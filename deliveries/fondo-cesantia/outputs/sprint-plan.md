# Sprint 1 — El operador registra un pago contra un préstamo activo y ve la distribución en la misma pantalla, el cliente obtiene comprobante e historial legibles, y los préstamos cancelados bloquean el pago.
**Capacidad:** 20 pts · **Comprometido:** 20 pts
| Historia | Pts | Épica | Prioridad |
|----------|-----|-------|-----------|
| US-01    | 8   | E-01  | 1         |
| US-03    | 5   | E-02  | 3         |
| US-04    | 5   | E-02  | 4         |
| US-07    | 2   | E-04  | 7         |

**Historias comprometidas:** US-01, US-03, US-04, US-07. Total = 20 pts.

---

## Justificación del Sprint Goal

El slice elegido es el único que entrega un flujo verificable de extremo a extremo (operador → cliente) y que, además, mueve la métrica principal declarada en `epics.md` ("caída de reclamos por 'pago no reflejado' en 7 días"): esa métrica solo se mueve si el cliente ve comprobante y puede consultar su historial. Incluir US-07 dentro del sprint es deliberado: a 2 pts blinda el dolor histórico "defectos-pagos-parciales" sin consumir capacidad significativa, y de lo contrario quedaría rezagado detrás de E-03 cuando su naturaleza es defensiva y pertence a E-04. Los slices B y C sacrifican la cara cliente por trabajo interno o defensivo, lo que retrasa la señal de valor del producto.

## Riesgos del sprint

- **US-01 concentra el riesgo del sprint.** A 8 pts es la pieza más grande y la única que crea el motor de aplicación; si se atrasa, el Sprint Goal completo está en juego porque US-03 y US-04 leen datos que produce.
- **US-04 tiene carga de UX no trivial.** El criterio exige "lenguaje claro, sin tecnicismos financieros"; si el equipo no tiene claro el modelo de etiquetas desde el inicio, puede generar retrabajo de presentación.
- **US-07 lee el estado del préstamo desde el motor de US-01.** Aunque su estimación es baja (2 pts), depende de que la consulta de estado del préstamo exista; conviene empezar US-07 solo cuando US-01 ya tenga el endpoint de lectura disponible.

## Notas para el equipo

- **Dependencias lógicas detectadas en `stories.md`** (informativas; el campo `dependencies` del backlog se mantiene vacío porque se resuelven dentro del mismo sprint):
  - US-03 lee datos que produce US-01 (sin pago no hay comprobante).
  - US-04 lee historial que produce US-01.
  - US-07 consulta el estado del préstamo que también usa US-01.
  - **Orden sugerido de construcción interno:** US-01 → US-03 → US-04 → US-07 (coincide con la prioridad y la dependencia lógica).
- **Supuestos abiertos documentados en `stories.md`** (no bloquean DoR, pero el Architect los resolverá en su fase): forma de captura de la autorización excepcional de US-07, mecanismo de la idempotency key, mecanismo de bloqueo de concurrencia, migración del historial previo del saldo a favor.
- **Recordatorio del Developer:** US-02 está fusionada en US-01 desde refinamiento; no existe como historia separada en el backlog.

## Lo que queda fuera (siguiente sprint)

| Historia | Pts | Épica | Prioridad | Motivo                                                   |
|----------|-----|-------|-----------|----------------------------------------------------------|
| US-05    | 3   | E-03  | 5         | Trazabilidad del saldo a favor — depende de US-01 ya hecho |
| US-06    | 5   | E-03  | 6         | Reconstrucción auditable — depende de US-01/US-08/US-09  |
| US-08    | 5   | E-04  | 8         | Idempotencia + concurrencia — blindaje defensivo         |
| US-09    | 5   | E-04  | 9         | Reverso con motivo — depende de la auditoría (US-06)     |
| US-10    | 3   | E-04  | 10        | Suite mínima — valida escenarios de US-01/07/08/09       |
| **Total fuera** | **21** | — | — | Capacidad sugerida para Sprint 2: 20 pts                 |

**Nota explícita:** el MVP completo requiere más de un sprint. El backlog total es 41 pts; con capacidad de 20 pts/sprint se necesitan al menos 3 sprints para entregar el núcleo + auditoría + blindajes de producción.