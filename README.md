# doom-gpt

I got the **real DOOM (1993)** running directly inside a regular ChatGPT conversation, starting in **E1M1: Hangar**, with the original graphics, enemies, HUD, sound effects, and music.

This is **not a clone** and ChatGPT didn't recreate the DOOM engine. The fun part was getting ChatGPT to build an interactive JavaScript interface around an existing open-source WebAssembly port: [wasmdoom](https://github.com/theMagicalKarp/wasmdoom).

## 1/3 — The prompt

You'll need to download the three files listed in **Credits and downloads** below before launching the game. Then paste this prompt into a ChatGPT conversation that supports executable HTML/JavaScript components:


> Mission: Run the real 1993 DOOM directly inside ChatGPT
> 
> I want you to create an interactive, executable game interface directly in our ChatGPT conversation, allowing us to play the original 1993 DOOM, specifically level E1M1 — Hangar.
> 
> Important: this method has already been successfully tested in another ChatGPT conversation. The real DOOM engine, compiled to WebAssembly, runs inside an HTML/JavaScript component embedded in the chat, including graphics, keyboard input, sound effects, and music.
> 
> You must NOT create a DOOM clone, a FPS inspired by DOOM, or a visual simulation. I want to run the real DOOM engine using its original assets.
> 
> 1. Engine to use
> 
> Use the open-source project:
> 
> https://github.com/theMagicalKarp/wasmdoom
> 
> This is a WebAssembly port of the original DOOM engine, with no WASI dependencies to import.
> 
> I own the following three files:
> 
> doom1.wad — original DOOM Shareware v1.9, 4,196,020 bytes.
> wasmdoom.wasm — the main game engine.
> wasmdoom.music.wasm — the OPL3 music synthesizer.
> 
> The interface must include three file selection fields, so I can load these three files directly from my computer after the component is displayed.
> 
> It must not depend on GitHub, external downloads, or a server: the files will be read locally via input type="file" HTML elements and File.arrayBuffer().
> 
> 2. ChatGPT interface
> 
> Create an interactive component embedded in the response, using ChatGPT's HTML/CSS/JavaScript execution support, notably AppBlock if available.
> 
> This component must include:
> 
> A 320 × 200 pixel Canvas screen, scaled up with pixelated rendering.
> The three file selection fields.
> A "Launch DOOM" button.
> A Pause button.
> A sound on/off toggle button.
> A technical log displaying engine messages and any errors.
> PC keyboard support.
> 
> I want to play directly in the chat, without needing to open an external web page.
> 
> 3. Technical engine initialization
> 
> After the files are selected:
> 
> Compile and instantiate wasmdoom.wasm with WebAssembly.instantiate().
> Retrieve instance.exports and its memory exports.memory.
> Allocate space for the WAD with wasmdoom_wad_alloc(wadBytes.length).
> Copy the bytes of doom1.wad into WebAssembly memory.
> Prepare the engine arguments at the address returned by wasmdoom_argv_ptr(), as UTF-8 strings separated by null bytes.
> Use the arguments -mode shareware -skill 3 -warp 1 1 to start directly on E1M1.
> Call wasmdoom_init().
> Run the game loop at 35 ticks per second with wasmdoom_tick() and requestAnimationFrame().
> 4. Displaying the real game
> 
> The engine exposes two essential functions:
> 
> wasmdoom_get_framebuffer(): pointer to 64,000 palette indices corresponding to the 320 × 200 image.
> wasmdoom_get_palette(): pointer to the 768 bytes of the original RGB palette.
> 
> On each frame, convert the palette indices into RGBA pixels using an ImageData object, then display the result with putImageData() on the Canvas.
> 
> 5. Keyboard controls
> 
> Forward input events to the real engine with wasmdoom_keydown() and wasmdoom_keyup().
> 
> Implement:
> 
> W/Z: move forward.
> S: move backward.
> A/Q and D: strafe left and right.
> Left/right arrows: turn.
> E: open doors and use objects.
> Space: fire.
> Shift: run.
> Keys 1 to 7: select weapons.
> Tab: map.
> Escape: menu.
> Enter: confirm.
> 
> Respect the key codes in lib/wasmdoom-keys.ts in the wasmdoom repository. The Canvas must be able to receive keyboard focus. Release any pressed keys when the focus is lost.
> 
> 6. Original audio
> 
> The engine generates audio events in its event buffer, accessible via:
> 
> wasmdoom_events_ptr()
> wasmdoom_events_len()
> wasmdoom_events_clear()
> 
> Read the events after initialization and after each tick.
> 
> Sound events are of types 10, 11, and 12. The original sound effects are DMX data, with an 8-byte header followed by 8-bit mono PCM samples at 11,025 Hz.
> 
> Use Web Audio to play them back.
> 
> For music, load wasmdoom.music.wasm into an AudioWorklet and forward music events 20 to 27. Use the functions wasmdoom_music_init, wasmdoom_music_alloc, wasmdoom_music_set_genmidi, wasmdoom_music_register, wasmdoom_music_play, and wasmdoom_music_render.
> 
> Queue music events if the synthesizer is not yet ready.
> 
> The AudioContext must be activated by a user interaction to comply with browser rules.
> 
> 7. Success criteria
> 
> The expected result is a genuine, playable DOOM E1M1 within the conversation, with:
> 
> The original level layout.
> The original monsters and their behaviors.
> The original weapons.
> Doors and collisions.
> The original HUD.
> The original sound effects and music.
> Smooth keyboard movement.
> 
> This solution has already worked in ChatGPT with these three files. You should therefore favor this existing architecture rather than inventing a new engine.
> 
> Proceed directly to building the playable interface. Provide the complete HTML/CSS/JavaScript code in an executable component embedded in this conversation. Do not merely explain how to do it, and do not replace the real DOOM with a clone.
> 
> If the environment of this chat does not allow the component to execute, explicitly state the limitation rather than pretending the game works.


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
