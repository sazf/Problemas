# Plantilla del `.md` maestro

Esta es la estructura exacta que debe tener `analisis-<modulo>.md`. No agregar secciones nuevas ni reordenar: la consistencia entre módulos es el valor del entregable. Si algo no aplica al dominio, dejar la sección presente con una nota explicando por qué no aplica, en lugar de borrarla.

```markdown
# Análisis de descubrimiento — Módulo <Nombre>

| Campo | Valor |
|---|---|
| Módulo | <nombre en voz de negocio> |
| Versión del documento | v0.1 — YYYY-MM-DD |
| Sistema destino | <ERP propio, greenfield, etc.> |
| Alcance (incluye) | <bullet corto> |
| Alcance (excluye) | <bullet corto> |
| Autor(es) | <usuario> |

## 0. Resumen ejecutivo

Tres a cinco bullets. Qué es el módulo, qué problema resuelve, cuáles son las decisiones de diseño no triviales que salen de este análisis, y qué queda abierto.

## 1. Benchmark de referentes

### 1.1 Referentes analizados

| Referente | Tipo | Documento consultado | Formato |
|---|---|---|---|
| SAP S/4HANA — EAM | Global | <título> | PDF offline / URL |
| Odoo Maintenance | Global | <título> | PDF offline / URL |
| Sinco ERP — Equipos | LatAm | <título> | PDF offline / URL |
| ... | ... | ... | ... |

### 1.2 Síntesis comparativa

Párrafo(s) con los **patrones comunes** (entidades, estados, integraciones, UX dominante) que aparecen en casi todos. No repetir features uno por uno: extraer la convergencia.

### 1.3 Divergencias interesantes

Cosas donde los grandes se separan, y por qué. Ejemplo: "SAP modela el mantenimiento como órdenes con tareas; Odoo lo simplifica a solicitudes. Para nuestro tamaño, la segunda encaja mejor porque ..."

### 1.4 Fuentes

Lista numerada. Para PDFs descargados: ruta relativa (`fuentes/sap/<archivo>.pdf`). Para manuales en línea: título, URL, fecha de consulta. Sin paywalls ni login.

## 2. Mapa de escenarios

Lista de 15–30 casos de uso reales, agrupados por tipo. Redactar en voz de negocio, no de software.

### 2.1 Operación normal
- E1 — <escenario>
- E2 — <escenario>
- ...

### 2.2 Excepciones y casos límite
- E# — <escenario>
- ...

### 2.3 Integraciones con otros módulos
- E# — <escenario con módulo>
- ...

## 3. Máquina de estados

Una subsección por entidad principal. Cada una referencia un `.mmd` externo.

### 3.1 Entidad: <Nombre>

- **Estados**: `borrador`, `activo`, `en_curso`, `cerrado`, `anulado`, ...
- **Diagrama**: [`estados-<entidad>.mmd`](./estados-<entidad>.mmd)

Tabla de transiciones:

| De | A | Evento | Guarda |
|---|---|---|---|
| borrador | activo | aprobar | tiene todos los campos obligatorios |
| ... | ... | ... | ... |

Notas sobre estados terminales, estados de excepción, rollback.

## 4. Restricciones de negocio

### 4.1 Reglas duras (bloquean)

1. **RD-1** — No se puede <acción> si <condición>. *Entidad: <E>. Estado afectado: <S>.*
2. **RD-2** — ...

### 4.2 Reglas blandas (advertencias / defaults)

1. **RB-1** — Advertir al usuario si <condición>. No bloquea.
2. **RB-2** — ...

### 4.3 Reglas cruzadas (afectan varios módulos)

1. **RC-1** — <regla> (involucra a <módulos>).

## 5. Roles y permisos

### 5.1 Roles identificados en el dominio

Lista con una línea por rol explicando qué hace en el negocio (no en el software).

### 5.2 Matriz acción × rol

| Acción | <Rol 1> | <Rol 2> | <Rol 3> | <Rol 4> |
|---|---|---|---|---|
| Crear <entidad> | ✅ | ✅ | ❌ | ❌ |
| Aprobar | ❌ | ⚠️ (con visto bueno) | ✅ | ✅ |
| Anular | ❌ | ❌ | ⚠️ | ✅ |
| ... | ... | ... | ... | ... |

### 5.3 Permisos por estado (si aplica)

Tabla estado × rol cuando la acción permitida depende del estado actual.

## 6. Flujos y mockups

### 6.1 Flujos críticos

Cada flujo en un `.mmd` aparte (vertical, `flowchart TD`).

- [`flujo-<nombre1>.mmd`](./flujo-<nombre1>.mmd) — <una línea de resumen>
- [`flujo-<nombre2>.mmd`](./flujo-<nombre2>.mmd) — <una línea de resumen>
- ...

### 6.2 Mockups de pantallas clave

Tres a cinco pantallas descritas en texto plano según `mockups-guia.md`. Cada una incluye: propósito, layout, datos visibles, acciones, comportamiento por estado.

#### 6.2.1 <Nombre de la pantalla>

<descripción estructurada>

#### 6.2.2 ...

## 7. Adaptación Colombia

Sección obligatoria con datos concretos del dominio. Cubrir lo que aplique:

- **Tributario**: régimen, IVA, retenciones (renta, IVA, ICA), autorretenciones.
- **Facturación electrónica DIAN**: tipo de documento, eventos, notas crédito/débito, documentos soporte.
- **Nómina electrónica** (si aplica).
- **RUT y terceros**: campos mínimos, validaciones.
- **SST** (Resolución 0312 cuando aplique): registros, evidencias.
- **Contratos y documentos**: firma, soporte físico/digital.
- **Centros de costo / proyectos**: cómo se cruza con contabilidad.
- **Particularidades sectoriales**: construcción (formatos de obra), transporte (RNDC), salud (RIPS), agro, etc.

No escribir "cumplir DIAN". Escribir qué documento, qué campos, qué eventos, qué frecuencia.

## 8. Preguntas abiertas y supuestos

Bullets de cosas que faltan resolver antes de implementar. Cada uno con owner sugerido si se conoce.

- **P1** — <pregunta>. *Sugerido: preguntar a <rol/persona>.*
- **S1** — Se asume que <supuesto> hasta que se confirme.

## 9. Próximos pasos sugeridos

Tres a cinco bullets. Qué hacer con este documento: llevarlo a Codex para modelado, validar con el dueño del proceso, hacer prototipo de UI, etc.
```
