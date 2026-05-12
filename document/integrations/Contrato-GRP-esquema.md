# Objeto **Contrato** — Esquema esperado (integración GRP ↔ SI-PSP)

**Audiencia:** equipo de desarrollo / integración **GRP** y **SI-PSP**.  
**Uso:** contrato de datos de la **respuesta** del *Servicio de Consulta* (RFC → contexto contractual) descrito en [`docs/RequirementsSpecification.md`](../../docs/RequirementsSpecification.md) §6 y [`PRD.md`](../PRD.md) §3.3.

**Versión:** 1.0 (2026-05-10).  
**Nota:** Los nombres JSON son **propuesta SI-PSP**; GRP puede proponer alias siempre que se publique **tabla de equivalencias** y OpenAPI compartido.

---

## 1. Alcance

| Entrada (SI-PSP → GRP) | Salida (GRP → SI-PSP) |
| --- | --- |
| Identificador del prestador: **RFC** (según reglas SAT de formato y normalización acordadas). | Objeto **Contrato** (o lista; ver §6) con vigencia, fechas para motor de periodos y campos de cabecera para PDF. |

Este documento define la **forma lógica** del recurso **Contrato** que SI-PSP necesita para: login/registro, modo solo consulta, motor de cortes mensuales, y generación de PDF (encabezado).

---

## 2. Request de consulta (referencia)

| Campo | Tipo | Obligatorio | Descripción |
| --- | --- | --- | --- |
| `rfc` | `string` | Sí | RFC del PSP, normalizado (mayúsculas, sin espacios) según acuerdo operativo. |

*Autenticación del cliente B2B, URL, método HTTP y versionado: **TBD** por GRP (OpenAPI).*

---

## 3. Objeto de respuesta: `Contract` (Contrato)

Representa **un** contrato asociado al RFC consultado. Si GRP puede devolver **varios** contratos activos para el mismo RFC, debe definirse una de las políticas del **§6**; hasta entonces SI-PSP asume **como máximo un contrato “elegible para operación”** por RFC en MVP.

### 3.1 Identificación y vínculo

| Campo JSON | Tipo | Obligatorio | Descripción (negocio) |
| --- | --- | --- | --- |
| `contractId` | `string` | Sí | Identificador **estable** del contrato en GRP (clave foránea en SI-PSP). |
| `rfc` | `string` | Sí | RFC del titular / prestador (eco o validación cruzada). |
| `contractNumber` | `string` | Condicional | Número o clave humana del contrato para PDF y soporte. **Obligatorio** si GRP lo usa en documentos oficiales. |
| `projectId` | `string` | Condicional | Identificador de proyecto en GRP, si aplica al modelo de datos. |

### 3.2 Vigencia y ciclo de vida (obligatorio para reglas SI-PSP)

| Campo JSON | Tipo | Obligatorio | Descripción |
| --- | --- | --- | --- |
| `lifecycleStatus` | `string` (enum) | Sí | Ver **§4**. Determina si el PSP puede crear informes o solo consultar. |
| `validFrom` | `date` | Sí | Inicio de vigencia del contrato (fecha civil). |
| `validTo` | `date` \| `null` | Sí | Fin de vigencia; `null` = vigencia abierta / sin fin conocido en GRP. |
| `contractStartDate` | `date` | Sí | **Fecha de inicio del contrato** usada por SI-PSP para calcular **día de corte** y **semanas** del informe mensual (puede coincidir con `validFrom` o ser distinta si negocio lo define así — **GRP debe documentar la semántica**). |

