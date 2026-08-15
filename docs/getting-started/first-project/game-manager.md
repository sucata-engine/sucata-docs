# Game Manager

Now we will create a **Game Manager**, responsible for managing core game systems such as:

- Meteor spawning
- Player health
- Player points

All of its behaviours are specific to this entity, so they live in `behaviours/game_manager/`.

---

## Meteor Spawner

First, we will create a system that spawns a new meteor every **5 seconds**.

Create the file `behaviours/game_manager/meteor_spawner.lua`:

```lua
-- Required lazily: entities require the behaviours aggregator, so requiring an entity
-- while `behaviours/init.lua` is still running would re-enter it.
local meteor

---@type Behaviour
return {
	init = function(state)
		state.spawner_time = state.spawner_time or 5 -- Seconds between meteor spawns

		-- Create a timer that spawns meteors
		state.spawner_timer_id = sucata.time.create_timer(function()
			meteor = meteor or require("entities.meteor")
			sucata.scene.spawn(meteor())
		end, {
			time = state.spawner_time,
			loop = true,
			auto_start = true
		})
	end,

	free = function(state)
		if state.spawner_timer_id then
			-- Stop spawning once the game manager is gone (e.g. game over)
			sucata.time.stop_timer(state.spawner_timer_id)
		end
	end
}
```

> **Important**
> Behaviours must never `require` an entity at the top of the file. `behaviours/init.lua` is loaded
> first, and entities require it back — requiring an entity while the aggregator is still building
> would re-enter it. Require it lazily inside `init`/`tick` instead, as above.

Create the subfolder aggregator `behaviours/game_manager/init.lua`:

```lua
-- Behaviours only the game manager uses.
return {
	meteor_spawner = require("behaviours.game_manager.meteor_spawner"),
}
```

And expose it in `behaviours/init.lua`:

```lua
return {
	draw_sprite           = require("behaviours.draw_sprite"),
	forces                = require("behaviours.forces"),
	random_start_position = require("behaviours.random_start_position"),

	game_manager = require("behaviours.game_manager"),
	meteor       = require("behaviours.meteor"),
	player       = require("behaviours.player"),
}
```

---

## Game Manager Entity

Now we will create the **Game Manager entity**. It has no position, so it's a plain table instead
of a factory function.

Create the file `entities/game_manager.lua`:

```lua
local behaviours = require("behaviours")

---@type Entity
return {
	state = {},
	behaviours = {
		behaviours.game_manager.meteor_spawner,
	}
}
```

Spawn the Game Manager in `main.lua`:

```lua
local game_manager = require("entities.game_manager")

sucata.scene.spawn(game_manager)
```

---

## Player health

Now we will implement the player's health system.

Health is another slice of the entity state, so it gets its own mutator.

Create `mutators/health.lua`:

```lua
local function remove(state, amount)
	if not state.health then -- Safe state modification
		return
	end

	state.health = state.health - (amount or 1)
end

local function is_dead(state)
	return (state.health or 0) <= 0
end

return {
	remove = remove,
	is_dead = is_dead,
}
```

Register it in `mutators/init.lua`:

```lua
return {
	forces = require("mutators.forces"),
	health = require("mutators.health"),
}
```

> **Note**
> *Mutators* are modules of stateless functions that read and mutate **one named slice** of an
> entity's state. They are the only thing that should touch that slice, so the same logic can be
> shared by any behaviour instead of being duplicated.

Now create the behaviour `behaviours/game_manager/player_health.lua`:

```lua
local mutators = require("mutators")

---@type Behaviour
return {
	init = function(state)
		state.health = state.health or 3 -- Default player health

		-- "meteor_reached" is emitted by behaviours.meteor.fall
		sucata.events.on(state, "meteor_reached", function(_)
			mutators.health.remove(state)

			if mutators.health.is_dead(state) then
				-- Game over logic (to be implemented)
			end
		end)
	end
}
```

Register the behaviour in `behaviours/game_manager/init.lua`:

```lua
return {
	meteor_spawner = require("behaviours.game_manager.meteor_spawner"),
	player_health  = require("behaviours.game_manager.player_health"),
}
```

Add it to the Game Manager in `entities/game_manager.lua`:

```lua
local behaviours = require("behaviours")

---@type Entity
return {
	state = {},
	behaviours = {
		behaviours.game_manager.player_health,
		behaviours.game_manager.meteor_spawner,
	}
}
```

---

## Player Points

Next, we will implement the player scoring system, with the same mutator + behaviour pair.

Create the mutator `mutators/points.lua`:

```lua
local function add(state, points)
	if not state.points then -- Safe state modification
		return
	end

	state.points = state.points + points
end

return {
	add = add,
}
```

Register it in `mutators/init.lua`:

```lua
return {
	forces = require("mutators.forces"),
	health = require("mutators.health"),
	points = require("mutators.points"),
}
```

Now create the behaviour `behaviours/game_manager/player_points.lua`:

```lua
local mutators = require("mutators")

---@type Behaviour
return {
	init = function(state)
		state.points = 0 -- Initial score

		-- "meteor_destroyed" will be emitted by the bullet, in the next section
		sucata.events.on(state, "meteor_destroyed", function(_)
			mutators.points.add(state, 5)
		end)
	end
}
```

Register the behaviour in `behaviours/game_manager/init.lua`:

```lua
return {
	meteor_spawner = require("behaviours.game_manager.meteor_spawner"),
	player_health  = require("behaviours.game_manager.player_health"),
	player_points  = require("behaviours.game_manager.player_points"),
}
```

Update the Game Manager in `entities/game_manager.lua`:

```lua
local behaviours = require("behaviours")

---@type Entity
return {
	state = {},
	behaviours = {
		behaviours.game_manager.player_health,
		behaviours.game_manager.player_points,
		behaviours.game_manager.meteor_spawner,
	}
}
```

---

## Drawing the UI

Finally, we will draw the player UI on the screen.

Create the behaviour `behaviours/game_manager/draw_ui.lua`:

```lua
local screen = require("commons.screen")

---@type Behaviour
return {
	draw = function(state)
		sucata.graphic.draw_text({
			text = "Health: " .. state.health,
			x = screen.WIDTH - screen.MARGIN,
			y = 10,
			size = 16,
			align = "right",
			fixed = true, -- Ignore the camera: this is UI
		})

		sucata.graphic.draw_text({
			text = "Points: " .. state.points,
			x = screen.WIDTH - screen.MARGIN,
			y = 40,
			size = 16,
			align = "right",
			fixed = true,
		})
	end
}
```

Register the behaviour in `behaviours/game_manager/init.lua`:

```lua
return {
	draw_ui        = require("behaviours.game_manager.draw_ui"),
	meteor_spawner = require("behaviours.game_manager.meteor_spawner"),
	player_health  = require("behaviours.game_manager.player_health"),
	player_points  = require("behaviours.game_manager.player_points"),
}
```

Add the UI behaviour to the Game Manager:

```lua
local behaviours = require("behaviours")

---@type Entity
return {
	state = {},
	behaviours = {
		behaviours.game_manager.player_health,
		behaviours.game_manager.player_points,
		behaviours.game_manager.meteor_spawner,
		behaviours.game_manager.draw_ui, -- Rendering should happen last
	}
}
```

The UI should now appear like this:

![Player UI](images/player-ui.png)
