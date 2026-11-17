# A minimal satellite, end to end

A complete resource that adds one item, one shop interaction, a settings page and a command.
Copy the shape, not the content. `whip_fuel` on the server is the real-world version of this.

## `fxmanifest.lua`

```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

name 'whip_lemonade'
description 'A lemonade stand: an item, a stand you buy from, and a price an admin can change.'
author 'you'
version '1.0.0'

dependency 'whip_core'

shared_script '@whip_core/lib/satellite.lua'   -- include it; never copy it
shared_script 'config.lua'
shared_script 'shared/settings.lua'   -- config.lua as a page in /whip -> Settings

server_scripts {
    'server/items.lua',   -- the item and its use handler
    'server/stand.lua',   -- buying: the money, the item, the log
}

client_scripts {
    'client/stand.lua',   -- the prompt on the stand
}
```

`lua54 'yes'` and `dependency 'whip_core'` are not optional. With the dependency, FiveM stops
your resource when the core stops and starts it again after — which is why you don't need
`Whip.onCoreLost` and why `Whip.register` is enough.

## `config.lua`

```lua
Config = Config or {}

Config.Stand = {
    coords = vec3(25.7, -1346.9, 29.5),
    radius = 1.5,
}

Config.Price = 4
```

## `shared/settings.lua`

```lua
Whip.configPage({ label = 'Lemonade', groups = {
    { label = 'Prices', settings = {
        { key = 'Price', type = 'int', min = 0, max = 500, unit = '$', label = 'A glass of lemonade' },
    } },
    { label = 'The stand', settings = {
        { key = 'Stand.radius', type = 'float', min = 0.5, max = 5.0, label = 'How close you must be' },
    } },
} })
```

Admins now change these in `/whip` → Settings → Lemonade, and `Config.Price` reads the saved
value from then on. Anything you *computed* from Config at load needs a hook:

```lua
Whip.onConfigChanged('Stand', function(value, old, key)
    rebuildTheStand()
end)
```

## `server/items.lua`

```lua
local function drinkLemonade(source, item)
    local ok = Whip.core:Progress(source, { label = 'Drinking', duration = 3000 })
    if not ok then return false end   -- cancelled, or 'whip_core stopped'

    Whip.core:AddStat(source, 'thirst', 25, 'item:lemonade')
    Whip.core:RemoveItem(source, 'lemonade', 1)
    return true
end

Whip.register(function()
    Whip.core:RegisterItem('lemonade', {
        name = 'Lemonade',
        weight = 0.3,
        description = 'Cold, sweet, and faintly of lemons.',
    })
    Whip.core:RegisterUseHandler('lemonade', drinkLemonade)
end)
```

Everything the core must remember goes inside `Whip.register` — it re-runs on every core restart.
Plain Lua helpers like `drinkLemonade` stay at file scope.

## `server/stand.lua`

```lua
local function buy(source)
    if not Whip.core:IsPlayerLoaded(source) then return end

    local where = Whip.core:GetPlayerCoords(source)
    if #(where - Config.Stand.coords) > Config.Stand.radius + 2.0 then
        Whip.core:NotifyPlayer(source, 'You are not at the stand.', 'error')
        return
    end

    if not Whip.core:CanFitItem(source, 'lemonade', 1) then
        Whip.core:NotifyPlayer(source, 'Your hands are full.', 'error')
        return
    end

    local receipt, why = Whip.core:ChargePlayer(source, Config.Price, { reason = 'lemonade' })
    if not receipt then
        Whip.core:NotifyPlayer(source, why or 'You cannot afford that.', 'error')
        return
    end

    local to, err = Whip.core:AddItem(source, 'lemonade', 1)
    if not to then
        Whip.core:RefundPlayer(receipt)   -- the receipt, not the amount
        Whip.core:NotifyPlayer(source, err or 'Something went wrong.', 'error')
        return
    end

    Whip.core:LogActivity(source, 'lemonade_bought', {
        detail = ('bought a lemonade for $%d'):format(Config.Price),
    })
end

Whip.register(function()
    Whip.core:SecureEvent('lemonade:buy', { rateLimit = 1000 }, buy)
end)

Whip.command('lemonadeprice', { dev = true }, function(source, args)
    print(('a lemonade is $%d'):format(Config.Price))
end)

Whip.sayStarted()
```

Note what the server does and the client never does: **check the distance, check the space, take
the money, give the item, and undo the charge if the give fails.** The client's only job is to
ask.

`Whip.command(name, opts, fn)` puts a command behind the same gate as a `SecureEvent` — `opts`
takes `dev`, `ace`, `whileDead`, `rateLimit`, `console`, a refusal is shown to the player as a
toast, and `fn` runs in its own thread so it may wait.

`Whip.sayStarted()` prints `whip_lemonade started successfully!` once your registrations have
run — and stays quiet if anything went wrong since the resource started. Report a problem you
find yourself with `Whip.startProblem('the stand has no coords')`.

## `client/stand.lua`

```lua
Whip.register(function()
    Whip.core:AddAction({
        id = 'lemonade.buy',
        at = { coords = Config.Stand.coords, heading = 90.0 },
        label = 'Buy a lemonade',
        icon = 'store',
        onSelect = function(ctx)
            TriggerServerEvent('lemonade:buy')
        end,
    })
end)
```

`AddAction` takes **one table** (`id`, `at`, `label`, `icon`, `require`, `onSelect`, ...) and
answers `id, why, warnings`. A field it does not know, a wrong kind or an out-of-range number is
*warned* about in F8 and in that third return — not refused — so read the warnings while
developing. The full shape is `WhipAction` in `whip_core/types/client.lua`.

## `.luarc.json`

Copy `whip_core/types/luarc.template.json` to your resource root as `.luarc.json` and fix
`workspace.library` if your folder layout differs from `../whip_core/types` or
`../[whip]/whip_core/types`. Then `Whip.core:` autocompletes with real signatures.

## Checklist before you call it done

- [ ] Every registration is inside `Whip.register`.
- [ ] Every core call goes through `Whip.core:` / `Whip.call`, not `exports.whip_core:`.
- [ ] Every return value from a core call is checked — `false, why` is a normal answer.
- [ ] Nothing the server decides is decided on the client.
- [ ] Anything a player's game can trigger is a `SecureEvent` / `SecureCallback` / `Whip.command`.
- [ ] Config is on a `Whip.configPage`, with `Whip.onConfigChanged` for anything derived.
- [ ] New manifest entries sit where their load-order dependencies demand, with a comment saying why.
- [ ] A money move that cannot be completed is refunded.
- [ ] `whipsetup` is clean and the console says the resource started successfully.
