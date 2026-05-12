# Opciones de decisión (A / B / C) — Open Questions SI-PSP

Documento de facilitación para cerrar [`open-questions.md`](open-questions.md). Cada sección mapea al **#** de esa tabla.

**Cómo usarlo:** elige **A**, **B** o **C** (o una variante) por número; luego esa elección se copia a **Resolution** en `open-questions.md` y al [`PRD.md`](PRD.md).

---

## #1 — KPI baselines y owners (KPI-3, KPI-4, KPI-5)

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Baseline “desde go-live”**: medir solo a partir del primer mes en producción; owners = PM + Tech lead. | Simple; no necesitas datos históricos manuales. | No comparas con “antes”; stakeholders pueden pedir línea base manual después. |
| **B** | **Baseline documental**: PM entrega números del proceso actual (encuesta/tiempos aprox.) una sola vez. | Negocio ve mejora vs “antes”. | Costo de recopilar datos que a veces no existen; riesgo de baseline poco riguroso. |
| **C** | **Diferir KPIs finos**: dejar KPI-3/4/5 como *TBD hasta 90 días post-MVP*; solo exigir KPI-1/2 en Gate-PRD. | Desbloquea implementación sin inventar metas. | Menos presión de resultado temprano; hay que calendarizar el cierre. |

**Sugerencia MVP:** **C** para no bloquear, con fecha de cierre de KPIs; o **A** si quieren métricas desde día uno sin trabajo extra de baseline histórico.

---

## #2 — GRP: OpenAPI, auth, SLAs, sandbox, códigos de error

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Contrato primero**: no arrancar integración real hasta OpenAPI firmado + sandbox; hasta entonces solo diseño y mocks. | Evita retrabajo por API volátil. | Puede retrasar E2E real si GRP tarda. |
| **B** | **Mock-driven + congelación**: implementar contra **OpenAPI interno** (stub) y ADR que declare “GRP debe alineararse a este contrato o negociar delta”. | Avanza backend/QA en paralelo. | Riesgo de desalineación si GRP entrega algo distinto; hace falta proceso de reconciliación. |
| **C** | **Spike + adaptador**: una iteración corta contra lo que sea que GRP tenga hoy (Postman/WSDL), encapsulado en un **adapter**; refactor cuando llegue el contrato formal. | E2E temprano si ya hay *algo*. | Deuda si el adapter queda como “producción temporal”. |

**Sugerencia MVP:** **B** (velocidad controlada) si GRP no está listo; **A** si ya hay compromiso de entrega de contrato en fecha fija.

---

## #3 — Corte mensual cuando el día **D** no existe en el mes **M** (ej. inicio 31 ene → febrero)

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Último día natural del mes**: si D no existe, usar 28/29/30/31 según corresponda. | Predecible; común en sistemas de facturación. | “Corte” ya no es siempre el mismo número de día visual; hay que explicarlo al PSP. |
| **B** | **Congelar siempre en D cuando existe, y si no existe usar D-1 recursivamente** (ej. 31→30→29…). | Intuición “retrocede hasta día válido”. | Más reglas y edge cases en tests; febrero bisiesto requiere cuidado. |
| **C** | **Meses afectados usan el mismo “áncla” que GRP**: copiar la regla exacta del ERP/GRP (documentada). | Una sola verdad de negocio con pagos. | Depende de que GRP documente la regla; si no, bloqueado. |

**Sugerencia MVP:** **A** (clara y testeable) salvo que **C** esté disponible y sea distinta.

---

## #4 — Zona horaria para el “día de corte”

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Zona fija del negocio** (ej. `America/Mexico_City`) para todos los PSP. | Implementación simple; un solo calendario. | Si hubiera operación multi-región, puede discutirse. |
| **B** | **Zona por sede/contrato** (dato desde GRP). | Alineado a operación real multi-sede. | Más complejidad; GRP debe exponer el dato. |
| **C** | **UTC internamente + presentación local** según A o B. | Buena práctica técnica; evita bugs de DST si se acota bien. | Misma decisión de negocio que A o B sigue siendo necesaria. |

**Sugerencia MVP:** **A** + **C** (almacenar instantes en UTC, interpretar “día civil” en la TZ de negocio fija).

---

## #5 — Política de edición del catálogo de actividades tras “completado”

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Inmutable** tras primer reporte enviado o tras marcar catálogo “cerrado”. | Máxima coherencia histórica en PDFs. | Errores de captura requieren soporte/admin o “versión 2” del catálogo (nuevo concepto). |
| **B** | **Editable con historial** (cambios versionados; reportes viejos referencian versión usada). | Flexibilidad sin romper histórico. | Más modelo de datos y UI. |
| **C** | **Editable libre** mientras no haya informe **APROBADO**; bloqueo después. | Balance simple. | Inconsistencia si hay informes en revisión con actividad renombrada (hay que definir snapshot). |

**Sugerencia MVP:** **C** (simple) o **A** (si negocio prioriza trazabilidad estricta); **B** si anticipan cambios frecuentes al contrato.

---

