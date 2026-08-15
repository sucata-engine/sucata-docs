# Sucata CLI

The Sucata CLI is a collection of helpers, installed as the `sucata` command.

| Command | Purpose |
|---|---|
| `sucata run <file> [--entity <file>]` | Run a game (or a single entity, for isolated testing) in dev mode |
| `sucata build <file> [--icon <path>] [--optimize]` | Package the game into a native distributable |
| `sucata shader build <file.glsl>` | Compile a `.glsl` shader into the engine's `.schd` format |
| `sucata shader create <file> [--post-processing\|-pp] [--font\|-f]` | Scaffold a starter `.glsl` shader |
| `sucata version` | Print the engine version and release date |

## sucata run

Runs a Sucata Lua script file

**arguments**

- `<file>` - It is the path of the main lua file
- `--entity <entity_file>` (optional) - It is the lua file of a entity (for testing)

```bash
sucata run main.lua
```

## sucata build

Builds Sucata into a binary game for the current OS

**arguments**

- `<file>` - It is the path of the main lua file
- `--icon <path>` (optional) - It is the icon for your game
- `--optimize` (optional) - Recompresses opaque images (PNG/BMP/TGA without an alpha channel) as JPEG to shrink the packaged assets. Images with transparency are left untouched, and dimensions are never changed.

```bash
sucata build main.lua --icon src://icon.png
```

> For while, this is a Sucata limitation, you can only build the game for the current OS.

## sucata version

Show the Sucata game engine version and release date

## sucata shader

Util to shaders on sucata

### sucata shader build

Builds a .glsl file to sucata shader file

**arguments**

- `<file>` - It is the path of the .glsl shader you wants do build

### sucata shader create

Create a base .glsl shader file

**arguments**

- `<file>` - It is the path for create a .glsl from template
- `--post-processing` / `-pp` (optional) - Its a flag to create a template for post-processing [WIP]
- `--font` / `-f` (optional) - Its a flag to create a template for font shader
