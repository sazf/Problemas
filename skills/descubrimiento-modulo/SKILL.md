---
name: descubrimiento-modulo
description: Esta skill debe usarse cuando el usuario pide "analizar un módulo de negocio", "hacer discovery de un módulo", "entender cómo funciona maquinaria / inventario / cartera / nómina / contratos en un ERP", "benchmark de ERPs para X", "diseñar estados y flujos de un módulo", "documento de descubrimiento de un módulo", o describe que quiere "ver cómo lo hacen los grandes" antes de implementar una funcionalidad en su sistema. Produce un entregable estándar de descubrimiento en español (benchmark + escenarios + máquina de estados + restricciones + roles + Mermaid + mockups + adaptación Colombia) como `.md` maestro + archivos `.mmd` sueltos, listo para servir de input a la fase de implementación (p. ej. con Codex).
---

# Descubrimiento de módulos de negocio

Convierte un módulo difuso de un ERP en seis entregables fijos y reutilizables: benchmark de referentes, mapa de escenarios, máquina de estados, restricciones de negocio, matriz de roles, flujos Mermaid + mockups, y una sección obligatoria de adaptación a Colombia.

El objetivo es **estandarizar la fase previa al código**: cuando termine este pipeline, el usuario debe tener un documento concreto y accionable para pasarle a un agente de implementación (Codex, Claude Code sobre el repo, un equipo humano) sin tener que re-explicar el dominio.

## Cuándo activar esta skill

Activar cuando el usuario:
- Pide analizar, diseñar o entender un **módulo** de negocio (maquinaria, inventario, cartera, compras, nómina, contratos, proyectos, activos, mantenimiento, facturación, etc.).
- Quiere **benchmarkear** cómo lo hacen SAP/Oracle/Odoo/Sinco/Siigo u otros antes de implementar.
- Habla de **estados**, **flujos**, **restricciones de negocio**, **roles y permisos** de un módulo.
- Pide un **documento de discovery** o un **documento vivo** de un módulo nuevo.

No activar para: bugs puntuales, refactors de código, preguntas sobre sintaxis, o tareas que ya están en fase de implementación. En esos casos, responder normalmente sin entrar al pipeline.

## Principios

- **El documento es el producto.** Esta skill produce texto, diagramas y mockups; no produce código. El código vendrá después, con otro agente.
- **Colombia no es opcional.** Ningún módulo se considera cerrado sin la sección de adaptación local con datos reales del dominio (DIAN, IVA, retenciones, SST, régimen tributario, documentos soporte). Ver `references/colombia-contexto.md`.
- **Benchmark con evidencia.** Los referentes se investigan con WebSearch y, cuando exista PDF oficial, se descarga para cita offline. Los manuales en línea sin PDF se referencian por URL. Ver `references/benchmark-guia.md`.
- **Mermaid vertical y en archivo aparte.** `flowchart TD` y `stateDiagram-v2` siempre, nunca horizontal. Cada diagrama va en su propio `.mmd` para poder renderizarlo fuera del `.md`.
- **Un ejemplo oro.** `assets/ejemplo-maquinaria/` es el patrón a imitar en profundidad, tono y formato. Revisarlo antes de empezar si es el primer uso de la skill en la sesión.

## Pipeline de 7 pasos

Ejecutar en orden. Entre pasos mayores (2, 4, 7) confirmar con el usuario antes de seguir; en los intermedios avanzar si no hay bloqueos evidentes, pero mostrar el avance para que pueda corregir.

### Paso 1 — Capturar intención

Preguntar al usuario, en una sola interacción si es posible:
- Nombre del módulo.
- Objetivo de negocio (qué problema resuelve, para quién).
- Alcance: qué entra y qué queda fuera.
- Sistema destino (stack, ERP propio, greenfield, etc.).
- Si ya existe documentación, repositorio o capturas previas que Claude deba leer.

Crear un directorio de trabajo `analisis-<modulo>/` (nombre en kebab-case) en el cwd y dejar ahí toda la salida. Si el usuario prefiere otra ubicación, respetarla.

### Paso 2 — Benchmark con WebSearch + recolección de documentación

Seleccionar 3–5 referentes combinando **globales** (SAP, Oracle, Odoo, Microsoft Dynamics, NetSuite) y **regionales/LatAm** cuando aplique (Sinco, Siigo, Alegra, World Office, Helisa, TNS). La elección depende del dominio; ver `references/benchmark-guia.md` para heurística.

Para cada referente:
1. Buscar documentación oficial del módulo con WebSearch.
2. Si hay **PDF oficial descargable**, guardarlo en `analisis-<modulo>/fuentes/<referente>/` para citación offline.
3. Si solo hay **manual en línea** (HTML, knowledge base pública), registrar URL + título + fecha de consulta en la sección "Fuentes" del entregable. No intentar descargar contenido que requiera login o esté tras paywall.
4. Extraer **patrones comunes**, no features: entidades que aparecen en todos, estados recurrentes, integraciones típicas, decisiones de UX dominantes, cosas que curiosamente ninguno resuelve.

