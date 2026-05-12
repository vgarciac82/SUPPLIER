# supplier (SI-PSP backend)

Repositorio del backend **SI-PSP** (*Sistema de Control de Informes para Prestadores de Servicios Profesionales*): API y servicios en **Spring Boot** para informes mensuales, integración **GRP** y ciclo de vida de documentos.

| | |
| --- | --- |
| **Maven** | `com.axtel:supplier:0.0.1-SNAPSHOT` |
| **Spring Boot** | 4.0.6 |
| **Java** | 25 |
| **Base de datos** | PostgreSQL (driver en runtime; configurar datasource en despliegue) |

---

## Requisitos

- **JDK 25**
- **Maven 3.9+** (o wrapper si lo añades al repo)
- **PostgreSQL** cuando conectes persistencia real (el `application.properties` actual es mínimo)

---

## Cómo ejecutar

```bash
mvn -q -DskipTests spring-boot:run
```

Compilar y tests:

```bash
mvn verify
```

Actuator y Web MVC están en el `pom.xml`. Puerto por defecto de Spring Boot: **8080** (salvo que lo cambies en `src/main/resources/application.properties`).

---

## Documentación del producto y SDD

| Ubicación | Contenido |
| --- | --- |
| [`docs/RequirementsSpecification.md`](docs/RequirementsSpecification.md) | Especificación funcional inicial (español) |
| [`document/PRD.md`](document/PRD.md) | Product Requirements Document |
| [`document/discovery-gate.md`](document/discovery-gate.md) | Cierre Fase 0 (Discovery) |
| [`document/gate-prd.md`](document/gate-prd.md) | Resultado Gate-PRD |
| [`document/PRD-analisis-critico.md`](document/PRD-analisis-critico.md) | Análisis crítico del PRD |
| [`document/open-questions.md`](document/open-questions.md) | Preguntas / decisiones registradas |
| [`document/decision-options.md`](document/decision-options.md) | Opciones A/B/C usadas en taller |
| [`document/integrations/Contrato-GRP-esquema.md`](document/integrations/Contrato-GRP-esquema.md) | DTO **Contrato** para alinear con equipo GRP |
| [`document/adrs/README.md`](document/adrs/README.md) | Índice de ADRs (integración, periodos, firma) |

La carpeta **`ai-specs-master/`** es una **plantilla** de Spec-Driven Development (comandos, skills, ejemplos); la fuente de verdad del producto para este repo está en **`document/`** y `docs/`.

---

## Código fuente

```
src/main/java/com/axtel/supplier/   # Aplicación Spring Boot
src/main/resources/application.properties
src/test/java/                       # Tests (cuando existan)
```

---

## Referencia Spring

Enlaces genéricos de arranque del proyecto: [`HELP.md`](HELP.md).

---

## Licencia y propiedad

Definir según política de **Axtel** / organización (el `pom.xml` deja placeholders de `license` y `scm`).
