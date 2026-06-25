# ADR-0002 · Transacción atómica para registrar y aplicar un pago
**Estado:** aceptado
**Fecha:** 2026-06-25

## Contexto y fuerza
El operador registra un pago (US-01) y espera ver, en la misma
pantalla, cómo se distribuyó el dinero: qué cuota quedó pagada, qué
cuota quedó parcial, qué monto se acreditó como saldo a favor. El
cliente (US-03) abre su comprobante inmediatamente después y debe ver
exactamente el mismo resultado. El supervisor (US-06) reconstruye ese
pago más tarde y debe encontrarlo en un estado coherente con lo que
ambos vieron en su momento.

Si el registro del pago, su distribución entre cuotas, la
actualización del saldo a favor y la confirmación al cliente se
ejecutan en pasos separados con confirmación parcial, basta con que
uno falle (latencia, timeout, crash) para que el cliente vea un
comprobante que no coincide con el saldo real, o que el supervisor
encuentre un pago sin la distribución esperada. Esto ya ha ocurrido
en producción (pain `inconsistencias-financieras`, pain
`estado-no-coincide`).

Las fuerzas que lo exigen son:

- **R-11** (consistencia financiera: ninguna vista debe mostrar
  estados contradictorios).
- **R-13** (rapidez de actualización: tras registrar un pago, lo que
  ve el cliente debe coincidir con la realidad sin esperas).
- **US-01** (el operador ve el desglose en la misma vista) y **US-03**
  (el cliente ve el comprobante al instante).
- Pains `inconsistencias-financieras` y `falta-comprobante-actualizacion`
  citados en `evidence-map.json`.

## Decisión
El caso de uso "registrar y aplicar un pago" se ejecuta dentro de
**una única transacción de base de datos** con los siguientes pasos
obligatorios, todos bajo el mismo commit:

1. Validar el préstamo y sus reglas de bloqueo (US-07).
2. Validar la `idempotency_key` (ADR-0003).
3. Adquirir el lock a nivel de préstamo (ADR-0003).
4. Insertar el `Pago`.
5. Calcular y persistir las `AplicaciónCuota` siguiendo el orden
   "vencidas primero, luego pendientes desde la más antigua"
   (US-01, US-02 fusionada en US-01).
6. Si hay excedente, registrar un `SaldoFavorEvento` de tipo
   `generación` por el monto del excedente.
7. Insertar los `EventoAuditoria` correspondientes (ADR-0004).
8. Confirmar la transacción y devolver al operador el desglose
   completo (cuotas afectadas con monto aplicado y saldo a favor
   resultante) en la misma respuesta, sin recargas (US-01).

El reverso (US-09) sigue el mismo principio: una transacción que
insertar un `Reverso`, anula la aplicación a nivel de
`AplicaciónCuota` (insertando contrapartes negativas o marcando el
pago original como reversado, según convenga al motor) y registra un
`SaldoFavorEvento` de tipo `anulación` si el pago original había
generado saldo a favor.

Si **cualquier** paso falla —validación, lock, escritura, evento de
auditoría— se hace rollback completo. No se devuelven estados
parciales al operador ni al cliente. La respuesta de éxito contiene
toda la información necesaria para que el front muestre el comprobante
del operador y el del cliente sin nuevas consultas.

## Alternativas consideradas
- **Pasos separados con compensación (saga).** Descartado: requiere
  lógica de compensación para errores parciales, y la evidencia
  (`inconsistencias-financieras`) muestra que precisamente eso es lo
  que ha fallado. Para un módulo con volumen bajo-medio y reglas
  cerradas, la transacción ACID es la opción más simple y más
  correcta.
- **Eventual consistency con cola/event-bus.** Descartado para el
  MVP: introduce latencia perceptible (rompe R-13) y abre una ventana
  en la que cliente y operador ven cosas distintas (rompe
  `falta-comprobante-actualizacion`). Se documenta como "no tomado"
  en `architecture.md`.
- **Doble confirmación (cliente → backend → confirmación manual).**
  Descartado: añade pasos manuales y contradice US-01 y US-03, que
  exigen ver el resultado de inmediato.

## Consecuencias
- **Positivas:** la consistencia entre saldos, cuotas, historial y
  reporte está garantizada por la base de datos, no por disciplina
  del equipo. El operador y el cliente ven exactamente el mismo
  estado al mismo tiempo. El supervisor reconstruye lo mismo que
  ambos vieron.
- **Costo:** la transacción incluye más escrituras (pago,
  aplicaciones, evento de saldo a favor, eventos de auditoría), por
  lo que el lock a nivel de préstamo dura lo que dura esa unidad de
  trabajo. Esto es aceptable para el volumen del MVP y se controla
  con ADR-0003.
- **Costo operativo:** cualquier nueva mutación del módulo
  (p. ej. consumo automático de saldo a favor) debe diseñarse
  también como parte de una transacción, no como job asíncrono.
  Esto se vigila desde la revisión de código y la suite mínima
  (ADR-0005).
