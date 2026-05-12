# Open Questions — SI-PSP

Companion to [`PRD.md`](PRD.md).

---

## Cómo se “cierran” las preguntas (y por qué no todas a la vez)

**Cerrar Fase 0 (Discovery)** ya está hecho: significa que el *problema, KPIs a alto nivel, restricciones y alcance* están anclados con fuente (`discovery-gate.md` + especificación + PRD).

**Cerrar cada fila de esta tabla** es otro paso: cada una es un **hueco de información** que solo se cierra cuando alguien con **autoridad** (negocio, GRP, arquitectura, seguridad) **toma una decisión** o **entrega un artefacto** (contrato de API, política, número). No se inventan en el vacío.

### Ciclo por pregunta (repetir hasta `Closed`)

1. **Asignar owner** — una persona o equipo responsable de *conseguir* la respuesta (ej. “GRP team”, “PM Cherlo”, “Tech lead”).
2. **Clarificar impacto** — qué se rompe o se bloquea si sigue abierta (ej. “bloquea ADR de integración”).
3. **Obtener decisión** — reunión, correo, doc de GRP, workshop de reglas de negocio.
4. **Registrar la respuesta** en esta tabla: columna **Resolution** (texto concreto: regla, número, enlace al doc).
5. **Propagar al PRD** — actualizar `PRD.md` (KPIs, AC, Technical Specifications) para que el PRD deje de decir TBD en ese punto.
6. **Marcar Status = Closed** y **Closed date** cuando 4 y 5 estén hechos.

No hace falta que las **10** estén en **Closed** antes de avanzar *algo* de diseño: lo habitual es **priorizar** (ver tabla abajo). Lo que el flujo SDD exige es que **no haya TBDs bloqueantes** sin dueño ni plan cuando pases **Gate-PRD** o cuando abras un ticket para implementar esa parte.

### Prioridad sugerida (para no sentirte bloqueado)

| Prioridad | Idea | Ejemplo en esta lista |
| --- | --- | --- |
| **P0 — Bloquea Gate-PRD o MVP** | Sin respuesta no puedes definir semántica verificable o contrato de sistema | #2 (GRP), #3–#4 (motor de fechas), #6 (idempotencia de borrador) |
| **P1 — Bloquea UX o permisos** | Afecta pantallas o roles | #7, #10 |
| **P2 — NFR / operación** | Se puede acordar antes de prod pero conviene antes de hardening | #8, #9 |
| **P3 — Medición** | Mejora KPIs; a veces se cierra cuando ya hay operación | #1 |

Puedes marcar explícitamente **Deferred** con *hasta cuándo* o *hasta qué gate* si negocio acuerda posponer (y entonces el PRD debe decir “comportamiento provisional: …”).

---

## Registro de preguntas

