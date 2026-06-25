# Requisitos candidatos — fondo-cesantia

> Evidencia extraída de `discoveries/fondo-cesantia/interviews/`.
> Conteo: **15 requisitos candidatos** (10 funcionales, 5 no funcionales).
> Numeración correlativa R-01 … R-15. Cada requisito cita la fuente que lo
> respalda y la persona/stakeholder asociado.

## Funcionales

- **[R-01]** Registrar un pago contra un préstamo vigente, validando que el préstamo exista y esté activo, y que el monto sea válido.
  - Tipo: funcional
  - Origen: `analista-qa.md` · `coordinador-proyectos.md` · persona Operador de pagos

- **[R-02]** Aplicar el pago siguiendo el orden de antigüedad: primero cuotas vencidas; si hay varias pendientes, desde la más antigua a la más reciente.
  - Tipo: funcional
  - Origen: `analista-qa.md` · `coordinador-proyectos.md` · personas Operador, Supervisor

- **[R-03]** Soportar pagos parciales sobre una cuota sin marcarla como pagada, manteniendo el saldo pendiente de la cuota y acumulando pagos posteriores.
  - Tipo: funcional
  - Origen: `analista-qa.md` · `operador.md` · `cliente.md` · persona Operador de pagos

- **[R-04]** Soportar pagos excedentes (monto mayor al de la cuota) registrando el remanente como saldo a favor disponible para próximas aplicaciones.
  - Tipo: funcional
  - Origen: `analista-qa.md` · `operador.md` · `cliente.md` · persona Operador de pagos

- **[R-05]** Mantener trazabilidad completa del saldo a favor: cuándo se generó, cuánto se utilizó, cuál es el saldo restante y en qué pagos se consumió.
  - Tipo: funcional
  - Origen: `analista-qa.md` · `supervisor.md` · persona Supervisor de pagos

- **[R-06]** Bloquear el registro de pagos sobre préstamos cancelados, salvo autorización explícita por un proceso excepcional supervisado.
  - Tipo: funcional
  - Origen: `analista-qa.md` · `coordinador-proyectos.md` · persona Especialista QA

- **[R-07]** Prevenir pagos duplicados cuando el operador envía el mismo registro varias veces (p. ej. doble clic por latencia) y cuando dos usuarios operan sobre el mismo préstamo a la vez.
  - Tipo: funcional
  - Origen: `analista-qa.md` · persona Especialista QA

- **[R-08]** Permitir reversar un pago previamente registrado, dejando evidencia de la operación original y de la corrección.
  - Tipo: funcional
  - Origen: `analista-qa.md` · `coordinador-proyectos.md` · persona Especialista QA

- **[R-09]** Mostrar al cliente, tras cada pago, un comprobante con el monto aplicado, las cuotas cubiertas, las fechas y el saldo pendiente actualizado.
  - Tipo: funcional
  - Origen: `cliente.md` · persona Cliente del préstamo

- **[R-10]** Exponer al cliente el historial completo de pagos (montos, fechas, cuotas afectadas, saldo a favor) con lenguaje claro y entendible.
  - Tipo: funcional
  - Origen: `cliente.md` · persona Cliente del préstamo

## No funcionales

- **[R-11]** Consistencia financiera: un movimiento registrado debe mantener coherencia inmediata entre saldos, cuotas, historial y reportes; ninguna vista debe mostrar estados contradictorios.
  - Tipo: no funcional (consistencia de datos)
  - Origen: `supervisor.md` · `coordinador-proyectos.md` · persona Supervisor de pagos

- **[R-12]** Trazabilidad: cada operación (registro, aplicación, reverso) debe poder reconstruirse mucho tiempo después, identificando quién, cuándo, qué cuotas afectó y con qué montos.
  - Tipo: no funcional (auditabilidad)
  - Origen: `supervisor.md` · `coordinador-proyectos.md` · persona Supervisor de pagos

- **[R-13]** Rapidez de actualización: tras registrar un pago, la información consultada por el cliente y por el operador debe reflejar los cambios sin esperas perceptibles.
  - Tipo: no funcional (rendimiento percibido)
  - Origen: `cliente.md` · `operador.md` · persona Cliente del préstamo

- **[R-14]** Claridad de la información: el sistema debe explicar de forma entendible cómo distribuyó el pago, qué cuotas afectó y cómo se calculó el saldo restante, sin requerir conocimientos financieros avanzados.
  - Tipo: no funcional (usabilidad/comprensibilidad)
  - Origen: `operador.md` · `cliente.md` · persona Operador de pagos

- **[R-15]** Cobertura de pruebas: todo cambio en el módulo de pagos debe validar, como mínimo, los escenarios de pago exacto, pago parcial, pago excedente, saldo a favor, reverso, múltiples cuotas vencidas y préstamo cancelado.
  - Tipo: no funcional (calidad / cobertura de pruebas)
  - Origen: `analista-qa.md` · persona Especialista QA
