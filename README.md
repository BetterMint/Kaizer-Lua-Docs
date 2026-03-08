# Kaizer Lua Scripting

## Overview

The **Lua** tab in Kaizer lets you write and run **Lua scripts** to add custom features, menu tabs, overlays, and in-game behavior. Scripts live in the **`lua`** folder (next to your Kaizer config folder) and must have the **`.lua`** extension.

---

## Where are my scripts?

| Item | Path |
|------|------|
| **Folder** | `lua` (same directory as the `Kaizer` config folder) |
| **Example** | If config is `C:\Game\Kaizer\`, scripts go in `C:\Game\Kaizer\lua\` |
| **Config per script** | Options are saved in `lua\config\<script_name>.json` (e.g. `myscript.json` for `myscript.lua`) |

Only **`.lua`** files are listed in the Lua menu.

---

## Lua menu (Lua tab)

- **Refresh list** – Rescan the `lua` folder for `.lua` files.
- **Checkbox** – Enable or disable a script. When enabled, the script’s `on_tick()` runs every frame.
- **Edit** – Open the script in the in-menu editor.
- **Stop** – Disable and stop the script immediately.
- **New script** – Create a new file (opens editor with a template).

---

## In-menu editor

- Edit script content and click **Save** to write to the `lua` folder.
- Create new scripts with **New script** and save with the filename you want (must end in `.lua`).
- **Syntax highlighting** for comments, strings, keywords, and numbers.
- **Error detection** – syntax errors are shown with a red marker on the error line. Runtime errors from the running script appear above the editor.
- **Auto-complete** – type `kaizer.` to see suggestions; Tab to accept.

---

## Script lifecycle

1. **Load** – Script is loaded from disk when you enable it or refresh the list.
2. **Run** – While enabled, `on_tick()` is called every frame.
3. **Stop** – Uncheck the script or click **Stop**; options are saved automatically when the script stops.

---

## API reference

Scripts receive a global **`kaizer`** table. All functions require `kaizer` to be available (e.g. check `if kaizer then ... end`).

---

### `kaizer.menu` (ImGui – draw inside the menu)

| Function | Description |
|----------|-------------|
| **`kaizer.menu.text(str)`** | Draw a line of text. |
| **`kaizer.menu.button(label)`** | Draw a button. Returns `true` when clicked. |
| **`kaizer.menu.checkbox(label, value)`** | Draw a checkbox. Pass current state; returns new state (boolean). |
| **`kaizer.menu.slider_float(label, value, min, max, format?)`** | Float slider. Returns new value. `format` optional (default `"%.2f"`). |
| **`kaizer.menu.slider_int(label, value, min, max, format?)`** | Integer slider. Returns new value. `format` optional (default `"%d"`). |
| **`kaizer.menu.input_text(label, default)`** | Single-line text input. Returns `(text, changed)`. |
| **`kaizer.menu.begin(name, open?)`** | Begin a window. Returns `true` if visible. `open` is optional and currently for internal use – you normally call `begin("My Window")`. |
| **`kaizer.menu.end()`** | End the current window. |
| **`kaizer.menu.same_line()`** | Next widget on the same line. |
| **`kaizer.menu.spacing()`** | Vertical spacing. |
| **`kaizer.menu.separator()`** | Horizontal separator. |
| **`kaizer.menu.tree_node(label)`** | Start a tree node. Returns `true` if open. |
| **`kaizer.menu.tree_pop()`** | End the current tree node. |
| **`kaizer.menu.collapsing_header(label)`** | Collapsing section. Returns `true` if open. |
| **`kaizer.menu.add_tab(name, callback)`** | Add a custom tab to the sidebar (next to Lua, Skins, etc.). Call each frame in `on_tick` with a function that draws the tab content. |
| **`kaizer.menu.begin_child(id, w?, h?, border?)`** | Begin a child region inside the current window. Returns `true` if visible. |
| **`kaizer.menu.end_child()`** | End the current child region. Must be paired with `begin_child`. |
| **`kaizer.menu.color_edit3(label, r, g, b)`** | RGB color picker. Returns `r, g, b` (0–1). |
| **`kaizer.menu.color_edit4(label, r, g, b, a?)`** | RGBA color picker. Returns `r, g, b, a`. |
| **`kaizer.menu.combo(label, current, items, height?)`** | Dropdown. `items` = `"opt1;opt2;opt3"`. Returns `(new_index, changed)`. |
| **`kaizer.menu.progress_bar(fraction, w?, h?, overlay?)`** | Progress bar (0–1). Optional overlay text. |
| **`kaizer.menu.bullet_text(str)`** | Bullet-point text. |
| **`kaizer.menu.dummy(w, h)`** | Invisible spacing (pixels). |
| **`kaizer.menu.selectable(label, selected)`** | Selectable row. Returns `(new_selected, clicked)`. |
| **`kaizer.menu.set_cursor_pos(x, y)`** | Set cursor position in window. |
| **`kaizer.menu.get_cursor_pos()`** | Returns `x, y` of cursor. |
| **`kaizer.menu.indent(w?)`** | Indent content. |
| **`kaizer.menu.unindent(w?)`** | Unindent content. |
| **`kaizer.menu.tooltip(str)`** | Show tooltip if previous item hovered. |
| **`kaizer.menu.set_tooltip(str)`** | Set tooltip for current window. |
| **`kaizer.menu.begin_tooltip()`** / **`kaizer.menu.end_tooltip()`** | Manual tooltip block. |
| **`kaizer.menu.is_item_hovered()`** | `true` if last item is hovered. |
| **`kaizer.menu.is_item_clicked(mouse?)`** | `true` if last item clicked. `mouse` = 0 LMB, 1 RMB. |
| **`kaizer.menu.is_item_active()`** | `true` if last item is being interacted with. |
| **`kaizer.menu.set_next_item_width(w)`** | Set width of next widget. |

#### Basic `kaizer.menu` patterns

**Simple settings group**

```lua
function on_tick()
    if not kaizer or not kaizer.menu_is_open() then return end

    kaizer.menu.text("My basic settings")
    kaizer.menu.separator()

    local enabled = kaizer.options.get("enabled")
    if enabled == nil then enabled = false end
    enabled = kaizer.menu.checkbox("Enable feature", enabled)
    kaizer.options.set("enabled", enabled)

    local strength = kaizer.options.get("strength") or 1.0
    strength = kaizer.menu.slider_float("Strength", strength, 0.0, 2.0)
    kaizer.options.set("strength", strength)

    kaizer.options.save()
