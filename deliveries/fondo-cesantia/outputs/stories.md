# Historias refinadas — fondo-cesantia

> Producto: **Módulo de aplicación de pagos del fondo de cesantía**.
> Historias refinadas bajo INVEST + Definition of Ready. Salida del Developer.
> Insumo: `inbox/mvp-canvas.md`, `inbox/user-stories.md`, `inbox/requisitos.md`,
> `inbox/personas.md`, `inbox/evidence-map.json`. Cero invención fuera de esas
> fuentes; las dudas de interpretación van al final, en "Supuestos y notas
> abiertas para el equipo".

---

### US-01 · Registrar pago y ver distribución en la misma pantalla   ·   épica E-01   ·   8 pts
**Como** Operador de pagos, **quiero** registrar un pago contra un préstamo vigente y ver de inmediato, en la misma pantalla y sin recargar la vista, el desglose exacto de la distribución del dinero (cuotas cubiertas, cuotas con saldo parcial, monto aplicado a cada cuota y saldo a favor acreditado si lo hubiera), aplicando el pago primero a las cuotas vencidas y, dentro de las pendientes, desde la más antigua a la más reciente, **para** validar la operación a la primera vista sin tener que revisar manualmente el historial del préstamo.

Criterios de aceptación (Gherkin):
- Dado un préstamo activo con al menos una cuota vencida y una cuota por vencer, cuando registro un pago parcial, entonces el dinero se aplica primero a la cuota vencida más antigua y, si sobra, continúa con la siguiente cuota pendiente en orden cronológico ascendente.
- Dado un préstamo activo con varias cuotas pendientes sin vencer, cuando registro un pago, entonces se aplica desde la más antigua hacia la más reciente.
- Dado un préstamo activo con una cuota pendiente, cuando registro un pago por el valor exacto de esa cuota, entonces esa cuota queda marcada como pagada, las demás mantienen su estado y el sistema muestra el desglose en la misma pantalla.
- Dado un préstamo activo con una cuota pendiente, cuando registro un pago que no alcanza para cubrirla completa, entonces esa cuota queda parcialmente pagada (no se marca como pagada) y se muestra el saldo pendiente de la cuota.
- Dado un pago cuyo monto supera el valor de la cuota más antigua, cuando confirmo el registro, entonces el excedente se aplica automáticamente a la siguiente cuota pendiente; si aún sobra, se acumula como saldo a favor con su monto y origen visible en el desglose.
- Dado que acabo de registrar un pago, cuando se muestra la confirmación, entonces la lista de cuotas afectadas, los montos aplicados y el saldo a favor resultante aparecen en la misma vista, sin necesidad de navegar a otra pantalla ni recargar.

Origen: `us:US-01`, `us:US-02` (fusionada), `req:R-01`, `req:R-02`, `req:R-03`, `req:R-04`, `req:R-14`, `pain:distribucion-opaca`, `pain:retrabajo-verificacion`.

---

### US-03 · Comprobante inmediato para el cliente   ·   épica E-02   ·   5 pts
**Como** Cliente del préstamo, **quiero** ver al instante, en mi comprobante, el monto aplicado, las cuotas cubiertas con sus montos individuales, la fecha y hora del movimiento y mi saldo pendiente actualizado después de pagar, **para** tener la certeza verificable de que el pago quedó bien registrado.

Criterios de aceptación (Gherkin):
- Dado que acabo de realizar un pago, cuando consulto el comprobante, entonces veo el monto total aplicado, el listado de cuotas afectadas con el monto aplicado a cada una, la fecha y hora del movimiento y el saldo pendiente resultante del préstamo.
- Dado que hice un pago parcial, cuando consulto el comprobante, entonces veo de forma explícita qué parte de la cuota quedó cubierta (monto aplicado) y cuánto falta (saldo pendiente de la cuota).
- Dado que hice un pago excedente, cuando consulto el comprobante, entonces veo el monto acreditado como saldo a favor con su origen (pago que lo generó).

Origen: `us:US-03`, `req:R-09`, `req:R-13`, `req:R-14`, `pain:incertidumbre-post-pago`, `pain:falta-comprobante-actualizacion`.

---

### US-04 · Historial legible para el cliente   ·   épica E-02   ·   5 pts
**Como** Cliente del préstamo, **quiero** consultar el historial completo de mis pagos (monto, fecha, cuotas afectadas y saldo a favor disponible) en lenguaje claro, sin tecnicismos financieros, **para** entender mi deuda y los movimientos de mi préstamo sin ayuda del operador.

