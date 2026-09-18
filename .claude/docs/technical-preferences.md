# Technical Preferences — CHROME HEART CITY

## Engine & Language

- **Engine**: RPG Maker MZ, template core 1.1.1 (refresh after editor project creation)
- **Language**: JavaScript ES6+
- **Script entry**: `js/plugins/*.js`
- **Data**: `data/*.json`

## Naming Conventions (MZ plugin defaults)

- Plugin files: descriptive names, one system per file (e.g., `ChromeHeart_HackBattle.js`)
- Aliased methods: `const _Class_method = Class.prototype.method;` + `.call(this, ...)`
- Plugin params: read once via `PluginManager.parameters` with safe defaults; command args arrive as strings
- No globals beyond documented singletons (`$game*`, `$data*`); note-tags parsed via regex
- Every plugin wrapped in an IIFE with `"use strict"`

## Input & Platform

- **Target Platforms**: PC
- **Input Methods**: Keyboard/Mouse, Gamepad (partial)
- **Primary Input**: Keyboard/Mouse
- **Gamepad Support**: Partial
- **Touch Support**: None
- **Platform Notes**: MZ default UI is mouse+keyboard friendly; confirm gamepad mapping for menus before Alpha.

## Performance Budgets

- **Frame rate**: 60fps target (16.6ms frame budget)
- **Maps**: keep parallel-process events minimal per map; heavy logic on switches, not every frame
- **Lighting**: one lighting overlay approach (plugin TBD in prototype); test on low-end PC before Alpha
- **Battles**: Hack-command object counts bounded per troop (exact cap set in combat GDD)

## Testing

- **Standard**: playtest discipline — F8/F9 debug scene, console-clean gates (no errors on boot, map load, battle), fresh-save testing for new state
- **Save safety**: initialize per-save state via `DataManager.createGameObjects` / `Game_System` aliases; test plugin commands from a fresh save

## Forbidden Patterns

[TO BE CONFIGURED]

## Allowed Libraries

[TO BE CONFIGURED — lighting plugin and any others added only when actively integrated, not speculatively]

## Engine Specialists

- No MZ specialist agents installed (template ships Godot/Unity/Unreal sets only).
- Routing: all MZ code work follows the rpgm-engine-coding skill; `code-review` skill for reviews.