end
```

**Two-column layout with `same_line`**

```lua
function on_tick()
    if not kaizer or not kaizer.menu_is_open() then return end

    kaizer.menu.text("Keybinds")
    kaizer.menu.separator()

    kaizer.menu.text("Main toggle")
    kaizer.menu.same_line()
    local active = kaizer.utils.key_down(kaizer.VK.X)
    kaizer.menu.text(active and "[X held]" or "[not held]")
end
```

**Custom sidebar tab (menu integration)**

```lua
function on_tick()
    if not kaizer then return end

    kaizer.menu.add_tab("My Lua Tab", function()
        kaizer.menu.text("Hello from my Lua tab")
        if kaizer.menu.button("Click me") then
            kaizer.log("My Lua button was clicked")
        end
    end)
end
```

---

### `kaizer.render` (draw on screen – overlay)

All coordinates and sizes are in screen pixels. Colors are **R, G, B, A** (0–255).

| Function | Description |
|----------|-------------|
| **`kaizer.render.rect(x1, y1, x2, y2, r, g, b, a, filled)`** | Axis-aligned rectangle. |
| **`kaizer.render.rect_rounded(x1, y1, x2, y2, rounding, r, g, b, a, filled)`** | Rounded rectangle. `rounding` = corner radius in pixels. |
| **`kaizer.render.line(x1, y1, x2, y2, r, g, b, a, thick)`** | Line. `thick` = thickness in pixels. |
| **`kaizer.render.circle(x, y, radius, r, g, b, a, filled, segs?)`** | Circle. `segs` = segment count (optional). |
| **`kaizer.render.triangle(x1, y1, x2, y2, x3, y3, r, g, b, a, filled)`** | Triangle (three corners). |
| **`kaizer.render.quad(x1, y1, x2, y2, x3, y3, x4, y4, r, g, b, a, filled)`** | Quad (four corners). |
| **`kaizer.render.text(x, y, text, r, g, b, a)`** | Text at position. |
| **`kaizer.render.text_size(text)`** | Returns `width, height` of the text in pixels. |
| **`kaizer.render.push_font(index)`** | Use a different ImGui font for subsequent `render.text` calls. `index` corresponds to fonts available in the menu (0 = default ESP font, etc.). |
| **`kaizer.render.pop_font()`** | Restore the previous font after `push_font`. |
| **`kaizer.render.image(texId, x, y, w, h)`** | Draw a loaded image. `texId` from `load_image`. |
| **`kaizer.render.load_image(path)`** | Load image (PNG/JPG/BMP). Relative path = under Lua folder. Returns texture id (0 on failure). Cached by path. |
| **`kaizer.render.arc(cx, cy, radius, a_min, a_max, r, g, b, a, thick?, segs?)`** | Arc (degrees). |
| **`kaizer.render.arc_filled(cx, cy, radius, a_min, a_max, r, g, b, a, segs?)`** | Filled pie slice. |
| **`kaizer.render.bezier(x1,y1, x2,y2, x3,y3, x4,y4, r,g,b,a, thick?, segs?)`** | Cubic bezier curve. |
| **`kaizer.render.polygon(points, r, g, b, a)`** | Convex polygon. `points` = table of `{x,y}`. |
| **`kaizer.render.ngon(cx, cy, radius, sides, r, g, b, a, filled?)`** | Regular polygon (e.g. hexagon). |
| **`kaizer.render.rect_multi_color(x1,y1, x2,y2, r1,g1,b1,a1, r2,g2,b2,a2)`** | Gradient rectangle. |

#### Basic `kaizer.render` patterns

**Static crosshair**

```lua
function on_tick()
    if not kaizer or not kaizer.render or not kaizer.sdk then return end

    local cx = kaizer.sdk.screen_center_x()
    local cy = kaizer.sdk.screen_center_y()
    local size = 8

    kaizer.render.line(cx - size, cy, cx + size, cy, 0, 255, 0, 255, 1.5)
    kaizer.render.line(cx, cy - size, cx, cy + size, 0, 255, 0, 255, 1.5)
