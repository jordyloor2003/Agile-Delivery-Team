# ADR-0003 · Idempotencia por clave de operación + lock pesimista por préstamo
**Estado:** aceptado
**Fecha:** 2026-06-25

## Contexto y fuerza
El operador registra pagos a través de un front (US-01) y los
requisitos y pains del inbox describen dos modos de duplicación
claramente distintos:

1. **Reintento del mismo operador.** Doble clic, latencia en la red
   que hace que el operador presione "Registrar" dos veces. El
   cliente paga una sola vez; el sistema debe registrar una sola
   vez.
2. **Concurrencia entre operadores distintos.** Dos operadores
   consultan el mismo préstamo, ambos ven el mismo conjunto de
   cuotas pendientes, ambos intentan registrar un pago casi al
   mismo tiempo. Sin coordinación, ambos pagos podrían aplicarse
   sobre la misma cuota, generando una doble aplicación que solo
   se detecta después y obliga a reversar (retrabajo del operador,
   daño al cliente).

Las fuerzas que lo exigen son:

- **R-07** (prevenir pagos duplicados por reintento del operador y
  por concurrencia).
- **US-08** (prevención de duplicados y bloqueo de concurrencia con
  rechazo explícito al segundo operador).
- Pain `pagos-duplicados-concurrencia` citado en `evidence-map.json`
  y descrito en `analista-qa.md`.

## Decisión
La unidad de concurrencia y deduplicación es el **préstamo**, no el
cliente ni el operador. Se combinan dos mecanismos:

### a) Idempotencia por `idempotency_key`
- El front genera un identificador de operación por cada intento de
  "Registrar pago" (por ejemplo, un UUID generado al abrir el
  formulario de pago y congelado al enviar). Se envía en la
  solicitud de pago.
- El backend persiste la `idempotency_key` junto al `Pago`.
- Una segunda solicitud con la **misma** `idempotency_key`, **mismo
  préstamo** y **mismo monto** se reconoce como duplicado: el
  backend no crea un `Pago` nuevo, no vuelve a aplicar a cuotas y
  devuelve la respuesta del registro original (mismo desglose, mismo
  evento de auditoría). Esto cubre el caso de doble clic y de
  reintento por timeout.
- Si llega la misma `idempotency_key` con un préstamo o monto
  distinto, el sistema rechaza con error claro: la clave se está
  reutilizando para una operación diferente.

### b) Lock pesimista por préstamo durante la transacción
- Al iniciar la transacción atómica del ADR-0002, el backend
  adquiere un lock pesimista sobre la fila del `Préstamo`. El lock
  se mantiene hasta el commit o rollback.
- Una segunda solicitud concurrente sobre el **mismo préstamo**
  queda esperando el lock. Una vez liberado, ve que ya existe un
  `Pago` reciente y, según el caso:
  - Si trae la misma `idempotency_key`, se trata como reintento
    idempotente y se devuelve la respuesta original.
  - Si trae una `idempotency_key` distinta, se interpreta como una
    operación nueva pero que ya no aplica sobre el estado actual;
    se rechaza con un mensaje claro de "conflicto de concurrencia:
    ya se registró un pago sobre este préstamo, recargue la vista"
    (criterio Gherkin de US-08).
- Esto cubre el caso de dos operadores en paralelo sobre el mismo
  préstamo.

La combinación de ambos mecanismos cumple los tres criterios de
aceptación de US-08: reintento idempotente, segundo operador
rechazado, y segundo operador bloqueado mientras el lock esté
activo.

## Alternativas consideradas
- **Lock optimista con columna de versión.** Descartado: requiere
  reintentos desde el front y complica la historia del operador
  cuando dos pantallas quedan "compitiendo" por el mismo préstamo.
  El volumen esperado del MVP no justifica esa complejidad y, peor,
  abriría una ventana en la que el operador cree que su pago se
  registró cuando en realidad fue rechazado.
- **Lock pesimista solo (sin idempotency_key).** Descartado: no
  distingue entre "el operador reintentó" y "otro operador
  realmente quiso pagar". Un doble clic legítimo quedaría
  registrado como conflicto y obligaría a reintentar, lo que rompe
  US-08 en su primer criterio.
- **Cola de mensajes con un único consumidor por préstamo.**
  Descartado para el MVP: introduce infraestructura nueva sin un
  beneficio claro sobre la combinación lock + idempotencia, y
  añade latencia (rompe R-13).
- **Bloqueo a nivel de cliente en lugar de préstamo.** Descartado:
  un mismo cliente puede tener varios préstamos, y el dolor del
  inbox es por préstamo, no por cliente. Bloquear al cliente
  serializa operaciones que no compiten.

## Consecuencias
- **Positivas:** US-08 se cumple con reglas claras y verificables.
  El doble clic y la concurrencia real quedan cubiertos por la
  misma maquinaria, sin pedirle al operador que distinga entre
  ambas.
- **Costo:** la transacción del ADR-0002 retiene el lock durante
  todo el procesamiento. En el MVP, con un solo motor de aplicación
  y un volumen bajo-medio, la espera es despreciable. Si en el
  futuro el lock se volviera un cuello de botella, la salida es
  cambiar a cola por préstamo, no relajar este ADR.
- **Costo operativo:** cualquier nueva operación de escritura sobre
  préstamo (pago, consumo de saldo a favor explícito, ajustes
  manuales del supervisor) debe participar del mismo lock. Esto
  queda registrado en la guía de revisión de código.
- **Deuda explícita:** la forma concreta de la `idempotency_key`
  (header HTTP, campo en el body, mecanismo de generación en el
  front) queda como open question para el equipo de desarrollo.
