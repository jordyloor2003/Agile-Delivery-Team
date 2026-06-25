# ADR-0004 · Auditoría como append-only log de eventos de pago
**Estado:** aceptado
**Fecha:** 2026-06-25

## Contexto y fuerza
El supervisor debe poder reconstruir cualquier movimiento de pago
"incluso meses después de ocurridas" (US-06) y el coordinador de
proyectos necesita que la información que alimenta cartera,
cobranza y contabilidad sea consistente y trazable (`coordinador-proyectos.md`).

El inbox documenta varios pains que apuntan a lo mismo: la
información existe, pero no se puede reconstruir quién hizo qué,
cuándo y con qué efecto — `falta-trazabilidad`,
`inconsistencias-financieras`, y la ausencia de evidencia
verificable que sufre el operador cuando el cliente pregunta por su
saldo a favor (`estado-no-coincide`).

Las fuerzas que lo exigen son:

- **R-12** (trazabilidad: cada operación debe poder reconstruirse
  mucho tiempo después, identificando quién, cuándo, qué cuotas
  afectó y con qué montos).
- **US-06** (reconstrucción auditable para el supervisor) y
  **US-09** (reverso con motivo, usuario y fecha, sin perder
  trazabilidad).
- Pains `falta-trazabilidad`, `inconsistencias-financieras` y
  `estado-no-coincide` de `evidence-map.json`.

## Decisión
Cualquier movimiento de saldo en el módulo de pagos se registra
como un **evento inmutable** en una tabla `EventoAuditoria` con la
siguiente forma mínima:

- `id` (autogenerado, monotónico).
- `tipo` (uno de: `pago_registrado`, `pago_aplicado`,
  `saldo_favor_generado`, `saldo_favor_consumido`,
  `saldo_favor_anulado`, `pago_reversado`,
  `autorizacion_excepcional_usada`).
- `usuario_id` (operador o supervisor que originó el movimiento).
- `fecha_hora` (timestamp del servidor, no editable).
- `referencias` (campos polimórficos que apuntan al `pago_id`,
  `cuota_id`, `saldo_favor_evento_id`, `autorizacion_id` o
  `reverso_id` afectados, según el tipo).
- `payload` (monto, motivo cuando aplique, idempotency_key del
  pago, etc., congelado al momento del evento).

Reglas:

1. **Append-only.** La tabla `EventoAuditoria` no tiene
   `UPDATE` ni `DELETE` en el código de aplicación. Corregir un
   error nunca es borrar o reescribir un evento: es insertar un
   evento nuevo que lo corrige (por ejemplo, `pago_reversado`
   referencia al `pago_registrado` original).
2. **Cobertura exhaustiva.** Todo lo que cambia saldos o
   distribución de un préstamo emite al menos un evento: la
   inserción del `Pago`, las `AplicaciónCuota` que produjo, los
   `SaldoFavorEvento` que originó, el `Reverso` cuando se
   ejecuta, y la `AutorizacionExcepcional` cuando se utiliza.
3. **Una sola fuente para vistas.** El "historial del supervisor"
   (US-06) se renderiza leyendo `EventoAuditoria` ordenado por
   `fecha_hora`. El "comprobante del cliente" (US-03) y el
   "historial del cliente" (US-04) leen los datos del `Pago` y
   de sus `AplicaciónCuota` y `SaldoFavorEvento` (que a su vez
   tienen su espejo en el log), de modo que toda vista que ve el
   cliente o el operador es derivable del log.
4. **Escritura dentro de la transacción del pago** (ADR-0002).
   Los eventos se insertan en la misma transacción que el `Pago`,
   las `AplicaciónCuota` y los `SaldoFavorEvento`. Si la
   transacción falla, no queda evento huérfano.

## Alternativas consideredas
- **Tabla de auditoría separada, llenada por triggers de la base
  de datos.** Descartado: añade acoplamiento con la tecnología
  concreta de la base de datos y complica pruebas y migraciones.
  Hacerlo desde la aplicación, dentro de la transacción, mantiene
  el log portable y verificable en código.
- **Event-bus externo (Kafka, RabbitMQ) como log de auditoría.**
  Descartado para el MVP: introduce infraestructura nueva
  (rompe el principio de "lo más simple que funcione") y
  duplica la verdad con la base transaccional. El log en base de
  datos ya cumple R-12.
- **Auditoría solo en la capa de aplicación sin tabla propia,
  reconstruyendo desde `Pago` y `Reverso`.** Descartado: la
  granularidad de US-06 (cuotas afectadas con monto individual,
  saldo a favor generado o consumido, reverso con su propio
  usuario y motivo) requiere una vista que una tabla `Pago`
  sola no ofrece sin recalcular. El log append-only es la forma
  más directa de cumplirlo.

## Consecuencias
- **Positivas:** el supervisor reconstruye cualquier movimiento
  en orden cronológico, con sus referencias y su payload. El
  reverso (US-09) no destruye información: añade un evento
  `pago_reversado` que apunta al original. El coordinador de
  proyectos tiene, en una sola tabla, la base para los futuros
  tableros de cartera y cobranza (US-13, fuera del MVP).
- **Costo:** más escrituras por operación (al menos un evento por
  mutación relevante). Aceptable en el MVP y dentro de la misma
  transacción del ADR-0002.
- **Costo operativo:** la política de retención del log
  (cuánto tiempo se conservan los eventos, qué se archiva)
  queda como open question; este ADR solo fija la inmutabilidad
  y la cobertura, no la ventana temporal.
- **Deuda explícita:** la decisión sobre notificaciones
  proactivas al cliente (push, email, polling) se documenta en
  `architecture.md` como open question; este ADR cubre la
  trazabilidad de los eventos, no su entrega.