end
```

**Outlined text label**

```lua
local function draw_text_outlined(x, y, text, r, g, b, a)
    -- black outline
    kaizer.render.text(x + 1, y + 1, text, 0, 0, 0, a)
    -- main text
    kaizer.render.text(x, y, text, r, g, b, a)
end

function on_tick()
    if not kaizer or not kaizer.render then return end
    draw_text_outlined(30, 40, "Kaizer Lua", 255, 255, 255, 255)
end
```

**Health bar style rectangle**

```lua
function draw_bar(x, y, w, h, pct)
    pct = math.max(0.0, math.min(1.0, pct))
    local fill_w = w * pct
    kaizer.render.rect(x, y, x + w, y + h, 0, 0, 0, 180, true)
    kaizer.render.rect(x + 1, y + 1, x + 1 + fill_w - 2, y + h - 1, 0, 200, 0, 220, true)
end
```

---

### `kaizer.options` (per-script saved settings)

Settings are stored in **`lua/config/<script_stem>.json`** (e.g. `myscript.lua` → `myscript.json`). They are loaded when the script runs and saved when the script is stopped or when you call `save()`.

| Function | Description |
|----------|-------------|
| **`kaizer.options.get(key)`** | Get value for `key`. Returns `nil` if not set. Value can be boolean, number, or string. |
| **`kaizer.options.set(key, value)`** | Set `key` to `value` (boolean, number, or string). In-memory until `save()`. |
| **`kaizer.options.save()`** | Write current options to this script’s config file. |
| **`kaizer.options.has(key)`** | Returns `true` if `key` exists in saved options. |
| **`kaizer.options.keys()`** | Returns table of all option keys. |

**Example – checkbox that persists:**

```lua
function on_tick()
    if not kaizer or not kaizer.menu_is_open() then return end
    local enabled = kaizer.options.get("enabled")
    if enabled == nil then enabled = false end
    enabled = kaizer.menu.checkbox("Enable feature", enabled)
    kaizer.options.set("enabled", enabled)
    kaizer.options.save()
end
```

---

### `kaizer.utils`

| Function | Description |
|----------|-------------|
| **`kaizer.utils.delta_time()`** | Time since last frame (seconds). |
| **`kaizer.utils.screen_size()`** | Returns `width, height` of the game window. |
| **`kaizer.utils.key_down(vk)`** | Returns `true` if key with virtual key code `vk` is currently down. Use `kaizer.VK` constants. |
| **`kaizer.utils.time()`** | ImGui time (seconds since start). |
| **`kaizer.utils.get_key_name(vk)`** | Human-readable key name for a VK code (e.g. for display). |
| **`kaizer.utils.clamp(val, min, max)`** | Clamp `val` between `min` and `max`. Returns clamped value. |
| **`kaizer.utils.lerp(a, b, t)`** | Linear interpolation: `a + (b - a) * t`. |
| **`kaizer.utils.distance_2d(x1, y1, x2, y2)`** | 2D distance between two points. |
| **`kaizer.utils.round(n)`** | Round to nearest integer. |
| **`kaizer.utils.floor(n)`** / **`kaizer.utils.ceil(n)`** | Floor/ceiling. |
| **`kaizer.utils.abs(n)`** | Absolute value. |
| **`kaizer.utils.min(a, b)`** / **`kaizer.utils.max(a, b)`** | Min/max. |
| **`kaizer.utils.sign(n)`** | -1, 0, or 1. |
| **`kaizer.utils.random()`** | Random 0–1. |
| **`kaizer.utils.random(lo, hi)`** | Random integer in [lo, hi] inclusive. |
| **`kaizer.utils.remap(v, in_lo, in_hi, out_lo, out_hi)`** | Map value between ranges. |
| **`kaizer.utils.angle_diff(a, b)`** | Shortest angle diff in degrees (-180 to 180). |
| **`kaizer.utils.format_time(seconds)`** | Format as `"M:SS"` or `"H:MM:SS"`. |
| **`kaizer.utils.should_tick(interval)`** | Helper to throttle work: returns `true` only when `interval` seconds have elapsed since last `true` for this script. |

#### Basic `kaizer.utils` patterns

**Timed toggle (cooldown-style)**

```lua
local last_toggle = 0
local enabled = false

