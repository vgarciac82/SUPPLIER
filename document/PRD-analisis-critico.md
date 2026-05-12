# Análisis crítico del PRD — SI-PSP

| Field | Value |
| --- | --- |
| **PRD analizado** | [`PRD.md`](PRD.md) |
| **Especificación fuente** | [`docs/RequirementsSpecification.md`](../docs/RequirementsSpecification.md) |
| **Fecha** | 2026-05-10 |
| **Propósito** | Detectar inconsistencias, ambigüedades y huecos **antes** de derivar ADRs, UX flows y tickets; complementa el **Gate-PRD** ya ejecutado ([`gate-prd.md`](gate-prd.md)). |

---

## Resumen ejecutivo

El PRD es **coherente** con la especificación inicial y **pasó Gate-PRD** tras fijar la máquina de estados en **§2.1.1**. Quedan **riesgos y ambigüedades acotables**: (1) casos límite de identidad/contrato (RFC sin contrato vs vencido), (2) primer mes sugerido sin historial, (3) fallos entre **APPROVED** y **`SIGNED_FINALIZED`**, (4) nomenclatura negocio (español) vs implementación (inglés), (5) TBDs técnicos explícitos (password policy, storage, admin de revisores) que deben cerrarse antes de producción o en fase de hardening.

**Corrección aplicada al PRD en esta misma iteración:** alinear el nombre de estado **`SIGNED_FINALIZED`** en **KPI-4** y **AC-08-2** (antes decía `SIGNED/FINALIZED`).

---

## Hallazgos

| # | Área | Severidad | Descripción | Acción sugerida |
| --- | --- | --- | --- | --- |
| 1 | **US-01 / US-02** | Media | **AC-01-2** remite a **US-02** para RFC “sin contrato activo”, pero **US-02** describe **contrato vencido** (solo lectura + historial). No queda del todo claro el comportamiento cuando **nunca hubo contrato**, **RFC inexistente en GRP**, o **error transitorio** vs “sin vigencia”. | En **PRD** o **UX flow auth**: tabla de resultados GRP → UX (mensaje, código HTTP, si se permite registro parcial). |
| 2 | **US-04** | Media | “**Siguiente mes elegible**” cuando **no hay historial** de informes: la regla está implícita en el motor pero no escrita como AC explícito (el Gate-PRD ya sugirió ADR + golden tests). | **ADR Period engine** + 1–2 AC en **US-04** o anexo de fórmula cuando se redacte el ADR. |
| 3 | **US-06 / US-08** | Media | Si el **polling de firma** falla por timeout o GRP devuelve error terminal, el informe queda en **APPROVED**: no hay transición documentada a un estado tipo **SIGNING_FAILED** ni flujo de reintento manual/automático. | Definir en **PRD** o **ADR Sign**: estado intermedio opcional o política “reintentar desde APPROVED” + límites; UX de recuperación. |
| 4 | **Trazabilidad legal** | Baja | La spec fuente usa estados en español (**BORRADOR**, **EN REVISIÓN**, …); el PRD usa enums en inglés. Riesgo de desalineación en documentación para usuario o contratos. | Tabla de equivalencias en **ADR** o anexo de **UX copy** (una fila por estado). |
| 5 | **US-07** | Baja | “**APPROVED** or signing-eligible report” en **AC-07-1**: en MVP, ¿se permite PDF previo a firma solo para vista previa interna? Límite de re-generación no especificado. | Aclarar: PDF “oficial” solo post-**APPROVED**; vista previa en **DRAFT** opcional y fuera de alcance si no se desea. |
| 6 | **US-04 discard** | Media | **Descartar borrador** borra contenido: puede chocar con **retención** o necesidad de auditoría si el borrador llegó a tocarse con evidencias subidas. | **ADR** o política: soft-delete + retención de blobs huérfanos alineada a **§3.5**; confirmar con legal si aplica. |
| 7 | **Revisor** | Baja | Mismo usuario **PSP** y **revisor** en la lista, o revisor en múltiples contratos: no es incorrecto, pero sin límites de segregación de funciones (SoD) si el negocio lo exige. | Si SoD es obligatorio, añadir **Non-goal** o regla “RFC del PSP no puede estar en allow list de revisor del mismo contrato”. |
| 8 | **Multi-contrato por RFC** | Media | El PRD asume un **contexto de contrato** por RFC; si GRP devuelve **varios** contratos activos, no hay regla de selección. | Pregunta a negocio/GRP: ¿siempre uno activo? Si no, añadir **US** + flujo de “selección de contrato”. |
| 9 | **§3.4** | Baja (hasta prod) | Password policy, cifrado, rol **ADMIN** para allow list: **TBD** — aceptable en MVP interno, bloqueante para go-live público. | Checklist **P3** + owners (Seguridad / IT). |
| 10 | **Evidencia vs spec** | Baja | Spec enfatiza evidencia por fila; **AC-06-2** deja evidencia **opcional** en MVP. Puede haber expectativa de negocio distinta. | Confirmar con PM; si deben ser obligatorias, ajustar **AC-06-2** y KPIs. |