Criterios de aceptación (Gherkin):
- Dado que he realizado varios pagos, cuando consulto mi historial, entonces veo cada movimiento con su monto total, fecha, las cuotas afectadas con su monto individual y el saldo a favor resultante tras ese pago, presentado con etiquetas en lenguaje claro.
- Dado un saldo a favor acumulado, cuando consulto el historial, entonces veo el saldo a favor disponible y el detalle de en qué pagos anteriores se generó, también con etiquetas en lenguaje claro.

Origen: `us:US-04`, `req:R-10`, `req:R-14`, `pain:opacidad-calculo-financiero`, `pain:incertidumbre-post-pago`.

---

### US-05 · Trazabilidad del saldo a favor para el operador   ·   épica E-03   ·   3 pts
**Como** Operador de pagos, **quiero** ver desglosada, en la consulta del préstamo, la trazabilidad del saldo a favor del cliente: fecha y pago que lo generó, monto original, cada consumo posterior con su monto y fecha, y saldo a favor restante, **para** responder con seguridad y con evidencia al cliente cuando pregunte por su saldo a favor.

Criterios de aceptación (Gherkin):
- Dado un préstamo con saldo a favor, cuando consulto el detalle, entonces veo el pago y fecha que lo generó, su monto original, cada consumo posterior con su monto y fecha, y el saldo a favor restante.
- Dado un préstamo con varios movimientos de saldo a favor (generaciones y consumos), cuando consulto el detalle, entonces veo cada movimiento en orden cronológico con su tipo (generación o consumo), monto, fecha y referencia al pago o movimiento que lo originó.

Origen: `us:US-05`, `req:R-05`, `req:R-14`, `pain:estado-no-coincide`.

---

### US-06 · Reconstrucción auditable para el supervisor   ·   épica E-03   ·   5 pts
**Como** Supervisor de pagos, **quiero** reconstruir cualquier movimiento de pago a posteriori, con todos sus datos: usuario que lo registró, fecha y hora, monto, cuotas afectadas con sus montos individuales, saldo a favor generado o consumido y, si fue reversado, los datos del reverso (usuario, fecha, motivo), **para** investigar inconsistencias financieras incluso meses después de ocurridas, sin perder el rastro.

Criterios de aceptación (Gherkin):
- Dado un préstamo, cuando consulto el historial completo de un pago, entonces veo el usuario operador que lo registró, fecha y hora, monto total, las cuotas afectadas con el monto aplicado a cada una y la referencia al posible reverso (con su propio usuario, fecha y motivo).
- Dado un pago reversado, cuando consulto su detalle, entonces veo tanto el registro del pago original como el registro del reverso vinculado, con motivo del reverso, usuario que lo autorizó y fecha.

Origen: `us:US-06`, `req:R-12`, `pain:falta-trazabilidad`, `pain:inconsistencias-financieras`.

---

### US-07 · Bloqueo de pagos en préstamos cancelados   ·   épica E-04   ·   2 pts
**Como** Especialista QA, **quiero** que el sistema bloquee el registro de pagos sobre préstamos cancelados, salvo que exista una autorización explícita de un proceso excepcional registrada con el ID del supervisor autorizante y el motivo, **para** evitar pagos fuera de política sin perder la trazabilidad de la excepción.

Criterios de aceptación (Gherkin):
- Dado un préstamo en estado "cancelado" y sin autorización excepcional registrada, cuando intento registrar un pago, entonces el sistema rechaza la operación y muestra un mensaje claro indicando el motivo del rechazo.
- Dado un préstamo en estado "cancelado" con una autorización excepcional registrada (con ID del supervisor autorizante, fecha y motivo), cuando intento registrar un pago indicando esa autorización, entonces el sistema permite el registro y deja constancia visible de la autorización utilizada (ID del supervisor y motivo).
- Dado un préstamo en estado "cancelado" con una autorización excepcional registrada, cuando intento registrar un pago SIN indicar la autorización, entonces el sistema rechaza la operación.

Origen: `us:US-07`, `req:R-06`, `pain:defectos-pagos-parciales`.

---

### US-08 · Prevención de pagos duplicados y concurrencia   ·   épica E-04   ·   5 pts
**Como** Operador de pagos, **quiero** que el sistema prevenga pagos duplicados cuando envío el mismo registro más de una vez (por ejemplo, doble clic) usando una clave de idempotencia por operación, y que también bloquee la concurrencia cuando dos operadores registran pagos sobre el mismo préstamo al mismo tiempo, **para** no tener que reversar manualmente pagos aplicados dos veces.

