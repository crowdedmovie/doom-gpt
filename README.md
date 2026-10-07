# doom-gpt

I got the **real DOOM (1993)** running directly inside a regular ChatGPT conversation, starting in **E1M1: Hangar**, with the original graphics, enemies, HUD, sound effects, and music.

This is **not a clone** and ChatGPT didn't recreate the DOOM engine. The fun part was getting ChatGPT to build an interactive JavaScript interface around an existing open-source WebAssembly port: [wasmdoom](https://github.com/theMagicalKarp/wasmdoom).

## 1/3 — The prompt

You'll need to download the three files listed in **Credits and downloads** below before launching the game. Then paste this prompt into a ChatGPT conversation that supports executable HTML/JavaScript components:

> Build an interactive, playable version of the **original DOOM (1993), E1M1: Hangar**, directly inside this ChatGPT conversation using an executable HTML/CSS/JavaScript component (AppBlock if available).
>
> **Do not make a clone or a simulation.** Use the actual `theMagicalKarp/wasmdoom` WebAssembly engine: https://github.com/theMagicalKarp/wasmdoom
>
> Include three local file inputs so I can upload `doom1.wad` (shareware v1.9, 4,196,020 bytes), `wasmdoom.wasm`, and `wasmdoom.music.wasm`. Read them with `File.arrayBuffer()`. Do not fetch any game assets or rely on an external server.
>
> Instantiate the engine with `WebAssembly.instantiate()`, allocate and copy the WAD with `wasmdoom_wad_alloc()`, stage null-separated arguments at `wasmdoom_argv_ptr()`, and call `wasmdoom_init()`. Use the flags `-mode shareware -skill 3 -warp 1 1`.
>
> Run `wasmdoom_tick()` at **35 ticks per second** with `requestAnimationFrame()`. Display the real 320×200 indexed framebuffer using `wasmdoom_get_framebuffer()` and `wasmdoom_get_palette()`, converting the palette to RGBA on a pixelated HTML Canvas.
>
> Implement keyboard controls through `wasmdoom_keydown()` and `wasmdoom_keyup()`, following the key codes in `lib/wasmdoom-keys.ts`. Support W/Z to move forward, S backward, A/Q and D to strafe, arrow keys to turn, E to use doors, Space to fire, Shift to run, 1–7 to select weapons, Tab for the map, Escape for the menu, and Enter to confirm. The Canvas should receive keyboard focus; release held keys when focus is lost.
>
> Implement the original audio: drain engine events using `wasmdoom_events_ptr()`, `wasmdoom_events_len()`, and `wasmdoom_events_clear()`. Play DMX PCM sound events (10–12) using Web Audio. Load `wasmdoom.music.wasm` into a locally created AudioWorklet for OPL3 music events (20–27), queueing music messages until the synth is ready. Enable audio from a user gesture.
>
> Add **Launch DOOM**, **Pause**, and **Sound on/off** buttons, plus a technical log for startup messages and errors. Return the **actual executable component in the conversation**, not just code instructions. If the chat environment blocks WebAssembly, local file inputs, or AudioWorklet, report that limitation instead of pretending it works.

Once the component appears, select the three files and click **Launch DOOM**. Click the game screen to focus the keyboard.

## 2/3 — Quick technical stack

- **Game engine:** id Software's original DOOM C code, adapted by **theMagicalKarp** into a standalone WebAssembly module (`wasmdoom.wasm`) with no WASI imports.
- **Game assets:** the original `doom1.wad` shareware file (levels, textures, sprites, sounds, music data).
- **Rendering:** JavaScript reads DOOM's **320×200 indexed framebuffer** and **256-color RGB palette**, then draws it through an HTML Canvas.
- **Game loop and controls:** `requestAnimationFrame()` drives the engine at **35 ticks/second**; keyboard events are forwarded to the actual engine.
- **Sound:** Web Audio plays the WAD's original DMX sound effects.
- **Music:** a second WebAssembly module (`wasmdoom.music.wasm`) handles MUS/GENMIDI playback using **Nuked OPL3** inside an AudioWorklet.
- **ChatGPT's role:** generating and hosting the interface that connects these pieces. **It isn't generating DOOM itself.**

The three files are selected locally by the user; the game does not need a game server once the files are loaded.

## 3/3 — Credits and downloads

### Credits

Huge thanks to the people who made this possible long before my ChatGPT experiment:

- **[id Software](https://github.com/id-Software/DOOM)** — original DOOM and its released engine source code; **John Carmack** led the engine programming.
- **John Romero** — designer of E1M1: Hangar.
- **Bobby Prince** — original DOOM soundtrack, including E1M1's *At Doom's Gate*.
- **[theMagicalKarp](https://github.com/theMagicalKarp/wasmdoom)** — the `wasmdoom` WebAssembly port and its browser integration.
- **[Nuke.YKT](https://github.com/nukeykt/Nuked-OPL3)** — the **Nuked OPL3** emulator used for FM music synthesis.
- And all the contributors to the original game, source ports, browser technologies, and open-source tools involved.

### Files you need

| File | What it does | Link |
| --- | --- | --- |
| `doom1.wad` | Original DOOM Shareware v1.9 assets (4,196,020 bytes) | [Download WAD](https://raw.githubusercontent.com/theMagicalKarp/wasmdoom/main/wads/doom1.wad) |
| `wasmdoom.wasm` | WebAssembly game engine | [Hosted binary](https://themagicalkarp.github.io/wasmdoom/wasmdoom.wasm) |
| `wasmdoom.music.wasm` | WebAssembly music synthesizer | [Hosted binary](https://themagicalkarp.github.io/wasmdoom/wasmdoom.music.wasm) |

**If the hosted WASM URLs no longer work:** check the project's [GitHub releases](https://github.com/theMagicalKarp/wasmdoom/releases) for published binaries, or [build from source](https://github.com/theMagicalKarp/wasmdoom) with `zig build`. The project's release workflow is set up to publish both WASM artifacts. The direct hosted binary URLs above are based on the project's published website layout and haven't been independently checked for availability here.

**Licensing note:** DOOM's released engine source code and the original game's WAD assets are not the same thing. The engine is released under GPL-2.0, Nuked OPL3 uses LGPL-2.1-or-later, and DOOM's shareware assets remain subject to their own terms. Credit the original authors and check the relevant licenses before redistributing files.

---

I didn't port DOOM or make a new game engine. I just wanted to see whether a regular ChatGPT conversation could run it — and it turns out it can, in a compatible chat runtime. Have fun on E1M1!
