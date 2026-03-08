# Kaizer – Discord documentation

Post this in your Discord for users. You can split into multiple messages by section.

**Lua scripting** is documented separately (link or attach `LUA_SCRIPTING.md` when you send it).

---

## Features overview

### Combo Builder

- **What it does:** Runs a **sequence of key presses** (and optional second key per step) with hold time, delay, and optional camera turn.
- **Modes:**
  - **Trigger** – Press the activation key once; the full sequence runs to the end.
  - **Hold** – Hold the activation key to run; release to stop.
- **Conditions:**
  - **Require Aimbot** – Only run when the aimbot key is held.
  - **Require Melee** – Only run when melee assist is active.
  - **Lock to Nearest** – Only run when there is a valid aimbot target in range.
  - **Max Target Dist** – Max distance to target (meters); 0 = no limit.
- **Steps:** Each step has **Key**, optional **Key2**, **Hold (ms)**, **Delay (ms)**, and optional **Yaw / Pitch** (camera turn in degrees).
- **Behavior:**
  - Combos run only when the **game window has focus**.
  - The combo’s activation key appears in the **Hotkeys overlay** when active.
  - L/R keys (e.g. LShift / RShift, LMB / RMB) are supported via the key picker.

---

### Hotkeys overlay

- Shows **all active keybinds** in one place: Aimbot, Heal Bot, Bullet TP, HealBot BTP, Melee, Combos, and **Lua-registered hotkeys** (including toggles).
- Scale and visibility are configurable in the menu.
- Lua scripts can add their own entries so users see every bind in one overlay.

---

### Menu and UI

- **Icons** – Sidebar uses Segoe MDL2 Assets icons (target, heart, camera, gear, etc.).
- **Search** – **Ctrl+F** in the sidebar to filter tabs by name.
- **Font** – In **Overlays → Font & appearance** you can choose the menu font and size.
- **Right-click settings panels** – Resizable; size is saved in config.
- **View / Overlays / Combo** – Each has its own main tab for easier access.
- **Custom tabs** – Lua scripts can add their own sidebar tabs (documented in the Lua scripting guide).

---

### Lua scripting (summary)

- **Lua tab** – Enables writing and running **Lua scripts** for custom features, menu tabs, overlays, and in-game logic.
- **In-menu editor** – Edit and save scripts without leaving the game.
- **Per-script options** – Each script can save its own settings in a separate config file (`lua/config/<script>.json`), like the main menu config.
- **Full Lua docs** – Sent separately (API reference, examples, options, render, SDK, hotkeys, etc.).

---

### Other features

- **Session stats** – Optional overlay for session statistics.
- **Keybinds overlay** – Single overlay listing all hotkeys (main + Lua).
- **Config save/load** – Save and load different config files from the Settings tab.
- **Theme and style** – Menu theme, accent colors, background effects (e.g. snow, glint), and animations.