Criterios de aceptación (Gherkin):
- Dado que presioné "Registrar pago" con un identificador de operación (idempotency key), cuando vuelvo a enviar el mismo registro con el mismo identificador de operación, mismo préstamo y mismo monto, entonces el sistema detecta el duplicado, no lo registra una segunda vez y devuelve el resultado del registro original.
- Dado que dos operadores consultan el mismo préstamo y ambos intentan registrar un pago casi simultáneamente, cuando el sistema procesa las dos solicitudes, entonces solo uno de los registros se aplica y el otro es rechazado con un mensaje claro de conflicto de concurrencia.
- Dado que ya existe un pago registrado para un préstamo y aún no se ha liberado el bloqueo sobre ese préstamo, cuando un segundo operador intenta registrar otro pago sobre el mismo préstamo, entonces el sistema rechaza la operación mientras el bloqueo está activo.

Origen: `us:US-08`, `req:R-07`, `pain:pagos-duplicados-concurrencia`.

---

### US-09 · Reverso con motivo y trazabilidad   ·   épica E-04   ·   5 pts
**Como** Operador de pagos o Supervisor de pagos, **quiero** reversar un pago previamente registrado indicando un motivo obligatorio, de modo que las cuotas afectadas vuelvan a su saldo anterior, se anule el saldo a favor que ese pago hubiera generado y el movimiento original quede marcado como reversado con su motivo, usuario y fecha, **para** corregir errores sin perder el historial ni la trazabilidad.

Criterios de aceptación (Gherkin):
- Dado un pago ya registrado, cuando solicito reversarlo indicando un motivo obligatorio, entonces las cuotas afectadas vuelven a su saldo anterior al pago, el movimiento original queda marcado como "reversado" con el motivo, el usuario que autorizó el reverso y la fecha del reverso.
- Dado un pago que al aplicarse generó saldo a favor, cuando reverso ese pago, entonces el saldo a favor generado por él queda anulado y la trazabilidad de esa anulación queda visible en la consulta del saldo a favor del préstamo.
- Dado un pago ya reversado, cuando intento reversarlo de nuevo, entonces el sistema rechaza la operación indicando que el pago ya fue reversado previamente.

Origen: `us:US-09`, `req:R-08`, `pain:defectos-pagos-parciales`, `pain:inconsistencias-financieras`.

---

### US-10 · Suite mínima de pruebas del módulo de pagos   ·   épica E-04   ·   3 pts
**Como** Especialista QA, **quiero** que la suite mínima de pruebas del módulo de pagos cubra, de forma repetible y ejecutable antes de cada liberación, los escenarios de pago exacto, pago parcial, pago excedente, saldo a favor, reverso, múltiples cuotas vencidas y préstamo cancelado, con aserciones explícitas de estado de cuota, saldo pendiente y saldo a favor, **para** que cada liberación valide los casos donde históricamente se han colado defectos y no se libere si alguno falla.

Criterios de aceptación (Gherkin):
- Dado un cambio en el módulo de pagos, cuando se ejecuta la suite mínima, entonces se ejecutan al menos los siete escenarios (pago exacto, pago parcial, pago excedente, saldo a favor, reverso, múltiples cuotas vencidas y préstamo cancelado) con aserciones explícitas de estado de cuota, saldo pendiente y saldo a favor.
- Dado que la suite mínima se ejecuta y todos los escenarios pasan, cuando termina la ejecución, entonces el sistema la marca como "suite verde" y permite la liberación.
- Dado que la suite mínima se ejecuta y al menos un escenario falla, cuando termina la ejecución, entonces el sistema reporta el escenario fallido con su aserción rota e impide la liberación.

Origen: `us:US-10`, `req:R-15`, `pain:defectos-pagos-parciales`, `pain:reglas-ocultas`.

---

## Tabla resumen

| ID    | Épica | Pts | Prioridad | Título                                                  |
|-------|-------|-----|-----------|---------------------------------------------------------|
| US-01 | E-01  | 8   | 1         | Registrar pago y ver distribución en la misma pantalla  |
| US-03 | E-02  | 5   | 3         | Comprobante inmediato para el cliente                   |
| US-04 | E-02  | 5   | 4         | Historial legible para el cliente                       |
| US-05 | E-03  | 3   | 5         | Trazabilidad del saldo a favor para el operador         |
| US-06 | E-03  | 5   | 6         | Reconstrucción auditable para el supervisor             |
| US-07 | E-04  | 2   | 7         | Bloqueo de pagos en préstamos cancelados                |
| US-08 | E-04  | 5   | 8         | Prevención de pagos duplicados y concurrencia           |
| US-09 | E-04  | 5   | 9         | Reverso con motivo y trazabilidad                       |
| US-10 | E-04  | 3   | 10        | Suite mínima de pruebas del módulo de pagos             |

