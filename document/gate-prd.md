# Gate-PRD — Resultado (SI-PSP)

| Field | Value |
| --- | --- |
| **Estándar** | `ai-specs-master/ai-specs/specs/flujo-desarrollo-standards.md` (Fase 2), `prd-requirements-standards.md` |
| **PRD evaluado** | [`PRD.md`](PRD.md) |
| **Fecha** | 2026-05-10 |
| **Resultado global** | **Passed** — todos los ítems del checklist en **Pass**; ajuste menor aplicado al PRD (§2.1.1 máquina de estados explícita) |

---

## Checklist Gate-PRD (6 ítems)

Criterios tomados del flujo SDD. Evidencia = sección del PRD o nota de cierre.

| # | Ítem | Criterio (resumen) | Resultado | Evidencia / notas |
| --- | --- | --- | --- | --- |
| 1 | **Semántica unívoca** | Reglas de negocio: qué se calcula, cómo y cómo se combina el resultado | **Pass** | Corte **D** vs último día de **M** (**US-04 AC-04-1**); semanas **US-04 AC-04-2**; TZ **`America/Mexico_City`** + UTC; catálogo inmutable (**US-03**); un borrador por `(contract, month)` (**AC-04-3**); límites de evidencia (**US-05**). Detalle fino del motor (primer mes elegible si no hay historial) se puede fijar en **ADR Period engine** + tests dorados (**KPI-1**); no bloquea semántica base. |
| 2 | **Estructura del resultado** | Entidades → atributos → valores esperados | **Pass** | **§3.2** (Contract context, Activity, MonthlyReport, PeriodRow, Evidence); estados en **§2.1.1**; secciones PDF (**US-07**). |
| 3 | **Condiciones “no aplica”** | Comportamiento cuando ninguna regla coincide / datos faltantes | **Pass** | Mes fuera de secuencia → rechazo (**AC-04-4**); cabecera PDF incompleta → bloqueo (**AC-07-2**); RFC sin contrato activo → **US-01 / US-02**; GRP no parseable → error sin cuenta (**AC-01-3**). |
| 4 | **Manejo de errores** | Qué falla, cómo se reporta, continuar vs abortar | **Pass** | AC con errores de validación (`409`/mensajes); **§3.5** (retries, outbox, uploads); login GRP fallo → abort crear cuenta (**AC-01-3**). Detalle “códigos GRP reales” sigue en dependencia externa; stub define errores surrogate (**§3.3**). |
| 5 | **Idempotencia** | Re-ejecución: reemplazo vs acumulación vs bloqueo | **Pass** | Catálogo (**AC-03-4**); un draft/mes (**AC-04-3**); segundo submit (**AC-06-6**); `receptionAttemptId` (**AC-08-4**); polling sign (**AC-08-1**). |
| 6 | **Estados / transiciones** | Todas las transiciones (reaperturas, bloqueos) | **Pass** | Tras ambigüedad “REJECTED” vs **DRAFT**, el PRD incorpora **§2.1.1** con estados persistidos y tabla de transiciones; rechazo → **DRAFT** con auditoría; fuera de alcance MVP explícito (withdraw, reopen). |

---

## Acciones tomadas en esta revisión

1. Añadido al PRD **`§2.1.1 MonthlyReport — state machine (normative)`** con estados persistidos y tabla de transiciones.
2. Ajustados **AC-06-3** y **AC-06-4** para alinear rechazo con **DRAFT** + auditoría (sin estado persistido **REJECTED** separado).
3. Actualizada la cabecera del **PRD** con fila **Gate-PRD (Fase 2): Passed** y enlace a este archivo.

---

## Condiciones de seguimiento (no revierten el Pass)

| Tema | Acción recomendada |
| --- | --- |
| Contrato GRP real | **ADR** de integración cuando exista OpenAPI/credenciales. |
| Primer mes sugerido sin historial | Definición explícita en **ADR Period engine** + fixtures **KPI-1**. |
| Password policy / encryption / admin de allow list | **TBD** en PRD §3.4 — cerrar antes de producción o en hardening (**P3**). |

---

## Regla SDD

Si en el futuro un cambio al PRD **rompe** alguno de estos ítems, repetir Gate-PRD antes de derivar UX/ADRs/tickets afectados.
