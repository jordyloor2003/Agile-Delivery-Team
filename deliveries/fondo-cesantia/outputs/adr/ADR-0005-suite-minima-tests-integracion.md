# ADR-0005 · Suite mínima de pruebas como tests de integración automatizados
**Estado:** aceptado
**Fecha:** 2026-06-25

## Contexto y fuerza
El Especialista QA (persona en `personas.md` con poder de veto
sobre la liberación) documenta explícitamente que el histórico de
defectos del módulo se concentra en escenarios muy concretos:
"pago exacto, pago parcial, pago excedente, saldo a favor, reverso,
múltiples cuotas vencidas y préstamo cancelado". Son los siete
escenarios de R-15. Si la cobertura de esos siete escenarios no es
automática y repetible, cada nueva versión del módulo vuelve a
colar los mismos defectos (`defectos-pagos-parciales`,
`reglas-ocultas`).

Las fuerzas que lo exigen son:

- **R-15** (cobertura mínima de los siete escenarios críticos
  antes de cada liberación).
- **US-10** (suite mínima repetible, ejecutada antes de cada
  liberación, que valida los siete escenarios con aserciones de
  estado de cuota, saldo pendiente y saldo a favor, y bloquea la
  liberación si alguno falla).
- Pains `defectos-pagos-parciales` y `reglas-ocultas`.

## Decisión
Los siete escenarios de R-15 / US-10 se implementan como **tests
de integración automatizados** del módulo de pagos (no como
checklist manual, no como pruebas unitarias aisladas del motor
de aplicación). Cada test:

1. Prepara un estado conocido de `Préstamo` y `Cuota` en una
   base de datos de prueba.
2. Ejecuta la operación real a través de la API o del caso de
   uso del motor de aplicación (mismo punto de entrada que
   producción).
3. Verifica aserciones explícitas sobre:
   - Estado de cada `Cuota` afectada (`pendiente`, `parcial`,
     `pagada`).
   - Saldo pendiente de cada `Cuota` y saldo pendiente global
     del préstamo.
   - `SaldoFavor` resultante y eventos `SaldoFavorEvento`
     registrados.
   - Eventos en `EventoAuditoria` (ADR-0004) asociados al
     movimiento.
4. Falla el build si alguna aserción no se cumple.

Los siete escenarios son obligatorios y cubren, como mínimo:

- Pago exacto de una cuota.
- Pago parcial sobre una cuota.
- Pago excedente sobre una cuota (genera saldo a favor).
- Consumo explícito de saldo a favor en un pago posterior
  (cuando aplique al flujo del MVP).
- Reverso de un pago con motivo.
- Pago parcial con múltiples cuotas vencidas (orden por
  antigüedad).
- Registro de pago sobre préstamo cancelado, con y sin
  autorización excepcional.

La suite se ejecuta en el pipeline de integración continua antes
de cada liberación. Si un escenario falla, la liberación se
bloquea; el criterio "suite verde" de US-10 no es optativo.

## Alternativas consideradas
- **Tests unitarios del motor de aplicación solamente.**
  Descartado: deja sin cubrir la integración con la
  transacción, el lock (ADR-0003), la persistencia del
  `SaldoFavorEvento` y la escritura en el log de auditoría
  (ADR-0004). Los defectos históricos son de integración, no
  de una función aislada.
- **Checklist manual ejecutado por QA en cada liberación.**
  Descartado: depende de la persona, no se repite igual en
  cada corrida, y la persona clave puede no estar disponible
  el día del release. El pain `reglas-ocultas` muestra que
  parte de la规则 está "en la cabeza" de los operativos y se
  pierde al rotar.
- **Pruebas end-to-end con interfaz de usuario.** Descartado
  para el MVP: añade fragilidad (selectores, navegación) y
  desacopla la verificación de la regla de negocio del
  escenario. Se reserva para una segunda iteración, si
  surge la necesidad.

## Consecuencias
- **Positivas:** cada uno de los siete escenarios críticos
  tiene un guardián automatizado. Un cambio que rompa el
  orden por antigüedad, que duplique un pago, que pierda el
  saldo a favor o que omita el log de auditoría no llega a
  producción. Esto cierra directamente los pains
  `defectos-pagos-parciales` y `reglas-ocultas`.
- **Costo:** hay que mantener fixtures y datos de prueba
  representativos (préstamos con varias cuotas, alguna
  vencida, alguna parcial, alguna con saldo a favor). Es
  trabajo extra en el primer sprint, pero se amortiza desde
  la segunda liberación.
- **Costo operativo:** la suite se ejecuta en cada PR y en
  cada release. Si crece demasiado, se debe volver a
  fragmentar — pero esto es un problema de otra iteración,
  no del MVP.
- **Acuerdo de equipo:** la suite es **la** definición
  operativa de "hecho" para el módulo de pagos. Cualquier
  historia nueva que toque el módulo debe agregar o
  actualizar al menos un test de la suite.