function on_tick()
    if not kaizer then return end
    local now = kaizer.utils.time()

    if kaizer.utils.key_down(kaizer.VK.F1) and now - last_toggle > 0.25 then
        enabled = not enabled
        last_toggle = now
    end

    if kaizer.menu_is_open() then
        kaizer.menu.text("F1 toggle: " .. tostring(enabled))
    end
end
```

**FPS / delta time display**

```lua
function on_tick()
    if not kaizer or not kaizer.render then return end

    -- Only update this overlay ~10 times per second to reduce cost
    if not kaizer.utils.should_tick(0.1) then return end

    local dt = kaizer.utils.delta_time()
    local fps = (dt > 0) and (1.0 / dt) or 0
    kaizer.render.text(10, 10, "dt=" .. tostring(dt) .. "  fps=" .. tostring(fps), 255, 255, 255, 255)
end
```

---

### `kaizer.sdk` (game data)

| Function | Description |
|----------|-------------|
| **`kaizer.sdk.camera_x/y/z()`** | Camera world position. |
| **`kaizer.sdk.camera_pitch/yaw()`** | Camera rotation (pitch, yaw). |
| **`kaizer.sdk.camera_info()`** | Returns table: `x`, `y`, `z`, `pitch`, `yaw`, `roll`, `fov`. |
| **`kaizer.sdk.screen_w/h()`** | Screen width and height. |
| **`kaizer.sdk.screen_center_x/y()`** | Center of the screen. |
| **`kaizer.sdk.has_world()`** | `true` if world is valid. |
| **`kaizer.sdk.has_player()`** | `true` if local player exists. |
| **`kaizer.sdk.player_x/y/z()`** | Local player world position. |
| **`kaizer.sdk.player_info()`** | Returns table: `has_player`, `x`, `y`, `z`, `distance_to_camera`. |
| **`kaizer.sdk.fov()`** | Current field of view. |
| **`kaizer.sdk.target_player()`** | `true` if aimbot has a current target. |
| **`kaizer.sdk.target_player_x/y/z()`** | World position of the current aimbot target (0 if no target). |
| **`kaizer.sdk.target_info()`** | Returns table: `has_target`, `x`, `y`, `z`, `distance_to_camera`, `screen_x`, `screen_y`, `on_screen`. |
| **`kaizer.sdk.locked_target()`** | `true` if a target is locked (target lock feature). |
| **`kaizer.sdk.local_hitscan()`** | `true` if local hero is hitscan. |
| **`kaizer.sdk.local_melee()`** | `true` if local hero is melee. |
| **`kaizer.sdk.melee_range()`** | Melee range value. |
| **`kaizer.sdk.enemy_overlay_count()`** | Number of enemies in the enemy overlay data. |
| **`kaizer.sdk.enemy_overlay_hero_id(i)`** | Hero ID for enemy at index `i`. |
| **`kaizer.sdk.enemy_overlay_name(i)`** | Hero name for enemy at index `i`. |
| **`kaizer.sdk.enemy_overlay_ult(i)`** | Ultimate percentage (0–100) for enemy at index `i`. |
| **`kaizer.sdk.enemy_overlay_player_name(i)`** | Player name for enemy at index `i`. |
| **`kaizer.sdk.enemy_overlay_is_bot(i)`** | `true` if enemy at index `i` is a bot. |
| **`kaizer.sdk.enemy_overlay_role(i)`** | Role for enemy at index `i`: 0 = Tank, 1 = Damage, 2 = Support, 3 = Unknown. |
| **`kaizer.sdk.enemy_overlay_ai_difficulty(i)`** | AI difficulty for enemy at index `i` (0 = human, >0 = bot level). |
| **`kaizer.sdk.enemy_overlay_info(i)`** | Returns a table with fields: `hero_id`, `name`, `ult`, `role`, `ai_difficulty`, `is_bot`, `player_name`. |
| **`kaizer.sdk.get_hero_name(heroId)`** | Hero name string for a hero ID. |
| **`kaizer.sdk.get_hero_role(heroId)`** | Role for a hero ID: 0 = Tank, 1 = Damage, 2 = Support, 3 = Unknown. |
| **`kaizer.sdk.world_to_screen(wx, wy, wz)`** | Project world position to screen. Returns `(sx, sy, on_screen)`. Uses game SDK projection. |
| **`kaizer.sdk.world_time()`** | Current world time in seconds (from game world). |
| **`kaizer.sdk.camera_roll()`** | Camera roll (rotation). |
| **`kaizer.sdk.ally_count()`** | Number of teammates (from live teammate names list). |
| **`kaizer.sdk.ally_name(i)`** | Teammate name at index `i` (hero name string). Use 1-based index. |
| **`kaizer.sdk.distance_3d(x1,y1,z1,x2,y2,z2)`** | 3D distance between two world positions. |
| **`kaizer.sdk.player_distance_to_camera()`** | Distance from local player to camera. |
| **`kaizer.sdk.target_distance()`** | Distance from local player to current target. |
| **`kaizer.sdk.target_distance_to_camera()`** | Distance from target to camera. |
| **`kaizer.sdk.is_valid_target()`** | `true` if current target is valid. |
| **`kaizer.sdk.set_control_rotation(pitch, yaw, roll?)`** | Set view angles (e.g. spinbot). Call each frame to override camera. |
| **`kaizer.sdk.get_control_rotation()`** | Returns `pitch, yaw, roll` (view angles from controller). |
| **`kaizer.sdk.angles_to_point(wx, wy, wz)`** | Returns `pitch, yaw` (degrees) to look at world point from camera. Use with target position for custom aim. |
| **`kaizer.sdk.fov_radius_pixels()`** | FOV circle radius in pixels (from current FOV). |
| **`kaizer.sdk.aimbot_fov_radius()`** | Current aimbot FOV circle radius in pixels. |

#### Basic `kaizer.sdk` patterns

**World/target info overlay**

```lua
function on_tick()
    if not kaizer or not kaizer.sdk or not kaizer.render then return end

    -- Camera and player info
    local px = kaizer.sdk.player_x()
    local py = kaizer.sdk.player_y()
    local pz = kaizer.sdk.player_z()
    local fov = kaizer.sdk.fov()

    kaizer.render.text(20, 40, "Player: " .. tostring(px) .. " " .. tostring(py) .. " " .. tostring(pz), 255, 255, 255, 255)
    kaizer.render.text(20, 60, "FOV: " .. tostring(fov), 255, 255, 255, 255)

    -- Target marker
    if kaizer.sdk.target_player() then
        local tx = kaizer.sdk.target_player_x()
        local ty = kaizer.sdk.target_player_y()
        local tz = kaizer.sdk.target_player_z()
        local sx, sy, on_screen = kaizer.sdk.world_to_screen(tx, ty, tz)
        if on_screen then
            kaizer.render.circle(sx, sy, 18, 255, 0, 0, 220, false, 32)
        end
    end
