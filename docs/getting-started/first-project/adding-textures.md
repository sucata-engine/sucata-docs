# Adding Textures

In this section we will add **textures** to our game.  
We will add textures for the **player**, **meteors**, and **bullets**.

You can download the assets used in this tutorial from  
[here](https://github.com/sucata-engine/meteors-sucata/tree/main/sprites).

---

## Bullet Texture

Let's start with the simplest texture: the **bullet**.

Update the state in `entities/bullet.lua`:

```lua
local behaviours = require("behaviours")

local function bullet(x, y)
	---@type Entity
	return {
		state = {
			x = x,
			y = y,
			texture = "src://sprites/bullet.png", -- Texture path
			width = 16,  -- Bullet width
			height = 16  -- Bullet height
		},

		behaviours = {
			behaviours.forces,
			behaviours.bullet.shot,
			behaviours.bullet.hit_meteor,
			behaviours.draw_sprite,
		}
	}
end

return bullet
```

> **Note**
> `src://` represents the **root directory of the project**.
> Use this prefix whenever referencing files inside the project.

The `draw_sprite` behaviour automatically renders textures when the `texture`, `width`, and `height` fields are present in the entity state.

The result should look like this:

![](./images/bullet-texture.png)

---

## Meteor Texture

The meteor texture uses a **texture atlas** so the meteor appearance changes based on its health.

First, define the texture in the meteor entity (`entities/meteor.lua`):

```lua
local behaviours = require("behaviours")

local function meteor()
	---@type Entity
	return {
		state = {
			y = -16,
			texture = "src://sprites/meteor.png", -- Meteor texture
			atlas_size = 8 -- Split the texture into 8 horizontal frames
		},

		behaviours = {
			behaviours.random_start_position,
			behaviours.forces,
			behaviours.meteor.fall,
			behaviours.meteor.damage,
			behaviours.draw_sprite,
		}
	}
end

return meteor
```

Now pick the frame from the remaining health. The health slice already has its own behaviour, so
the atlas frame goes there — update `behaviours/meteor/damage.lua`:

```lua
---@type Behaviour
return {
	init = function(state)
		state.health = state.health or math.random(1, 5) -- Random meteor health
	end,

	tick = function(state)
		state.atlas_x = state.health - 1 -- Select the frame based on meteor health
	end
}
```

Now the meteor sprite will change depending on its health:

![](./images/meteor-texture.gif)

---

## Player Texture

For the player we will add a **texture atlas** that represents the ship inclination.

This is player-specific, so create the behaviour in `behaviours/player/inclination.lua`:

```lua
---@type Behaviour
return {
	init = function(state)
		state.inclination = 2 -- Middle frame of the ship atlas
	end,

	tick = function(state)
		local delta = sucata.time.get_delta()

		if sucata.input.is_held("left", "a") then
			state.inclination = sucata.math.clamp(
				state.inclination - (15 * delta),
				0,
				4
			)

		elseif sucata.input.is_held("right", "d") then
			state.inclination = sucata.math.clamp(
				state.inclination + (15 * delta),
				0,
				4
			)

		else
			state.inclination = sucata.math.lerp(
				state.inclination,
				2,
				delta * 10
			)
		end

		state.atlas_x = math.floor(state.inclination) -- Frame of the texture atlas
	end
}
```

Register the behaviour in `behaviours/player/init.lua`:

```lua
return {
	controller  = require("behaviours.player.controller"),
	inclination = require("behaviours.player.inclination"),
	shooter     = require("behaviours.player.shooter"),
}
```

Now update the player entity in `entities/player.lua`:

```lua
local behaviours = require("behaviours")

local function player(x, y)
	---@type Entity
	return {
		state = {
			x = x,
			y = y,
			texture = "src://sprites/ship.png", -- Player texture
			atlas_size = 8 -- Split texture into 8 frames
		},

		behaviours = {
			behaviours.forces,
			behaviours.player.controller,
			behaviours.player.inclination,
			behaviours.player.shooter,
			behaviours.draw_sprite,
		}
	}
end

return player
```

Now the player ship will tilt depending on movement:

![](./images/player-animation.gif)
