# Problemas — Skills de descubrimiento

Repositorio con skills personales de Claude Code para análisis de problemas de negocio antes de implementación.

## Skills incluidas

- [`skills/descubrimiento-modulo/`](./skills/descubrimiento-modulo/) — convierte un módulo difuso de un ERP en seis entregables estándar (benchmark de referentes, mapa de escenarios, máquina de estados, restricciones, matriz de roles, flujos Mermaid + mockups), con adaptación obligatoria a Colombia. Pensada para fase previa a pasarle el trabajo a Codex o a implementación.

## Instalación en un PC nuevo

Claude Code carga skills desde `~/.claude/skills/`. Para dejarlas disponibles desde este repo hay tres caminos; **se recomienda la Opción A** por su simplicidad y porque incluye actualización en un solo comando.

### Opción A — `npx skills` (recomendada)

Usa el CLI [`skills`](https://github.com/vercel-labs/skills) de Vercel Labs. Requiere Node.js instalado (viene con Claude Code si lo instalaste por npm).

```bash
npx skills add sazf/Problemas -g --agent claude-code -y
```

Eso clona el repo, detecta la skill y la monta en `~/.claude/skills/descubrimiento-modulo/` (por defecto como symlink). No hay que elegir carpeta ni recordar rutas.

Comandos útiles:

```bash
npx skills ls -g                                   # listar instaladas
npx skills update descubrimiento-modulo -g         # actualizar a última versión del repo
npx skills remove descubrimiento-modulo -g -y      # desinstalar
```

Si algún día cambias de agente, el mismo repo sirve para otros (`--agent cursor`, `--agent codex`, etc.).

### Opción B — Git + symlink (fallback, sin dependencia de npx)

Útil si prefieres no depender de una herramienta de terceros, o si vas a editar la skill a menudo en tu repo clonado.

```bash
mkdir -p ~/Projects && cd ~/Projects
git clone https://github.com/sazf/Problemas.git
mkdir -p ~/.claude/skills
ln -s ~/Projects/Problemas/skills/descubrimiento-modulo ~/.claude/skills/descubrimiento-modulo
```

Los cambios en el repo se reflejan al instante en Claude Code.

### Opción C — Git + copia (la más manual)

```bash
mkdir -p ~/Projects && cd ~/Projects
git clone https://github.com/sazf/Problemas.git
mkdir -p ~/.claude/skills
cp -r Problemas/skills/descubrimiento-modulo ~/.claude/skills/
```

Es la más simple conceptualmente pero obliga a repetir el `cp -r` tras cada `git pull`, y si editas en `~/.claude/skills/` en vez de en el repo, pierdes los cambios al siguiente copy.

### Verificación

Abre una sesión de Claude Code y lanza un prompt de prueba:

> "Quiero hacer un análisis de descubrimiento del módulo de inventario."

La skill `descubrimiento-modulo` debe activarse sola y comenzar por el Paso 1 del pipeline (capturar intención).

## Actualizar la skill

Editas siempre dentro del repo, nunca directo en `~/.claude/skills/`. Luego:

| Instalada con | Propagar cambios |
|---|---|
| Opción A (`npx skills`) | `npx skills update descubrimiento-modulo -g` |
| Opción B (symlink) | `git pull` (ya está; el symlink apunta al repo) |
| Opción C (copia) | `git pull && cp -r skills/descubrimiento-modulo ~/.claude/skills/` |

Flujo de autor (la máquina donde editas):

```bash
cd ~/Projects/Problemas
# edita skills/descubrimiento-modulo/...
git add skills/
git commit -m "..."
git push
```

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
