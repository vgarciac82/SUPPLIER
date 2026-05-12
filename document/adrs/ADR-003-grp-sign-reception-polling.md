# ADR-003 — Firma electrónica y recepción GRP: polling, idempotencia y fallos

## Metadatos

| Campo | Valor |
| --- | --- |
| ID | ADR-003 |
| Fecha | 2026-05-10 |
| Estado | Aceptado |
| Decisores | Equipo SI-PSP (pendiente ratificación formal) |
| PRD Ref | US-08; §3.5; decisiones **#9-A** |
| Ticket Redmine | N/A |

---

## Contexto

Tras **APPROVED**, SI-PSP debe:

1. Enviar el PDF al **servicio de firma** de GRP.
2. Obtener el PDF firmado y pasar el agregado a **`SIGNED_FINALIZED`** (PRD §2.1.1).
3. Llamar al **servicio de recepción** para notificar listo para pago.

GRP puede exponer firma **síncrona** o **asíncrona** (correlation / job id). La decisión de producto **#9-A** impone **polling** desde SI-PSP hasta estado terminal, con backoff y timeout.

Riesgos sin ADR: estados ambiguos entre “firmando” y “aprobado”, doble notificación de recepción, o bloqueo permanente si GRP falla.

---

## Opciones consideradas

### Opción A — Solo webhook desde GRP

**Ventajas:** Eficiente.  
**Desventajas:** Contradice decisión de producto #9-A; expone endpoint público y depende de garantías de GRP.

### Opción B — Polling desde SI-PSP (elegida en producto)

**Ventajas:** Alineado a #9-A; funciona sin callback.  
**Desventajas:** Latencia y carga; requiere límites claros.

### Opción C — Polling + webhook posterior

**Ventajas:** Evolución.  
**Desventajas:** Dos caminos hasta converger; fuera de MVP salvo necesidad.

---

## Decisión

**Opción B**, con los siguientes lineamientos.

### 1. Modelo de datos de seguimiento (sin nuevo estado de negocio obligatorio)

- El informe permanece en **`APPROVED`** hasta que exista **PDF firmado persistido** y la transición a **`SIGNED_FINALIZED`** sea atómica.
- Paralelamente se persiste un **SignJob** (tabla o subdocumento): `signJobId` (externo o UUID interno), `status` (`PENDING` | `SUCCEEDED` | `FAILED`), `lastPolledAt`, `failureReason` (texto seguro), `rawLastHttpCode` (opcional, no PII).
- La UI puede mostrar “Firma en proceso” derivado de `SignJob.status === PENDING` sin cambiar el enum público del informe si se desea simplificar APIs; alternativa: exponer campo derivado `signingInProgress` en DTO de lectura.

### 2. Polling (valores por defecto; ajustables por config)

| Parámetro | Valor default |
| --- | --- |
| Primer delay post-submit | 30 s |
| Backoff | Exponencial con **jitter**; techo entre polls **5 min** |
| Timeout total desde inicio de firma | **24 h** |
| Tras timeout | `SignJob.status = FAILED`; informe **permanece APPROVED** |

### 3. Reintento manual

- Si `FAILED` o timeout: el PSP (o proceso) puede **reintentar** firma desde **APPROVED** con un **nuevo** `signJobId` interno; límites de reintentos por día **TBD** en configuración (evitar abuso).

### 4. Recepción (notify payment-ready)

- Tras persistir PDF firmado y ejecutar transición a **`SIGNED_FINALIZED`**, invocar recepción.
- **`receptionAttemptId`:** UUID generado **una vez** por informe finalizado; todas las reintentos de envío incluyen el mismo id (PRD AC-08-4).
- Hasta que GRP confirme idempotencia por clave propia: registrar resultado en log + tabla `ReceptionAttempt` para **reconciliación manual** si hay duda de doble aviso.

### 5. Outbox (recomendado)

- Encolar efectos secundarios (poll siguiente tick, llamada recepción) vía **outbox pattern** en la misma transacción que el cambio de estado persistido, para no perder pasos tras crash (detalle en implementación; alineado a PRD §3.5).

### 6. Errores GRP

- Mapear a errores de dominio; no exponer stack ni cuerpo crudo a cliente PSP.
- Stub debe simular: éxito sync, éxito async tras N polls, fallo terminal, timeout.

---

## Consecuencias

### Positivas

- Comportamiento acorde a **US-08** y decisión **#9-A**.
- Recuperación clara si GRP cae (sigue en **APPROVED** + job **FAILED**).

### Negativas / trade-offs

- Workers / scheduler para polls (infra adicional).
- Sin webhook, latencia hasta “firmado” depende del intervalo de poll.

### Acciones de seguimiento

- [ ] Definir en implementación si `SignJob` es tabla dedicada o JSON en `MonthlyReport`.
- [ ] Cuando GRP confirme **idempotencia** en recepción, actualizar este ADR y AC-08-4.
- [ ] Alinear **ADR-001** `GrpClientHttp` con paths reales del OpenAPI GRP.

---

## Referencias

- [`PRD.md`](../PRD.md) US-08, §2.1.1, §3.5
- [`PRD-analisis-critico.md`](../PRD-analisis-critico.md) hallazgo #3
- [`open-questions.md`](../open-questions.md) decisión #9
- [`ADR-001-grp-integration-adapter.md`](ADR-001-grp-integration-adapter.md)
