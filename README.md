# NEON FLUX

A 3D, real-time sound visualizer that runs in your browser and reacts to **whatever your PC is already playing** — Spotify, YouTube, a game, your DAW, anything. No install, no build step, no virtual audio cables. It's one HTML file.

Three visual modes, four color themes, mouse-orbit camera, and bloom that pulses on the beat.

![Nebula mode](img/nebula.png)

---

## Quick start

1. Download or clone this repo.
2. Double-click `index.html` (it opens in your default browser — use **Chrome** or **Edge**).
3. Start playing music.
4. Click **🔊 System Audio**. In the share dialog that appears, choose **Entire screen** and tick **"Also share system audio"**, then click Share.

That's it. The visuals should start moving with the music.

> **Not sure it's working?** Click **▶ Demo Signal** first. That plays a built-in synthetic 120 BPM pattern (silently — it's generated math, not sound) so you can confirm the graphics run before wrestling with audio permissions.

### Alternative: run it from a local server

Opening the file directly works fine, but a few browsers restrict microphone and screen-capture APIs on `file://` URLs. If a source refuses to start, serve it over `localhost` instead:

```bash
python -m http.server 8080
```

Then open <http://localhost:8080>. Any static server works — `npx serve`, `php -S localhost:8080`, VS Code's Live Server extension, whatever you have.

---

## How the system audio capture works

This is the part people get stuck on, so it's worth explaining.

Browsers have no direct API for "give me the sound my speakers are making." The workaround is the **screen sharing** API: when you share a screen or a tab, Chrome and Edge on Windows will optionally include the system audio stream alongside the video. The visualizer grabs that stream, throws the video away, and analyzes only the audio.

Practical consequences:

- **You must tick the "Also share system audio" checkbox.** If you miss it, you'll get an error telling you no audio track was shared. Just click the source button again and retry.
- **Pick "Entire screen," not "Window."** Window-sharing does not carry audio on Windows. Sharing a specific *browser tab* does work and is a good option if you only want one tab's sound (e.g. just YouTube, ignoring Discord notifications).
- **Chrome and Edge only.** Firefox and Safari don't support system audio capture. Both still work fine with the microphone and demo sources.
- **Nothing is recorded or uploaded.** The stream goes to an analyzer node and nowhere else. The page has no network code at all beyond loading the Three.js library. It's never connected to your speakers either, so there's no echo or feedback.
- Windows shows a "sharing your screen" indicator while it runs. That's expected — it's the price of the loopback trick.

**Microphone mode** is the fallback if system capture won't cooperate, and it's genuinely more fun with speakers in a room — it picks up the actual acoustics, room reverb and all. It will also pick up you talking, so mind that if you're streaming.

---

## Controls

| Input | Action |
|---|---|
| Drag | Orbit the camera |
| Scroll | Zoom in / out |
| `1` `2` `3` | Switch visual mode |
| `T` | Cycle color theme |
| `H` | Hide / show the UI panel |
| `F` | Toggle fullscreen |

The camera slowly auto-rotates until the first time you drag it, then it stays where you put it. Reload the page to get the drift back.

### Panel controls

- **Source** — switch between System / Mic / Demo at any time without reloading.
- **Mode** and **Theme** — same as the keyboard shortcuts.
- **Sensitivity** — the master gain on everything reactive. Turn it **up** for quiet or heavily-compressed tracks, **down** if loud passages look like a solid blown-out wall. This is the single most useful knob; expect to touch it when you change genres.
- **Glow** — bloom intensity. Low values look sharp and technical, high values look hazy and dreamlike.
- The bar meters at the bottom show live bass / mid / treble levels. If they're flat while music is playing, the audio source isn't connected — recheck the share dialog.

---

## The visual modes

**1 · Nebula** — 14,000 particles in a three-armed spiral galaxy. Radius maps to frequency: the inner core rides the bass, the outer arms shimmer with the treble. The whole galaxy spins faster as the track gets more energetic, and particles bloom on detected beats. Best for ambient, drum & bass, anything with a wide frequency spread.

**2 · City** — a 25×25 grid of neon blocks arranged as a cyberpunk skyline. Height and color follow the spectrum radially outward from the center, so kick drums punch up the middle and hi-hats ripple around the edges. Try zooming down to street level and looking up.

![City mode](img/city.png)