end
```

**Enemy list using overlay data**

Use concatenation instead of `string.format` so hero names with `%` (e.g. from game data) don't cause errors.

```lua
function on_tick()
    if not kaizer or not kaizer.sdk or not kaizer.render then return end

    local n = kaizer.sdk.enemy_overlay_count()
    if n <= 0 then return end

    local y = 80
    for i = 1, n do
        local name = kaizer.sdk.enemy_overlay_name(i)
        local ult = kaizer.sdk.enemy_overlay_ult(i)
        local role = kaizer.sdk.enemy_overlay_role(i)
        local role_name = (role == 0 and "Tank")
            or (role == 1 and "DPS")
            or (role == 2 and "Support")
            or "Unknown"

        local line = name .. " [" .. role_name .. "] ult=" .. tostring(ult) .. "%"
        kaizer.render.text(20, y, line, 255, 255, 255, 255)
        y = y + 16
    end
end
```

---

### `kaizer.input`

| Function | Description |
|----------|-------------|
| **`kaizer.input.press_key(vk)`** | Simulate key press (down). |
| **`kaizer.input.release_key(vk)`** | Simulate key release (up). |
| **`kaizer.input.tap_key(vk)`** | Press and release a key (single tap). |
| **`kaizer.input.mouse_move(dx, dy)`** | Relative mouse movement in pixels. |
| **`kaizer.input.mouse_position()`** | Returns `x, y` (screen coords). |
| **`kaizer.input.mouse_left_down()`** | `true` if left mouse button down. |
| **`kaizer.input.mouse_right_down()`** | `true` if right mouse button down. |

#### Basic `kaizer.input` patterns

```lua
function on_tick()
    if not kaizer then return end

    -- Tap key when F2 is held (use tap_key for convenience)
    if kaizer.utils.key_down(kaizer.VK.F2) then
        kaizer.input.tap_key(kaizer.VK.Space)
    end
