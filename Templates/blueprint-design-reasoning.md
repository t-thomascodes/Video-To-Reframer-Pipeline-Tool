# Blueprint Design Reasoning

## The Core Principle

**The blueprint captures WHAT Gemini sees. Claude Code decides HOW to build it.**

Think of it like an architect looking at a building photo vs. the construction blueprints:
- Photo shows: "glass facade, 20 stories, curved top"  
- Construction plans show: "use 12mm tempered glass, steel I-beams at 3m intervals"

Gemini is taking the photo. Claude Code draws the construction plans.

---

## What I Removed and Why

### 1. `implementationGuidelines` — REMOVED

**Original:**
```json
"implementationGuidelines": {
  "critical": [
    "MUST use @remotion/three for the Canvas setup.",
    "MUST use <Suspense> when loading the GLTF model and Fonts.",
    ...
  ]
}
```

**Why removed:** Gemini cannot observe library names, React components, or code patterns from watching a video. This is Claude Code's domain knowledge.

**The test:** Can you see `<Suspense>` by watching a video? No → Remove it.

---

### 2. `antiPatterns` — REMOVED

**Original:**
```json
"antiPatterns": [
  {
    "wrong": "const model = useLoader(GLTFLoader, url)",
    "right": "const { scene } = useGLTF(url)",
    ...
  }
]
```

**Why removed:** This is code review guidance, not video analysis. Gemini has no way to know which React hooks are "better." This belongs in Claude Code's training or a separate Remotion best-practices document.

---

### 3. `implementation.code` blocks — REMOVED

**Original:**
```json
"implementation": {
  "code": "const float = Math.sin((frame / fps) * 2) * 10;",
  "note": "Subtle hovering effect"
}
```

**Replaced with:**
```json
{
  "motion": "oscillating",
  "amplitude": 10,
  "period": 3.0
}
```

**Why:** Gemini can observe "this thing bobs up and down slowly, about this much." It cannot (and should not) write JavaScript. By describing the motion semantically, Claude Code can implement it correctly using Remotion's APIs.

**Translation Claude Code performs:**
- `oscillating` → `Math.sin()`
- `amplitude: 10` → multiply by 10
- `period: 3.0` → `(frame / fps) * (2 * Math.PI / 3.0)`

---

### 4. Library-specific notes — REMOVED

**Original:**
```json
"note": "Use staticFile() if local, or useGLTF() for remote"
```

**Why removed:** `staticFile()` and `useGLTF()` are Remotion/drei API calls. Gemini observing a video has no concept of these. The note was helpful but belongs in Claude Code's context, not the video blueprint.

---

### 5. `threeSpecific` guidance — REMOVED

**Original:**
```json
"threeSpecific": {
  "text3D": "Use <Text3D> from @react-three/drei...",
  "lighting": "Apply a MeshStandardMaterial..."
}
```

**Why removed:** Same principle—library recommendations aren't observable from video. Gemini can say "the text has 3D depth and reacts to light." Claude Code knows that means `<Text3D>` + `MeshStandardMaterial`.

---

## What I Kept and Why

### 1. `source` metadata — KEPT (simplified)

```json
"source": {
  "description": "3D cinematic text reveal...",
  "duration": 5.0,
  "dimensions": { "width": 1920, "height": 1080 },
  "fps": 60,
  "background": "#000000"
}
```

**Why:** All observable. Gemini can detect video dimensions, duration, frame rate, and dominant background color.

---

### 2. `assets` with descriptions — KEPT (restructured)

**Original:**
```json
{
  "id": "brand-logo-model",
  "type": "gltf",
  "url": "https://assets.example.com/models/logo-mesh.glb",
  "note": "Use staticFile() if local..."
}
```

**Cleaned:**
```json
{
  "id": "logo-model",
  "type": "3d-model",
  "description": "Stylized brand logo, appears metallic/reflective",
  "position": "background, centered"
}
```

**Why:** Gemini can see "there's a 3D model that looks metallic." It cannot see file URLs or know which loader to use. The description gives Claude Code enough to work with—if the user provides the actual GLB file, Claude Code will handle loading it correctly.

---

### 3. `layers` — ADDED

```json
"layers": [
  { "order": 0, "assetId": "logo-model", "depth": "back" },
  { "order": 1, "assetId": "main-text", "depth": "front" },
  { "order": 2, "assetId": "sweep-light", "depth": "illumination" }
]
```

**Why added:** Z-ordering is visually observable. Gemini can see "the text is in front of the logo." This helps Claude Code structure the scene graph correctly.

---

### 4. `animations` with semantic motion types — KEPT (restructured)

**Keyframe-based (when motion has clear start/end):**
```json
{
  "target": "main-text",
  "property": "position.z",
  "keyframes": [
    { "time": 0, "value": -500 },
    { "time": 1.5, "value": 0 }
  ],
  "easing": "ease-out-strong"
}
```

**Continuous motion (oscillations, rotations):**
```json
{
  "target": "logo-model",
  "property": "position.y",
  "motion": "oscillating",
  "amplitude": 10,
  "period": 3.0
}
```

**Why this structure:** 
- Time in seconds (observable) not frames (implementation detail)
- Easing as descriptive strings (`ease-out-strong`) not bezier arrays
- Motion types (`oscillating`) instead of code (`Math.sin()`)

---

### 5. `snapshots` — KEPT (this is Gemini's superpower)

```json
"snapshots": [
  {
    "time": 2.0,
    "description": "Light sweep beginning from left edge, creating highlight on text surface"
  }
]
```

**Why:** Natural language descriptions of key moments. This is what Gemini does best—describe what it sees. These serve as:
1. Validation checkpoints for Claude Code
2. Context that pure numbers can't capture
3. Disambiguation when keyframes are ambiguous

---

### 6. `visualNotes` — ADDED

```json
"visualNotes": {
  "colorPalette": ["#000000", "#60a5fa", "#ffffff"],
  "mood": "cinematic, tech, premium",
  "lighting": "dark environment with dramatic spotlight",
  "motionCharacter": "smooth, polished, professional"
}
```

**Why added:** Holistic observations that don't fit into structured fields. Gemini can perceive "this feels premium and cinematic"—that context helps Claude Code make stylistic decisions (smooth easings, not bouncy ones; subtle effects, not flashy).

---

## The Litmus Tests

When deciding if something belongs in the blueprint, apply these tests:

| Test | If YES → | If NO → |
|------|----------|---------|
| Can Gemini observe this by watching the video? | Keep | Remove |
| Does it contain `()` parentheses? | Remove (it's code) | Maybe keep |
| Does it contain `<>` angle brackets? | Remove (it's JSX) | Maybe keep |
| Does it mention a library name? | Remove | Maybe keep |
| Would a non-programmer understand it? | Keep | Reconsider |

---

## The Handoff Contract

**Gemini's job:**
- Watch the video
- Describe what exists (objects, colors, positions)
- Describe how things move (keyframes, easing character, motion types)
- Describe what it looks like at key moments (snapshots)
- Capture the vibe (visual notes)

**Claude Code's job:**
- Convert seconds to frames using `fps`
- Map easing strings to `Easing.*` functions
- Map motion types to mathematical implementations
- Choose appropriate Remotion/Three.js components
- Handle deterministic randomness
- Apply all Remotion best practices (clamping, memoization, etc.)

The blueprint is the creative brief. Claude Code is the technical executor.
