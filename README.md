# whip-skill

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) for developing
resources on **WHIP** — the FiveM framework built around [`whip_core`](https://github.com/WHIPFX).
Credit:  [`fozenistaken`](https://github.com/fozenistaken/whip-skill)

It teaches the agent the satellite contract (`@whip_core/lib/satellite.lua`, `Whip.register`,
`Whip.core`), the hard rules (server owns state, `SecureEvent` over `RegisterNetEvent`, manifest
load order), the framework's extension points (Director cards and tasks, skill check games, add-on
weapons, settings pages), and where to look up the real export and event signatures instead of
inventing them.

## Install

**One server / project:**

```
<your server>/.claude/skills/whip/
```

**Every project on your machine:**

```
~/.claude/skills/whip/
```

Clone or copy this repo's contents (`SKILL.md` and `references/`) into that `whip/` folder:

```bash
git clone https://github.com/fozenistaken/whip-skill ~/.claude/skills/whip
```

Claude picks it up automatically from `SKILL.md`'s frontmatter — no registration step. Check it
loaded with `/skills`, or invoke it directly with `/whip`.

## Layout

| File | What's in it |
| --- | --- |
| `SKILL.md` | The contract, the rules, and where to read the real API |
| `references/exports.md` | All 589 core exports, grouped by domain, for grepping |
| `references/events.md` | Every event `whip_core` fires, with payload shapes |
| `references/extension-points.md` | What each `Whip.*` extension call takes |
| `references/satellite-skeleton.md` | A complete minimal satellite to copy |

## Keeping it honest

The skill deliberately treats the `whip_core` on your own server as the source of truth —
`types/server.lua`, `types/client.lua`, `types/events.lua`, `lib/satellite.lua` and `docs.html` —
and tells the agent to grep those before writing code. The references here are a map, not a
replacement: when `whip_core` adds exports, the type stubs are updated in the same change, so a
stale list in this repo degrades into "grep for it" rather than into wrong code.

Written against **whip_core 1.2.0** (383 server exports, 206 client).
