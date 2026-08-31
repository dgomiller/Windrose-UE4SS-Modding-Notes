# Windrose UE4SS Modding Notes

Durable, engine-level facts about modding **Windrose** (Kraken Express, UE 5.6) with **UE4SS** — both Lua scripting and compiled C++ mods. Not a tutorial and not a wrapper library — just the hard-won, confirmed-live findings that would otherwise get rediscovered by every modder independently: what crashes, what silently no-ops, what actually works, and the exact recipe for each.

Everything here was learned building real, shipping mods (**[Living Base Enhanced](https://github.com/dgomiller/Living-Base-Enhanced-Windrose)** and its C++ companion, **[LivingBaseSpawnMenu](https://github.com/dgomiller/Living-Base-Spawn-Menu-Windrose)**), then generalized and stripped of anything specific to those mods' own code. A few entries use a specific function name as a worked example of applying a technique — the technique is what's durable, not the name.

## What's in it

`Windrose_Modding_Notes.txt` — one plain-text file, organized into numbered sections:

1. Spawning an actor that actually works
2. The composite (appearance) system — what sticks and what doesn't
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
17. A third-party companion mod (Windrose Mod Settings) can probably render a real slider and dropdown widget — a single unconfirmed exploratory test, not a proven recipe
18. Line-trace-based targeting: object-type queries aren't a strict superset of channel-based ones, and a "does this component exist" check needs a validity check, not just a nil check
19. Constructing a composite outfit from scratch: the real 3-level asset structure, what's safe to build via Lua, and what crashes

Every entry is a specific, confirmed-live finding — not a guess, not "should work in theory." Where something was tried and failed, that's recorded too (a documented dead end saves someone else the same hours).

`pakcontents.xlsx` — an export of every asset name inside the game's `.pak`/`.utoc` files, one sheet per file. Useful for finding a class/asset path to spawn or reference without digging through the raw archives yourself.

## Using it

It's one file, so pick whatever fits your workflow:

- **Just read it** — browse `Windrose_Modding_Notes.txt` directly here on GitHub.
- **Pull it into your own project as a submodule**, so it updates when this repo does:
  ```
  git submodule add https://github.com/dgomiller/Windrose-UE4SS-Modding-Notes.git external/windrose-notes
  ```
- **Clone or `curl` the raw file** if you just want a local copy:
  ```
  curl -O https://raw.githubusercontent.com/dgomiller/Windrose-UE4SS-Modding-Notes/main/Windrose_Modding_Notes.txt
  ```

## Staying up to date

This isn't a one-time snapshot — it gets updated whenever new findings come out of active mod development. Pull/re-fetch periodically (or watch the repo) if you want the latest.

## License

Public domain (CC0) — see [`LICENSE`](LICENSE). Use it, copy it, fork it, mirror it, build on it, no attribution required. It's meant to be freely useful to the whole Windrose modding community, not just this project.
