---
description: El Developer refina las historias candidatas hasta que cumplan INVEST + Definition of Ready. Pasa por el gate dor-invest.
argument-hint: "<carpeta del delivery, p. ej. deliveries/citasalud>"
---

# Refinar historias (INVEST + Definition of Ready)

**Delivery:** `$ARGUMENTS`
Prerrequisito: `$ARGUMENTS/outputs/backlog.json` (de `/delivery:generate-epics`).

Pasos:
1. Lanza el subagente **developer** (herramienta Task) para refinar cada historia
   del `backlog.json`: criterios de aceptación en Gherkin, estimación en puntos,
   división de las que superen 8 pts, resolución de dependencias/preguntas abiertas.
   Debe **actualizar `backlog.json`** con los campos completos y escribir
   `$ARGUMENTS/outputs/stories.md`.
2. (Opcional) Lanza el **scrum-master** para una pasada de Definition of Ready
   antes de escribir.
3. Al escribir `stories.md` se ejecuta el hook **dor-invest-gate**. Si **bloquea**,
   NO evadas: lee el motivo, corrige las historias señaladas (criterios faltantes,
   sin estimación, demasiado grandes, formato incompleto) y reintenta.
4. Cuando pase, resume: cuántas historias quedaron listas y los puntos totales.

Recuerda: una historia vaga no es testeable. Reescríbela o pártela.