| # | Topic | Priority | Status | Owner | Impact (short) | Resolution (decisión o enlace; vacío si Open) | PRD / other updated | Closed date |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | KPI baselines and owners (KPI-3, KPI-4, KPI-5) | P3 | Closed | PM + Tech lead | KPIs no auditables contra baseline | **Opción A:** Baseline **desde go-live** (primer mes completo en producción para agregar métricas); owners **PM** (negocio) + **Tech lead** (instrumentación). Metas numéricas de KPI-3/4 se **fijan** tras el mes 1 de operación. | PRD §1.3 | 2026-05-10 |
| 2 | GRP OpenAPI/schemas, auth, SLAs, sandbox, error codes | P0 | Closed | Tech lead + GRP | Bloquea ADR integración y pruebas E2E | **Opción B:** **Mock-driven:** implementar y probar contra **OpenAPI interno / stub** congelado; **ADR** documenta alineación o delta cuando GRP entregue contrato real. Auth/SLA/códigos finales siguen del proveedor GRP cuando existan. | PRD §3.3, §4.2 | 2026-05-10 |
| 3 | Cutover when start day D missing in month M (e.g. Jan 31 → Feb) | P0 | Closed | Stakeholder | Motor de periodos / tests dorados | **Opción A:** Si el día **D** no existe en el mes **M**, el día de corte es el **último día natural del calendario** de **M** (28, 29, 30 o 31 según corresponda). | PRD US-04 | 2026-05-10 |
| 4 | Timezone for calendar cutover | P0 | Closed | Stakeholder | Alineación fecha límite / cortes | **Opción A:** Zona fija de negocio **`America/Mexico_City`** para interpretar “día civil” de corte y periodos. Persistencia de instantes en **UTC** en backend (implementación). | PRD §3.6, US-04 | 2026-05-10 |
| 5 | Policy for editing activity catalog after first completion | P1 | Closed | Stakeholder | UX + reglas de datos maestros | **Opción A:** Catálogo **inmutable** tras marcarse “completado” (sin edición ni borrado de actividades usadas como maestro). Correcciones posteriores vía **cambio de proceso / soporte** fuera de MVP si negocio lo exige. | PRD US-03 | 2026-05-10 |
| 6 | One vs multiple drafts per month | P0 | Closed | Stakeholder | Idempotencia y flujo de envío | **Opción A:** **Un solo borrador activo** por par (`contractId`, mes de reporte). “Nuevo” para ese mes **reanuda** el existente o **descartar** explícitamente con confirmación (definir copia en PRD si aplica). | PRD US-04 | 2026-05-10 |
| 7 | Reviewer assignment and permissions | P1 | Closed | PM + Stakeholder | Modelo de seguridad y pantallas | **Opción A:** **Lista fija de revisores por contrato o proyecto** en SI-PSP (configuración); solo esos usuarios pueden aprobar/rechazar informes de ese alcance. Sincronización desde GRP **fuera de MVP** salvo que ya exista el dato. | PRD §2.1, §3.4 | 2026-05-10 |
| 8 | Evidence limits (size, count, formats) and retention | P2 | Closed | Tech lead + PM | Límites upload, PDF, compliance | **Opción A (MVP):** Formatos **JPEG, PNG, WebP**; máx **5 MB** por archivo; máx **5 imágenes** por fila de periodo; retención de adjuntos e informes **vida del contrato + 1 año** (luego política corporativa puede ajustar). | PRD US-05, US-02 | 2026-05-10 |
| 9 | Async signing: polling vs webhook (GRP) | P2 | Closed | Tech lead | Diseño de colas / timeouts | **Opción A:** **Polling** con backoff exponencial hasta estado terminal o timeout global; intervalos y timeout máximo en **ADR / NFR** (p. ej. poll cada 30s–5m, timeout 24h salvo otro acuerdo con GRP). | PRD US-08 | 2026-05-10 |
| 10 | Read-only mode: full vs partial history | P1 | Closed | Stakeholder | Alcance consulta contrato vencido | **Opción C:** **Misma UI** que contrato activo con **acciones de mutación deshabilitadas**; el PSP ve **historial completo** de informes (todos los estados) en solo lectura. Endpoints de escritura denegados (`403`). | PRD US-02 | 2026-05-10 |

---

## Opciones A / B / C (trade-offs)

Tabla ampliada por cada pregunta: **[`decision-options.md`](decision-options.md)**.

## Cómo te puedo ayudar en Cursor

- **Facilitar decisiones:** opciones A/B/C en [`decision-options.md`](decision-options.md); eliges letra o variante y propagamos a esta tabla y al PRD.
- **Redactar Resolution:** cuando tengas la decisión verbal, la dejo en texto firme en esta tabla y en el PRD.
- **Separar “cerrado” de “diferido”:** si algo se pospone, documentamos el comportamiento provisional en el PRD para no bloquear tickets.

Cuando quieras cerrar una fila concreta, dime por ejemplo: *“cerramos la #3 así: último día hábil del mes”* (o lo que acuerden) y actualizo **Resolution**, **PRD**, y **Closed date**.
