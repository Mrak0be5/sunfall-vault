# STARFALL — The Wishkeeper’s Vault

An original treasure-vault reward animation with a procedural Three.js chest and Astral Crown, four rarity tiers, GSAP choreography, Canvas particles, and synthesized Web Audio.

Open **`starfall.html`** directly in a modern browser with WebGL enabled. The built page includes its scripts, styles, fonts, and artwork, so it works offline without a server or build step. Sound starts only after interaction. Public page: [Play STARFALL](https://mrak0be5.github.io/sunfall-vault/starfall.html).

## Play

- **One tap:** Open the vault once to play every upgrade and the complete reveal.
- **Tap to upgrade:** Each tap advances one rarity: Common green → Rare blue → Epic purple → Legendary gold. Tap again at Legendary to reveal the rewards.
- **Claim your rewards:** Coins and gems curve into the HUD balances; a replay button appears when collection completes. Balances are a local session demo and reset when the page reloads.
- The upper-right sound button mutes or unmutes synthesis. Playback buttons lock during transitions to prevent overlapping sequences.
- Phone layouts are supported. With the system’s **Reduce Motion** preference enabled before loading, screen shake and camera push are disabled, idle motion is reduced, and fewer particles are emitted. The core reward sequence still plays.

## Timing

Times below are relative to the first tap in One tap mode; Ready waits indefinitely for input.

| Phase | Duration | Timeline |
| --- | ---: | ---: |
| Ready | Until tapped | Before 0 s |
| Upgrade × 3 | 1.48 s each + 0.24 s pause each | 0–5.16 s |
| Charge-up | 1.15 s | 5.16–6.31 s |
| Burst | 1.00 s | 6.31–7.31 s |
| Centerpiece reveal | 3.15 s | 7.31–10.46 s |
| Results entrance and counting | 1.65 s | 10.46–12.11 s |

The claim action becomes available at approximately **12.11 seconds**. Results then wait for input. Collection takes roughly **1.8 seconds**, or less with reduced motion. Manual mode inserts an unlimited input wait between upgrades.

## Adjust the presentation

Edit `CONFIG` at the top of `src/app.js`, then rebuild the standalone file. It is also exposed as `window.VAULT_CONFIG` for inspection.

| Parameter | Default | Effect |
| --- | --- | --- |
| `intensity` | `1` | Multiplies burst particle and sound strength. |
| `tierBurstStrength` | `[0.7, 1, 1.4, 1.9]` | Burst strength by rarity; the normal journey bursts at Legendary. |
| `shake` | `1` | Scales the strongest screen-shake impulses. |
| `flashOpacity` | `0.70` | Caps radial bloom opacity; an additional hard limit keeps it at or below `0.8`. |
| `cameraPush` | `0.55` | Camera travel toward the chest during the burst. |
| `upgradeDuration`, `tierPause` | `1.48`, `0.24` | Upgrade motion and spacing. |
| `chargeDuration`, `burstDuration` | `1.15`, `1.00` | Charge and burst timing. |
| `revealDuration`, `resultsDuration` | `3.15`, `1.65` | Reveal hold and results/claim timing. Keep long enough for their internal animations to finish. |
| `outerLine`, `innerLine` | `2.1`, `0.8` | Post-process silhouette and interior outline widths. |
| `supersampling`, `msaa` | `1.5`, `4` | Render resolution multiplier and WebGL2 multisample count. Change before renderer initialization. |
| `coins`, `gems` | `2400`, `120` | Reward quantities and claim totals. |

The `tiers` array controls the main color, light color, background tint, and copy. `src/models.js` controls the solid ornaments and metal palettes. Particle caps, drag, gravity, lifetimes, suction, orbit, and curved flights are in `src/effects.js`; soft backdrop rays and beam styling are in `src/style.css`.

## Build and capture

Run `python3 starfall/build.py` from the workspace root. The builder embeds `src/` plus the existing Three.js, GSAP, and font files in `highlight/vendor/` into `starfall/starfall.html`.

`window.__vault` exposes the actual animation state, pose, history, and renderer. For repeatable phase capture, start on a fresh load and call:

```js
__vault.setMuted(true);
__vault.play();
__vault.advance(5.96); __vault.pause(); // charge
__vault.advance(0.58); __vault.pause(); // burst
__vault.advance(3.1); __vault.pause(); // reveal
__vault.advance(3.0); __vault.pause(); // results
```

Capture a screenshot after each line. `advance(seconds)` steps GSAP and rendering at 1/60-second intervals, avoiding wall-clock screenshot races; it does not replace the rendered artwork. Particle positions remain randomized. Call `__vault.resume()` to return to normal playback.

`check.mjs` can capture through the existing browser bridge and its already attached background tab. `node starfall/check.mjs audit` writes:

- `evidence/charge.png`
- `evidence/burst.png`
- `evidence/reveal.png`
- `evidence/results.png`
- `evidence/audit.json` — phase, visibility, tap-lock, claim, and manual-mode observations.

`evidence/ready.png` records the idle screen. The four phase screenshots were individually reviewed: the models stay visible, bloom stays controlled, the reveal background stays clear, text remains legible, and results have zero residual shake. Additional phone captures cover 390px and 360px widths. Automated playback checks cover repeated taps, both modes, replay visibility and exact currency collection. Reduced-motion behavior is checked separately in `evidence/phone-audit.json`.

Rendering targets 60 fps using normal/depth edge detection, 1.5× supersampling, and 4× MSAA on WebGL2. **No measured 60 fps benchmark is claimed.** Performance depends on GPU, viewport, and device pixel ratio; the built-in frame samples are diagnostic data, not a representative benchmark.
