# Starting the Project

## Getting Started

First, make sure that the **Sucata Engine** is installed on your system.  
You can follow the installation guide on the [installation page](../installation.md).

To verify that Sucata is installed correctly, run the following command in your terminal:

```bash
sucata version
```

If the installation was successful, the engine version will be printed in the terminal.

Next, create a new folder for your project. Inside this folder, create a file called `main.lua`.
This file will be the **main entry point of your game**.

To run the game at any point during this tutorial, use:

```bash
sucata run main.lua
```

## Project layout

Sucata doesn't enforce a scaffold, but this tutorial follows the layout every Sucata project uses.
We will create these folders as we need them:

```
meteors/
├── main.lua               Entry point — the file you pass to `sucata run`
├── config.lua             Window setup
├── behaviours/            Reusable init/tick/draw/free tables
│   ├── init.lua           Aggregator: exposes every behaviour in one table
│   └── player/            Behaviours only one entity uses live in a subfolder named after it
│       └── init.lua       Nested aggregator — reached as `behaviours.player.*`
├── entities/              Factory functions: fn(...) -> { state = {...}, behaviours = {...} }
├── mutators/              Pure functions that read/mutate one slice of an entity's state
│   └── init.lua           Aggregator — reached as `mutators.forces.*`, `mutators.health.*`
├── commons/               Shared constants — data only, no logic, no state
├── scenes/                Functions returning a list of entities
├── sprites/               Texture assets
└── sounds/                Audio assets
```

Two naming rules keep this predictable:

- Files are `snake_case.lua`, and the key a module is exposed under in `init.lua` is exactly its
  file name (`draw_sprite.lua` → `draw_sprite`).
- A folder is always reached through its `init.lua`, never file by file: `require("behaviours")`
  and `require("mutators")` are the only entry points into those trees.

## Constants

Start with the screen size, since a lot of the game logic depends on it.

Create the file `commons/screen.lua`:

```lua
-- Data only: constants shared by behaviours, entities and mutators.
return {
	WIDTH = 960,
	HEIGHT = 540,
	MARGIN = 16,
}
```

> **Tip**
> Keeping constants in `commons/` instead of repeating `960` across a dozen files means changing
> the window size later is a one-line change.

## Configuration

To configure the game window, you can use functions from `sucata.window`.

It is recommended to place these settings in a separate file called `config.lua`.

```lua
local screen = require("commons.screen")

sucata.window.set_window_size(screen.WIDTH, screen.HEIGHT) -- Game window size, in pixels
sucata.window.set_keep_aspect(1) -- Maintain aspect ratio when resizing the window
-- (0 = off, 1 = keep aspect with bars, 2 = keep aspect with crop)

sucata.window.set_window_title("My First Game") -- Window title
sucata.window.set_max_fps(0) -- Frame rate cap (0 = uncapped)
sucata.window.set_vsync(1) -- Enable VSync (0 = off, 1 = on, higher values for specific intervals)

sucata.window.show_debug_info(true) -- Display debug information such as FPS and engine stats
sucata.window.set_window_icon("src://icon.png") -- Window icon
sucata.window.set_fullscreen(false) -- Enable or disable fullscreen mode
```

> **Note**
> Due to limitations of [sokol](https://github.com/floooh/sokol), VSync will always remain enabled.

> **Tip**
> Remember to disable `sucata.window.show_debug_info()` when releasing your game.

Finally, load the configuration file inside `main.lua`:

```lua
require("config")
```
