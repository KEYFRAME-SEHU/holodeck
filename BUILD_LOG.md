# Project Holodeck — Build Log

Star Trek TNG-style holodeck WebXR demo for the Meta Connect WebXR workshop
(Sept 24, 2026). Built with IWSDK 0.5.3, VR target, locomotion, no physics,
no grabbing.

## Phase 1 — Grid room + voice link + Arch (completed 2026-09-16)

### Scope (strict)
1. IWSDK VR app, locomotion, no physics, no grabbing
2. Canonical resting grid room with visible entrance door/arch
3. Web Speech API for "Computer, ..." commands
4. Push-to-talk recording fallback where speech recognition is unavailable
5. Transcript, canned computer responses, best-effort speechSynthesis
6. World-space Arch panel summoned by "Computer, show me the arch" or button
7. Placeholder programs only — no world generation, no deployment, no paid APIs

### Files
- `src/scene-assets/holodeck-room.scene-asset.ts` — 8m x 8m x 3.2m room.
  Black floor/ceiling/walls, yellow 0.5m grid on all surfaces, dark door slab
  on the front wall with yellow frame and center seam. Clear player origin
  at [0,0,0]. Procedural parentless geometry, Three.js imported from
  `@iwsdk/core` only.
- `public/ui/voice-bar.uikitml` — "HOLODECK VOICE LINK" panel: transcript,
  status, Enable Voice / Push to Talk / Show Arch buttons.
- `public/ui/arch.uikitml` — "HOLODECK ARCH" panel: three placeholder program
  buttons (Dixon Hill 1941 San Francisco; Ancient West, Tombstone 1881;
  Beach Resort, Risa), status field, Dismiss button. Hidden on load.
- `src/voice.ts` — event bus (transcript/status/computer/arch-show/arch-hide),
  SpeechRecognition + webkitSpeechRecognition detection, continuous
  recognition for "Computer"-prefixed commands, canned responses ("Yes?",
  "Acknowledged.", "Arch displayed.", "Arch dismissed.", Phase 1 program
  placeholder), best-effort speechSynthesis, single-shot Push to Talk where
  supported, MediaRecorder fallback (records duration, no transcription —
  deferred to Phase 2). Browser/editor runtime guards.
- `src/panel.ts` — HolodeckPanelSystem wires both panels: Arch starts hidden,
  button click handlers, placeholder program responses, voice-event
  subscription, cleanup registrations.
- `src/index.ts` — creates the world, registers HolodeckPanelSystem.
- `src/assets.ts` — registers holodeck-room, voice-bar, arch-panel.
- `public/scenes/main.iwsdk.scene.json` — room at origin with
  LocomotionEnvironment, world-space voice + Arch panels with RayInteractable.
- `src/vite-env.d.ts` — minimal Web Speech API declarations.

### Issues encountered and resolutions
1. **npm install proxy failure.** Initial `npm install` failed with
   `ERR_PROXY_TUNNEL`. Retry succeeded; 271 packages installed.
2. **Managed browser Chromium download failures.** The managed Chromium
   download repeatedly failed (closed network connections), and manual
   extraction hit /tmp's 512MB limit. Resolved by extracting under
   `/home/hatch` instead of /tmp. Installed the exact Playwright 1.63.0
   revision: Chromium 1243 / Chrome for Testing 153.0.8010.12 (headed
   Chromium + headless shell). Headless launch verified OK.
3. **UIKitML `flex` shorthand unsupported.** The voice bar initially rendered
   black because `flex: 1` is not a valid UIKitML declaration. Replaced with
   explicit `width`. Lesson: UIKitML supports `display: flex` but not the
   `flex` shorthand; numeric sizes are centimetre-like units.
4. **Horizon kit theming.** The Horizon spatial UI kit defaults to a light
   color scheme (white panels) and its Button has a large fixed min-width
   that overflowed a 3-button row. Resolved with
   `<meta preferred-color-scheme="dark" />` on both panels (dark glass,
   yellow accents) and a vertical button stack on the voice bar. Both panels
   verified with `npx iwsdk ui render-preview`:
   `artifacts/voice-bar-preview.png`, `artifacts/arch-preview.png`.
5. **Emulator UI click testing.** Entered emulated XR (`npx iwsdk xr enter`,
   sessionActive: true, immersive-vr, Meta Quest 3), aimed the right
   controller at buttons (ray cursor confirmed on target), and tried four
   input paths: `xr select`, `xr set-gamepad-state` trigger press (1s and
   0.4s), `xr set-select-value`. None produced a UI click in the emulator,
   though the input pipeline (MultiPointer HOVER -> SELECT -> pointer
   down/up -> click synthesis) was traced through the SDK source and the
   wiring (`addEventListener('click', ...)` on UIKit elements) matches the
   documented API. Ray hover on buttons confirmed visually. The click ->
   toggle path remains unverified in the emulator; it should be smoke-tested
   on a real Quest. All other Phase 1 behaviors verified (see below).

### Verification (2026-09-16)
- `npx tsc --noEmit` — pass.
- `npm run build` — pass (43.8s).
- Runtime browser connected, `browserCommandReady: true`.
- Console: zero application errors/warnings at steady state. (After XR
  exit/reload only environmental entries: controller .glb visuals failing to
  fetch from the jsdelivr CDN in the sandbox, and a transient WebGLRenderer
  resize warning during XR teardown. No app errors.)
- Runtime ECS: 14 entities, including Holodeck Room, Voice Bar, Arch Panel,
  LevelRoot, xr-origin-head.
- Screenshots:
  - `artifacts/runtime-hero.png` — resting state: grid room, door, voice bar
    visible, Arch hidden.
  - `artifacts/scene-quarter.png` — scene validation render.
  - `artifacts/arch-visible-test.png`, `artifacts/click-test-*.png`,
    `artifacts/trigger-test*.png`, `artifacts/selectvalue-test.png`,
    `artifacts/aim-check.png` — XR emulator interaction attempts.
- XR: `npx iwsdk xr enter` -> `sessionActive: true`, mode immersive-vr;
  interior HMD-view screenshots captured; `npx iwsdk xr exit` clean.

### Findings for later phases
- **Multiplayer:** IWSDK reference searches found no native multiplayer /
  shared-state API. Multiplayer for up to 5 people is possible but needs a
  separate networking backend (room membership, avatar transforms, voice/state
  sync, shared world changes). Do not claim IWSDK provides this natively.
- **World generation:** World Labs Marble API is the real path (async
  text-to-world ~5 min, draft ~230 credits; wallet held 7,000 credits at last
  test 2026-09-15). Meta WorldGen is research-only; AssetGen 2.0 is
  Horizon-creators-only later in 2026, neither is a public text-to-world API.
  Demo strategy: pre-generated library for instant transitions + "compiling
  program" progress for novel prompts; panoramas-as-skyboxes first, full
  Gaussian splats deferred. No World Labs calls or credit spend in Phase 1.
- **Voice:** speech recognition state handling is code-complete but
  untestable headless (no mic); MediaRecorder fallback records duration only.
  Transcription of fallback audio deferred to Phase 2.

### Not done in Phase 1
- End-to-end button click -> Arch toggle in the XR emulator (see issue 5).
- Voice command recognition against a live microphone.
- Any deployment, GitHub/HTTPS work (Julian is learning this step by step).
- Any World Labs generation or credit spend.