end
```

---

### `kaizer.hotkey`

| Function | Description |
|----------|-------------|
| **`kaizer.hotkey.add(label, vk, is_hold, is_active)`** | Show this keybind in the **Hotkeys** overlay. Call each frame in `on_tick`. `is_hold` = hold-to-activate; `is_active` = currently active (e.g. key held or toggle on). |

#### Basic `kaizer.hotkey` pattern

```lua
local enabled = false

function on_tick()
    if not kaizer then return end

    if kaizer.utils.key_down(kaizer.VK.F3) then
        enabled = true
    else
        enabled = false
    end

    kaizer.hotkey.add("My F3 feature", kaizer.VK.F3, true, enabled)
end
```

---

### `kaizer.download`

| Function | Description |
|----------|-------------|
| **`kaizer.download.url(url, path)`** | Download `url` to file `path`. Returns `(ok, error_message)`. |

**Example – simple HTTP download into the Lua folder**

```lua
function on_tick()
    if not kaizer or not kaizer.menu_is_open() then return end

    if kaizer.menu.button("Download example file") then
        local ok, err = kaizer.download.url(
            "https://example.com/test.txt",
            "downloads/test.txt"
        )
        if ok then
            kaizer.log("Download succeeded")
        else
            kaizer.log("Download failed: " .. tostring(err))
        end
    end
end
```

---

### `kaizer.file`

| Function | Description |
|----------|-------------|
| **`kaizer.file.read(name)`** | Read file from Lua folder. Returns content or `nil, err`. |
| **`kaizer.file.write(name, content)`** | Write content to a file in the Lua folder. Returns success (boolean). |
| **`kaizer.file.folder()`** | Returns the Lua folder path. |
| **`kaizer.file.exists(name)`** | Returns `true` if file exists in Lua folder. |
| **`kaizer.file.list(subdir?)`** | Returns table of filenames in folder (optionally in subdir). |
| **`kaizer.file.append(name, content)`** | Append content to file. Returns success. |
| **`kaizer.file.create_folder(name)`** | Create folder (and parents). Returns success. |
| **`kaizer.file.folder_exists(name)`** | Returns `true` if folder exists. |

**Example – write and read a small text file**

```lua
function on_tick()
    if not kaizer or not kaizer.menu_is_open() then return end

    if kaizer.menu.button("Write hello.txt") then
        local ok = kaizer.file.write("hello.txt", "Hello from Kaizer Lua!\n")
        kaizer.log(ok and "Wrote hello.txt" or "Failed to write hello.txt")
    end

    if kaizer.menu.button("Read hello.txt") then
        local content, err = kaizer.file.read("hello.txt")
        if content then
            kaizer.log("hello.txt content: " .. tostring(content))
        else
            kaizer.log("Read failed: " .. tostring(err))
        end
    end
end
```

---

### `kaizer.VK` (virtual key codes)

Use these with **`kaizer.utils.key_down(vk)`**, **`kaizer.hotkey.add`**, and **`kaizer.input.press_key`** / **`release_key`**.

| Keys | Constants |
|------|-----------|
| Mouse | **LMB**, **RMB**, **MMB**, **Mouse4**, **Mouse5** |
| Modifiers | **Shift**, **Ctrl**, **Alt**, **Space** |
| Navigation | **Tab**, **Enter**, **Escape**, **Backspace**, **Insert**, **Delete**, **Home**, **End**, **PageUp**, **PageDown**, **Left**, **Right**, **Up**, **Down** |
| Function | **F1** … **F24** |
| Numpad | **Numpad0** … **Numpad9** |
| Letters & digits | **A**–**Z**, **0**–**9** |

---

### Other

| Function | Description |
|----------|-------------|
| **`kaizer.log(text)`** | Log a message to the console. |
| **`kaizer.menu_is_open()`** | Returns `true` if the Kaizer menu is visible. |

---

## Examples

### 1. Toggle overlay with saved option

```lua
function on_tick()
    if not kaizer then return end
    local enabled = kaizer.options.get("enabled")
    if enabled == nil then enabled = false end
    if kaizer.menu_is_open() then
        enabled = kaizer.menu.checkbox("Show circle", enabled)
        kaizer.options.set("enabled", enabled)
        kaizer.options.save()
    end
    if enabled and kaizer.sdk and kaizer.render then
        local cx = kaizer.sdk.screen_center_x()
        local cy = kaizer.sdk.screen_center_y()
        kaizer.render.circle(cx, cy, 50, 255, 0, 0, 150, true)
    end
