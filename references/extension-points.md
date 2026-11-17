# Extension points: what each `Whip.*` call takes

Every one of these is fully documented in its own banner block in
`whip_core/lib/satellite.lua` — read that block before using it; the options below are the
headline ones, not the whole set.

Two rules apply to all of them:

- **Namespaced ids.** `'<your namespace>.<name>'`. The first running resource to use a namespace
  owns it, for its cards and its events; it's released when that resource stops.
- **Your functions stay in your resource.** whip_core receives your definition as *plain data*
  plus one function to call back into you. It runs your callbacks inside a `pcall` on your side,
  so a mistake comes back in your own words. Anything you hand over must be plain data — no
  functions, no vectors, 8 levels deep at most.

Declare them at file scope or inside `Whip.register`; they're re-sent whenever whip_core restarts,
and a refusal is printed in your console with the reason.

## `Whip.configPage(def)` — your `config.lua` in `/whip` → Settings

In a shared script **after** `config.lua`, at file scope:

```lua
Whip.configPage({ label = 'Autos', groups = {
    { label = 'Prices', settings = {
        { key = 'Tuning.EngineSwapPrice', type = 'int', min = 0, max = 1000000,
          unit = '$', label = 'Engine swap' },
    } },
} })

Whip.onConfigChanged('Shop', function(value, old, key) ... end)
```

Each `key` is a path in your `Config`; its default is whatever `config.lua` says. What an admin
saves is written back into `Config` **in place** when the page is declared and on every change, so
code holding a reference to `Config.Shop` sees new values and your code keeps reading `Config.X`
as before. Use `Whip.onConfigChanged(path, fn)` for anything you *derived* from Config at load (a
price table, a thread that only runs when an option is on) — it hears its own path, anything
under it, and changes to a table it sits inside. A `'record'` entry is one card for several values
of one table (`key = 'Turbo'`, `fields = { { key = 'TorqueGain', ... } }`, each stored as
`'Turbo.TorqueGain'`).

## `Whip.playerSetting(def)` — a row on the pause menu's Server Settings page (client)

A choice each **player** makes for themselves; your resource still stores the value (its own KVP).

```lua
Whip.playerSetting({
    key = 'chat.visibility', group = 'Chat & voice', label = 'Chat messages',
    help = 'New messages fade away, stay on screen, or chat is hidden.',
    type = 'choice', options = { { value = 0, label = 'Fade away' }, { value = 1, label = 'Stay' } },
    default = 0,
    get = function() return hideState end,
    set = function(value) hideState = value; SetResourceKvp(...) end,
})
```

`type`: `'toggle'`, `'choice'` (`options`), `'slider'` (`min`, `max`, `step`, `unit`).
`group`: one of `Controller`, `Vehicles`, `Map & HUD`, `Chat & voice`, or a new tab of yours.

## `Whip.card(type, def)` — a Director card (server)

```lua
Whip.card('police.notify', {
    title = 'Notify Police', desc = 'Sends a dispatch call to on-duty officers.',
    outputs = { 'sent', 'nobody' },
    labels = { sent = 'Sent', nobody = 'Nobody on duty' },
    bad = { 'nobody' },
    props = { { key = 'message', kind = 'text', label = 'Message',
                default = '10-90 alarm', maxLength = 120 } },
    testSays = 'Would send "{message}" to the police on duty',
    run = function(card)
        -- may wait: Whip.core, the database
        return 'sent'   -- or return nothing and Whip.answerCard(card.callId, 'sent') later
    end,
})
```

Also: `category` / `categoryTitle` / `color` (`blue teal green amber red violet slate`) / `icon` /
`help`; `timeout` (seconds before Took Too Long, 30); `instant = true`;
`event = { name, match = 'player'|'run'|'any', once?, filter? }` (with no `run`, the card goes on
when you send that event); `check = function(props, doc)` returning a message; a field's
`showIf = '<a bool field key>'`; a field of kind `'entries'` filled by one of your Director
buttons; `onEnd = function(callId, why)`; `test = 'run'`; `v = 2` (bump when fields change);
`renamed = { ... }`; `once = false` / `onceOut = 'sent'`.

The `card` your `run` gets is a copy (`callId`, `runId`, `cardId`, `scenario`, `name`, `rev`,
`test`, `tag`, `player`, `starter`, `players`, `trigger`, `props`) with methods that call back into
whip_core and are refused once the card has finished: `card:var(name)` / `card:setVar(name, v)`,
`card:state()` / `card:setState(t)`, `card:place(id)` and more — see the banner block.

**A fail condition** is a card with no `outputs` and no `run`, just `fail`:

```lua
Whip.card('police.failArrested', {
    title = 'Fail If a Player Is Arrested', desc = '...',
    fail = {
        event = 'police.arrested',   -- yours: SendDirectorEvent('police.arrested', { source = src })
        match = 'player',            -- 'player' (default) | 'run' | 'any'
        ends  = 'them',              -- 'everyone' (run fails) or 'them' ('player' match only)
        reason = 'You were arrested',
        filter = function(card, event) return event.data.felony and 'Arrested for a felony' end,
    },
})
```

Related: `Whip.removeCard`, `Whip.scenario`, `Whip.template`, `Whip.answerCard`.

## `Whip.task(type, def)` — a Director task for NPCs and cars (both sides)

ONE call, in a file both sides load:

```lua
Whip.task('autos.flash', {
    label = 'Flash the Lights', group = 'car', kinds = { 'vehicle' },
    timing = 'timed', sec = 4,
    fields = { { key = 'times', kind = 'number', label = 'Times', min = 1, max = 20, default = 5 } },
    run = function(car, args, info)   -- on the game that owns the car, in a thread
        for _ = 1, args.times do
            SetVehicleLights(car, 2) Wait(200) SetVehicleLights(car, 1) Wait(200)
        end
        SetVehicleLights(car, 0)
    end,
})
```

