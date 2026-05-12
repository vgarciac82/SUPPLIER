# ADR-002 — Motor de periodos: corte mensual y renglones semanales

## Metadatos

| Campo | Valor |
| --- | --- |
| ID | ADR-002 |
| Fecha | 2026-05-10 |
| Estado | Aceptado |
| Decisores | Equipo SI-PSP (pendiente ratificación formal) |
| PRD Ref | US-04; §3.6; decisiones **#3-A**, **#4-A** |
| Ticket Redmine | N/A |

---

## Contexto

Los informes mensuales exigen:

- **Día de corte** alineado al **día del mes** de `contractStartDate` (**D**).
- Si **D** no existe en el mes **M**, usar el **último día civil** de **M** (decisión de producto #3-A).
- **Renglones semanales** según reglas de la especificación fuente: semana 1 (desde el día siguiente al inicio en el alcance del mes hasta el domingo siguiente), semanas intermedias lunes–domingo, semana de cierre desde el último lunes hasta el corte.
- Zona horaria civil: **`America/Mexico_City`**; persistencia de instantes en **UTC** (#4-A).

Sin ADR, cada desarrollador puede interpretar distinto los bordes del mes, el primer mes reportable o la secuencia “siguiente mes”, rompiendo **KPI-1** y los AC de **US-04**.

---

## Opciones consideradas

### Opción A — Implementar reglas solo en texto del PRD / tests dispersos

**Ventajas:** Sin documento extra.  
**Desventajas:** Semántica difícil de revisar y de alinear con QA; regresiones frecuentes.

### Opción B — ADR + motor de dominio puro + golden fixtures (elegida)

**Ventajas:** Una sola fuente normativa complementaria al PRD; tests dorados versionados.  
**Desventajas:** Mantener ADR cuando cambie legislación o reglas de negocio.

---

## Decisión

**Opción B.** Se define la semántica operativa siguiente.

### 1. Entradas normativas

| Entrada | Tipo | Origen |
| --- | --- | --- |
| `contractStartDate` | `LocalDate` | GRP `contractStartDate` (ver ADR-001 / esquema Contrato); misma TZ civil **America/Mexico_City**. |
| `reportingYearMonth` | `YearMonth` | Mes calendario **M** del informe que el usuario crea o reanuda. |

### 2. Corte del mes **M**

```
cutoverDate(M, D) =
  if M tiene día D then fecha(D) en M
  else último día civil de M
```

Donde **D** = día del mes extraído de `contractStartDate`.

### 3. Alcance temporal del informe para el mes **M**

- **Inicio del bloque mensual en M:** `max(firstDayOf(M), contractStartDate)` — no se generan renglones antes del inicio de contrato.
- **Fin del bloque mensual en M:** `cutoverDate(M, D)`.

*(Si `contractStartDate` es posterior a `cutoverDate` en un caso patológico de datos, el motor debe fallar de forma explícita con error de dominio “datos de contrato inválidos”; no debe inventar renglones.)*

### 4. Generación de renglones semanales (alto nivel)

1. **Semana 1:** desde el **día siguiente** al inicio efectivo del alcance (ver spec) hasta el **domingo** inmediato (inclusive), recortado al rango [inicio bloque, fin bloque].
2. **Semanas intermedias:** bloques completos **lunes–domingo** dentro del rango.
3. **Semana de cierre:** desde el **último lunes** ≤ fin de bloque hasta **cutoverDate** (inclusive).

La implementación debe descomponer en filas con etiqueta de periodo **no editable** (etiqueta derivada para UI/PDF). Detalle de strings de etiqueta puede vivir en i18n; la **secuencia y cantidad** de filas es la fuente de verdad para tests.

### 5. Primer mes sugerido (sin historial de informes)

**Regla MVP:** el primer `YearMonth` sugerido para un contrato nuevo es **`YearMonth.from(contractStartDate)`** (el mes calendario que contiene la fecha de inicio).

Ejemplo: inicio **2024-02-15** → primer mes sugerido **2024-02**.

### 6. Siguiente mes en secuencia (con historial)

**Regla MVP:** sea `H` el conjunto de meses (`YearMonth`) que ya tienen un informe cuyo estado **no** sea solo borrador descartado sin envío — en la práctica: considerar meses con informe en `IN_REVIEW`, `APPROVED`, o `SIGNED_FINALIZED` (y opcionalmente `DRAFT` si política de producto lo excluye de “consumido”; **por defecto ADR:** solo estados **≥ IN_REVIEW** cierran el mes para secuencia).

`nextSuggested = max(H) + 1 mes` en orden calendario. Si no hay `H`, usar §5.

**Conflicto:** si existe `DRAFT` activo para un mes (decisión #6-A), la UI reanuda ese borrador; no crear segundo draft.

*(Ajuste fino de “qué meses cuentan en H” puede refinarse en ticket de implementación si negocio exige contar `DRAFT` enviado una vez; el PRD actual enfatiza evitar duplicados y saltos.)*

### 7. Pruebas

- **Golden fixtures** obligatorios: mínimo **inicio 15-feb**, **inicio 31-ene** (febrero bisiesto y no), **inicio 1-ene**, y un caso de contrato que empieza a mitad de mes con corte en mes corto.
- Ubicación sugerida: `src/test/resources/period-engine/` (o equivalente cuando exista el módulo).

---

## Consecuencias

### Positivas

- KPI-1 verificable con fixtures estables.
- Comportamiento acotado para soporte (“por qué mi febrero tiene N semanas”).

### Negativas / trade-offs

- Cualquier cambio de regla de negocio requiere actualizar **este ADR** + fixtures + posiblemente PRD AC-04.

### Acciones de seguimiento

- [ ] Implementar motor como **función pura** testeable sin Spring cuando sea posible.
- [ ] Si GRP redefine `contractStartDate` vs `validFrom`, actualizar §1 y tabla de mapeo en esquema Contrato.

---

## Referencias

- [`docs/RequirementsSpecification.md`](../../docs/RequirementsSpecification.md) §3.A
- [`PRD.md`](../PRD.md) US-04, §3.6
- [`gate-prd.md`](../gate-prd.md)
- [`open-questions.md`](../open-questions.md) decisiones #3, #4, #6