end
```

### 2. Custom sidebar tab and hotkey in overlay

```lua
function on_tick()
    if not kaizer then return end
    kaizer.menu.add_tab("My Tab", function()
        kaizer.menu.text("Hello from Lua")
        if kaizer.menu.button("Click me") then
            kaizer.log("Button clicked!")
        end
    end)
    local active = kaizer.utils.key_down(kaizer.VK.X)
    kaizer.hotkey.add("My Feature", kaizer.VK.X, true, active)
end
```

### 3. Slider + options and image

```lua
local texId = 0
function on_tick()
    if not kaizer or not kaizer.render then return end
    if texId == 0 then texId = kaizer.render.load_image("my_icon.png") end
    if kaizer.menu_is_open() then
        local size = kaizer.options.get("size")
        if size == nil then size = 1.0 end
        size = kaizer.menu.slider_float("Size", size, 0.5, 2.0)
        kaizer.options.set("size", size)
        kaizer.options.save()
        if texId and texId ~= 0 then
            kaizer.render.image(texId, 100, 100, 64 * size, 64 * size)
        end
    end
end
```

### 4. Enemy overlay list

**Important:** Always guard with `if not kaizer or not kaizer.sdk then return end` so the script exits safely when the game world is not ready.

```lua
function on_tick()
    if not kaizer or not kaizer.sdk or not kaizer.render then return end

    local n = kaizer.sdk.enemy_overlay_count()
    if n <= 0 then return end

    local y = 50
    for i = 1, n do  -- 1-based index (Lua convention)
        local name = kaizer.sdk.enemy_overlay_name(i)
        local ult = kaizer.sdk.enemy_overlay_ult(i)
        local role = kaizer.sdk.enemy_overlay_role(i)
        local role_name = (role == 0 and "Tank")
            or (role == 1 and "DPS")
            or (role == 2 and "Support")
            or "Unknown"

        local line = name .. " [" .. role_name .. "] ult=" .. tostring(ult) .. "%"
        kaizer.render.text(20, y, line, 255, 255, 255, 255)
        y = y + 18
    end
end
```

### 5. Target position and world-to-screen (game SDK)

```lua
function on_tick()
    if not kaizer or not kaizer.sdk or not kaizer.render then return end
    if kaizer.sdk.target_player() then
        local tx = kaizer.sdk.target_player_x()
        local ty = kaizer.sdk.target_player_y()
        local tz = kaizer.sdk.target_player_z()
        local sx, sy, on_screen = kaizer.sdk.world_to_screen(tx, ty, tz)
        if on_screen then
            kaizer.render.circle(sx, sy, 20, 255, 0, 0, 200, false)
        end
    end
    local t = kaizer.sdk.world_time()
    kaizer.render.text(20, 20, "World time: " .. tostring(math.floor(t)), 255, 255, 255, 255)
end
```

### 6. More `kaizer.sdk` usage (distances, allies, FOV ring)

```lua
function on_tick()
    if not kaizer or not kaizer.sdk or not kaizer.render then return end

    -- Show basic camera / player info
    if kaizer.sdk.has_world() and kaizer.sdk.has_player() then
        local px = kaizer.sdk.player_x()
        local py = kaizer.sdk.player_y()
        local pz = kaizer.sdk.player_z()
        local dist_cam = kaizer.sdk.player_distance_to_camera()
        kaizer.render.text(20, 80,
            "Player: " .. tostring(px) .. " " .. tostring(py) .. " " .. tostring(pz) ..
            "  dist_cam=" .. tostring(math.floor(dist_cam)),
            200, 255, 200, 255)
    end

    -- Ally list (names only)
    local ally_count = kaizer.sdk.ally_count()
    local y = 100
    for i = 1, ally_count do
        local name = kaizer.sdk.ally_name(i)
        kaizer.render.text(20, y, "[ALLY] " .. tostring(name), 100, 200, 255, 255)
        y = y + 14
    end

    -- FOV circle (uses fov_radius_pixels and aimbot_fov_radius)
    local cx = kaizer.sdk.screen_center_x()
    local cy = kaizer.sdk.screen_center_y()
    local base_radius = kaizer.sdk.fov_radius_pixels()
    local aim_radius = kaizer.sdk.aimbot_fov_radius()
    if base_radius > 0 then
        kaizer.render.circle(cx, cy, base_radius, 0, 150, 255, 120, false, 64)
    end
    if aim_radius > 0 then
        kaizer.render.circle(cx, cy, aim_radius, 255, 255, 0, 180, false, 64)
    end