**3 · Core** — a plasma sphere wrapped in a neon lattice. Bass inflates it, mids ripple the surface into churning noise, treble drives the orbiting sparkle field, and the whole thing kicks on each beat. The most "reactive-looking" of the three; good for bass-heavy music.

![Core mode](img/core.png)

### Themes

**Cyberpunk** (cyan/magenta), **Synthwave** (orange/purple), **Matrix** (green), and **Ice** (blue/white). Press `T` to cycle. Here's Nebula again under Synthwave — same geometry, completely different mood:

![Synthwave theme](img/synthwave.png)

---

## Tips

- **Press `H` then `F`.** Hiding the UI and going fullscreen is what makes this feel like a real visualizer rather than a web demo. This is the intended way to actually use it.
- **Put it on a second monitor.** Fullscreen it on a spare display and leave it running while you work or play. It'll happily run for hours.
- **Match the mode to the music.** Sparse, slow tracks look best in Core, where a single bass hit visibly deforms the geometry. Dense, busy tracks look better in Nebula or City, which have enough elements to show detail.
- **Sensitivity is genre-dependent.** Modern loudness-war masters sit near the sensitivity floor — drop to ~0.7. Classical, jazz, and vinyl rips often need 1.8+ to come alive.
- **Fixing choppy playback:** the visualizer is GPU-bound. If it stutters, make a smaller browser window (fewer pixels), or drop Glow toward 0 — bloom is by far the most expensive effect. Nebula is the heaviest mode, Core the lightest.
- **Verify your GPU is actually being used.** If everything is slow, check `chrome://gpu` for "Hardware accelerated" on WebGL. Some Windows setups run browsers on the integrated GPU; in Windows Settings → System → Display → Graphics, set your browser to High performance.
- **Sharing a browser tab instead of the whole screen** gives you cleaner audio when you want to visualize one source and ignore system notification sounds.

---

## Customizing it

Everything lives in `index.html` — no build step, so edit and refresh.

- **Add a theme:** append an entry to the `THEMES` array near the top of the script. Each has an accent color `a`, secondary `b`, background `bg`, and fog color, all as hex numbers. The UI picks up new entries automatically, though you'll want to add a matching button in the Theme row.
- **Change particle count:** `const N = this.N = 14000` in the `Nebula` class. Push it to 40,000 on a strong GPU, drop to 5,000 on a laptop.
- **Change the city grid:** `const S = this.S = 25` in the `City` class. Note it's the square of this, so 40 means 1,600 blocks.
- **Add your own mode:** subclass `VisualMode`, build your geometry in the constructor, and implement `update(freq, levels, dt, t, sensitivity)`. `freq` is the raw 1024-bin FFT as a `Uint8Array`; `levels` gives you smoothed `bass` / `mid` / `treble` / `energy` floats plus a `beat` boolean. Then add your class to the `MODES` array and drop a button in the Mode row.
- **Tune beat detection** in `AudioEngine.update()` — it compares current bass against a rolling 50-frame average with a 160 ms refractory period. Lower the `1.35` multiplier for more sensitive triggering.

---

## Requirements

- A modern browser with WebGL2. Chrome or Edge recommended; required for system audio capture.
- An internet connection **on first load** — Three.js is fetched from a CDN. If it can't load, a red banner appears at the bottom of the page rather than a mysterious black screen. To run fully offline, download `three.module.js` and the four addon files and repoint the import map.

Built with [Three.js](https://threejs.org/) r160 and the Web Audio API. No frameworks, no bundler, no dependencies to install.

## Troubleshooting

| Symptom | Fix |
|---|---|
| "No audio track was shared" | You missed the **Also share system audio** checkbox. Retry and tick it. |
| Black screen, red banner at the bottom | Three.js couldn't load from the CDN. Check your connection and reload. |
| Visuals run but never move | Wrong source, or the media is muted. Check the bass/mid/treble meters; try Demo to confirm the graphics work. |
| "Permission denied" | You dismissed the browser prompt. Click the source button again and allow it. |
| Everything is a blown-out white blob | Sensitivity too high. Slide it down. |
| Barely any movement on loud music | Sensitivity too low, or you're capturing a silent source. |
| Choppy / low frame rate | Lower Glow, shrink the window, or switch from Nebula to Core. |

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, rip the modes out for your own project.
