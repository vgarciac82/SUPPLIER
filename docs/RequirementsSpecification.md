# Documento de Especificación de Requerimientos: Sistema SI-PSP

**Proyecto:** Sistema de Control de Informes para Prestadores de Servicios Profesionales  
**PM:** Cherlo  
**Consultoría:** Business Analysis Senior  

---

## 1. Arquitectura de Acceso y Usuarios

### Identificador Único

El identificador único del prestador será su **RFC**.

### Lógica de Registro

El sistema consultará al **GRP** si existe un contrato activo para el RFC ingresado.

- Si la consulta es positiva y corresponde al primer ingreso del prestador, el sistema solicitará la creación de una contraseña.
- Si el contrato se encuentra vencido, el sistema inhabilitará la creación de nuevos reportes, manteniendo únicamente el modo consulta.

---

## 2. Módulo: Configuración de Proyecto  
### Registro de Actividades

Este módulo funcionará como el catálogo maestro que alimentará los reportes mensuales.

### Funcionalidad

El PSP deberá dar de alta las actividades o cláusulas específicas de su contrato.

### Campos

| Campo | Descripción |
|---|---|
| ID de Actividad | Identificador único de la actividad |
| Descripción de la Actividad | Descripción conforme al contrato |

### Regla Crítica

Este paso será obligatorio y se realizará una sola vez por contrato.

Los reportes mensuales únicamente podrán vincularse a actividades previamente registradas, con el objetivo de evitar el uso de texto libre incoherente o no relacionado con el contrato.

---

## 3. Módulo: Gestión de Reportes Mensuales

---

### A. Lógica Automática de Periodos y Cortes

El sistema calculará los renglones de la **Sección 2** con base en la fecha de inicio del contrato.

**Ejemplo:** Si el contrato inicia el día 15 de febrero, el día de corte será el día 15 de cada mes.

### Día de Corte

El día de corte será el día natural igual al día de inicio del contrato.

**Ejemplo:**  
Si el contrato inicia el día 15, el corte mensual será el día 15.

### Generación de Renglones Semanales

| Tipo de Semana | Regla |
|---|---|
| Semana 1 | Desde el día siguiente al inicio del contrato hasta el domingo inmediato |
| Semanas Intermedias | Bloques completos de lunes a domingo |
| Semana de Cierre | Desde el último lunes hasta el día de corte |

### Control de Flujo

El sistema sugerirá el mes y periodo a reportar con base en el historial del prestador, con el objetivo de evitar:

- Duplicidad de reportes.
- Saltos de meses.
- Captura de periodos fuera de secuencia.

---

### B. Captura de Actividades  
### Sección 2 y Sección 3

La captura de actividades se realizará mediante una tabla estructurada.

### Tabla de Actividades

| Columna | Campo | Comportamiento |
|---|---|---|
| 1 | Periodo | Autogenerado, no editable |
| 2 | Actividad | Lista desplegable con actividades del Registro de Proyecto |
| 3 | Descripción | Campo de texto para capturar el logro alcanzado en el periodo |
| 4 | Evidencia | Botón para adjuntar imágenes relacionadas |

### Anexo Fotográfico

El sistema generará automáticamente una sección al final del documento con las imágenes cargadas por el PSP.

Las imágenes deberán:

- Reducirse automáticamente para su inclusión en el PDF.
- Asociarse al ID de la actividad correspondiente.
- Mostrarse dentro del anexo fotográfico del informe.

---

## 4. Ciclo de Vida del Reporte  
### Workflow

Se define una máquina de estados estricta para garantizar la validez del informe.

| Estado | Descripción |
|---|---|
| BORRADOR | El PSP puede editar y guardar el informe. El botón "Enviar" validará que todos los periodos del mes estén llenos |
| EN REVISIÓN | El informe se envía al Área Técnica o Responsable de Proyecto. El PSP pierde permisos de edición |
| RECHAZADO | El revisor regresa el informe con comentarios. El estado vuelve a BORRADOR para corrección |
| APROBADO | El informe queda bloqueado. Se habilita el envío al servicio de firma electrónica |
| FIRMADO / FINALIZADO | El sistema envía el PDF al API del GRP, recibe el documento sellado y lo marca como listo para pago |

---

## 5. Estructura del Output  
### PDF Estándar

El documento final generado por el sistema tendrá 4 secciones obligatorias.

| Sección | Descripción |
|---|---|
| Encabezado | Datos del proveedor, contrato, proyecto, unidad y ubicación. Estos datos serán recuperados desde el GRP |
| Descripción de Actividades | Tabla detallada con los periodos semanales y actividades reportadas |
| Anexo Fotográfico | Imágenes de evidencia con sus respectivos IDs de actividad |
| Hoja de Firmas | Espacio técnico para la firma electrónica SAT/GRP |

---

## 6. Integración Técnica  
### APIs con GRP

Se requiere que el equipo de desarrollo del GRP entregue los siguientes servicios.

| Servicio | Descripción |
|---|---|
| Servicio de Consulta | Permite extraer datos del contrato y vigencia por RFC |
| Servicio de Firma | Recibe el PDF generado por el sistema SI-PSP y lo devuelve firmado digitalmente |
| Servicio de Recepción | Notifica al GRP que el informe está listo y aprobado para procesar el pago |

---

# Resumen General

El sistema SI-PSP permitirá que los Prestadores de Servicios Profesionales capturen, documenten, validen y firmen electrónicamente sus informes mensuales.

El flujo principal será:

```mermaid
flowchart TD
    A[Ingreso del PSP con RFC] --> B[Consulta de contrato en GRP]
    B --> C{¿Contrato activo?}
    C -- No --> D[Modo consulta únicamente]
    C -- Sí --> E{¿Primer ingreso?}
    E -- Sí --> F[Crear contraseña]
    E -- No --> G[Acceso al sistema]

    G --> H[Registrar actividades del contrato]
    H --> I[Generar reporte mensual]
    I --> J[Capturar actividades y evidencias]
    J --> K[Enviar a revisión]

    K --> L[Área Técnica revisa informe]
    L --> M{¿Aprueba?}
    M -- No --> N[Rechazado con comentarios]
    N --> I
    M -- Sí --> O[Informe aprobado]

    O --> P[Generar PDF]
    P --> Q[Enviar a firma electrónica GRP]
    Q --> R[Recibir PDF firmado]
    R --> S[Notificar informe listo para pago]
