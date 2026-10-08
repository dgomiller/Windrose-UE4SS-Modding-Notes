# Windrose UE4SS Modding Notes

Durable, engine-level facts about modding **Windrose** (Kraken Express, UE 5.6) — two related but distinct workflows, one file each: **UE4SS** (Lua scripting and compiled C++ mods, running against the live game) and a **real Unreal Editor / SDK-stub project** (offline content authoring against generated header stubs, no live game process involved). Not a tutorial and not a wrapper library — just the hard-won, confirmed-live findings that would otherwise get rediscovered by every modder independently: what crashes, what silently no-ops, what actually works, and the exact recipe for each.

Everything here was learned building real, shipping mods (**[Living Base Enhanced](https://github.com/dgomiller/Living-Base-Enhanced-Windrose)** and its C++ companion, **[LivingBaseSpawnMenu](https://github.com/dgomiller/Living-Base-Spawn-Menu-Windrose)**, plus the spin-off **[LBE: Summonable Ghost Fighters](https://github.com/dgomiller/LBE-Summonable-Ghost-Fighters)**), then generalized and stripped of anything specific to those mods' own code. A few entries use a specific function name as a worked example of applying a technique — the technique is what's durable, not the name.

## What's in it

`Windrose_Modding_Notes.txt` — the UE4SS side (Lua + compiled C++ mods against the running game), organized into numbered sections, grouped into eleven parts by topic:

**Part 1 — Working method and tooling**

1. Development workflow that works
2. Test on the scripting loader your players actually use — a locally built copy can silently end up installed
3. Reloading a script mod can leave its timers stale — relaunch instead (2026-10-01)
4. A Lua syntax check built on a Python Lua binding can never fail — loadfile returns a (nil, message) tuple (2026-10-02)
5. A large single-file scripting mod can hit the scripting language's own hard local-variable ceiling — pack new state onto an existing table instead of adding new top-level variables
6. A "not yet started" note is a snapshot, not a durable fact
7. A generated data file that looks like a full history may only hold recent state — diff against known-good data before regenerating/overwriting from it
8. When auditing captured data against a reference catalog, compare literal strings from the real source — never a reconstructed or templated name

**Part 2 — Reading live state correctly**

9. A live reflection probe that resolves cleanly but reads empty can mean the data doesn't exist to read, not that the property path is wrong
10. Before concluding a live write reverted, rule out that the read is looking at a different object
11. A class-hierarchy check is a cheap, decisive way to rule out "this class structurally can't do that" before spending more time tuning a property that silently does nothing
12. When an engine's own "give me the current live value" readback proves unreliable, tracking the same value yourself from a known start point plus elapsed time is a robust, low-effort substitute
13. A single shared condition can silently gate two logically separate behaviors — exempting a new caller from one can disable the other without anyone noticing until live testing catches it

**Part 3 — The UE4SS Lua binding: calls, comparisons and quirks**

14. Comparing two independently-obtained UE4SS component references with `==` is unreliable — compare `GetFName()` instead
15. A native function call through a script-reflection binding can require stricter argument handling than its own C++ signature promises
16. A documented, real engine function can be completely non-functional through a script-reflection binding, with no error and no partial effect — confirmed only by a 100% failure rate across repeated live attempts
17. A script-reflection method can be unbound on one actor class while others on the same reference work

**Part 4 — Threads, timers and the script runtime**

18. `ExecuteWithDelay`'s callback does not run on the game thread — and nesting it inside `ExecuteInGameThread` is a separate, differently-broken thing
19. Delayed script callbacks run on a separate thread that shares the interpreter with the game thread — the likely source of random scripting-state corruption
20. Quiet mode for a script mod: defer every timer callback while heavy game-thread work runs — and why moving timers onto the game thread breaks rendering (2026-10-01)
21. A render-state property written from a non-game-thread callback can be stored but never shown
22. Loading a class from disk in the middle of a staggered restore lets the engine run the queued callbacks INSIDE the running one
23. A script-side array walked repeatedly in a tight loop destabilises the scripting bridge — read it once and cache the answer
24. A self-rescheduling timer chain with an unguarded error dies silently — and leaves its temporary state switched on forever

**Part 5 — Crashes: traps and triage**

25. THE CRASH TRAPS (each cost real debugging hours)
26. An uncatchable native crash chased for a full session — it was a use-after-free from capturing a console command's output object in a delayed timer callback, not the mesh rebuild everyone blamed
27. A third-party cosmetic replacer mod silently broke a live customization tool AND caused an intermittent native crash — check installed content mods before chasing an engine theory
28. A struct field that reads as a small numeric "progress" range can be unsafe to overwrite live in EITHER direction — treat it as read-only unless a documented safe setter exists
29. Updating render components in place instead of rebuilding them on every edit is cheap hygiene — but check a crash address against history before blaming it
30. A loader you built yourself lets you turn crash addresses into function names
31. A crash dialog freezes only the thread that crashed — that tells you which thread (2026-10-01)
32. A crash with no dump file still leaves a stack in the game's own log, and offsets can be computed from module bases in any dump from the same boot (2026-10-02)

**Part 6 — Assets, classes and pak files**

33. Useful class paths
34. Resolving a Blueprint's own generated CLASS via the asset registry needs its exact indexed name — not the bare asset name
35. A Blueprint class can refuse a plain asset load while the game's AssetRegistry returns it — and a menu-time prewarm does not survive the world load (2026-10-02)
36. Content-replacer paks (asset overrides) — what's possible from Lua and what isn't
37. A binary-asset editing library needs the game's own type-mappings file loaded explicitly, or a native asset silently degrades to an unreadable raw blob
38. Patching a single package inside an already-shipped Zen/IoStore container, without its original staging tree or a full re-cook
39. Constructing a genuinely new tagged property from scratch in a cooked asset needs an extra type-descriptor field the editing library won't infer for you

**Part 7 — Spawning, the world and persistence**

40. Spawning an actor that actually works
41. A deferred-spawn transaction has a real window for assigning properties before an actor's own construction-time subsystems initialize — use it instead of patching a live instance afterward
42. Reacting to things the game itself spawns
43. Native spawn/scouting machinery (partially explored)
44. Placing an actor relative to a moving ship
45. Line-trace-based targeting: object-type queries aren't a strict superset of channel-based ones, and a "does this component exist" check needs a validity check, not just a nil check
46. An actor's "can be damaged" flag only gates the generic damage pipeline — a component can carry its own independent hit-point value that a custom gameplay system decrements directly
47. Raw static meshes as spawnable, text-bearing decor: rotate the mesh component, key the type on class plus mesh, and read bounds with the out-table form
48. A raw mesh spawned inside a loot/pickup wrapper is passthrough until its collision is forced to Block (2026-10-01)
49. Finding which "world" (save) is currently loaded
50. Persisting and restoring actor state across a world load
51. Saved per-object data keyed by position loses its owner when the object moves — re-key it (2026-10-01)
52. Sequential restore chains: make the completion callback idempotent — duplicate timer chains share the cursor and each one fires it (2026-10-01)

**Part 8 — Characters: appearance and animation**

53. The composite (appearance) system — what sticks and what doesn't, a working technique for retargeting a class's body archetype/mesh (plus how to tell whether a source class is actually stable), and the real mechanism for size control
54. Cross-skeleton re-skinning — what actually determines the result
55. Constructing a composite outfit from scratch: the real 3-level asset structure, what's safe to build via Lua, what crashes, and how to automate the whole offline authoring/cook pipeline from the command line
56. Runtime body-shape morphing on AI-controlled NPCs is not reachable from outside the engine — the deformation is baked once at construction by a compiled rig graph that never re-evaluates
57. Playing a specific canned animation on a live Character

**Part 9 — AI, movement and factions**

58. Peace / faction mechanics
59. A spawned/summoned actor's native AI targeting system can be a completely separate layer from its faction/damage-relationship data — fixing "it damages the wrong thing" does not fix "it targets the wrong thing"
60. Movement — this game does not use the UE navmesh
61. This game's AI runs a modern State Tree, not a Blackboard/Behavior Tree — its compiled-tree reference is populated at runtime, not baked; plus the native levers to force a tree onto a runtime-spawned controller, and why a hot-reload never re-deploys your files
62. A live probe showing a real AI/movement component stack on an actor class is not proof a specific behavior is active — confirm by observing native, unmodified behavior directly
63. A standard "stop AI logic" call doesn't reliably freeze every actor's movement — zeroing the movement component's own speed directly is a robust, mechanism-agnostic fallback
64. Forcing a foreign controller/animation-instance pair onto an actor can fix locomotion while leaving its real combat animation completely unreachable — the attack sequence can be baked directly into that specific character family's own animation graph, not driven through a generic system
65. Out-of-combat AI characters are capped by their walk gait, not by the max-walk-speed property — measure before tuning, and check a property exists before trusting a write (2026-10-02)
66. Where NPC stats live: per-class ability-system params and default-attribute effects, not a level number (2026-10-02, from the file inventory only — not tried live)

**Part 10 — Rendering: text, fonts, lighting, camera and time**

67. A text-render component only draws fonts that were baked offline — runtime/UI fonts load fine and produce zero glyphs
68. Baking a custom offline font headlessly with the editor's Python, and the four ways the conversion clips or smears glyphs
69. Font licences decide whether a font can go in a distributed mod — "free" and "non-commercial" are not the same as "redistributable"
70. Colour written to a text-render component is treated as linear, not sRGB
71. Making one object's light independent of the world's lighting: lighting channels plus zeroing the indirect bounce
72. A spring-arm/boom camera's two offset properties live in different reference frames — mixing them up produces an "orbiting" camera that looks centered only while facing one direction
73. A collision volume set to "query only" doesn't physically obstruct movement, but it still blocks a third-person camera's own collision-avoidance trace
74. A day/night cycle's "current time" can be computed from real elapsed time rather than accumulated per-tick — disabling the component's tick then only pauses the VISIBLE application of that value, not its underlying progression

**Part 11 — UI, menus and companion mods**

75. Compiled C++ UE4SS mods — rendering an interactive overlay safely
76. A UI helper function called more than once per frame with a literal widget identifier will eventually collide with itself
77. An immediate-mode GUI library's "pin this widget to the trailing/leading end of a bar" flag does not right/left-align it into unused space — it only prevents overflow
78. Scaling an entire Dear ImGui window (v1.92) without touching any hard-coded pixel sizes
79. In Dear ImGui a row's height is its tallest item
80. A tree UI built by splitting a delimited display string has no escaping — a literal delimiter character inside what's meant to be one leaf label silently creates extra nesting
81. Held-down UI buttons feeding a script mod: append to a queue file and drain it in batches, never a single-slot file (2026-10-01)
82. A persistent status file one side writes and another polls needs an explicit resync on load, not just a write-on-change
83. A compiled/hardcoded UI list generated once from a spreadsheet needs its own regeneration and rebuild step — editing the interpreted-language source alone does nothing for it
84. Extending an existing array anywhere but its true trailing end silently shifts every later entry's flattened index, corrupting an already-generated external mapping
85. A curated menu config cannot be rebuilt from game config: map old indices to new by identity, and never insert mid-order (2026-10-01)
86. Integrating with the "R5 Mod Settings" companion mod — registering a settings page, reading values back, keybind vs. toggle live-update rules (and a real crash trap), and a more scalable auto-generated-manifest pattern
87. A UE4SS-reflected UFunction call requires every parameter explicitly, even ones with a C++ default value
88. A widget object surviving IsValid() across a level transition is not proof it's still mounted in the viewport — own your own overlay widget instead of adopting one the engine created
89. When a fix "doesn't work" and new diagnostic logging added to explain why also never fires, suspect an upstream wrapper silently dropping data — grep every call site before adding more logging at the same layer
90. A relationship/faction system does not preclude a hardcoded "enemy by literal native class" rule sitting alongside it and evaluated first — confirm with an actual asset export, not by assuming a faction copy is sufficient
91. A per-session cache reloaded on "has this ever loaded" rather than "does the id match what's currently cached" silently serves stale data after a context switch — and the same bug can exist in one function while a sibling function in the same file already has the correct check

Every entry is a specific, confirmed-live finding — not a guess, not "should work in theory." Where something was tried and failed, that's recorded too (a documented dead end saves someone else the same hours).

`Windrose_Unreal_SDK_Notes.txt` — the OTHER side: setting up a real, standalone Unreal Editor project against generated header stubs for the game's own reflected classes, and using it to author, cook, and package genuinely new content entirely offline. Organized into twelve numbered sections:

1. Setting up the project
2. Authoring content headlessly
3. Cooking
4. Retargeting a soft reference to the real target asset
5. The one process rule that actually matters: cook once, edit once
6. Verification: script it, don't eyeball it
7. Worked example: retargeting which body mesh a class resolves to
8. Worked example: real per-size material control (and why it's needed at all)
9. Packaging and installing
10. Authoring a genuinely NEW native class (not just a new data-only asset), confirmed live
11. A genuinely new native-class Blueprint spawns fine — making it look and act like a real one is a separate, ongoing task
12. Baking a fix instead of writing it at runtime: an inherited component's own defaults, and a new class-reference retarget technique

Covers the two real limitations of headless scripting (constructing a `GameplayTag` from a string; setting a soft-object reference to an external/unmounted asset) and their workarounds, several general Unreal build-environment gotchas, and a technique for retargeting a hard CLASS reference (not just a soft object value) via a cooked package's own import table. Cross-references `Windrose_Modding_Notes.txt` where the two overlap rather than duplicating content.

`pakcontents.xlsx` — an export of every asset name inside the game's `.pak`/`.utoc` files, one sheet per file. Useful for finding a class/asset path to spawn or reference without digging through the raw archives yourself.

The Nexus download (`Windrose_Modding_Notes.zip`) bundles all of the above plus the [`LICENSE`](LICENSE) file (see License below) — the GitHub repo is always the canonical, most current copy.

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

## Credits

Living Base Enhanced began as a fork of the original [Living Base](https://www.nexusmods.com/windrose/mods/519) by [me123420](https://www.nexusmods.com/windrose/users/4796505), and some of these findings build on that mod's original work — thank you.

## License

Public domain (CC0) — see [`LICENSE`](LICENSE). Use it, copy it, fork it, mirror it, build on it, no attribution required. It's meant to be freely useful to the whole Windrose modding community, not just this project.
