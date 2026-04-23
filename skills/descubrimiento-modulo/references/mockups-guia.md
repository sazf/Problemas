# Guía de mockups en texto plano

Los mockups de este entregable **no son UI real**. Son descripciones estructuradas en texto que cualquier diseñador o agente frontend puede convertir después en Figma o componentes. La ventaja: son rápidos de escribir, diffables en Git y no sesgan decisiones de visual design.

## Estructura obligatoria por pantalla

Cada pantalla se describe con **cinco bloques**, en este orden:

### 1. Propósito

Una a dos líneas. Qué tarea permite completar, para qué rol principal, en qué momento del flujo aparece. Ejemplo: *"Tablero operativo para el despachador. Permite ver en tiempo real la disponibilidad de toda la flota y tomar decisiones de asignación de máquinas a obras."*

### 2. Layout

Un bloque ASCII de una pantalla típica (desktop, salvo que el módulo sea móvil-first). No pixel-perfect: secciones con nombres. Entre 10 y 20 líneas máximo.

```
+------------------------------------------------------------------+
| Top bar: logo | nombre módulo | buscador global | usuario+rol    |
+-------+----------------------------------------------------------+
|       | Filtros: estado ▾ | tipo ▾ | obra ▾ | operario ▾  [Aplicar] |
| Nav   +----------------------------------------------------------+
| late- | Tarjetas-resumen: Disponibles (12) | En operación (34) | En mnto (5) |
| ral   +----------------------------------------------------------+
|       | Tabla: # interno | Tipo | Estado | Obra | Operario | Horóm. | Acciones |
|       |   ...fila 1...                                            |
|       |   ...fila 2...                                            |
|       |   (scroll)                                                |
|       +----------------------------------------------------------+
|       | Footer: paginación | total filtrados: 58                  |
+-------+----------------------------------------------------------+
```

Si la pantalla se divide en varios paneles (split view, wizard, tabs), incluir **un bloque por panel** o explicitar las tabs.

### 3. Datos visibles

Lista de campos que aparecen en la pantalla, agrupados por zona. Incluir tipo, formato y si son clickeables/editables.

- **Tabla principal**:
  - `# interno` — texto corto, clickeable → abre ficha.
  - `tipo` — chip de color según categoría.
  - `estado` — chip; mapea a estado de la máquina de estados (ver §3).
  - `horómetro_actual` — número con separador de miles, lectura solo.
  - ...
- **Tarjetas-resumen**:
  - Conteos por estado, clickeables → filtran la tabla.

### 4. Acciones disponibles

Cada acción con: etiqueta, rol que puede ejecutarla, efecto, y qué restricciones aplican (regla dura del §4).

- **"Asignar a obra"** — rol: despachador, supervisor. Abre modal con selector de obra y operario. Sujeto a **RD-3** (no asignar si la máquina está en `en_mantenimiento`) y **RD-7** (operario con SST vigente).
- **"Marcar en mantenimiento"** — rol: taller. Transición `disponible → en_mantenimiento`.
- **"Exportar CSV"** — cualquier rol con acceso. Respeta los filtros activos.

### 5. Comportamiento por estado y vacíos

Cómo se ve la pantalla en los casos no-felices:

- **Sin datos**: mensaje empty-state + CTA primaria (p. ej. "Aún no hay maquinaria registrada. [Registrar primera]").
- **Cargando**: skeleton de tabla + tarjetas en placeholder.
- **Error de red**: banner rojo con reintento, sin borrar filtros del usuario.
- **Sin permisos**: el usuario ve la pantalla pero las acciones no permitidas aparecen deshabilitadas con tooltip explicando por qué.

## Qué NO hacer

- **No describir colores exactos ni tipografías.** Decir "chip de color según estado" es suficiente; el mapeo concreto es decisión de diseño.
- **No dibujar ASCII pixel-perfect.** El bloque es esquemático, no un render. Si cuesta más de 5 minutos, se está dibujando en lugar de analizar.
- **No repetir toda la máquina de estados.** Referenciar el `.mmd` correspondiente y mostrar solo cómo se visualiza el estado en pantalla.
- **No asumir un framework.** El entregable debe poder implementarse en React, Vue, Blazor o UI nativa.

## Cuántas pantallas incluir

Entre 3 y 5 por módulo. Típicamente:

1. **Listado / tablero principal** — la puerta de entrada del módulo.
2. **Ficha de entidad** — vista detallada con toda la información.
3. **Acción crítica** — modal/pantalla del flujo más importante (crear contrato, liquidar alquiler, cerrar obra).
4. **Pantalla de excepción** — p. ej. anulación, rechazo, devolución.
5. (Opcional) **Reporte / resumen** — si el módulo tiene una vista agregada relevante.

Más de 5 empieza a duplicar y dispersa la atención. Si el módulo parece necesitar más, probablemente hay que desagregarlo en submódulos.
