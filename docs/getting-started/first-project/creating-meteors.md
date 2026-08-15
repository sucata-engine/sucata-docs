# Creating Meteors

Now we will create the enemies of the game: **meteors** that fall from the sky.

## Movement with named forces

Instead of writing `state.x` and `state.y` from every behaviour, Sucata projects accumulate
**named forces** in `state.forces` and integrate them once per frame. Gravity, a jump and player
input can then coexist without overwriting each other.

The named-force table is a *slice* of the entity state, and slices are read and written through
**mutators**: plain modules of stateless functions, kept in `mutators/`.

Create the file `mutators/forces.lua`:

```lua
local function get_force(state, force_type)
	if not state.forces then
		return { x = 0, y = 0 }
	end

	return state.forces[force_type] or { x = 0, y = 0 }
end

local function set_force(state, force_type, value)
	if not state.forces then -- Safe state modification
		return
	end

	state.forces[force_type] = value
end

local function add_force(state, force_type, value)
	local force = get_force(state, force_type)

	set_force(state, force_type, {
		x = force.x + (value.x or 0),
		y = force.y + (value.y or 0),
	})
end

local function clear_force(state, force_type)
	set_force(state, force_type, { x = 0, y = 0 })
end

return {
	get_force = get_force,
	set_force = set_force,
	add_force = add_force,
	clear_force = clear_force,
}
```

> **Note**
> Every mutator guards defensively (`if not state.forces then return end`) — a mutator can be called
> against an entity whose behaviour list never initialised that slice.

Mutators have an aggregator too. Create `mutators/init.lua`:

```lua
return {
	forces = require("mutators.forces"),
}
```

From now on, any file that needs a mutator does:

```lua
local mutators = require("mutators")

mutators.forces.set_force(state, "gravity", { x = 0, y = 100 })
```

Now the behaviour that integrates those forces. Create `behaviours/forces.lua`:

```lua
---@type Behaviour
return {
	init = function(state)
		state.x = state.x or 0
		state.y = state.y or 0
		state.forces = state.forces or {} -- The slice mutators.forces writes into
		state.velocity_x = 0
		state.velocity_y = 0
	end,

	tick = function(state)
		local delta = sucata.time.get_delta()
		local velocity_x, velocity_y = 0, 0

		for _, force in pairs(state.forces) do
			velocity_x = velocity_x + force.x
			velocity_y = velocity_y + force.y
		end

		state.velocity_x = velocity_x
		state.velocity_y = velocity_y

		state.x = state.x + (velocity_x * delta)
		state.y = state.y + (velocity_y * delta)
	end
}
```

Register it in `behaviours/init.lua`:

```lua
return {
	draw_sprite = require("behaviours.draw_sprite"),
	forces      = require("behaviours.forces"),

	player = require("behaviours.player"),
}
```

> **Important**
> `behaviours.forces` creates `state.forces` in its `init`, so it must be listed **before** any
> behaviour that pushes a force during its own `init`. Otherwise the mutator's guard silently
> discards the force and the entity never moves.

---

## Moving the player with forces

Now update `behaviours/player/controller.lua` to push a named force instead of writing `state.x`:

```lua
local mutators = require("mutators")
local screen = require("commons.screen")

local BOUND = 20

---@type Behaviour
return {
	init = function(state)
		state.speed = state.speed or 200
	end,

	tick = function(state)
		local direction = 0

		if sucata.input.is_held("left", "a") and state.x > BOUND then
			direction = -1
		elseif sucata.input.is_held("right", "d") and state.x < screen.WIDTH - BOUND then
			direction = 1
		end

		-- Setting the force to zero when no key is held stops the player,
		-- without touching whatever else is pushing the entity.
		mutators.forces.set_force(state, "movement", {
			x = direction * state.speed,
			y = 0,
		})
	end,
}
```

And add the `forces` behaviour to `entities/player.lua`:

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
			behaviours.forces,            -- First: creates state.forces and integrates it
			behaviours.player.controller, -- Pushes the "movement" force
			behaviours.draw_sprite,       -- Rendering last
		}
	}