**Total de puntos: 41** (9 historias refinadas).

---

## Decisiones de refinamiento (notas internas)

- **US-01 se mantiene como una sola historia (8 pts).** El `want` describe una sola acción cohesiva (registrar y ver distribución). Los seis criterios cubren exacto, parcial, excedente, orden por antigüedad, visibilidad sin recarga y casos con cuota vencida + por vencer. Dividirla en "registrar" y "mostrar desglose" rompe la verificación end-to-end del comportamiento que el operador necesita dar por bueno a la primera vista. Los criterios están lo bastante granulares para que un sprint de una semana la entregue testeable.
- **US-02 se fusiona en US-01.** La regla "primero vencidas, después pendientes desde la más antigua" es el mismo motor de aplicación que US-01; mantenerla como historia separada duplicaba la verificación de la misma regla. Sus dos criterios quedaron integrados como los dos primeros criterios de US-01 ("cuota vencida más antigua primero" y "desde la más antigua a la más reciente"). Se elimina la entrada `US-02` del backlog. El PO debería reflejarlo en `epics.md` cuando haga su próxima pasada (nota: el Developer no reordena prioridades; el PO lo hará en su fase).

## Notas para arquitectura/sprint

Dependencias lógicas detectadas (el Architect las resolverá en su fase; aquí no bloquean INVEST porque el campo `dependencies` del JSON se mantiene vacío):

- **US-03 depende lógicamente de US-01:** no hay comprobante del cliente sin que el pago se haya registrado. La pantalla del comprobante puede construirse sobre los datos que ya produce US-01.
- **US-05 depende lógicamente de US-01 y US-08:** la trazabilidad del saldo a favor se alimenta del motor de aplicación de US-01 y debe coexistir con la idempotencia de US-08 (un pago duplicado no debe generar dos veces saldo a favor).
- **US-06 depende lógicamente de US-01, US-08 y US-09:** la auditoría del supervisor es la vista agregada de los registros que producen esas tres historias.
- **US-09 depende lógicamente de US-06:** el reverso es auditable porque la historia de auditoría ya existe; el reverso añade un movimiento nuevo enlazado al original.
- **US-10 depende lógicamente de US-01, US-07, US-08 y US-09:** la suite valida los escenarios que esas historias implementan; no puede ejecutarse verde si las historias precedentes no existen.

## Supuestos y notas abiertas para el equipo (no bloquean la DoR)

Estos puntos no se declaran como `open_questions` porque el backlog no depende de resolverlos para estar "ready"; se documentan para que el Architect y el Scrum Master los consideren en su fase:

- **Reverso parcial:** la evidencia del inbox no distingue entre reverso total y reverso parcial. US-09 aterriza el reverso total (el comportamiento que aparece citado en `analista-qa.md` y `coordinador-proyectos.md`). Si el negocio requiere reverso parcial de un subconjunto de cuotas de un pago, eso sería una historia nueva (US-09b) que se discutiría en la fase de Sprint Plan.
- **Forma de la autorización excepcional de US-07:** los criterios exigen ID del supervisor autorizante, fecha y motivo, y que se indique al registrar el pago. El detalle de cómo se captura y se persiste esa autorización (pantalla, modelo, flujo) es decisión de arquitectura.
- **Mecanismo de la idempotency key de US-08:** los criterios exigen que el cliente (front) envíe un identificador de operación y que el backend lo dedupe. El detalle técnico (header HTTP, token, etc.) es decisión de arquitectura; aquí se aterriza la regla de negocio.
- **Bloqueo de concurrencia de US-08:** se describe a nivel de comportamiento (otro operador es rechazado mientras el bloqueo esté activo). El mecanismo (lock optimista, pesimista, versionado) es decisión de arquitectura.
- **Migración del historial previo del saldo a favor (Riesgo del mvp-canvas):** US-05 y US-06 asumen que la trazabilidad histórica del saldo a favor es consultable. Si la migración del histórico falla, la primera versión arrancará con saldo a favor desde cero; el comportamiento de las historias no cambia, solo el dato de partida.
