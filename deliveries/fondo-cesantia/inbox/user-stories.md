# User stories — fondo-cesantia

> Historias INVEST generadas a partir de `personas.md`, `requisitos.md` y
> `evidence-map.json`. Ordenadas por prioridad para el MVP: primero lo que
> ataca el **núcleo de valor** (registrar un pago correcto, visible y trazable),
> luego lo que evita regresiones y atiende los escenarios de excepción.

Convención:
- **Núcleo MVP** = historias indispensables para validar la propuesta de valor.
- **Endurecimiento** = historias que blindan la calidad y la consistencia del
  núcleo; sin ellas, el MVP funciona en demos pero falla en producción.
- **Después del MVP** = historias valiosas pero que pueden esperar a una
  segunda iteración para no inflar el primer entregable.

---

## Núcleo MVP

- **[US-01]** Como **Operador de pagos**, quiero registrar un pago contra un préstamo vigente y ver de inmediato cómo se distribuyó el dinero entre las cuotas (cuáles se cubrieron, cuáles quedaron con saldo y, si hay excedente, cuánto se acreditó como saldo a favor) para validar la operación sin tener que revisar manualmente el historial.
  - Criterios de aceptación:
    - Dado un préstamo activo con cuotas pendientes, cuando registro un pago por el valor exacto de la cuota más antigua, entonces esa cuota queda marcada como pagada, las demás mantienen su estado y el sistema muestra el desglose de la distribución.
    - Dado un préstamo con varias cuotas pendientes, cuando registro un pago que no alcanza para cubrir la más antigua completa, entonces esa cuota queda parcialmente pagada (no se marca como pagada) y se muestra el saldo pendiente de la cuota.
    - Dado un pago cuyo monto supera el valor de la cuota más antigua, cuando confirmo el registro, entonces el excedente se aplica automáticamente a la siguiente cuota pendiente; si aún sobra, se acumula como saldo a favor.
    - Fuente: `operador.md` · `analista-qa.md` · requisitos R-01, R-02, R-03, R-04, R-14

- **[US-02]** Como **Operador de pagos**, quiero que el sistema aplique los pagos siguiendo un orden predecible (primero las cuotas vencidas, después las pendientes desde la más antigua) para confiar en la distribución sin tener que reconstruirla a mano.
  - Criterios de aceptación:
    - Dado un préstamo con cuotas vencidas y cuotas por vencer, cuando registro un pago parcial, entonces el dinero se aplica primero a la cuota vencida más antigua.
    - Dado un préstamo con varias cuotas pendientes sin vencer, cuando registro un pago, entonces se aplica desde la más antigua hacia la más reciente.
    - Fuente: `analista-qa.md` · `coordinador-proyectos.md` · requisito R-02

- **[US-03]** Como **Cliente del préstamo**, quiero ver al instante un comprobante con el monto aplicado, las cuotas cubiertas, la fecha y mi saldo pendiente actualizado después de pagar, para tener la certeza de que el pago quedó bien registrado.
  - Criterios de aceptación:
    - Dado que acabo de realizar un pago, cuando consulto el comprobante, entonces veo el monto aplicado, las cuotas afectadas, la fecha del movimiento y el saldo pendiente resultante.
    - Dado que hice un pago parcial, cuando consulto el comprobante, entonces entiendo qué parte de la cuota quedó cubierta y cuánto falta.
    - Dado que hice un pago excedente, cuando consulto el comprobante, entonces veo cuánto se acreditó como saldo a favor.
    - Fuente: `cliente.md` · requisitos R-09, R-13, R-14

- **[US-04]** Como **Cliente del préstamo**, quiero consultar el historial completo de mis pagos (monto, fecha, cuotas afectadas y saldo a favor disponible) en lenguaje claro, para entender mi deuda sin necesidad de conocimientos financieros.
  - Criterios de aceptación:
    - Dado que he realizado varios pagos, cuando consulto mi historial, entonces veo cada movimiento con monto, fecha, cuotas afectadas y saldo a favor resultante de ese pago.
    - Dado un saldo a favor acumulado, cuando consulto el historial, entonces puedo ver el saldo a favor disponible y en qué pagos anteriores se generó.
    - Fuente: `cliente.md` · requisitos R-10, R-14

- **[US-05]** Como **Operador de pagos**, quiero ver desglosada la trazabilidad del saldo a favor de un cliente (cuándo se generó, cuánto se ha consumido y en qué pagos) para responder con seguridad cuando el cliente pregunte por él.
  - Criterios de aceptación:
    - Dado un préstamo con saldo a favor, cuando consulto el detalle, entonces veo el origen (pago y fecha) y cada consumo posterior con su monto y fecha.
    - Fuente: `operador.md` · `analista-qa.md` · `supervisor.md` · requisitos R-05, R-14

