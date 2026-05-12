# ADR-001 — Integración GRP: adaptador, stub y contrato API

## Metadatos

| Campo | Valor |
| --- | --- |
| ID | ADR-001 |
| Fecha | 2026-05-10 |
| Estado | Aceptado |
| Decisores | Equipo SI-PSP (pendiente ratificación formal) |
| PRD Ref | US-01, US-02, US-07, US-08; §3.3 |
| Ticket Redmine | N/A |
| Producto | Decisión **#2-B** — [`open-questions.md`](../open-questions.md) |

---

## Contexto

SI-PSP depende de **GRP** para consulta de contrato por RFC, firma de PDF y notificación de recepción. El **OpenAPI oficial**, credenciales y SLAs de GRP **aún no están** disponibles para el arranque del desarrollo.

Sin una decisión explícita, el equipo implementaría contra APIs inestables o bloquearía el avance hasta la entrega de GRP.

Restricciones:

- El PRD exige **adapter** y pruebas reproducibles (**decisión #2-B**: mock-driven + contrato interno congelado).
- El objeto lógico **Contrato** ya está descrito para alinear a GRP: [`integrations/Contrato-GRP-esquema.md`](../integrations/Contrato-GRP-esquema.md).

---

## Opciones consideradas

### Opción A — Esperar solo a OpenAPI real de GRP

**Ventajas:** Cero desalineación con el proveedor.  
**Desventajas:** Paraliza backend, pruebas E2E y definición de errores hasta la entrega externa.

### Opción B — Adaptador + OpenAPI interno + implementación stub (elegida en producto)

**Ventajas:** Desarrollo y CI paralelos; contrato versionado en repo; migración controlada cuando GRP entregue spec.  
**Desventajas:** Posible delta entre stub y real; costo de reconciliación documentada.

### Opción C — Spike directo contra Postman/colección no versionada

**Ventajas:** Feedback temprano si ya hay algo usable.  
**Desventajas:** Deuda y riesgo de “stub en producción”; no recomendado como línea base.

---

## Decisión

**Se adopta la Opción B**, alineada a la decisión de producto **#2-B**:

1. **Capa `GrpClient` (interfaz)** en el backend con operaciones mínimas: `queryContractByRfc`, `submitSignJob`, `pollSignJob`, `notifyReception` (nombres ilustrativos; ajustar al paquete real).
2. **Implementaciones:**
   - **`GrpClientStub`:** respuestas deterministas para CI/UAT local; errores **surrogate** documentados (códigos y cuerpos de ejemplo) que el dominio debe mapear igual que en producción.
   - **`GrpClientHttp`:** implementación real detrás de feature flag / configuración por entorno; activa cuando exista URL + auth + OpenAPI **ratificado** con GRP.
3. **Artefacto de contrato:** mantener en repo un **OpenAPI interno** (p. ej. `document/integrations/openapi-grp-stub.yaml` en iteración futura) o equivalente que refleje [`Contrato-GRP-esquema.md`](../integrations/Contrato-GRP-esquema.md) para **Query**; Sign/Reception en **ADR-003**.
4. **Mapeo de errores:** tabla interna `grpErrorCode` / HTTP → errores de dominio SI-PSP (`GRP_UNAVAILABLE`, `CONTRACT_NOT_FOUND`, etc.); stub y Http **misma tabla**.
5. **Congelación:** cambios al OpenAPI interno pasan por revisión y notificación a GRP (“delta” explícito).

---

## Consecuencias

### Positivas

- Desbloquea implementación de **US-01**, **US-02**, cabecera PDF y flujos que llaman a GRP.
- Tests de contrato sin red externa.
- Criterio claro de “done” para integración: **paridad** stub ↔ Http según checklist en [`Contrato-GRP-esquema.md`](../integrations/Contrato-GRP-esquema.md) §8.

### Negativas / trade-offs

- Mantener **dos** implementaciones hasta estabilizar GRP.
- Riesgo de reinterpretación cuando el spec real difiera: mitigación = **ADR de migración** o actualización de este ADR con sección “Supersedido / Delta”.

### Acciones de seguimiento

- [ ] Añadir `openapi-grp-stub.yaml` (o ubicación acordada) cuando se bootstrap el módulo.
- [ ] Cuando GRP entregue OpenAPI oficial: diff, actualizar mapeos y marcar en README de ADRs si ADR-001 recibe anexo “Delta GRP”.
- [ ] Coordinar con GRP la cardinalidad **un RFC → contratos** (ver §5–6 del esquema Contrato).

---

## Referencias

- [`PRD.md`](../PRD.md) §3.3, §3.5
- [`integrations/Contrato-GRP-esquema.md`](../integrations/Contrato-GRP-esquema.md)
- [`PRD-analisis-critico.md`](../PRD-analisis-critico.md) hallazgos #1, #8
- [`open-questions.md`](../open-questions.md) decisión #2
