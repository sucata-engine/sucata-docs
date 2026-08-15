# Playing Sounds

Now let's add **sound effects and music** to our game.

All audio assets used in this tutorial are **CC0** and were obtained from  
[freesound](https://freesound.org/).

---

## Health Loss Sound

This sound will play when a meteor reaches the end of the screen.

Update the behaviour `behaviours/meteor/fall.lua`:

```lua
local mutators = require("mutators")
local screen = require("commons.screen")

---@type Behaviour
return {
	init = function(state)
		sucata.scene.add_tag(state, "meteor")
		state.speed = state.speed or math.random(100, 200)

		mutators.forces.set_force(state, "gravity", { x = 0, y = state.speed })
	end,

	tick = function(state)
		if state.y > screen.HEIGHT then
			sucata.events.emit("meteor_reached", state)

			sucata.audio.play({
				sound = "src://sounds/lose.ogg"
			})

			sucata.scene.destroy(state)
		end
	end
}
```

> **Note**
> `src://` represents the **root directory of the project**.
> Use this prefix whenever referencing files inside the project.

---

## Shooting Sound

This sound will play when the player fires a bullet.

Update `behaviours/bullet/shot.lua`:

```lua
local mutators = require("mutators")

---@type Behaviour
return {
	init = function(state)
		state.speed = state.speed or 400

		mutators.forces.set_force(state, "shot", { x = 0, y = -state.speed })

		-- Play the shooting sound when the bullet is spawned
		sucata.audio.play({
			sound = "src://sounds/shoot.ogg"
		})
	end,

	tick = function(state)
		if state.y < -16 then
			sucata.scene.destroy(state)
		end
	end
}
```

---

## Meteor Hit Sound

Now we will add a sound effect when a meteor is hit by a bullet.

Update `behaviours/bullet/hit_meteor.lua`:

```lua
local mutators = require("mutators")

---@type Behaviour
return {
	tick = function(state)
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
				mutators.health.remove(meteor)

				if mutators.health.is_dead(meteor) then
					sucata.events.emit("meteor_destroyed", meteor)
					sucata.scene.destroy(meteor)
				end

				-- Play the explosion sound when the meteor is hit
				sucata.audio.play({
					sound = "src://sounds/explosion.ogg"
				})

				sucata.scene.destroy(state)
				break
			end
		end
	end
}
```

---

## Playing Music

Now we will add **background music** to the game. Any entity could reuse this behaviour, so it
stays at the top level of `behaviours/`.

Create a new behaviour `behaviours/music.lua`:

```lua
---@type Behaviour
return {
	init = function(state)
		if state.music then
			state.music_id = sucata.audio.play({
				sound = state.music,
				loop = true,
				volume = 0.5
			})
		end
	end,

	free = function(state)
		if state.music_id then
			sucata.audio.stop(state.music_id) -- Stop the music when the entity is removed
		end
	end
}
```

Register the behaviour in `behaviours/init.lua`:

```lua
return {
	draw_sprite           = require("behaviours.draw_sprite"),
	forces                = require("behaviours.forces"),
	music                 = require("behaviours.music"),
	random_start_position = require("behaviours.random_start_position"),

	bullet       = require("behaviours.bullet"),
	game_manager = require("behaviours.game_manager"),
	meteor       = require("behaviours.meteor"),
	player       = require("behaviours.player"),
}
```

Now create an entity responsible for playing the background music.

Create `entities/music.lua`:

```lua
local behaviours = require("behaviours")

local function music()
	---@type Entity
	return {
		state = {
			music = "src://sounds/music.ogg"
		},

		behaviours = {
			behaviours.music,
		}
	}
end

return music
```

Finally, load the music entity in `main.lua`:

```lua
local music = require("entities.music")

sucata.scene.load_global("music", music())
```

> **Tip**
> `sucata.scene.load_global` keeps the entity alive across scene reloads, so the music doesn't
> restart every time you load a new scene.

The background music will now **play in a loop** while the game is running.
