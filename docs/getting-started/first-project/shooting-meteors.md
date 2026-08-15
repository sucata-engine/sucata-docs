# Shooting Meteors

Now it's time to **shoot some meteors**.  
In this section we will create bullets and implement the logic required to destroy meteors and earn points.

---

## Creating the Bullet Behaviours

Bullets need two behaviours of their own, so they go in `behaviours/bullet/`.

The first one shoots the bullet upwards and cleans it up when it leaves the screen.

Create the file `behaviours/bullet/shot.lua`:

```lua
local mutators = require("mutators")

---@type Behaviour
return {
	init = function(state)
		state.speed = state.speed or 400 -- Bullet speed (default: 400)

		-- Named force, pointing up
		mutators.forces.set_force(state, "shot", { x = 0, y = -state.speed })
	end,

	tick = function(state)
		-- Destroy bullet when it leaves the screen
		if state.y < -16 then
			sucata.scene.destroy(state)
		end
	end
}
```

The second one handles the collision against every meteor.

Create the file `behaviours/bullet/hit_meteor.lua`:

```lua
local mutators = require("mutators")

---@type Behaviour
return {
	tick = function(state)
		-- Get all meteors in the scene
		local meteors = sucata.scene.get_entities_by_tag("meteor")

		for _, id in ipairs(meteors) do
			local meteor = sucata.scene.find_by_id(id)

			if meteor and sucata.math.overlapping({
				x = state.x - 8,
				y = state.y - 8,
				width = 16,
				height = 16
			}, {
				x = meteor.x - 16,
				y = meteor.y - 16,
				width = 32,
				height = 32
			}) then
				-- Damage the meteor
				mutators.health.remove(meteor)

				if mutators.health.is_dead(meteor) then
					sucata.events.emit("meteor_destroyed", meteor)
					sucata.scene.destroy(meteor)
				end

				-- Destroy the bullet
				sucata.scene.destroy(state)
				break
			end
		end
	end
}
```

> **Note**
> The bullet never writes `meteor.health` itself — it goes through `mutators.health`, the same
> module the game manager uses. That's the point of mutators: one place per slice of state.

Create the subfolder aggregator `behaviours/bullet/init.lua`:

```lua
-- Behaviours only bullets use.
return {
	hit_meteor = require("behaviours.bullet.hit_meteor"),
	shot       = require("behaviours.bullet.shot"),
}
```

And expose it in `behaviours/init.lua`:

```lua
return {
	draw_sprite           = require("behaviours.draw_sprite"),
	forces                = require("behaviours.forces"),
	random_start_position = require("behaviours.random_start_position"),

	bullet       = require("behaviours.bullet"),
	game_manager = require("behaviours.game_manager"),
	meteor       = require("behaviours.meteor"),
	player       = require("behaviours.player"),
}
```

---

## Creating the Bullet Entity

Now create the bullet entity in `entities/bullet.lua`:

```lua
local behaviours = require("behaviours")

local function bullet(x, y)
	---@type Entity
	return {
		state = {
			x = x,
			y = y
		},

		behaviours = {
			behaviours.forces,            -- Before bullet.shot, which pushes a force on init
			behaviours.bullet.shot,       -- Upward force and offscreen cleanup
			behaviours.bullet.hit_meteor, -- Collision against every meteor
			behaviours.draw_sprite,       -- Render the bullet
		}
	}
end

return bullet
```

---

## Creating the Shooter Behaviour

Now we will allow the player to shoot bullets. This one belongs to the player.

Create the file `behaviours/player/shooter.lua`:

```lua
-- Required lazily: entities require the behaviours aggregator, so requiring an entity
-- while `behaviours/init.lua` is still running would re-enter it.
local bullet

---@type Behaviour
return {
	tick = function(state)
		if sucata.input.is_pressed("space", "enter") then
			bullet = bullet or require("entities.bullet")
			sucata.scene.spawn(bullet(state.x, state.y - 16))
		end
	end
}
```

Register the behaviour in `behaviours/player/init.lua`:

```lua
return {
	controller = require("behaviours.player.controller"),
	shooter    = require("behaviours.player.shooter"),
}
```

---

## Adding the Shooter to the Player

Now add the shooter behaviour to the player entity in `entities/player.lua`:

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
			behaviours.forces,
			behaviours.player.controller, -- Player movement
			behaviours.player.shooter,    -- Shooting logic
			behaviours.draw_sprite,       -- Render player
		}
	}
end

return player
```

> **Note**
> Behaviours are executed in order.
> The player logic runs first, followed by shooting logic, and finally rendering.

---

Now when you run the game, it should look like this:

![](./images/player-shooting.gif)
