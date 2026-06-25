# ADR-0001 · Modelo de dominio único para el módulo de pagos
**Estado:** aceptado
**Fecha:** 2026-06-25

## Contexto y fuerza
El módulo de pagos es consumido por el operador (registra), el cliente
(consulta comprobante e historial) y el supervisor (reconstruye movimientos).
Tres roles distintos leen los mismos datos: saldos, cuotas, pagos, saldo a
favor, reversos. Si el modelo se fragmenta — por ejemplo, mantener un
"saldo a favor" como columna calculada en una tabla y, al mismo tiempo,
como vista materializada en otra — el histórico deja de coincidir y la
auditoría del supervisor (US-06) no puede reconstruir lo que pasó.

Las fuerzas que lo exigen son:

- **R-05** (trazabilidad completa del saldo a favor: generación, consumo,
  saldo restante) y **R-11** (consistencia inmediata entre saldos, cuotas,
  historial y reportes): si el saldo a favor se calcula a mano en cada
  vista, se desincroniza.
- **R-12** (reconstruir cada operación mucho tiempo después) y la US-06:
  el supervisor necesita ver generaciones y consumos de saldo a favor con
  su pago de origen, no un número agregado.
- **US-01, US-05, US-06** y los pains `estado-no-coincide`,
  `inconsistencias-financieras` y `falta-trazabilidad` citados en
  `evidence-map.json`: tres historias que tocan la misma entidad
  (`SaldoFavor`) desde tres ángulos distintos, lo que obliga a un solo
  modelo de eventos.

## Decisión
Se define un único modelo de dominio para el módulo de pagos, con las
siguientes entidades y reglas:

- **`Préstamo`** — entidad raíz. Tiene `estado` (`activo` / `cancelado` /
  `cerrado`), `cliente_id` y `condiciones` (moneda, periodicidad). El
  estado del préstamo es lo que US-07 consulta para bloquear pagos.
- **`Cuota`** — pertenece a un `Préstamo`. Tiene `número`, `vencimiento`,
  `monto_esperado`, `monto_pagado` y `saldo_pendiente`. La `Cuota` no
  guarda cómo se pagó: eso lo registra `AplicaciónCuota`.
- **`Pago`** — registra el ingreso de dinero contra un `Préstamo`. Tiene
  `monto_total`, `fecha_hora`, `usuario_operador`, `idempotency_key`,
  `canal` (en el MVP siempre "front operador"), y un puntero opcional a
  `autorizacion_excepcional_id` cuando el préstamo está cancelado.
- **`AplicaciónCuota`** — entidad puente. Por cada `Pago` y por cada
  `Cuota` que ese pago tocó, una fila con `monto_aplicado`. Es la fuente
  de verdad de la distribución que el operador ve en US-01 y de la
  trazabilidad que US-06 reconstruye.
- **`SaldoFavor`** — saldo a favor disponible, agregado, calculado a
  partir de sus eventos (`SaldoFavorEvento`):
  - `tipo = generación` cuando un pago deja excedente.
  - `tipo = consumo` cuando se aplica saldo a favor a un pago posterior.
  - `tipo = anulación` cuando se reversa el pago que lo generó
    (US-09).
  El saldo a favor que el cliente ve en su comprobante (US-03) y su
  historial (US-04), y el que el operador explica con US-05, son siempre
  el mismo agregado derivado de estos eventos.
- **`Reverso`** — evento que marca un `Pago` como reversado, con
  `motivo`, `usuario_autorizante`, `fecha_hora` y referencia al pago
  original. Existe como entidad separada para que US-06 distinga "el
  pago original" de "el evento que lo anuló", tal como pide US-09.
- **`AutorizacionExcepcional`** — entidad independiente con
  `supervisor_id`, `fecha_hora`, `motivo` y `prestamo_id`. El `Pago` la
  referencia por `id` cuando US-07 lo permite. Existe ya desde el MVP
  aunque US-11 (reglas por convenio) quede fuera: es el primer
  ladrillo para soportar excepciones, y US-07 la necesita.
- **`EventoAuditoria`** — log append-only; ver ADR-0004 para el detalle.
  Todas las mutaciones de las entidades anteriores emiten un evento
  aquí; las vistas (comprobante, historial, vista supervisor) leen
  estas tablas y, cuando conviene, el log.

Reglas del modelo:
1. Una sola fuente de verdad para el saldo a favor: `SaldoFavorEvento`.
   El saldo disponible en pantalla es una vista derivada; no se persiste
   como columna acumulable.
2. La distribución de un pago se reconstruye siempre desde
   `AplicaciónCuota`, nunca recalculando reglas a mano en cada vista.
3. `Pago`, `Reverso` y `AutorizacionExcepcional` son inmutables: solo
   se insertan y se consultan; corregir es siempre un evento nuevo.

## Alternativas consideradas
- **Calcular el saldo a favor como columna en `Préstamo`.** Descartado:
  impide ver generaciones y consumos individuales (rompe US-05 y
  R-05), y se desincroniza ante reversos (rompe R-11 y
  `inconsistencias-financieras`).
- **Vistas materializadas por rol** (una para operador, otra para
  cliente, otra para supervisor). Descartado: introduce duplicación de
  la verdad y reabre `estado-no-coincide` apenas una vista se
  desactualice.
- **Reutilizar el modelo de cartera existente del fondo sin adaptarlo.**
  Descartado porque el inbox no describe ese modelo; sería
  inventarlo. Se documenta la dependencia con el sistema de cartera
  como open question.

## Consecuencias
- **Positivas:** US-01, US-05, US-06 leen del mismo origen, lo que
  cierra `estado-no-coincide` y `inconsistencias-financieras` desde el
  diseño. El reverso (US-09) es trivial: se inserta un
  `SaldoFavorEvento.tipo = anulación` y un `Reverso`; nada se borra ni
  se sobreescribe.
- **Costo:** el modelo es más rico que un "tabla Pago + columna saldo".
  El equipo de desarrollo debe entender que la distribución no se
  recalcula en cada vista, y que el saldo a favor siempre sale del log
  de eventos. Esto se mitiga porque US-10 (suite mínima) cubre
  explícitamente el escenario "saldo a favor".
- **Deuda explícita:** este modelo no toca la integración con el
  sistema de cartera/legado del fondo; queda como open question.
