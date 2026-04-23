# Problemas — Skills de descubrimiento

Repositorio con skills personales de Claude Code para análisis de problemas de negocio antes de implementación.

## Skills incluidas

- [`skills/descubrimiento-modulo/`](./skills/descubrimiento-modulo/) — convierte un módulo difuso de un ERP en seis entregables estándar (benchmark de referentes, mapa de escenarios, máquina de estados, restricciones, matriz de roles, flujos Mermaid + mockups), con adaptación obligatoria a Colombia. Pensada para fase previa a pasarle el trabajo a Codex o a implementación.

## Instalación en un PC nuevo

Las skills de Claude Code se cargan desde `~/.claude/skills/`. Para que este repo se active, cada skill tiene que estar accesible desde ahí.

### Opción A — Copia simple (recomendada si actualizas poco)

```bash
git clone git@github.com:<tu-usuario>/Problemas.git ~/Projects/Problemas
cp -r ~/Projects/Problemas/skills/descubrimiento-modulo ~/.claude/skills/
```

Cada vez que hagas `git pull`, repite el `cp -r` para propagar los cambios.

### Opción B — Symlink (sin duplicar; recomendado si editas a menudo)

```bash
git clone git@github.com:<tu-usuario>/Problemas.git ~/Projects/Problemas
ln -s ~/Projects/Problemas/skills/descubrimiento-modulo ~/.claude/skills/descubrimiento-modulo
```

Así los cambios en el repo se reflejan al instante en Claude Code y viceversa.

### Verificación

Abre una sesión de Claude Code y ejecuta cualquier prompt que dispare la skill (p. ej. *"quiero hacer un análisis de descubrimiento del módulo de inventario"*). La skill `descubrimiento-modulo` debe aparecer en la lista de skills disponibles.

## Actualizar la skill

El flujo depende de la opción de instalación:

### Si usaste copia (Opción A)

1. Edita los archivos bajo `skills/descubrimiento-modulo/` **en este repo**.
2. Commit y push.
3. En cada PC donde esté instalada, traer los cambios:
   ```bash
   cd ~/Projects/Problemas && git pull
   cp -r skills/descubrimiento-modulo ~/.claude/skills/
   ```

Importante: si alguna vez editas directo en `~/.claude/skills/descubrimiento-modulo/` en vez de en el repo, **perderás** los cambios al siguiente `cp -r`. Regla simple: edita siempre en el repo.

### Si usaste symlink (Opción B)

1. Edita los archivos bajo `skills/descubrimiento-modulo/` en este repo (o desde `~/.claude/skills/descubrimiento-modulo/`, ambos apuntan al mismo sitio).
2. Commit y push.
3. En cada PC: `git pull` y listo, no hay paso de copia.

## Estructura de la skill

```
skills/descubrimiento-modulo/
├── SKILL.md                         # orquestador + pipeline de 7 pasos
├── references/
│   ├── plantilla-entregable.md
│   ├── benchmark-guia.md
│   ├── estados-patrones.md
│   ├── colombia-contexto.md
│   └── mockups-guia.md
└── assets/
    └── ejemplo-maquinaria/          # patrón oro para tono y profundidad
        ├── analisis-maquinaria.md
        ├── estados-maquinaria.mmd
        └── flujo-alquiler.mmd
```

Ver [`skills/descubrimiento-modulo/SKILL.md`](./skills/descubrimiento-modulo/SKILL.md) para el detalle del pipeline y cómo invocarla.
