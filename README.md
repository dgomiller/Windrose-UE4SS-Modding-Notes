# Windrose UE4SS Modding Notes

Durable, engine-level facts about modding **Windrose** (Kraken Express, UE 5.6) — two related but distinct workflows, one file each: **UE4SS** (Lua scripting and compiled C++ mods, running against the live game) and a **real Unreal Editor / SDK-stub project** (offline content authoring against generated header stubs, no live game process involved). Not a tutorial and not a wrapper library — just the hard-won, confirmed-live findings that would otherwise get rediscovered by every modder independently: what crashes, what silently no-ops, what actually works, and the exact recipe for each.

Everything here was learned building real, shipping mods (**[Living Base Enhanced](https://github.com/dgomiller/Living-Base-Enhanced-Windrose)** and its C++ companion, **[LivingBaseSpawnMenu](https://github.com/dgomiller/Living-Base-Spawn-Menu-Windrose)**), then generalized and stripped of anything specific to those mods' own code. A few entries use a specific function name as a worked example of applying a technique — the technique is what's durable, not the name.

## What's in it

`Windrose_Modding_Notes.txt` — the UE4SS side (Lua + compiled C++ mods against the running game), organized into numbered sections:

1. Spawning an actor that actually works
2. The composite (appearance) system — what sticks and what doesn't, a working technique for retargeting a class's body archetype/mesh (plus how to tell whether a source class is actually stable), and the real mechanism for size control
3. THE CRASH TRAPS (each cost real debugging hours)
4. Persisting and restoring actor state across a world load
5. Peace / faction mechanics
5b. Movement — this game does not use the UE navmesh
6. Development workflow that works
7. Useful class paths
7b. Reacting to things the game itself spawns
8. Native spawn/scouting machinery (partially explored)
9. Cross-skeleton re-skinning — what actually determines the result
10. Finding which "world" (save) is currently loaded
11. Content-replacer paks (asset overrides) — what's possible from Lua and what isn't
12. Compiled C++ UE4SS mods — rendering an interactive overlay safely
13. Placing an actor relative to a moving ship
14. Playing a specific canned animation on a live Character
15. `ExecuteWithDelay`'s callback does not run on the game thread — and nesting it inside `ExecuteInGameThread` is a separate, differently-broken thing
16. Comparing two independently-obtained UE4SS component references with `==` is unreliable — compare `GetFName()` instead
17. Line-trace-based targeting: object-type queries aren't a strict superset of channel-based ones, and a "does this component exist" check needs a validity check, not just a nil check
18. Constructing a composite outfit from scratch: the real 3-level asset structure, what's safe to build via Lua, what crashes, and how to automate the whole offline authoring/cook pipeline from the command line
19. Integrating with the "R5 Mod Settings" companion mod — registering a settings page, reading values back, keybind vs. toggle live-update rules (and a real crash trap), and a more scalable auto-generated-manifest pattern
20. A "not yet started" note is a snapshot, not a durable fact
21. A live reflection probe that resolves cleanly but reads empty can mean the data doesn't exist to read, not that the property path is wrong
22. Before concluding a live write reverted, rule out that the read is looking at a different object
23. Resolving a Blueprint's own generated CLASS via the asset registry needs its exact indexed name — not the bare asset name
24. This game's AI runs a modern State Tree, not a Blackboard/Behavior Tree — its compiled-tree reference is populated at runtime, not baked; plus the native levers to force a tree onto a runtime-spawned controller, and why a hot-reload never re-deploys your files
25. An uncatchable native crash chased for a full session — it was a use-after-free from capturing a console command's output object in a delayed timer callback, not the mesh rebuild everyone blamed

Every entry is a specific, confirmed-live finding — not a guess, not "should work in theory." Where something was tried and failed, that's recorded too (a documented dead end saves someone else the same hours).

`Windrose_Unreal_SDK_Notes.txt` — the OTHER side: setting up a real, standalone Unreal Editor project against generated header stubs for the game's own reflected classes, and using it to author, cook, and package genuinely new content entirely offline. Covers project setup, the headless Python authoring + cook pipeline, the two real limitations of headless scripting (constructing a `GameplayTag` from a string; setting a soft-object reference to an external/unmounted asset) and their workarounds, packaging/installing, several general Unreal build-environment gotchas, two full worked examples (retargeting which body archetype/mesh a class resolves to, and the real mechanism for per-size material/skin control), authoring a genuinely NEW native class (not just a new data-only asset) by reimplementing a stub of the game's own base class under its real module/class name, and baking a fix directly into a Blueprint's own class defaults instead of at runtime — including a technique for retargeting a hard CLASS reference (not just a soft object value) via a cooked package's own import table. Cross-references `Windrose_Modding_Notes.txt` where the two overlap rather than duplicating content.

`pakcontents.xlsx` — an export of every asset name inside the game's `.pak`/`.utoc` files, one sheet per file. Useful for finding a class/asset path to spawn or reference without digging through the raw archives yourself.

## Using it

Plain text files, so pick whatever fits your workflow:

- **Just read them** — browse either file directly here on GitHub.
- **Pull the repo into your own project as a submodule**, so it updates when this repo does:
  ```
  git submodule add https://github.com/dgomiller/Windrose-UE4SS-Modding-Notes.git external/windrose-notes
  ```
- **Clone or `curl` the raw file(s)** if you just want a local copy:
  ```
  curl -O https://raw.githubusercontent.com/dgomiller/Windrose-UE4SS-Modding-Notes/main/Windrose_Modding_Notes.txt
  curl -O https://raw.githubusercontent.com/dgomiller/Windrose-UE4SS-Modding-Notes/main/Windrose_Unreal_SDK_Notes.txt
  ```

## Staying up to date

This isn't a one-time snapshot — it gets updated whenever new findings come out of active mod development. Pull/re-fetch periodically (or watch the repo) if you want the latest.

## License

Public domain (CC0) — see [`LICENSE`](LICENSE). Use it, copy it, fork it, mirror it, build on it, no attribution required. It's meant to be freely useful to the whole Windrose modding community, not just this project.
