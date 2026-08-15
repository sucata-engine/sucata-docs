# Creating the Player

In this section, we will create the first entity of the game: the **player**.  
We will also add movement so the player can move around the screen.

## Creating the Draw Behaviour

First, we need a behaviour responsible for rendering a sprite.

Create the file `behaviours/draw_sprite.lua`:

```lua
---@type Behaviour
return {
	init = function(state) -- Called once when the entity enters the scene
		-- Try to get values from the state, fallback to defaults if missing
		state.x = state.x or 0
		state.y = state.y or 0
		state.width = state.width or 32
		state.height = state.height or 32
		state.texture = state.texture or ""
		state.atlas_x = state.atlas_x or 0
	end,

	draw = function(state) -- Called every frame during rendering
		sucata.graphic.draw_rect({
			x = state.x,
			y = state.y,
			width = state.width,
			height = state.height,
			z_index = state.z_index or 0,
			origin = 0.5, -- Center the sprite on x,y instead of the top-left corner
			texture = state.texture,
			atlas_size = state.atlas_size,
			atlas_x = state.atlas_x,
			atlas_y = state.atlas_y,
			opacity = state.opacity,
			fixed = state.fixed,
		})
	end
}
```

> **Tip**
> The `---@type Behaviour` annotation lets the [sucata Lua addon](../installation.md) type-check the
> table for you. Add it to every behaviour, and `---@type Entity` to every entity.

Now we need to register this behaviour in the **aggregator**.

Create the file `behaviours/init.lua`:

```lua
-- Aggregator: every behaviour, under a key equal to its file name.
return {
	draw_sprite = require("behaviours.draw_sprite"),
}
```

Every file that needs a behaviour requires the **folder**, never the file:

```lua
local behaviours = require("behaviours")
```

> **Important**
> Sucata reuses behaviours that share the same Lua table *pointer identity*.
> `require` caches by module name, so going through the aggregator means every entity in your game
> shares the exact same behaviour tables — that's what makes the reuse work.
> Never `require("behaviours.draw_sprite")` directly from an entity or another behaviour.

---

## Creating the Player Entity

Now we will create the player entity.

Create the file `entities/player.lua`:

```lua
local behaviours = require("behaviours")

local function player(x, y)
	---@type Entity
	return {
		state = {
			x = x, -- Player position
			y = y
		},

		behaviours = {
			behaviours.draw_sprite -- Render the player
		}
	}
end

return player
```

Now spawn the player in `main.lua`:

```lua
require("config")

local screen = require("commons.screen")
local player = require("entities.player")

sucata.scene.spawn(player(screen.WIDTH / 2, screen.HEIGHT - 40))
```

The game should now look like this:

![White square in the center of the screen](./images/first-player-draw.png)

---

## Creating the Player Behaviour

Next, we will add player movement using the keyboard.

Since this behaviour only makes sense for the player, it goes in a **subfolder named after the
entity**: `behaviours/player/`.

Create the file `behaviours/player/controller.lua`:

```lua
---@type Behaviour
return {
	init = function(state)
		state.speed = state.speed or 200 -- Default movement speed
	end,

	tick = function(state)
		local delta = sucata.time.get_delta() -- Time between frames

		if sucata.input.is_held("left", "a") then
			state.x = state.x - state.speed * delta
		elseif sucata.input.is_held("right", "d") then
			state.x = state.x + state.speed * delta
		end
	end
}
```

Player input is handled through the `sucata.input` module.

Subfolders have their own aggregator. Create `behaviours/player/init.lua`:

```lua
-- Behaviours only the player uses.
return {
	controller = require("behaviours.player.controller"),
}
```

And expose the subfolder from the main aggregator, `behaviours/init.lua`:

```lua
return {
	draw_sprite = require("behaviours.draw_sprite"),

	player = require("behaviours.player"), -- Resolves to behaviours/player/init.lua
}
```

Then add the behaviour to the entity in `entities/player.lua`:

```lua
local behaviours = require("behaviours")

local function player(x, y)
	---@type Entity
	return {
		state = {
			x = x,
			y = y
		},

		behaviours = {
			behaviours.player.controller, -- Handle player logic
			behaviours.draw_sprite        -- Render player
		}
	}
end

return player
```

> **Note**
> Behaviours are executed **in order**, so the player logic runs before rendering.

Now the player can move:

![White square moving](./images/first-player-movement.gif)

---

## Adding Screen Boundaries

To finish the initial player implementation, we will prevent the player from leaving the screen.

Update `behaviours/player/controller.lua`:

```lua
local screen = require("commons.screen")

local BOUND = 20 -- Distance kept from the screen edges

---@type Behaviour
return {
	init = function(state)
		state.speed = state.speed or 200
	end,

	tick = function(state)
		local delta = sucata.time.get_delta()

		-- Add horizontal boundaries
		if sucata.input.is_held("left", "a") and state.x > BOUND then
			state.x = state.x - state.speed * delta
		elseif sucata.input.is_held("right", "d") and state.x < screen.WIDTH - BOUND then
			state.x = state.x + state.speed * delta
		end
	end
}
```

Now we have the **first working version of the player**!

In the next section we will make meteors fall, and while doing that we will replace this direct
`state.x` manipulation with **named forces** — the way movement is normally handled in Sucata.