## #6 — Un borrador vs varios borradores por mes

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Un solo borrador activo** por (contrato, mes); “nuevo” reanuda o descarta explícitamente. | Evita duplicados; idempotencia clara. | Menos flexibilidad si quieren experimentar en paralelo. |
| **B** | **Varios borradores** pero solo **uno** puede enviarse a revisión; los demás archivados o merge manual. | Permite borradores experimentales. | UX y reglas más pesadas; riesgo de confusión. |
| **C** | **Un borrador + versiones internas** (autosave/historial sin múltiples entidades “report”). | UX fluida; un `reportId` estable. | Costo de implementar historial si se expone. |

**Sugerencia MVP:** **A** o **C**; evitar **B** salvo requisito explícito.

---

## #7 — Asignación y permisos del revisor

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Lista fija por contrato/proyecto** (roles `REVIEWER` asignados en SI-PSP o sincronizados desde GRP). | Control fino; auditable. | Mantenimiento de listas; depende de dato maestro. |
| **B** | **Cualquier usuario con rol global “Área técnica”** ve la cola de todos los contratos. | Menos configuración inicial. | Riesgo de privacidad/alcance si la cola es masiva; filtrado obligatorio. |
| **C** | **Híbrido**: revisor “por defecto” del proyecto + suplentes; escalación si no hay respuesta en N días (**N TBD**). | Operación realista. | Más reglas y posible SLA interno. |

**Sugerencia MVP:** **A** si GRP o negocio ya tiene “responsable de proyecto”; si no, **B** con filtros estrictos por proyecto/contrato.

---

## #8 — Límites de evidencia (tamaño, cantidad, formatos) y retención

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Cuotas fijas MVP**: ej. JPG/PNG/WebP, máx 5 MB por archivo, máx 5 imágenes por fila, retención = vida del contrato + 1 año. | Fácil de implementar y comunicar. | Puede quedarse corto o largo vs necesidad real. |
| **B** | **Cuotas por plan/tipo de contrato** (tabla configurable). | Flexible sin redeploy. | Necesita UI admin o datos de configuración. |
| **C** | **Alineado a política corporativa de datos** (DPO define números). | Compliance fuerte. | Depende de stakeholder legal; puede retrasar. |

**Sugerencia MVP:** **A** con números conservadores; revisión legal puede ajustar después (**C**).

---

## #9 — Firma asíncrona: polling vs webhook (GRP)

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Polling** desde SI-PSP (job o front con backoff) hasta estado terminal. | Funciona aunque GRP no llame de vuelta; simple detrás de firewalls propios. | Más carga y latencia; definir intervalos y timeout. |
| **B** | **Webhook/callback** de GRP a endpoint SI-PSP. | Eficiente; estado en tiempo casi real. | Exige URL pública, seguridad (firma HMAC), reintentos de GRP. |
| **C** | **Híbrido**: iniciar con polling; migrar a webhook cuando GRP lo soporte. | Pragmático en integraciones lentas de madurar. | Dos caminos de código a mantener brevemente. |

**Sugerencia MVP:** **A** o **C**; **B** si GRP ya lo ofrece de forma estable y documentada.

---

## #10 — Modo consulta con contrato vencido: historial completo vs parcial

| Opción | Descripción | Pros | Contras |
| --- | --- | --- | --- |
| **A** | **Historial completo** de informes del PSP (todos los estados) solo lectura. | Mejor experiencia y auditoría para el PSP. | Mayor superficie de exposición de datos; revisar retención. |
| **B** | **Solo últimos N meses** o solo **FIRMADO/FINALIZADO**. | Menor riesgo y menos consultas pesadas. | El PSP puede no ver borradores viejos o rechazos lejanos. |
| **C** | **Igual que activo pero sin acciones** (misma UI, botones deshabilitados). | Menos bifurcación de UI. | Hay que asegurar que ningún endpoint permita mutación. |

**Sugerencia MVP:** **C** con reglas de datos de **A** o **B** según compliance (si hay duda legal, **B** conservador).

---

## Resumen rápido (si quieres un “paquete” MVP por defecto)

| # | Paquete MVP sugerido |
| --- | --- |
| 1 | **C** (calendarizar cierre de KPIs) o **A** |
| 2 | **B** + fecha objetivo de contrato real GRP |
| 3 | **A** (si no hay regla GRP) |
| 4 | **A** + **C** |
| 5 | **C** |
| 6 | **A** o **C** |
| 7 | **A** si hay “responsable”; si no **B** con filtro por proyecto |
| 8 | **A** |
| 9 | **A** o **C** |
| 10 | **C** + alcance de datos tipo **A** salvo legal pida **B** |

---

## Siguiente paso

Responde en una sola línea por número, por ejemplo:  
`#3=A, #4=A+C, #6=A, …`  

Con eso actualizo `open-questions.md` (**Resolution**, **Closed date**) y las secciones correspondientes del **PRD**.

---

## Aplicado en el repositorio

| Fecha | Elección |
| --- | --- |
| **2026-05-10** | **1A 2B 3A 4A 5A 6A 7A 8A 9A 10C** — propagado a [`open-questions.md`](open-questions.md) y [`PRD.md`](PRD.md). |