end

return player
```

---

## Creating the Meteor Behaviours

Meteors need two behaviours of their own, so they go in `behaviours/meteor/`.

The first one makes the meteor fall and reports when it reaches the ground.

Create the file `behaviours/meteor/fall.lua`:

```lua
local mutators = require("mutators")
local screen = require("commons.screen")

---@type Behaviour
return {
	init = function(state)
		sucata.scene.add_tag(state, "meteor")              -- Tag the entity as a meteor
		state.speed = state.speed or math.random(100, 200) -- Random fall speed

		mutators.forces.set_force(state, "gravity", { x = 0, y = state.speed })
	end,

	tick = function(state)
		if state.y > screen.HEIGHT then -- Reached the ground
			sucata.events.emit("meteor_reached", state) -- Emit an event
			sucata.scene.destroy(state)                 -- Destroy the meteor
		end
	end
}
```

The second one holds the meteor's health, which bullets will damage later.

Create the file `behaviours/meteor/damage.lua`:

```lua
---@type Behaviour
return {
	init = function(state)
		state.health = state.health or math.random(1, 5) -- Random meteor health
	end
}
```

Create the subfolder aggregator `behaviours/meteor/init.lua`:

```lua
-- Behaviours only meteors use.
return {
	damage = require("behaviours.meteor.damage"),
	fall   = require("behaviours.meteor.fall"),
}
```

And expose it in `behaviours/init.lua`:

```lua
return {
	draw_sprite = require("behaviours.draw_sprite"),
	forces      = require("behaviours.forces"),

	meteor = require("behaviours.meteor"),
	player = require("behaviours.player"),
}
```

---

## Creating the Meteor Entity

Now we will create the meteor entity.

Create the file `entities/meteor.lua`:

```lua
local behaviours = require("behaviours")

local function meteor(x, y)
	---@type Entity
	return {
		state = {
			x = x, -- Meteor position
			y = y
		},

		behaviours = {
			behaviours.forces,        -- Before meteor.fall, which pushes a force on init
			behaviours.meteor.fall,   -- Fall speed and reaching the ground
			behaviours.meteor.damage, -- Health
			behaviours.draw_sprite,   -- Render the meteor
		}
	}
end

return meteor
```

Spawn a meteor in `main.lua`:

```lua
local meteor = require("entities.meteor")

sucata.scene.spawn(meteor(300, 100))
```

The game should now look like this:

![White square falling](images/meteor-falling.gif)

---

## Creating a Random Start Position Behaviour

Now we will create a behaviour that spawns entities at a **random position**. Any entity could
reuse this one, so it stays at the top level of `behaviours/`.

Create the file `behaviours/random_start_position.lua`:

```lua
local screen = require("commons.screen")

---@type Behaviour
return {
	init = function(state)
		-- When x/y are missing, pick a random point inside the screen bounds
		state.x = state.x or math.random(screen.MARGIN, screen.WIDTH - screen.MARGIN)
		state.y = state.y or math.random(screen.MARGIN, screen.HEIGHT - screen.MARGIN)
	end
}
```

Register the behaviour in `behaviours/init.lua`:

```lua
return {
	draw_sprite           = require("behaviours.draw_sprite"),
	forces                = require("behaviours.forces"),
	random_start_position = require("behaviours.random_start_position"),

	meteor = require("behaviours.meteor"),
	player = require("behaviours.player"),
}
```

---

## Spawning Meteors at Random Positions

Now we will update the meteor entity so it spawns at a random X position.

Update `entities/meteor.lua`:

```lua
local behaviours = require("behaviours")

local function meteor()
	---@type Entity
	return {
		state = {
			y = -16 -- Start slightly above the screen
		},

		behaviours = {
			behaviours.random_start_position, -- Random x (y is already set)
			behaviours.forces,
			behaviours.meteor.fall,
			behaviours.meteor.damage,
			behaviours.draw_sprite,
		}
	}
end

return meteor
```

Now meteors will spawn with a **random X position** and fall from the top of the screen.