El paso se cierra con un bloque de **síntesis comparativa** (no un copy-paste de cada uno) + una lista de fuentes.

### Paso 3 — Mapa de escenarios

Listar casos de uso reales, exhaustivos, en la voz del negocio. No features del software: situaciones que ocurren en la operación. Cada escenario en una línea, agrupado por tipo (operación normal, excepción, integración con otros módulos).

Apuntar a 15–30 escenarios. Si salen menos de 10, probablemente falta profundidad; pedir más contexto al usuario.

### Paso 4 — Máquina de estados

Para cada entidad principal del módulo:
- Listar estados con nombres en snake_case.
- Listar transiciones con el evento disparador y la guarda (condición).
- Identificar estados terminales y estados de excepción.

Escribir el diagrama en `analisis-<modulo>/estados-<entidad>.mmd` usando `stateDiagram-v2`. El `.md` maestro referencia el archivo, no lo embebe. Ver `references/estados-patrones.md` para vocabulario reutilizable.

Confirmar con el usuario antes de seguir: una máquina de estados incorrecta contamina todo lo siguiente.

### Paso 5 — Restricciones de negocio

Dividir en dos bloques:
- **Reglas duras** (bloquean la operación): "no se puede X si Y". Formato imperativo, cada regla numerada.
- **Reglas blandas** (advertencias, defaults, sugerencias): mismo formato pero etiquetadas como advertencia.

Relacionar cada regla con la entidad/estado afectado. Si una regla cruza varios módulos (p. ej. no facturar sin inventario disponible), marcarla como **regla cruzada** para que quien implemente sepa que toca más de un componente.

### Paso 6 — Matriz de roles y permisos

Tabla con filas = acciones, columnas = roles. Los roles se extraen del dominio del usuario (operario, supervisor, administrativo, facturación, gerencia, auditor) y no de un catálogo genérico. Usar marcas simples: ✅ permitido, ⚠️ permitido con aprobación, ❌ prohibido.

Si el módulo tiene permisos dependientes del estado (p. ej. solo se puede editar en borrador), crear una tabla adicional estado × rol.

### Paso 7 — Flujos Mermaid, mockups y adaptación Colombia

Tres entregables finales que cierran el documento:

**a) Flujos Mermaid verticales.** Un `.mmd` por flujo crítico (3–6 flujos típicamente). Usar `flowchart TD`. Nombrar `analisis-<modulo>/flujo-<nombre>.mmd`. El `.md` maestro los lista con una línea de resumen cada uno.

**b) Mockups textuales.** 3–5 pantallas clave descritas en texto plano siguiendo `references/mockups-guia.md`. No dibujar UI real; describir layout, datos visibles, acciones disponibles y comportamiento en cada estado. Esto es suficiente para que un diseñador o un agente frontend lo convierta después.

**c) Adaptación Colombia.** Sección obligatoria. Cubrir, mínimo, lo que aplique al dominio: régimen tributario, IVA y retenciones, facturación electrónica DIAN, nómina electrónica, RUT, SST (Resolución 0312 cuando aplique), contratos, documentos soporte, centros de costo, particularidades sectoriales (construcción, agro, transporte, salud, etc.). Ver `references/colombia-contexto.md`. La sección no es un checklist genérico: cada ítem debe estar resuelto con datos del dominio concreto.

## Contrato de salida

El entregable final siempre es:

```
analisis-<modulo>/
├── analisis-<modulo>.md           # .md maestro, única puerta de entrada
├── estados-<entidad>.mmd          # uno por entidad con máquina de estados
├── flujo-<nombre>.mmd             # uno por flujo crítico
└── fuentes/
    └── <referente>/
        └── <documento>.pdf        # solo PDFs oficiales descargables
```

La estructura del `.md` maestro está fijada en `references/plantilla-entregable.md`. No improvisar secciones ni cambiar el orden: el valor de la skill está en la consistencia.

## Recursos

- `references/plantilla-entregable.md` — estructura exacta del `.md` maestro.
- `references/benchmark-guia.md` — cómo investigar referentes y manejar fuentes.
- `references/estados-patrones.md` — vocabulario y patrones para máquinas de estado.
- `references/colombia-contexto.md` — checklist de contexto local.
- `references/mockups-guia.md` — convenciones para mockups en texto plano.
- `assets/ejemplo-maquinaria/` — análisis completo del módulo de maquinaria, usado como patrón oro.

## Señales de que el entregable no está listo

Antes de declarar el análisis terminado, verificar:

- Menos de 10 escenarios → falta profundidad.
- Máquina de estados sin estado terminal o sin estado de anulación/error → revisar.
- Restricciones sin regla cruzada siendo un módulo integrado → probablemente se olvidó pensar en integraciones.
- Matriz de roles con una sola columna → faltan actores reales del negocio.
- Sección Colombia con frases genéricas ("cumplir con DIAN") → rehacer con datos concretos.
- Fuentes vacías → no hubo benchmark real.

Si alguna señal aparece, volver al paso correspondiente y completar antes de entregar.
