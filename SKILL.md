---
name: whip
description: Build and debug resources for WHIP, the FiveM framework (whip_core). Use when writing, reviewing or fixing any resource that depends on whip_core — satellites that call its exports (inventory, money, jobs, vehicles, garages, properties, zones, doors, shops, stats, UI), register items, Director cards, Director tasks, skill check games, add-on weapons, object viewer options or settings pages, or when working inside whip_core itself. Triggers on whip_core, whip satellite, Whip.register, Whip.core, lib/satellite.lua, SecureEvent, SecureCallback, an fxmanifest with dependency 'whip_core', resources named whip_*, and on "the Director", "the whip panel", "/w", "/whip".
---

# WHIP (whip_core) resource development

WHIP is a complete FiveM framework: characters, inventory, money and banks, jobs, shops,
vehicles and garages, properties, doors, zones, stats, the UI, the Director (an in-game mission
builder) and an admin panel. `whip_core` owns all of it; every other resource is a **satellite**
that talks to it through exports.

## Before writing code

1. **Read the real API; don't guess an export name.** The source of truth is the `whip_core` on
   the server you are working on:
   - `whip_core/types/server.lua` and `types/client.lua` — every export with its parameters and
     returns (383 server, 206 client).
   - `whip_core/types/events.lua` — every event whip fires, with its payload shape.
   - `whip_core/lib/satellite.lua` — the `Whip` helper, heavily commented; each extension point
     (cards, tasks, weapons, skill checks, settings pages) has its own banner block.
   - `whip_core/docs.html` — the full reference with guides and search.
   - `whip_core/docs/*.md` — `nui-architecture`, `nui-theming`, `nui-inventory`,
     `director-architecture`, `properties`.

   Grep those for the domain you need before inventing anything. `references/exports.md` lists
   the export names by domain so you know what to grep for.

2. **Never reimplement what the core owns.** No second inventory, target, money, notification or
   progress bar. If a satellite needs behaviour the core does not expose, the change belongs in
   `whip_core` and its type stubs, not in a local copy.

3. **Copy an existing satellite's shape.** `whip_fuel` is the best small model: a config page, a
   shared file per concept, thin client files, all state on the server.

## The satellite contract

In `fxmanifest.lua`:

```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

name 'my_resource'
version '1.0.0'

dependency 'whip_core'
shared_script '@whip_core/lib/satellite.lua'   -- include it; never copy it
shared_script 'config.lua'
```

Then declare everything you add to the core inside `Whip.register`:

```lua
Whip.register(function()
    Whip.core:RegisterItem('lemonade', { name = 'Lemonade', weight = 0.3 })
    Whip.core:RegisterUseHandler('lemonade', drinkLemonade)
    Whip.core:SecureEvent('lemonade:buy', { rateLimit = 1000 }, buyLemonade)
end)
```

**Why it must be in `Whip.register`:** `whip_core` forgets every registration when it restarts.
`Whip.register` re-runs your function on each core start, so your items, events and cards come
back. It runs immediately if the core is already up. Running it twice is harmless — registration
is idempotent. Inside it, a refused declaring call (`false, why`) is printed in your console with
the export's name; the same call made straight through `exports.whip_core` is silent.

**Always call through `Whip.core:Name(...)`, not `exports.whip_core:Name(...)`.** It is the same
call plus two things that matter: it is typed (the editor shows params and returns), and a call
that waits (`Progress`, `SkillCheck`, anything that reads the database) returns
`false, 'whip_core stopped'` if the core dies mid-wait, instead of never returning and leaving
your handler stuck holding a latch. `'whip_core stopped'` means *no answer*, not *did not
happen* — a charge whose write already reached the database landed. Check; don't blindly retry.

The rest of the helper: `Whip.ready(ms)` / `Whip.isReady()`, `Whip.call(name, ...)` for a name
built at runtime, `Whip.requires('1.2.0')`, `Whip.has(exportName)`, `Whip.caller()`,
`Whip.registerWith(resource, fn)`, `Whip.sayStarted()`, `Whip.startProblem(fmt, ...)`,
`Whip.registry(name)`, `Whip.command(...)`, `Whip.onCoreLost(fn)`.