end
```

**Note:** Use `..` concatenation instead of `string.format` when displaying game data (names, etc.) so values containing `%` do not cause errors.

**Spinbot / set view angles (e.g. spin yaw without moving camera)**

```lua
function on_tick()
    if not kaizer or not kaizer.sdk then return end
    local t = kaizer.utils.time() * 360.0  -- degrees per second
    local pitch = 0
    local yaw = t % 360.0
    kaizer.sdk.set_control_rotation(pitch, yaw, 0)
end
```

**Aim at current target from Lua (angles to target)**

```lua
function on_tick()
    if not kaizer or not kaizer.sdk or not kaizer.render then return end
    if kaizer.sdk.target_player() then
        local tx = kaizer.sdk.target_player_x()
        local ty = kaizer.sdk.target_player_y()
        local tz = kaizer.sdk.target_player_z()
        local pitch, yaw = kaizer.sdk.angles_to_point(tx, ty, tz)
        -- e.g. draw indicator or set view: kaizer.sdk.set_control_rotation(pitch, yaw, 0)
        kaizer.render.text(20, 60, "Pitch: " .. tostring(math.floor(pitch)) .. " Yaw: " .. tostring(math.floor(yaw)), 255, 255, 255, 255)
    end
end
```

---

## Complete function list (quick reference)

| Category | Functions |
|----------|-----------|
| **kaizer.menu** | `text`, `button`, `checkbox`, `slider_float`, `slider_int`, `input_text`, `begin`, `end`, `same_line`, `spacing`, `separator`, `tree_node`, `tree_pop`, `collapsing_header`, `add_tab`, `color_edit3`, `color_edit4`, `combo`, `progress_bar`, `bullet_text`, `dummy`, `selectable`, `set_cursor_pos`, `get_cursor_pos`, `indent`, `unindent`, `tooltip`, `set_tooltip`, `begin_tooltip`, `end_tooltip`, `is_item_hovered`, `is_item_clicked`, `is_item_active`, `set_next_item_width` |
| **kaizer.render** | `rect`, `rect_rounded`, `line`, `circle`, `triangle`, `quad`, `text`, `text_size`, `image`, `load_image`, `arc`, `arc_filled`, `bezier`, `polygon`, `ngon`, `rect_multi_color` |
| **kaizer.options** | `get`, `set`, `save`, `has`, `keys` |
| **kaizer.utils** | `delta_time`, `screen_size`, `key_down`, `time`, `get_key_name`, `clamp`, `lerp`, `distance_2d`, `round`, `floor`, `ceil`, `abs`, `min`, `max`, `sign`, `random`, `remap`, `angle_diff`, `format_time` |
| **kaizer.sdk** | `camera_x`, `camera_y`, `camera_z`, `camera_pitch`, `camera_yaw`, `camera_roll`, `screen_w`, `screen_h`, `screen_center_x`, `screen_center_y`, `has_world`, `has_player`, `player_x`, `player_y`, `player_z`, `fov`, `target_player`, `target_player_x/y/z`, `locked_target`, `local_hitscan`, `local_melee`, `melee_range`, `enemy_overlay_*`, `get_hero_name`, `get_hero_role`, `world_to_screen`, `world_time`, `ally_count`, `ally_name`, `distance_3d`, `player_distance_to_camera`, `target_distance`, `target_distance_to_camera`, `is_valid_target`, `set_control_rotation`, `get_control_rotation`, `angles_to_point`, `fov_radius_pixels`, `aimbot_fov_radius` |
| **kaizer.input** | `press_key`, `release_key`, `tap_key`, `mouse_move`, `mouse_position`, `mouse_left_down`, `mouse_right_down` |
| **kaizer.hotkey** | `add` |
| **kaizer.file** | `read`, `write`, `folder`, `exists`, `list`, `append`, `create_folder`, `folder_exists` |
| **kaizer.download** | `url` |
| **Other** | `kaizer.log`, `kaizer.menu_is_open` |

**Tip:** Always guard scripts with `if not kaizer then return end`. Use `..` concatenation instead of `string.format` when displaying game data (names can contain `%`).

---

## Libraries (require)

- **`package.path`** includes the Lua folder and **`lua/libs/`**. Put shared modules in **`lua/libs/`** and use **`require("mymod")`**.
- Use **`kaizer.download.url(url, path)`** to download libraries into the Lua folder or **libs**.

---