**Semántica de fechas:** fechas **civiles** interpretadas en **`America/Mexico_City`** (decisión de producto #4-A). Formato recomendado: **ISO 8601** `YYYY-MM-DD`.

### 3.3 Cabecera del PDF (Servicio de Consulta / enriquecimiento)

Campos alineados a [`docs/RequirementsSpecification.md`](../../docs/RequirementsSpecification.md) §5 (Encabezado: proveedor, contrato, proyecto, unidad, ubicación).

| Campo JSON | Tipo | Obligatorio | Mapa negocio (spec) |
| --- | --- | --- | --- |
| `supplierName` | `string` | Sí | Datos del **proveedor** |
| `supplierTaxId` | `string` | Sí | RFC (puede repetir `rfc`) |
| `contractLabel` | `string` | Sí | Identificación legible del **contrato** (p. ej. número + descripción corta) |
| `projectName` | `string` | Condicional | **Proyecto** |
| `organizationalUnit` | `string` | Condicional | **Unidad** |
| `location` | `string` | Condicional | **Ubicación** |

**Regla SI-PSP:** si algún campo marcado **Sí** para el PDF falta o viene vacío, la generación del PDF puede **bloquearse** con error explícito (**PRD** AC-07-2). GRP debe listar qué campos garantiza en cada escenario.

### 3.4 Metadatos opcionales (recomendados para soporte y trazabilidad)

| Campo JSON | Tipo | Obligatorio | Descripción |
| --- | --- | --- | --- |
| `lastUpdatedAt` | `date-time` | No | Última modificación de datos de contrato en GRP. |
| `queryCorrelationId` | `string` | No | ID de correlación devuelto por GRP para trazas entre sistemas. |

---

## 4. Enum `lifecycleStatus`

Valores propuestos para mapeo unívoco a comportamiento SI-PSP:

| Valor | Significado | Comportamiento esperado en SI-PSP |
| --- | --- | --- |
| `ACTIVE` | Contrato vigente y operativo para informes | Acceso completo (tras catálogo, etc.). |
| `EXPIRED` | Vigencia terminada | Solo lectura; sin nuevos informes (**US-02**). |
| `SUSPENDED` | Suspendido administrativamente | **TBD** con negocio: tratar como `EXPIRED` para escritura o estado aparte. |
| `NOT_FOUND` | No existe contrato para el RFC | Sin alta operativa; mensaje de negocio (**aclarar vs RFC inválido**). |

GRP puede usar otros literales si documenta la **tabla de equivalencia** hacia estos cuatro comportamientos.

---

## 5. Cardinalidad: un RFC, varios contratos

| Opción | Descripción | Requisito para GRP |
| --- | --- | --- |
| **A — Uno a uno (MVP asumido)** | A lo sumo un contrato `ACTIVE` por RFC en un instante dado. | Respuesta: objeto único `Contract` o arreglo de longitud 0–1. |
| **B — Uno a muchos** | Varios contratos activos posibles. | Respuesta: `contracts[]` + campo `isPrimary` o criterio de **default**; o endpoint adicional “listar contratos por RFC” + “seleccionar contrato”. **SI-PSP** debe extender PRD/UX. |

**Pendiente de confirmación con GRP:** opción A o B (hallazgo análisis crítico #8).

---

## 6. Ejemplo JSON (ilustrativo — no normativo hasta OpenAPI firmado)

```json
{
  "contract": {
    "contractId": "GRP-CT-2024-00088421",
    "rfc": "XAXX010101000",
    "contractNumber": "CT-88421",
    "projectId": "PRJ-552",
    "lifecycleStatus": "ACTIVE",
    "validFrom": "2024-02-15",
    "validTo": null,
    "contractStartDate": "2024-02-15",
    "supplierName": "Proveedor Ejemplo S.A. de C.V.",
    "supplierTaxId": "XAXX010101000",
    "contractLabel": "CT-88421 — Servicios profesionales zona norte",
    "projectName": "Proyecto Integración Red",
    "organizationalUnit": "Dirección de Operaciones",
    "location": "Ciudad de México",
    "lastUpdatedAt": "2026-05-01T12:00:00-06:00"
  }
}
```

Para `EXPIRED`, mismos campos salvo `lifecycleStatus` y `validTo` acotada en el pasado.

---

## 7. Errores esperados (solicitud a GRP)

GRP debe documentar códigos HTTP y cuerpo de error. Mínimo deseado para SI-PSP:

| Situación | Comportamiento deseado |
| --- | --- |
| RFC mal formado | `4xx` + mensaje de validación; SI-PSP no crea sesión. |
| RFC válido sin contrato | `404` o payload con `lifecycleStatus: NOT_FOUND` (acordar uno). |
| Servicio no disponible | `5xx` o timeout; SI-PSP: error genérico + reintentos (**PRD**). |

---

## 8. Checklist de entrega GRP hacia SI-PSP

- [ ] OpenAPI 3.x del endpoint de consulta por RFC.
- [ ] Semántica exacta de `contractStartDate` vs `validFrom` / `validTo`.
- [ ] Enum o catálogo de `lifecycleStatus` y equivalencias.
- [ ] Lista de campos de cabecera **garantizados** vs **opcionales**.
- [ ] Política **un contrato vs varios** por RFC.
- [ ] Autenticación, ambientes (sandbox/prod), SLAs y límites de tasa.

---

## 9. Referencias internas

| Documento | Rol |
| --- | --- |
| [`PRD.md`](../PRD.md) §3.2, §3.3 | Contexto de producto |
| [`PRD-analisis-critico.md`](../PRD-analisis-critico.md) | Hallazgos #1, #8 |
| [`open-questions.md`](../open-questions.md) | Cierre decisiones #1–#10; reabrir si GRP cambia cardinalidad |

---

*Fin del esquema. Ajustar versiones cuando GRP publique el contrato oficial.*