---

## Fortalezas (para balance)

- Trazabilidad **US ↔** `RequirementsSpecification.md`.
- Decisiones **#1–#10** cerradas y reflejadas en AC y **§3.3**.
- **§2.1.1** desbloquea implementación del motor de estados sin ambigüedad de **REJECTED** persistido.
- **Gate-PRD** documentado con evidencia por ítem.

---

## Preguntas abiertas derivadas

| Tema | ¿Ir a `open-questions.md`? | Nota |
| --- | --- | --- |
| Hallazgos **#1, #8** (semántica GRP / multi-contrato) | **Sí**, si negocio no responde en 1 ciclo — reabrir fila con owner PM/GRP | Impacto en **US-01** y modelo de sesión. **DTO de contrato para GRP:** [`integrations/Contrato-GRP-esquema.md`](integrations/Contrato-GRP-esquema.md). |
| Hallazgos **#3, #6** (firma fallida, discard + auditoría) | Opcional en **open-questions**; puede resolverse en **ADR** sin reabrir OQ masivo | Preferir **ADR Sign** + **ADR Storage** |
| Hallazgos **#7, #9, #10** | Seguimiento en hardening / workshop | No bloquean tabla de decisión SDD si quedan documentados |

*No se reabrieron automáticamente las filas cerradas del 2026-05-10; el equipo puede añadir filas nuevas en [`open-questions.md`](open-questions.md) para **#1** y **#8** si se decide.*

---

## Recomendación de siguiente paso (SDD)

1. ~~**ADR-001** (o similar): **Integración GRP** — stub, adaptador, errores surrogate, estrategia de congelación vs contrato real.~~ **Hecho:** [`adrs/ADR-001-grp-integration-adapter.md`](adrs/ADR-001-grp-integration-adapter.md)  
2. ~~**ADR-002**: **Period engine** — primer mes sin historial, golden fixtures, límites de `M`.~~ **Hecho:** [`adrs/ADR-002-period-engine-monthly-weeks.md`](adrs/ADR-002-period-engine-monthly-weeks.md)  
3. ~~**ADR-003**: **Sign + Reception** — polling, timeouts, fallo desde **APPROVED**, `receptionAttemptId`.~~ **Hecho:** [`adrs/ADR-003-grp-sign-reception-polling.md`](adrs/ADR-003-grp-sign-reception-polling.md)  
4. **UX flow** (si se desea): autenticación + resultado GRP (**hallazgo #1**); opcionalmente ciclo completo del informe.

---

## Referencias cruzadas

| Artefacto | Uso |
| --- | --- |
| [`PRD.md`](PRD.md) | Fuente normativa de producto |
| [`gate-prd.md`](gate-prd.md) | Checklist Gate-PRD |
| [`open-questions.md`](open-questions.md) | Registro de preguntas cerradas / futuras |
| [`discovery-gate.md`](discovery-gate.md) | Cierre Fase 0 |
