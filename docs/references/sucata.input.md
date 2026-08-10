# sucata.input

The input module of the Sucata.

---

## Key

Input key and button names.

**values**

- `"mouse_left"`
- `"mouse_right"`
- `"mouse_middle"`
- `"a"` to `"z"`
- `"space"`
- `" "` (space character)
- `"escape"`
- `"esc"`
- `"enter"`
- `"return"`
- `"shift"`
- `"ctrl"`
- `"control"`
- `"alt"`
- `"apostrophe"`
- `"tab"`
- `"up"`
- `"down"`
- `"left"`
- `"right"`

---

## sucata.input.get_mouse_position

Gets the current mouse position.

**return**

- x `number` - The x coordinate of the mouse  
- y `number` - The y coordinate of the mouse  

---

## sucata.input.get_relative_mouse_position

Gets the current mouse position based on the camera position, rotation and zoom.

**return**

- x `number` - The x coordinate of the mouse  
- y `number` - The y coordinate of the mouse  

---

## sucata.input.get_mouse_scroll

Gets the current mouse scroll values.

**return**

- x `number` - The horizontal scroll value  
- y `number` - The vertical scroll value  

---

## sucata.input.get_key

Gets the last pressed key character code.

**return**

- key_code `number` - The character code of the last pressed key  

---

## sucata.input.is_pressed

Checks if any of the given keys or buttons were pressed this frame.

**parameters**

- ... `Key` - Key or button names to check  

**return**

- pressed `boolean` - Whether any of the keys/buttons were pressed  

---

## sucata.input.is_held

Checks if any of the given keys or buttons are currently held down.

**parameters**

- ... `Key` - Key or button names to check  

**return**

- held `boolean` - Whether any of the keys/buttons are held  

---

## sucata.input.is_released

Checks if any of the given keys or buttons were released this frame.

**parameters**

- ... `Key` - Key or button names to check  

**return**

- released `boolean` - Whether any of the keys/buttons were released  

---

## sucata.input.is_hover

Registers an area as hoverable for the current frame and returns whether the mouse
is currently over it. When multiple registered areas overlap, only the one with the
highest `z_index` reports `true`, so call this every frame for each interactive
region you want the mouse to reach (e.g. inside a `draw` behaviour).

**parameters**

- area `table`
  - id `string` - Unique id, can be the entity id  
  - x `number` - X position of the area  
  - y `number` - Y position of the area  
  - width `number` - Width of the area  
  - height `number` - Height of the area  
  - z_index? `number` - Z-index used to break ties between overlapping areas (optional)  
  - fixed? `boolean` - Whether the area is fixed to the screen (optional)  

**return**

- hovering `boolean` - Whether the mouse is hovering over this area (and no higher-z area covers it)  
