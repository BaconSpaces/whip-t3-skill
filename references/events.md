# Events whip_core fires

Plain `AddEventHandler` local events — no network trip, no `RegisterNetEvent`. Payload types are
in `whip_core/types/events.lua`; annotate the handler to get them:

```lua
AddEventHandler('whip_core:item:added', function(e) ---@param e WhipEvtItemChanged
    -- e.source, e.item, e.amount, ...
end)
```

**The naming rule, which your own events should follow too:** `whip_core:<domain>:<pastTense>`,
with exactly **one table payload**. Names containing `client:` or `server:` are whip's own network
messages — not for you.

`S` = fired on the server. `C` = fired on the client.

## Readiness

- `S+C whip_core:onReady` `{ reason, resources }` — ready for registrations. `reason` is
  `'coreStarted'` (the core restarted and remembers nothing) or `'resourceStarted'` (`resources`
  names which just started). **Prefer `Whip.register`**, which filters this for you.

## Players & characters

- `S whip_core:player:loaded` — character fully loaded. `C` too: the character is in the world
  and the picture is back.
- `S whip_core:player:logout` — the character is leaving. `C`: the local player is going back to
  character select.
- `S whip_core:character:created` / `S whip_core:character:deleted`
- `S whip_core:character:renamed` — an admin renamed a character.
  `C whip_core:player:renamed` — the local player's, with `PlayerData` already updated.
- `C whip_core:player:spawned` — put into the world (every load, and each respawn).
- `S/C whip_core:player:died`, `S/C whip_core:player:revived`
- `C whip_core:death:screenChanged` — for a resource drawing its own death screen.
- `C whip_core:appearance:modelChanged` — the ped was rebuilt with a new model.
- `S whip_core:flag:changed` `{ source, charId, name, value, was }`

## Inventory & items

- `S whip_core:inventory:changed` `{ source }` — anything in a player's pockets changed, any way.
  The catch-all; prefer a narrower event when one fits.
- `S whip_core:item:added` / `S whip_core:item:removed` — the player carries more / less of an item.
- `S whip_core:item:used`, `S whip_core:item:given` (one player to another)
- `C whip_core:hotbar:slotUsed` — a number key or the pad's hotkeys wheel.
- `C whip_core:hands:blocked` / `C whip_core:hands:unblocked` `{}` — put away what you hold that a
  block covers (every block covers weapons); `unblocked` fires when no block is left.
- `S whip_core:heldProps:lost` `{ source, tag, reason, by }`
- `S whip_core:container:opened` `{ containerId, source, charId }`

## Money & banking

- `S whip_core:bank:accountChanged` — an account changed with no money moving.
- `S whip_core:bank:balanceChanged` — any deposit, withdrawal or transfer.

## Shops

- `S whip_core:shop:opened` / `:closed` / `:purchased` / `:created` / `:updated` / `:deleted` /
  `:priceChanged`

## Jobs

- `S whip_core:job:created` / `:updated` / `:deleted` / `:hired` / `:fired` / `:roleChanged` /
  `:clockedOn` / `:clockedOff` — each carries `jobId` and `key`; the ones about a person carry
  `charId` and `source` (`source` is nil while they're offline).
- `C whip_core:job:clockedOn` / `:clockedOff` — the local player went on or off duty.
- `C whip_core:job:listChanged` — hired, fired, promoted or renamed.

## Vehicles

- `S whip_core:vehicle:entered` / `:exited` `{ vehicleId, plate, source, seat }`
  (`previousSeat` on a seat change); also `:plateChanged` `{ vehicleId, from, to }`. `C` fires
  the same pair for the local player.
- `C whip_core:vehicle:copied` `{ entity, of }` — this game made its own copy of a vehicle.

## Properties

- `S whip_core:property:ownerChanged` — given to someone, or made unowned.
- `S whip_core:property:keyGranted` / `:keyRevoked`
- `C whip_core:property:entered` / `:exited` — the local player walked in or out.
- `C whip_core:property:furnished` / `:unfurnished` / `:furnishStateChanged`

## World

- `S whip_core:time:changed`, `S whip_core:time:freezeChanged` (frozen **or** unfrozen),
  `S whip_core:weather:changed`
- `C whip_core:weather:layerChanged` — a local time/weather layer set or removed (`layer` nil).
- `C whip_core:zone:entered` / `:exited` — the local player. `S` fires the same pair for zones
  with `trackPlayers` on.
- `S whip_core:door:lockChanged` `{ doorId, locked, source, cause }` — only when it really
  changed. `S whip_core:door:denied` `{ doorId, source, reason }`
- `S whip_core:npc:spawned` `{ npcId, tag, owner, netId, model, coords }`,
  `:despawned` `{ ..., reason }` (`'removed'|'stopped'|'released'|'lost'|'corpse'`),
  `:damaged` `{ ..., amount, health, armour, attacker, group }`,
  `:killed` `{ ..., killer, weapon, coords }` — `attacker`/`killer` is a server id or nil.

## UI & input

- `C whip_core:ui:stateChanged` — a screen or dialog opened, or everything closed.
- `C whip_core:ui:cleanScreen` — the screen went clean for a recording.
- `C whip_core:pad:deviceChanged` — keyboard ⇄ pad, once the answer has settled.
- `C whip_core:keyboard:opened` / `:closed` — whip's on-screen keyboard (`OpenKeyboard`).
- `C whip_core:nui:capsDetected` — what the game's browser can do.
- `C whip_core:voice:rangeChanged` (the player pressed Z), `C whip_core:voice:systemChanged`
- `C whip_core:ask:cancelled` `{ token }` — the core gave up on a question it asked this player.

## Permissions

- `S whip_core:permission:changed` `{ userId, permission, granted }`
- `C whip_core:permission:changed` `{ permission, granted }` — on every game of that account.
  (`whip_core:client:permissionChanged` is the network message that carries it; not for you.)

## The Director

- `S whip_core:run:started` `{ run, startedBy = { mode, by } }`
- `S whip_core:run:ended` `{ run, outcome, reason, vars, payout, faultCard, stopping }`
- `S whip_core:run:playerJoined` `{ run, source, charId, by }`
- `S whip_core:run:playerLeft` `{ run, source, charId, why, by, runEnds }`
- `S whip_core:run:signalled` `{ run, name, data = { value }, source }` — a Send Signal card.
- `S whip_core:run:thingMade` `{ run, world, kind, label, netId, npcId, vehicleId, containerId,
  groundId, doorId, ... }`
- `S whip_core:scenario:switchedOn` `{ scenario, name, rev, replaced, by }` / `:switchedOff`
  `{ scenario, name, rev, why, by }`
- `S whip_core:lore:changed` — a lore character was made, changed or retired.
- `C whip_core:run:joined` — this player's own runs (see `GetPlayerScenarioRuns`).
- `C whip_core:director:builderCopy` `{ id, kind, model, entity, ... }` — the world builder.

## Dev & profiling

- `C whip_core:devtools:modeChanged` `{ on }` — this player's developer mode switched.
- `C whip_core:cost:started` `{ seconds }` / `:stopped` `{}` — `/whipcost` timing.
- `C whip_core:cost:reported` `{ resource, ms, runs, waited, kb, kbRuns, memory }`
