# ArkWarden

A bare-metal game server manager for Windows, starting with ARK: Survival Ascended, built for higher, steadier tick rates.

## What it is

ArkWarden runs game servers as plain Windows processes, no containers, no compatibility layer, and wraps each one in the controls a serious cluster needs. It looks at the CPU and memory in the machine and uses that to decide which CCD each server runs on and how much memory it gets. It keeps servers from stepping on each other, automates the setup that usually gets done by hand, and gives you one desktop app to run it all from. ARK is the first game module; the design is multi-game from the start.

## What it does today

**Hardware-aware placement**
- Maps the CPU: processor groups, NUMA nodes, cores, and every cache level, using the shared L3 to work out which cores sit on the same die.
- Pins each server to its own CCD inside a Windows Job Object, and spreads servers across dies so no two share a cache.
- Enforces a hard per-server memory ceiling in the kernel, so one leaky map can't take the box down with it.
- Logs each server's real peak memory and recommends a right-sized ceiling per map.
- Reads the DIMM layout and warns when memory channels are under-populated or one DIMM dips below its rated speed.

**Setup and safety**
- Each server runs on its own game install or shares one with another server. Add a server by reusing an existing install, sharing another server's, or downloading a fresh one through SteamCMD.
- Imports an existing server's settings instead of starting from scratch.
- Picks and claims ports, keeps a registry of who owns what, and flags conflicts before a server starts.
- Writes the firewall rules and notices when they've been changed or removed behind its back.
- Manages the ini settings it understands and passes everything else through untouched, with a backup and a verify step on every write.
- Keeps every server's saves in its own folder outside the game install, and refuses to launch any two servers that would write to the same save.
- One-click backups and a save history per server.
- Reinstalls a server's game files, and deletes a server with a plain count of what will be removed before anything is.

**Running the cluster**
- Graceful stop and restart over RCON: warn the players, save the world, exit cleanly. The loader and the server are tied together at the kernel, so when one goes down the other goes with it, and you never find a dead server still holding its ports in Task Manager.
- Five status lights per server: running state, install, ports, firewall and reachability, all read from live state.
- Installs and updates AsaApi, the add-on that lets ARK servers run plugins, detects the plugins you've got, and lets you turn them on or off.
- Switches seasonal event mods on and off by date.
- Relays chat between servers, so a cluster feels like one place.
- Watches each server's process and catches crashes.
- Serves a local API with every server's state, which the desktop app reads and a web UI could use later.

## Status

In progress since May 2026 and not finished. The engine is mostly there; the desktop app is being built. It's a personal project that may never be released. If it is, it will be free.

## Planned

- Mod-update awareness: read-only CurseForge metadata checks and a scheduled restart with warnings. Optional: a receipt of what each restart installed.
- Staggered backup schedules, so there's always more than one point to roll back to.
- Live tick-rate monitoring per server and per CCD, with health judged by the game's heartbeat, not just "the process is running."
- Auto-restart on crash, with a cooldown.
- A guided setup: dropdowns and sensible defaults instead of blank boxes you're expected to know the right values for.
- Hardware profiles for other CPUs. Today it's tuned around one EPYC layout; a public release would need profiles for other CCD and cache layouts.

## Tech

Go engine, builds to a single exe. Desktop app in Tauri + SvelteKit.