`Whip.onCoreLost` only fires for a resource **without** `dependency 'whip_core'` — with the
dependency, FiveM stops you first and you start fresh. Every satellite should declare the
dependency.

## Hard rules

- **The server owns state; the client asks.** Money, items, vehicle ownership, keys, job rank —
  the client may display them and never decide them. A client that needs a value calls a
  `SecureCallback`; a client that wants a change triggers a `SecureEvent` the server validates.
- **`SecureEvent` / `SecureCallback`, not `RegisterNetEvent`,** for anything a player's game can
  trigger. They give you rate limiting, payload shape checks and per-player sanity checks before
  your handler runs, and the core keeps errors thrown inside them under your resource's name.
- **One table payload, `<domain>:<pastTense>` names** for events you fire yourself — the rule the
  core follows (`whip_core:item:added`, `whip_core:zone:entered`).
- **Errors are already captured.** `lib/satellite.lua` wraps your threads, timers, commands,
  statebag handlers, NUI callbacks and red prints, hands failures to the core (`GetErrorLog`,
  `whipsupport`) and prints them as before. Don't add your own wrapper; opt out with
  `whip_error_tap 'no'` only if you genuinely wrap these yourself.
- **Manifest load order is load-bearing.** Files in whip manifests are ordered because each one
  uses tables the earlier ones build *at file scope*. Add a file in the position its dependencies
  demand, with the one-line comment saying why, as the surrounding lines do.
- **Settings go through `Whip.configPage`,** so admins change them in `/whip` → Settings instead
  of editing `config.lua` on disk. React to changes with `Whip.onConfigChanged(path, fn)` at file
  scope. Per-player options use `Whip.playerSetting`.

## Extension points

Each is declared through the helper and documented in its banner block in
`whip_core/lib/satellite.lua`. See `references/extension-points.md` for what each takes.

| Call | Adds |
| --- | --- |
| `Whip.card` / `Whip.removeCard` | A Director card in the palette (server) |
| `Whip.scenario` / `Whip.template` / `Whip.answerCard` | Director scenarios, templates, card answers |
| `Whip.task` / `Whip.removeTask` | A task in the Director's NPC and car Tasks list (both sides) |
| `Whip.skillCheckGame` | A skill check minigame played wherever whip runs one (both sides) |
| `Whip.weapons` | Add-on weapons your resource streams, offered wherever whip lists guns |
| `Whip.directorButton` | A button on a placed thing's Director page (client) |
| `Whip.objectViewerOption` | An option in the `/w` object viewer's list (client) |
| `Whip.configPage` / `Whip.playerSetting` | Pages in `/whip` → Settings |
| `Whip.padScreen` | Controller input for your own NUI screen (client) |
| `Whip.voiceSystem` / `Whip.voice` | Talk to whatever voice system is running |

Namespaced ids (`'<your resource>.<name>'`) are claimed by the first running resource to use the
namespace, and released when it stops.

## Editor setup

Copy `whip_core/types/luarc.template.json` to your resource root as `.luarc.json`. It points
`workspace.library` at `../whip_core/types` or `../[whip]/whip_core/types`; fix the path for any
other layout. Add a FiveM natives extension separately — these stubs cover whip only.

**When you add an export to `whip_core`, add its stub in `types/` in the same change.** That is
the only thing keeping the stubs from rotting.

## Debugging on a live server

| Command | Use |
| --- | --- |
| `whipsetup` (console) | Re-print the start-up report; every `FIX` line says what to change |
| `whipsupport` (console) | Version, setup report and recent errors in one block |
| `/w` | Dev tools: noclip, coords, object viewer, debug overlays, Dungeon Mode |
| `/whip` | Admin panel: players, items, vehicles, economy, world, Director, settings |
| `/whipcost` | What each resource is asked to run, by event handler and by inbound call |

`/w` and `/whip` need the `whip.devtools` ACE, and developer mode must be switched on in `/w`
(it resets every join). Anything a dev tool triggers on the server must still check `HasDevTools`
itself — the client being in dev mode proves nothing.

## References

- `references/exports.md` — export names by domain, both sides
- `references/events.md` — every event the core fires
- `references/extension-points.md` — what each `Whip.*` extension call takes
- `references/satellite-skeleton.md` — a complete minimal satellite to copy