`group`: one of the Director's (`move look fight act car body wait`) or your own with
`groupTitle`. `kinds`: `npc`, `vehicle`. `timing`: `'instant'`, `'timed'` (`sec`, default 3),
`'until'` (give up after `giveUp`, default 30), `'either'`. Also `lasting = true`, `once = true`,
`standing = false`. At most 8 `fields`, each `{ key, kind, label, hint?, default?, required? }`
with `kind` one of `number` (`min`, `max`, `slider`, `step`, `unit`), `bool`, `select`
(`options`, `look = 'seg'`), `text` (`max`), `world` (`worldKind?`), `point` (key `'at'`),
`target` (key `'target'`).

`run(entity, args, info)`: a `world` field arrives as that thing's entity on this game (nil when
it isn't here), a `target` as `{ kind, entity, coords }`, a `point` as
`{ x, y, z, heading }`. `info` is `{ type, kind, timing, ms, preview }`. An `'until'` step ends
when `run` returns, unless you supply `done(entity, args)` (polled four times a second; `true` =
done, `'failed'` = failed). **A new owner replays a timed or lasting step from its start**, so
keep `run` to what the entity does.

## `Whip.skillCheckGame(id, def)` — a minigame for every skill check (both sides)

Played wherever whip runs a check: `SkillCheck(source, 'hard', { game = 'phone.reroute' })` and
the Director's Skill Check card.

```lua
Whip.skillCheckGame('phone.reroute', {
    label = 'Reroute (phone)', desc = 'Turn the tiles until power reaches the exit.',
    levels = {   -- server: minMs = faster than this cannot be honest; timeoutMs = the server gives up
        easy   = { minMs = 2500, timeoutMs = 47000 },
        medium = { minMs = 2500, timeoutMs = 57000 },
        hard   = { minMs = 2500, timeoutMs = 67000 },
    },
    canStart = function(source)   -- server, optional
        if not hasPhone(source) then return false, 'You need a phone!' end
        return true
    end,
    run = function(difficulty, token)   -- client: play it, in a thread
        return passed, why   -- why: 'failed' | 'cancelled' | 'timeout' | 'busy'
    end,
    cancel = function(token) end,   -- client, optional: make run return
})
```

Someone `canStart` refuses is shown the words, nothing starts, and the check answers
`false, 'not started'`.

## `Whip.weapons(list, opts)` — add-on weapons (both sides)

Weapons your resource streams (its own `WEAPON_*` in its `weapons.meta`), offered beside GTA's
wherever whip lists weapons: the Director's NPC weapons and Hold a Weapon step, Fixed Weapons,
Dungeon Mode, the Object Viewer's Give a weapon.

```lua
Whip.weapons({
    { name = 'WEAPON_MY_PUMP', label = 'Pump Shotgun', group = 'shotgun', model = 'w_sg_my_pump' },
    { name = 'WEAPON_MY_KNIFE', label = 'Knife', group = 'melee' },
}, { tag = 'mine' })
```

`group`: `pistol smg rifle machine gun shotgun sniper heavy thrown stun gun melee`. `model` is
the world model (firearms only; needed to be usable as a Fixed Weapon). Calling again replaces
the list. An NPC given one carries the gun itself, never your item — your ammo and reload code is
not on it.

## `Whip.directorButton(id, def)` — a button on a placed thing's page (client)

```lua
Whip.directorButton('autos.customise', {
    label = 'Customise', icon = 'turbo', kinds = { 'vehicle' },
    where = 'both',        -- 'page' | 'bubble' (world builder) | 'both'
    takesOver = true,      -- the Director steps aside until thing:back()
    press = function(thing)
        thing:back({ paint = 27 })
    end,
})
```

`thing`: `{ id, kind, label, model, coords, scenario, data, entity, builder }`. `thing:save(t)`
keeps plain data on the thing (4 KB of JSON at most), saved and undone like any other scenario
edit, and handed to your server side on a run. `kinds = { 'card' }` makes it a button for a
card's `entries` field instead.

## `Whip.objectViewerOption(id, def)` — an option in the `/w` object viewer (client)

```lua
Whip.objectViewerOption('autos.whiplash', {
    label = 'Open Whiplash', group = 'Whiplash', tint = 'violet',
    hint = 'Takes the driver\'s seat and opens the shop on this car.',
    kinds = { 'vehicle' },        -- vehicle | player | npc | object
    whip = true,                  -- whip's own things only; false = the game's only; omit = both
    select = function(target) ... end,   -- in its own thread
})
```

`target`: `{ kind, entity, netId, whip, vehicleId, serverId, model, name, coords }`. Shown to
anyone with dev tools on **and to a Dungeon Master in Dungeon Mode** — so anything `select` does
on the server must check `HasDevTools` or its own permission itself.

## `Whip.padScreen(...)` — controller input for your NUI screen (client)

The browser can't read a gamepad inside FiveM. This reads the pad in Lua while your screen is open
and posts `{ action = 'pad', cmd = 'up', down = true, rep = 2 }` to your page per press. **LB is
whip's combo key:** a press while LB is held that whip_core claims as its combo never reaches you.

## `Whip.voiceSystem()` / `Whip.voice(name, ...)`

`voiceSystem()` returns the voice resource running now, or `false`. `voice(name, ...)` calls one
of its exports and answers its answer, or `false` and why.

## `Whip.registry(name, opts)`

A registry of your own that follows whip's rule: other resources register into it through
`Whip.registerWith(yourResource, fn)`, and their entries come back when you restart.
