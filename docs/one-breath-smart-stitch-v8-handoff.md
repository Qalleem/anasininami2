# RAP STUDIO v8 — ONE BREATH Smart Stitch Handoff

This handoff is matched to the actual `rapstudio_v8.html` app, not a generic audio-app redesign.

## Actual app direction

RAP STUDIO v8 is a single-file mobile-first browser HTML app.

Style lock:

- Black background
- Neon yellow accent: `#e8ff00`
- Compact mono typography: Space Mono / Courier-style
- Faint white borders
- Mobile-first screen width around 480px
- Route strip at top
- Bottom nav
- Existing sections: Home/Record, Beat, Performance, Export/Mix, Tools

Existing One Breath UI language already exists around the Performance screen:

- `.ob-block`
- `.ob-title`
- `.ob-tog`
- `.ob-joins`
- `.ob-join`
- `.ob-badge`
- `.c-clean`
- `.c-ok`
- `.c-rough`
- `.ob-actions`
- `.ob-btn`
- `.ob-btn.primary`

Do not redesign this into a desktop DAW. Do not introduce red/violet visual direction.

---

## Claude implementation prompt

```text
You are patching my existing RAP STUDIO v8 single-file HTML app.

Target file:
rapstudio_v8.html

Task:
Integrate ONE BREATH Smart Stitch into the existing RAP STUDIO v8 Performance screen, playback system, and export system.

Do not create a separate demo.
Do not rewrite the whole app.
Do not redesign the UI.
Do not add external libraries.
Do not add paid APIs.
Use plain HTML, CSS, JavaScript, and Web Audio API only.
Make the smallest safe production patch possible.

==================================================
REAL APP STYLE LOCK
==================================================

The real app is not a generic dark red/violet audio app.

Preserve this visual language:
- black background
- neon yellow accent #e8ff00
- Space Mono / Courier-style mono labels
- compact uppercase labels
- faint white borders
- mobile-first max-width screens around 480px
- top route strip
- bottom nav
- existing Home / Beat / Performance / Export / Tools structure

Preserve existing Performance screen structure:
- parts scroll lane
- part pips
- part detail block
- existing One Breath block
- vocal engine block

Use the existing One Breath block style instead of creating a new DAW timeline.

Existing classes to preserve and reuse:
.ob-block
.ob-title
.ob-tog
.ob-joins
.ob-join
.ob-badge
.c-clean
.c-ok
.c-rough
.ob-actions
.ob-btn
.ob-btn.primary

Do not add red/violet UI.
Do not add a desktop DAW layout.
Do not expose technical debug data to the user.

==================================================
PRODUCT GOAL
==================================================

The user records multiple rap parts/takes.
ONE BREATH Smart Stitch combines them into one continuous rap performance.

This is:
- punch-in recording
- vocal comping
- smart splice
- equal-power crossfade stitching

User-facing idea:
Separate recorded parts should feel like one flowing performance.

Main user-facing actions should fit existing app language:
- ONE BREATH
- ON / OFF
- STITCH
- PLAY MIX
- EXPORT SAFE
- CLEAN / OK / ROUGH

==================================================
CORE AUDIO RULE
==================================================

Raw recorded parts/takes must stay as source material.

The app must render one final stitched vocal buffer:

oneBreathBuffer

This oneBreathBuffer must be the single source of truth for BOTH:
1. preview playback
2. final export / download

Playback and export must use the exact same oneBreathBuffer.

Do not create separate stitching logic for playback.
Do not create separate stitching logic for export.
Do not play raw takes separately in ONE BREATH mode.

==================================================
AUDIO FLOW
==================================================

recorded parts/takes
→ filter active usable audio
→ ignore empty/silent audio
→ RMS envelope analysis
→ detect vocal start/end
→ trim silence with safety guards
→ stitch audio in app order
→ micro-overlap at boundaries
→ equal-power crossfade
→ create oneBreathBuffer
→ preview uses oneBreathBuffer
→ export uses same oneBreathBuffer
→ mix with beat, vocal effects, and master limiter

==================================================
GLOBAL STATE
==================================================

Add or adapt:

let oneBreathBuffer = null;
let oneBreathRenderInfo = null;
let oneBreathDirty = true;

Keep existing raw part/take storage as source data.
Do not replace raw session data with the stitched buffer.

Add:

function markOneBreathDirty() {
  oneBreathDirty = true;
  oneBreathBuffer = null;
  oneBreathRenderInfo = null;
  updateOneBreathUI({ dirty: true });
}

Call markOneBreathDirty() after:
- recording a new part/take
- deleting audio
- undo
- mute change if applicable
- solo change if applicable
- replacing a take
- session restore

==================================================
STITCH OPTIONS
==================================================

const STITCH_OPTIONS = {
  analysisWindowMs: 8,
  leadingGuardMs: 60,
  trailingGuardMs: 120,
  minTakeDurationMs: 250,
  defaultCrossfadeMs: 35,
  maxCrossfadeMs: 80,
  boundarySearchMs: 25,
  silenceThresholdMultiplier: 1.8
};

==================================================
REQUIRED FUNCTIONS
==================================================

Implement these if missing. If similar functions already exist, adapt them safely.

function getRmsEnvelope(buffer, windowMs) {}
function detectVocalRegion(buffer, options) {}
function trimTakeBuffer(buffer, vocalRegion, options) {}
function findNearestZeroCrossing(channelData, sampleIndex, searchSamples) {}
function getActiveAudioForOneBreath() {}
function stitchTakes(takes, options) {}
async function renderOneBreathMix() {}
async function playOneBreathMix() {}
async function exportOneBreathMix() {}
function updateOneBreathUI(report) {}
function buildStitchReport(activeTakes, ignoredTakes, transitionInfo) {}
function markOneBreathDirty() {}

==================================================
RMS / TRIM RULES
==================================================

Use RMS windows around 5–10 ms.
Use each recording’s own noise floor.
Do not use one fixed global threshold.
Detect sustained vocal energy above the noise floor.
Ignore tiny clicks and short noise bursts.
Do not cut inside strong syllables.

Trim leading silence.
Trim trailing silence.
Do not trim aggressively.

Use:
- leadingGuardMs before detected vocal start
- trailingGuardMs after detected vocal end

Prefer trim points around:
- low-energy areas
- breath gaps
- near zero-crossing zones

==================================================
STITCH / CROSSFADE RULES
==================================================

For every boundary between two usable parts/takes:
- create a small overlap
- apply equal-power fade-out to the previous audio
- apply equal-power fade-in to the next audio
- sum both during overlap
- avoid clicks and pops

Use equal-power crossfade:

fadeOut = Math.cos(t * Math.PI / 2)
fadeIn = Math.sin(t * Math.PI / 2)

where t moves from 0 to 1 across the crossfade length.

Default crossfade:
35 ms

If transition is harsh:
allow up to 80 ms

Do not:
- time-stretch
- pitch-shift
- fake missing audio
- add placeholder audio
- expose technical debug data in the UI

==================================================
renderOneBreathMix()
==================================================

Use this central logic:

async function renderOneBreathMix() {
  if (!oneBreathDirty && oneBreathBuffer) return oneBreathBuffer;

  const activeTakes = getActiveAudioForOneBreath();
  const result = await stitchTakes(activeTakes, STITCH_OPTIONS);

  oneBreathBuffer = result.buffer || result;
  oneBreathRenderInfo = result.report || buildStitchReport(activeTakes);
  oneBreathDirty = false;

  updateOneBreathUI(oneBreathRenderInfo);

  return oneBreathBuffer;
}

Edge cases:
- zero usable audio: show a compact status and do not crash
- one usable audio item: return safely trimmed audio, no crossfade needed
- multiple usable items: stitch smoothly in order

==================================================
PLAYBACK INTEGRATION
==================================================

In ONE BREATH mode, playback must call:

const vocalBuffer = await renderOneBreathMix();

Then use vocalBuffer in the existing playback chain.

Do not play raw audio separately in ONE BREATH mode.

==================================================
EXPORT INTEGRATION
==================================================

Final export must also call:

const vocalBuffer = await renderOneBreathMix();

Then use vocalBuffer in the existing export/mixdown chain.

Do not duplicate stitching inside export.
Do not export raw audio separately in ONE BREATH mode.

If playback and export both use the same oneBreathBuffer, show:
EXPORT SAFE

==================================================
UIX REQUIREMENTS FOR EXISTING v8 UI
==================================================

Use the existing `.ob-block` area in the Performance screen.

Do not add a full timeline.
Do not add a new desktop section.
Do not use red/violet styling.

Use compact v8-style labels:
- ONE BREATH
- ON / OFF
- STITCH
- PLAY MIX
- CLEAN
- OK
- ROUGH
- EXPORT SAFE
- NEEDS REFRESH

Status mapping:
CLEAN = smooth transition, ready
OK = usable transition
ROUGH = audible transition, retake may be needed

Do not show:
- RMS numbers
- sample indexes
- zero-crossing values
- AudioBuffer debug data
- long technical logs

==================================================
FEATURE PROTECTION
==================================================

Preserve:
- recording
- beat upload
- beat playback
- beat loop
- beat volume
- BPM detection
- beat flash if present
- input meter
- vocal engine presets
- effects chain
- master limiter
- save/restore
- export
- existing route strip
- existing bottom nav
- existing Performance screen layout

Do not change beat quality.
Do not break playback/export parity.

==================================================
TESTING
==================================================

Verify:
1. One recorded part still plays normally.
2. One usable part shows ready state.
3. Two or more parts stitch in correct order.
4. Silent audio is ignored.
5. No click/pop at transitions.
6. CLEAN / OK / ROUGH status updates.
7. STITCH renders oneBreathBuffer.
8. PLAY MIX uses oneBreathBuffer.
9. Export uses the same oneBreathBuffer.
10. EXPORT SAFE appears only when playback/export share the same buffer.
11. Beat mix remains stable.
12. Existing export still works.
13. Existing recording still works.
14. Save/restore still works.
15. Mobile layout remains usable.

==================================================
OUTPUT FORMAT
==================================================

Return only exact code changes.

Use labels:

PASTE THIS NEAR GLOBAL STATE
PASTE THIS NEAR AUDIO HELPERS
PASTE THIS NEAR UI HELPERS
PASTE THIS CSS IN THE STYLE SECTION
ADD THIS HTML INSIDE EXISTING .ob-block OR PERFORMANCE SCREEN
REPLACE THIS FUNCTION
ADD THIS INSIDE RECORDING COMPLETE HANDLER
ADD THIS INSIDE DELETE/MUTE/SOLO HANDLERS
ADD THIS INSIDE PLAYBACK FUNCTION
ADD THIS INSIDE EXPORT FUNCTION

Do not explain generally first.
Do not redesign unrelated parts.
Do not add libraries.
Do not add paid APIs.
Preserve everything that already works.
```
