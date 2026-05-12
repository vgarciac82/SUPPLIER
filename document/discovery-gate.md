# Discovery Gate — Cierre Fase 0 (SI-PSP)

| Field | Value |
| --- | --- |
| **Proceso** | SDD — `ai-specs-master/ai-specs/specs/flujo-desarrollo-standards.md` (Fase 0) |
| **Fecha de cierre documental** | 2026-05-10 |
| **Fuentes** | [`docs/RequirementsSpecification.md`](../docs/RequirementsSpecification.md), [`document/PRD.md`](PRD.md) |

Este artefacto **cierra la Fase 0** registrando las respuestas a las preguntas núcleo del Discovery Gate. Las respuestas se basan en la especificación inicial acordada y en el PRD derivado; lo no sustentado allí permanece como **TBD** en el PRD o en [`open-questions.md`](open-questions.md) (regla anti-alucinación SDD).

---

## 1. Preguntas núcleo (mínimo 2) — Respuestas

### 1.1 Core problem — ¿Qué dolor resolvemos y qué pasa si no lo hacemos?

**Respuesta**

- Los PSP deben entregar **informes mensuales estructurados**, alineados al contrato, con **evidencia**, **revisión** y **firma electrónica** antes del pago. Sin SI-PSP, el proceso es **fragmentado**, propenso a **texto libre incoherente**, **periodos incorrectos**, **duplicidad o saltos de meses**, y depende de coordinación manual con **GRP**.
- Si no se implementa: se degrada la **calidad y auditabilidad** de los informes y el **tiempo hasta pago** (riesgo operativo y cumplimiento frente al flujo oficial GRP).

**Trazabilidad:** `docs/RequirementsSpecification.md` (§1–§6, Resumen General); `document/PRD.md` §1.1.

---

### 1.2 Success metrics — ¿Cómo medimos éxito (KPIs) y contra qué baseline?

**Respuesta**

- Criterios medibles propuestos en el PRD: **correctez de periodos** (muestra de auditoría / golden fixtures), **100% de líneas ligadas al catálogo**, **tiempo de ciclo de revisión** (mediana; baseline y meta **TBD** con negocio), **tasa de finalización firma E2E** (**TBD** con SLA GRP), **incidentes de duplicado/fuera de secuencia** (objetivo 0 críticos; definición de “crítico” **TBD**).

**Trazabilidad:** `document/PRD.md` §1.3 (KPI-1 a KPI-5); refinamiento de baseline y owners en [`open-questions.md`](open-questions.md) ítem 1.

---

### 1.3 Constraints — ¿Restricciones de tiempo, costo, stack, dependencias, integraciones?

**Respuesta**

- **Integración obligatoria con GRP:** consulta por RFC, firma de PDF, recepción / listo para pago (`docs/RequirementsSpecification.md` §6).
- **Identidad del prestador:** **RFC** como identificador único (misma fuente §1).
- **Stack acordado en repo:** backend **Spring Boot 4.x**, **Java 25**, **PostgreSQL** (`pom.xml` del proyecto `supplier`). Frontend **TBD** en PRD.
- **Dependencia crítica:** entrega de **contratos/API** y ambientes GRP (sandbox, auth, SLAs) — **TBD**; riesgo registrado en PRD §4.2.

**Trazabilidad:** especificación §1 y §6; `document/PRD.md` §3.3–§3.5; `open-questions.md` ítem 2.

---

### 1.4 Alcance — ¿Qué incluye y qué NO incluye (non-goals)?

**Respuesta**

- **Incluye (MVP lógico según especificación):** acceso por RFC y validación de contrato; catálogo único de actividades por contrato; reportes mensuales con **corte** y **semanas** según fecha de inicio; captura tabular con evidencias; **máquina de estados** del informe; PDF estándar en cuatro secciones; orquestación hacia **firma GRP** y notificación de **listo para pago**.
- **No incluye (non-goals del PRD):** reemplazar GRP; ejecutuar pago completo dentro de SI-PSP; actividades en texto libre en líneas mensuales; offline-first móvil; multi-tenant más allá del alcance acordado (**TBD** si se expande).

**Trazabilidad:** `docs/RequirementsSpecification.md` global; `document/PRD.md` §2.3 (Non-Goals).

---

## 2. Criterio de cierre Fase 0

| Criterio | Estado |
| --- | --- |
| Al menos **2** preguntas núcleo respondidas con fuente explícita | **Cumplido** (se documentaron las 4) |
| Supuestos no sustentados marcados como **TBD** / Open Questions | **Cumplido** (PRD + `open-questions.md`) |
| Listo para **Gate-PRD** sobre el texto del PRD | **Sí** — proceder a checklist Gate-PRD |

---

## 3. Firma / reconocimiento (opcional)

| Rol | Nombre | Fecha | Firma / comentario |
| --- | --- | --- | --- |
| Product / PM | *Pendiente* |  |  |
| Negocio / Stakeholder | *Pendiente* |  |  |
| Líder técnico | *Pendiente* |  |  |

*Hasta completar la tabla, el cierre es **documental**; la firma formal valida alineación con negocio.*

---

## 4. Referencias cruzadas

- PRD: [`PRD.md`](PRD.md) — campo **Discovery Gate** en la cabecera.
- Preguntas abiertas derivadas del descubrimiento: [`open-questions.md`](open-questions.md).