- **[US-06]** Como **Supervisor de pagos**, quiero reconstruir cualquier movimiento de pago a posteriori (quién lo registró, cuándo, qué cuotas afectó, con qué montos y si fue reversado) para investigar inconsistencias incluso meses después de ocurridas.
  - Criterios de aceptación:
    - Dado un préstamo, cuando consulto el historial completo de un pago, entonces veo usuario, fecha y hora, monto, cuotas afectadas, monto aplicado a cada cuota y referencia al posible reverso.
    - Fuente: `supervisor.md` · `coordinador-proyectos.md` · requisito R-12

---

## Endurecimiento (calidad indispensable del MVP)

- **[US-07]** Como **Especialista QA**, quiero que el sistema bloquee el registro de pagos sobre préstamos cancelados, salvo que exista una autorización explícita de un proceso excepcional, para evitar pagos fuera de política.
  - Criterios de aceptación:
    - Dado un préstamo cancelado, cuando intento registrar un pago, entonces el sistema lo rechaza y muestra un mensaje claro.
    - Fuente: `analista-qa.md` · `coordinador-proyectos.md` · requisito R-06

- **[US-08]** Como **Operador de pagos**, quiero que el sistema prevenga pagos duplicados cuando envío el mismo registro más de una vez (por ejemplo, doble clic) o cuando dos operadores trabajan sobre el mismo préstamo, para no tener que reversar manualmente.
  - Criterios de aceptación:
    - Dado que presioné "Registrar pago" y la operación se procesa, cuando vuelvo a presionar el mismo botón sobre el mismo préstamo con el mismo monto dentro de la misma operación, entonces el sistema detecta el duplicado y no lo registra dos veces.
    - Dado que dos operadores consultan el mismo préstamo, cuando ambos intentan registrar un pago casi simultáneamente, entonces solo uno de los registros se aplica y el otro es rechazado con un mensaje claro.
    - Fuente: `analista-qa.md` · requisito R-07

- **[US-09]** Como **Operador de pagos** o **Supervisor de pagos**, quiero reversar un pago previamente registrado con un motivo registrado, dejando trazabilidad del pago original y de la corrección, para corregir errores sin perder el historial.
  - Criterios de aceptación:
    - Dado un pago ya registrado, cuando solicito reversarlo indicando un motivo, entonces el préstamo vuelve al estado previo al pago y queda constancia del pago original y del reverso.
    - Fuente: `analista-qa.md` · `coordinador-proyectos.md` · requisito R-08

- **[US-10]** Como **Especialista QA**, quiero que la suite mínima de pruebas del módulo de pagos cubra, de forma automática o repetible, los escenarios de pago exacto, pago parcial, pago excedente, saldo a favor, reverso, múltiples cuotas vencidas y préstamo cancelado, para que cada liberación valide los casos donde históricamente se han colado defectos.
  - Criterios de aceptación:
    - Dado un cambio en el módulo de pagos, cuando se ejecuta la suite, entonces se ejecutan al menos los siete escenarios listados con sus aserciones de estado de cuota, saldo pendiente y saldo a favor.
    - Fuente: `analista-qa.md` · requisito R-15

---

## Después del MVP (no en este entregable)

- **[US-11]** Como **Supervisor de pagos**, quiero registrar y consultar reglas especiales de tratamiento (refinanciamientos, reestructuraciones, convenios, acuerdos de mora) por préstamo, para que la aplicación de pagos respete excepciones documentadas.
  - Por qué no entra: requiere un discovery adicional específico sobre esas reglas; la evidencia actual las menciona pero no las describe.
  - Fuente: `supervisor.md` · `analista-qa.md` · `coordinador-proyectos.md` · requisito asociado a las reglas no documentadas

- **[US-12]** Como **Operador de pagos**, quiero registrar pagos desde múltiples canales (caja, transferencia, débito automático) con la misma lógica de aplicación, para no duplicar流程 por canal.
  - Por qué no entra: la evidencia menciona que "el pago puede llegar desde diferentes canales" pero no hay detalle de cuáles ni de su integración; requiere discovery técnico.
  - Fuente: `coordinador-proyectos.md` · requisito R-01 (extensión)

- **[US-13]** Como **Coordinador de proyectos**, quiero reportes y tableros de control (cartera, cobranza, contabilidad) que reflejen en tiempo real los pagos aplicados, para tomar decisiones con información consistente.
  - Por qué no entra: depende de US-01…US-06 estables y de acordar con contabilidad los indicadores; va en una segunda iteración.
  - Fuente: `coordinador-proyectos.md` · requisito R-11 (visualización)
