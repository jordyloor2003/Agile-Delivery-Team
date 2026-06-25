# Agile Delivery Team

Equipo ágil de entrega impulsado por IA que convierte el descubrimiento de un producto en un **plan de entrega listo para construir**: épicas, backlog priorizado, historias refinadas, arquitectura con ADRs y Sprint Plan.

No escribe código de producción. **Planifica el delivery** con criterio ágil.

---

## El equipo

| Rol | Responsabilidad | Pregunta clave |
|-----|----------------|----------------|
| **Product Owner** | Descompone el MVP en épicas y ordena el backlog por valor | ¿Estamos construyendo lo correcto? |
| **Developer** | Refina historias hasta cumplir INVEST + DoR | ¿Cómo lo construimos bien? |
| **Scrum Master** | Custodia la Definition of Ready y el flujo del backlog | ¿Qué frena al equipo? |
| **Architect** | Decide la estructura técnica y registra el porqué en ADRs | ¿Qué estructura sostiene esto en el tiempo? |

---

## Flujo de trabajo

```
/delivery:generate-epics <delivery>    → PO: epics.md + backlog.json
/delivery:generate-stories <delivery>  → Developer + SM: stories.md   (gated por DoR/INVEST)
/delivery:architecture <delivery>      → Architect: architecture.md + ADRs
/delivery:sprint-plan <delivery>       → SM + Developer: sprint-plan.md (gated)
/delivery:report <delivery>            → Reporte HTML visual del delivery
```

Cada comando toma como insumo el `inbox/` del delivery. Si está vacío, copia ahí las salidas del Discovery Agent antes de continuar.

---

## Estructura de un delivery

```
deliveries/<nombre>/
├── inbox/                      ← entradas del Discovery Agent (solo lectura)
│   ├── mvp-canvas.md
│   ├── user-stories.md
│   ├── requisitos.md
│   ├── personas.md
│   └── evidence-map.json
└── outputs/                    ← artefactos generados por el equipo
    ├── epics.md                (Product Owner)
    ├── backlog.json            (Product Owner · lo lee el gate y el reporte)
    ├── stories.md              (Developer · GATED por DoR/INVEST)
    ├── architecture.md         (Architect)
    ├── adr/ADR-NNNN-*.md      (Architect)
    ├── sprint-plan.md          (Scrum Master/Developer · GATED)
    ├── sprint-plan.json        (estructurado)
    └── report.html             (generado por scripts/build-report.py)
```

---

## Reglas no negociables

1. **Cero invención.** Todo debe trazarse al `inbox/`. Si el descubrimiento no respalda algo, se declara como `open_questions`, no como hecho.
2. **Trazabilidad.** Épica → MVP · Historia → requisito/user story · ADR → fuerza que lo motiva.
3. **INVEST + DoR es ley.** El hook `dor-invest-gate.py` bloquea `stories.md` y `sprint-plan.md` si alguna historia no cumple. No hay forma de evitarlo.
4. **Valor antes que solución.** Prioridad por valor (output → outcome → impact), no por facilidad técnica.
5. **Aislamiento.** Cada delivery vive en `deliveries/<nombre>/`. No se mezclan datos entre deliveries.

---

## Inicio rápido

```bash
# 1. Crear la estructura del delivery
mkdir -p deliveries/mi-producto/inbox
mkdir -p deliveries/mi-producto/outputs

# 2. Copiar las salidas del Discovery Agent al inbox
cp /ruta/discovery/* deliveries/mi-producto/inbox/

# 3. Ejecutar el flujo completo en Claude Code
/delivery:generate-epics mi-producto
/delivery:generate-stories mi-producto
/delivery:architecture mi-producto
/delivery:sprint-plan mi-producto
/delivery:report mi-producto
```

El reporte HTML final se genera en `deliveries/mi-producto/outputs/report.html`.

---

## Ejemplo incluido

El delivery `fondo-cesantia` contiene un ejemplo completo con inbox y outputs generados, útil como referencia del formato esperado en cada artefacto.