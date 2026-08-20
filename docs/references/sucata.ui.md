# sucata.ui

The immediate-mode UI module of Sucata, backed by [microui](https://github.com/rxi/microui). Every `draw_*`/`popup_open` call must happen inside a behaviour's `draw(state)` function, same as `sucata.graphic.draw_rect`/`draw_text` — calling them elsewhere is a no-op (logs an error).

**No window needed**: a widget drawn without an open `draw_window`/`draw_popup` around it lands on an implicit full-screen root canvas automatically — there's no `draw_root()` to call.

**Contract**: `draw_window`, `draw_popup`, and `begin_treenode` return whether the container is open. Only call the matching `end_window`/`end_popup`/`end_treenode` if that call returned `true` — this mirrors vanilla microui's own C API (`if (mu_begin_window(...)) { ...; mu_end_window(...); }`). Calling `end_*` unconditionally, or skipping it after a `true`, will desync the UI state. The same applies to `layout_begin_column`/`layout_end_column`, except that pair is unconditional (always call both, one after the other).

---

## UIStyle

Shared optional overrides accepted by every widget call (`draw_label`, `draw_text`, `draw_button`, `draw_checkbox`, `draw_slider`, `draw_textbox`, `draw_header`, `begin_treenode`). All fields are optional and combinable.

**fields**
- x? `number` - Overrides the widget's auto-layout x position; must be set together with `y`/`width`/`height`
- y? `number` - Overrides the widget's auto-layout y position; must be set together with `x`/`width`/`height`
- width? `number` - Overrides the widget's auto-layout width; must be set together with `x`/`y`/`height`
- height? `number` - Overrides the widget's auto-layout height; must be set together with `x`/`y`/`width`
- text_size? `number` - Text pixel height (default font height is 18); scales the bitmap font — large values look blocky since it's a small fixed-size atlas, not a scalable font
- color? `{r, g, b, a?}` - Text color, 0.0-1.0 per channel
- background_color? `{r, g, b, a?}` - Widget frame/background color, 0.0-1.0 per channel
- border_color? `{r, g, b, a?}` - Border color, 0.0-1.0 per channel; set alpha to 0 to hide the border

---

## UIWindowProps

**fields**
- title? `string` - Window title, also used as its unique id (default: `"Window"`)
- x? `number` - The x position (default: 40)
- y? `number` - The y position (default: 40)
- width? `number` - The width (default: 200)
- height? `number` - The height (default: 150)
- transparent? `boolean` - Hide the window body background, keeps the title bar (default: false)
- movable? `boolean` - Whether the title bar can be dragged to move the window (default: true)
- resizable? `boolean` - Whether the resize handle is shown (default: true)
- color? `{r, g, b, a?}` - Title text color
- background_color? `{r, g, b, a?}` - Window body background color
- border_color? `{r, g, b, a?}` - Window border color

---

## sucata.ui.draw_window

Begins a window. Widgets drawn between this and `end_window()` are placed inside it.

**parameters**

- props `UIWindowProps`

**return**

- open `boolean`

---

## sucata.ui.end_window

Ends a window. Only call this if the matching `draw_window()` call returned `true`.

---

## sucata.ui.popup_open

Triggers a popup to open at the current mouse position. Call this from e.g. a button click, then draw the popup with `draw_popup` using the same `name` on a later call (typically later in the same frame, or subsequent frames).

**parameters**

- props `{name: string}`

---

## sucata.ui.draw_popup

Begins a popup window. Only actually open after a matching `popup_open()` call; closes automatically when the user clicks outside it.

**parameters**

- props `{name: string}`

**return**

- open `boolean`

---

## sucata.ui.end_popup

Ends a popup. Only call this if the matching `draw_popup()` call returned `true`.

---

## sucata.ui.draw_label

Draws a text label (no word wrap).

**parameters**

- props `UIStyle & {text?: string}`

---

## sucata.ui.draw_text

Draws word-wrapped text.

**parameters**

- props `UIStyle & {text?: string}`

---

## sucata.ui.draw_button

Draws a button.

**parameters**

- props `UIStyle & {text?: string}`

**return**

- clicked `boolean`

---

## sucata.ui.draw_checkbox

Draws a checkbox. State persists across frames keyed by `props.id`.

**parameters**

- props `UIStyle & {id: string, text?: string}`

**return**

- changed `boolean`
- checked `boolean`

---

## sucata.ui.draw_slider

Draws a slider. State persists across frames keyed by `props.id`.

**parameters**

- props `UIStyle & {id: string, value?: number, low?: number, high?: number, step?: number}` - `value` is the initial value, only used the first time this `id` is seen; `low`/`high` default to 0/100, `step` defaults to 0 (continuous)

**return**

- changed `boolean`
- value `number`

---

## sucata.ui.draw_textbox

Draws a text input box. State persists across frames keyed by `props.id`.

**parameters**

- props `UIStyle & {id: string, text?: string}` - `text` is the initial value, only used the first time this `id` is seen

**return**

- changed `boolean`
- submitted `boolean` - `true` on the frame Enter was pressed
- text `string` - the current text

---

## UIHeaderProps

**fields**
- text? `string` - The header label, also used as its unique id (default: `""`)
- expanded? `boolean` - Whether it's expanded the first time this label is seen (default: false)

---

## sucata.ui.draw_header

Draws a collapsible header/section title. Expanded state is tracked internally by microui, keyed by `props.text` — no `id` field needed.

**parameters**

- props `UIStyle & UIHeaderProps`

**return**

- open `boolean`

---

## UITreeNodeProps

**fields**
- text? `string` - The tree node label, also used as its unique id (default: `""`)
- expanded? `boolean` - Whether it's expanded the first time this label is seen (default: false)

---

## sucata.ui.begin_treenode

Begins a collapsible tree node. Widgets drawn between this and `end_treenode()` are indented and only rendered while it's expanded. Expanded state is tracked internally by microui, keyed by `props.text`.

**parameters**

- props `UIStyle & UITreeNodeProps`

**return**

- open `boolean`

---

## sucata.ui.end_treenode

Ends a tree node. Only call this if the matching `begin_treenode()` call returned `true`.

---

## sucata.ui.layout_row

Starts a new row of widgets with fixed/relative column widths, similar to vanilla microui's `mu_layout_row`. Every widget drawn after this call occupies the next column; once `#widths` widgets have been placed, the row wraps and reuses the same widths again. Calling it again resets the columns.

**parameters**

- props `{widths?: number[], height?: number}` - Each entry in `widths` is a column width in pixels: positive is a fixed width, negative fills remaining row space (e.g. `-1` takes the rest, two negative columns split what's left), `0`/omitted uses the widget's default width. `height` is the row height in pixels (`0`/omitted uses the widget's default height).

**example**

```lua
-- 3 columns: 86px fixed, then split the remaining space two ways
sucata.ui.layout_row({ widths = { 86, -110, -1 } })
sucata.ui.draw_label({ text = "Test buttons:" })
sucata.ui.draw_button({ text = "Button 1" })
sucata.ui.draw_button({ text = "Button 2" })
```

---

## sucata.ui.layout_begin_column

Begins a sub-layout column, occupying the current layout cell (as produced by the enclosing `layout_row`). Useful for stacking widgets vertically inside one column of a row while a sibling column has its own widgets. Always pair with `layout_end_column()`.

---

## sucata.ui.layout_end_column

Ends a column opened with `layout_begin_column()`.

**example**

```lua
sucata.ui.layout_row({ widths = { 140, -1 } })

sucata.ui.layout_begin_column()
if sucata.ui.begin_treenode({ text = "Test 1" }) then
  sucata.ui.draw_label({ text = "Hello" })
  sucata.ui.draw_label({ text = "world" })
  sucata.ui.end_treenode()
end
sucata.ui.layout_end_column()

sucata.ui.layout_begin_column()
sucata.ui.draw_text({ text = "Some wrapped text next to the tree, in its own column." })
sucata.ui.layout_end_column()
```

---

## Example

```lua
-- widgets placed directly on screen, no visible window - no draw_root() needed
sucata.ui.draw_label({ text = "Free-floating widgets" })

if sucata.ui.draw_button({ text = "Click me", background_color = { 0.85, 0.25, 0.25, 1 } }) then
  print("clicked")
end

-- widgets inside a window
local open = sucata.ui.draw_window({ title = "Demo", x = 40, y = 80, width = 300, height = 200 })
if open then
  local changed, checked = sucata.ui.draw_checkbox({ id = "opt1", text = "Enable thing" })
  local _, vol = sucata.ui.draw_slider({ id = "vol", value = 50, low = 0, high = 100 })

  sucata.ui.end_window()
end
```
