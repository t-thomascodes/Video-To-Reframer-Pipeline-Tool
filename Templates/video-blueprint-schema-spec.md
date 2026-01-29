# Video Inspiration Blueprint Schema Specification

**Version:** 1.0  
**Purpose:** Define a structured format for AI-extracted video analysis that enables deterministic recreation in Remotion.

---

## Overview

This schema captures **observable visual properties** from reference videos. It is designed for:

- **Producer:** Gemini (or similar multimodal AI) analyzing video
- **Consumer:** Claude Code generating Remotion components

**The golden rule:** If you can't see it by watching the video, it doesn't belong in the blueprint.

---

## Schema Definition

```json
{
  "$schema": "video-inspiration-blueprint-v1.0",
  
  "source": { ... },
  "scenes": [ ... ],
  "assets": [ ... ],
  "layers": [ ... ],
  "animations": [ ... ],
  "snapshots": [ ... ],
  "visualNotes": { ... }
}
```

---

## Section: `source`

Global video metadata.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `description` | string | ✓ | One-sentence summary of the video content |
| `duration` | number | ✓ | Total duration in seconds |
| `dimensions` | object | ✓ | `{ "width": number, "height": number }` |
| `fps` | number | ✓ | Frames per second (detect or assume 30/60) |
| `background` | string | ✓ | CSS color, gradient, or `"transparent"` |

**Example:**
```json
"source": {
  "description": "App explainer with 3 scenes: value prop, CTA, and incentive offer",
  "duration": 8.0,
  "dimensions": { "width": 1920, "height": 1080 },
  "fps": 30,
  "background": "linear-gradient(135deg, #4c1d95 0%, #2563eb 100%)"
}
```

---

## Section: `scenes`

Explicit timeline segments. Helps Claude Code structure `<Sequence>` components.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✓ | Unique identifier (kebab-case) |
| `name` | string | ✓ | Human-readable scene name |
| `start` | number | ✓ | Start time in seconds |
| `end` | number | ✓ | End time in seconds |
| `transition` | string | | How this scene enters: `"cut"`, `"fade"`, `"slide-left"`, etc. |

**Example:**
```json
"scenes": [
  { "id": "value-prop", "name": "Value Proposition", "start": 0, "end": 3.0, "transition": "cut" },
  { "id": "cta", "name": "Call to Action", "start": 3.0, "end": 5.0, "transition": "cut" },
  { "id": "incentive", "name": "Incentive Offer", "start": 5.0, "end": 8.0, "transition": "fade" }
]
```

---

## Section: `assets`

All visual elements that appear in the video.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✓ | Unique identifier (kebab-case) |
| `type` | enum | ✓ | See Asset Types below |
| `sceneId` | string | | Which scene this belongs to (omit if global) |
| `sourceType` | enum | ✓ | `"user-provided"`, `"generate"`, `"built-in"` |
| `description` | string | ✓ | Visual description of the asset |
| `position` | object | ✓ | See Position Format below |
| `style` | object | | Visual styling properties |
| `content` | string | | For text assets, the actual text |

### Asset Types

| Type | Description |
|------|-------------|
| `text` | Plain text element |
| `text-container` | Text with background shape (pill, badge, etc.) |
| `image` | Raster image (PNG, JPG) |
| `vector-art` | SVG or illustration |
| `shape` | Primitive shape (rect, circle, etc.) |
| `shape-group` | Collection of decorative shapes |
| `video` | Embedded video element |
| `3d-model` | GLTF/GLB model |
| `3d-text` | Extruded 3D typography |
| `lottie` | Lottie animation file |

### Source Types

| Type | Meaning for Claude Code |
|------|------------------------|
| `user-provided` | Expect user to supply this file |
| `generate` | Claude Code should create this (SVG, shape, etc.) |
| `built-in` | Use standard component (gradient, solid color) |

### Position Format

Always use explicit coordinates with units.

```json
"position": {
  "anchor": "center",           // Reference point: "center", "top-left", "bottom-right", etc.
  "x": 0,                       // Offset from anchor (pixels, 0 = at anchor)
  "y": -50,                     // Offset from anchor (pixels, negative = up)
  "z": 0                        // Layer depth for 3D scenes (optional)
}
```

**Anchor values:** `"center"`, `"top-left"`, `"top-center"`, `"top-right"`, `"center-left"`, `"center-right"`, `"bottom-left"`, `"bottom-center"`, `"bottom-right"`, `"fullscreen"`

**Example Asset:**
```json
{
  "id": "text-pill-earn",
  "type": "text-container",
  "sceneId": "value-prop",
  "sourceType": "generate",
  "description": "Pink rounded pill with white bold text",
  "content": "Earn money",
  "position": { "anchor": "center-left", "x": 200, "y": -50 },
  "style": {
    "color": "#ffffff",
    "backgroundColor": "#ec4899",
    "borderRadius": 50,
    "paddingX": 40,
    "paddingY": 20,
    "fontSize": 60,
    "fontWeight": "bold"
  }
}
```

---

## Section: `layers`

Z-ordering of assets. Lower order = further back.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order` | number | ✓ | Stack order (0 = backmost) |
| `assetId` | string | ✓ | Reference to asset id |

**Example:**
```json
"layers": [
  { "order": 0, "assetId": "bg-gradient" },
  { "order": 1, "assetId": "city-skyline" },
  { "order": 2, "assetId": "decor-crosses" },
  { "order": 3, "assetId": "text-pill-earn" },
  { "order": 3, "assetId": "text-pill-ads" },
  { "order": 4, "assetId": "icon-megaphone" }
]
```

---

## Section: `animations`

All motion in the video.

### Common Fields (all animation types)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✓ | Unique identifier |
| `target` | string or string[] | ✓ | Asset id(s) this animation applies to |
| `property` | string | ✓ | What's being animated (see Properties) |
| `description` | string | | Human-readable description of the motion |

### Animatable Properties

| Property | Unit | Description |
|----------|------|-------------|
| `position.x` | pixels | Horizontal position |
| `position.y` | pixels | Vertical position |
| `position.z` | pixels | Depth (3D scenes) |
| `scale` | ratio | Uniform scale (1 = 100%) |
| `scale.x` | ratio | Horizontal scale |
| `scale.y` | ratio | Vertical scale |
| `rotation` | degrees | Z-axis rotation |
| `rotation.x` | degrees | X-axis rotation (3D) |
| `rotation.y` | degrees | Y-axis rotation (3D) |
| `rotation.z` | degrees | Z-axis rotation (3D) |
| `opacity` | 0-1 | Transparency |
| `blur` | pixels | Gaussian blur amount |

### Animation Type A: Keyframe Animation

For motion with discrete start/end states.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `keyframes` | array | ✓ | Array of `{ time, value }` objects |
| `easing` | string | | Easing function (see Easing Values) |

**Example:**
```json
{
  "id": "pill-slide-in",
  "target": "text-pill-earn",
  "property": "position.x",
  "description": "Slides in from left with bounce",
  "keyframes": [
    { "time": 0.0, "value": -500 },
    { "time": 0.8, "value": 200 }
  ],
  "easing": "ease-out-elastic"
}
```

### Animation Type B: Continuous Motion

For ongoing oscillations, rotations, or loops.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `motion` | enum | ✓ | `"oscillating"`, `"rotating"`, `"pulsing"` |
| `amplitude` | number | | Size of motion (pixels, degrees, or ratio) |
| `period` | number | | Duration of one cycle in seconds |
| `lifespan` | [number, number] | | `[startTime, endTime]` when motion is active |

**Example:**
```json
{
  "id": "megaphone-wiggle",
  "target": "icon-megaphone",
  "property": "rotation",
  "description": "Playful back-and-forth wiggle",
  "motion": "oscillating",
  "amplitude": 5,
  "period": 0.5,
  "lifespan": [0, 3.0]
}
```

### Animation Type C: Multi-Keyframe with Segments

For complex animations with multiple phases.

**Example:**
```json
{
  "id": "text-pop-in-out",
  "target": "text-free-reg",
  "property": "scale",
  "description": "Pops in, holds, then shrinks out",
  "keyframes": [
    { "time": 5.0, "value": 0, "easing": "ease-out-back" },
    { "time": 5.5, "value": 1 },
    { "time": 7.5, "value": 1, "easing": "ease-in" },
    { "time": 8.0, "value": 0 }
  ]
}
```

Note: `easing` on a keyframe applies to the transition *from* that keyframe to the next.

---

## Easing Values

Use descriptive names. Claude Code maps these to Remotion's `Easing.*` or `spring()`.

| Value | Motion Character | Remotion Mapping |
|-------|-----------------|------------------|
| `"linear"` | Constant speed | `Easing.linear` |
| `"ease-in"` | Slow start | `Easing.in(Easing.ease)` |
| `"ease-out"` | Slow end | `Easing.out(Easing.ease)` |
| `"ease-in-out"` | Slow both ends | `Easing.inOut(Easing.ease)` |
| `"ease-out-quad"` | Gentle deceleration | `Easing.out(Easing.quad)` |
| `"ease-out-cubic"` | Medium deceleration | `Easing.out(Easing.cubic)` |
| `"ease-out-expo"` | Strong deceleration | `Easing.out(Easing.exp)` |
| `"ease-out-back"` | Overshoot then settle | `Easing.out(Easing.back(1.7))` |
| `"ease-out-elastic"` | Bouncy overshoot | `spring({ damping: 10 })` |
| `"ease-in-back"` | Pull back then go | `Easing.in(Easing.back(1.7))` |
| `"bounce"` | Ball bounce | `Easing.bounce` |
| `"step-end"` | Instant jump at end | `Easing.step1` |
| `"step-start"` | Instant jump at start | `Easing.step0` |

---

## Section: `snapshots`

Natural language descriptions at key moments. Gemini's strength.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `time` | number | ✓ | Timestamp in seconds |
| `description` | string | ✓ | What's visible on screen |

**Guidelines for Gemini:**
- Include one snapshot per scene transition
- Include one snapshot at the visual "peak" of each scene
- Describe spatial relationships ("X is to the left of Y")
- Note any effects visible (blur, glow, shadows)

**Example:**
```json
"snapshots": [
  { "time": 0.0, "description": "Purple-blue gradient background, no content visible yet" },
  { "time": 1.5, "description": "Two text pills visible: 'Earn money' (pink, left) and 'By seeing ads!' (indigo, right). Megaphone icon bottom-right, wiggling. Money bill top-left." },
  { "time": 3.0, "description": "Hard cut - previous elements gone" },
  { "time": 4.0, "description": "Phone mockup centered-right showing app logo. 'Download PplAd' pink pill centered-left." },
  { "time": 6.0, "description": "City skyline silhouette at bottom. Five illustrated people standing in front. Text above: 'Free registration to the first 1000 users'" }
]
```

---

## Section: `visualNotes`

Holistic observations that inform stylistic decisions.

| Field | Type | Description |
|-------|------|-------------|
| `colorPalette` | string[] | Dominant colors (hex values) |
| `atmosphere` | string | Emotional tone (e.g., "energetic", "calm", "corporate") |
| `style` | string | Visual style (e.g., "flat design", "3D realistic", "hand-drawn") |
| `lighting` | string | Lighting approach (e.g., "flat", "dramatic spotlight", "soft ambient") |
| `motionCharacter` | string | How things move (e.g., "bouncy", "smooth", "snappy") |
| `typography` | string | Font style observations (e.g., "bold sans-serif", "elegant serif") |

**Example:**
```json
"visualNotes": {
  "colorPalette": ["#4c1d95", "#2563eb", "#ec4899", "#ffffff"],
  "atmosphere": "Energetic, corporate-friendly, persuasive",
  "style": "Flat design, vector illustrations, vibrant gradients",
  "lighting": "Flat 2D, no dynamic shadows",
  "motionCharacter": "Bouncy and elastic with quick cuts between scenes",
  "typography": "Bold rounded sans-serif, high contrast white on color"
}
```

---

## Complete Example

```json
{
  "$schema": "video-inspiration-blueprint-v1.0",

  "source": {
    "description": "App explainer with value prop, CTA, and incentive scenes",
    "duration": 8.0,
    "dimensions": { "width": 1920, "height": 1080 },
    "fps": 30,
    "background": "linear-gradient(135deg, #4c1d95 0%, #2563eb 100%)"
  },

  "scenes": [
    { "id": "value-prop", "name": "Value Proposition", "start": 0, "end": 3.0, "transition": "cut" },
    { "id": "cta", "name": "Call to Action", "start": 3.0, "end": 5.0, "transition": "cut" },
    { "id": "incentive", "name": "Incentive Offer", "start": 5.0, "end": 8.0, "transition": "cut" }
  ],

  "assets": [
    {
      "id": "bg-gradient",
      "type": "shape",
      "sourceType": "generate",
      "description": "Full-screen purple to blue diagonal gradient",
      "position": { "anchor": "fullscreen" }
    },
    {
      "id": "text-pill-earn",
      "type": "text-container",
      "sceneId": "value-prop",
      "sourceType": "generate",
      "content": "Earn money",
      "description": "Pink rounded pill with white text",
      "position": { "anchor": "center-left", "x": 200, "y": -50 },
      "style": {
        "color": "#ffffff",
        "backgroundColor": "#ec4899",
        "borderRadius": 50,
        "paddingX": 40,
        "paddingY": 20,
        "fontSize": 60,
        "fontWeight": "bold"
      }
    },
    {
      "id": "text-pill-ads",
      "type": "text-container",
      "sceneId": "value-prop",
      "sourceType": "generate",
      "content": "By seeing ads!",
      "description": "Indigo rounded pill with white text",
      "position": { "anchor": "center-right", "x": -200, "y": 50 },
      "style": {
        "color": "#ffffff",
        "backgroundColor": "#4f46e5",
        "borderRadius": 50,
        "paddingX": 40,
        "paddingY": 20,
        "fontSize": 60,
        "fontWeight": "bold"
      }
    },
    {
      "id": "icon-megaphone",
      "type": "vector-art",
      "sceneId": "value-prop",
      "sourceType": "user-provided",
      "description": "Red and white megaphone illustration",
      "position": { "anchor": "bottom-right", "x": -100, "y": -100 }
    },
    {
      "id": "phone-mockup",
      "type": "image",
      "sceneId": "cta",
      "sourceType": "user-provided",
      "description": "Smartphone displaying PplAd app logo",
      "position": { "anchor": "center-right", "x": -200, "y": 0 }
    },
    {
      "id": "text-download",
      "type": "text-container",
      "sceneId": "cta",
      "sourceType": "generate",
      "content": "Download PplAd",
      "description": "Pink rounded pill CTA",
      "position": { "anchor": "center-left", "x": 200, "y": 0 },
      "style": {
        "color": "#ffffff",
        "backgroundColor": "#ec4899",
        "borderRadius": 50,
        "fontSize": 60
      }
    },
    {
      "id": "city-skyline",
      "type": "vector-art",
      "sceneId": "incentive",
      "sourceType": "user-provided",
      "description": "Cyan stylized city building silhouettes",
      "position": { "anchor": "bottom-center", "x": 0, "y": 0 }
    },
    {
      "id": "people-group",
      "type": "vector-art",
      "sceneId": "incentive",
      "sourceType": "user-provided",
      "description": "Five diverse illustrated office workers",
      "position": { "anchor": "bottom-center", "x": 0, "y": -50 }
    },
    {
      "id": "text-incentive",
      "type": "text",
      "sceneId": "incentive",
      "sourceType": "generate",
      "content": "Free registration to\nthe first 1000 users",
      "description": "White centered text above people",
      "position": { "anchor": "top-center", "x": 0, "y": 200 },
      "style": {
        "color": "#ffffff",
        "fontSize": 50,
        "textAlign": "center",
        "lineHeight": 1.3
      }
    }
  ],

  "layers": [
    { "order": 0, "assetId": "bg-gradient" },
    { "order": 1, "assetId": "city-skyline" },
    { "order": 2, "assetId": "text-pill-earn" },
    { "order": 2, "assetId": "text-pill-ads" },
    { "order": 2, "assetId": "icon-megaphone" },
    { "order": 2, "assetId": "phone-mockup" },
    { "order": 2, "assetId": "text-download" },
    { "order": 2, "assetId": "people-group" },
    { "order": 3, "assetId": "text-incentive" }
  ],

  "animations": [
    {
      "id": "pill-earn-enter",
      "target": "text-pill-earn",
      "property": "position.x",
      "description": "Slides in from left with elastic bounce",
      "keyframes": [
        { "time": 0.0, "value": -600 },
        { "time": 0.8, "value": 200 }
      ],
      "easing": "ease-out-elastic"
    },
    {
      "id": "pill-ads-enter",
      "target": "text-pill-ads",
      "property": "position.x",
      "description": "Slides in from right with elastic bounce",
      "keyframes": [
        { "time": 0.3, "value": 600 },
        { "time": 1.1, "value": -200 }
      ],
      "easing": "ease-out-elastic"
    },
    {
      "id": "megaphone-wiggle",
      "target": "icon-megaphone",
      "property": "rotation",
      "description": "Continuous playful wiggle",
      "motion": "oscillating",
      "amplitude": 8,
      "period": 0.4,
      "lifespan": [0.5, 3.0]
    },
    {
      "id": "scene1-exit",
      "target": ["text-pill-earn", "text-pill-ads", "icon-megaphone"],
      "property": "opacity",
      "description": "Hard cut to next scene",
      "keyframes": [
        { "time": 2.95, "value": 1 },
        { "time": 3.0, "value": 0 }
      ],
      "easing": "step-end"
    },
    {
      "id": "phone-enter",
      "target": "phone-mockup",
      "property": "position.y",
      "description": "Slides up from bottom",
      "keyframes": [
        { "time": 3.0, "value": 600 },
        { "time": 3.6, "value": 0 }
      ],
      "easing": "ease-out-expo"
    },
    {
      "id": "download-enter",
      "target": "text-download",
      "property": "position.x",
      "description": "Slides in from left",
      "keyframes": [
        { "time": 3.0, "value": -600 },
        { "time": 3.6, "value": 200 }
      ],
      "easing": "ease-out-expo"
    },
    {
      "id": "scene2-exit",
      "target": ["phone-mockup", "text-download"],
      "property": "opacity",
      "description": "Cut to final scene",
      "keyframes": [
        { "time": 4.95, "value": 1 },
        { "time": 5.0, "value": 0 }
      ],
      "easing": "step-end"
    },
    {
      "id": "city-fade-in",
      "target": "city-skyline",
      "property": "opacity",
      "description": "Background fades in",
      "keyframes": [
        { "time": 5.0, "value": 0 },
        { "time": 5.5, "value": 1 }
      ],
      "easing": "linear"
    },
    {
      "id": "people-enter",
      "target": "people-group",
      "property": "position.y",
      "description": "Pop up from bottom with overshoot",
      "keyframes": [
        { "time": 5.0, "value": 400 },
        { "time": 5.7, "value": -50 }
      ],
      "easing": "ease-out-back"
    },
    {
      "id": "incentive-text-enter",
      "target": "text-incentive",
      "property": "scale",
      "description": "Pops in with bounce",
      "keyframes": [
        { "time": 5.3, "value": 0 },
        { "time": 5.9, "value": 1 }
      ],
      "easing": "ease-out-back"
    }
  ],

  "snapshots": [
    { "time": 0.0, "description": "Purple-blue gradient only, scene empty" },
    { "time": 1.5, "description": "Two text pills centered: 'Earn money' pink left, 'By seeing ads!' indigo right. Megaphone wiggling bottom-right." },
    { "time": 3.0, "description": "Instant cut - scene 1 elements gone" },
    { "time": 4.0, "description": "Phone mockup right side, 'Download PplAd' pill left side" },
    { "time": 5.0, "description": "Instant cut - scene 2 elements gone" },
    { "time": 6.5, "description": "City skyline background, five people at bottom, 'Free registration...' text above them" }
  ],

  "visualNotes": {
    "colorPalette": ["#4c1d95", "#2563eb", "#ec4899", "#4f46e5", "#ffffff"],
    "atmosphere": "Energetic, friendly, sales-focused",
    "style": "Flat vector design with bold colors and rounded shapes",
    "lighting": "Flat 2D, no shadows or highlights",
    "motionCharacter": "Bouncy elastic entrances, snappy hard cuts between scenes",
    "typography": "Bold rounded sans-serif, white on vibrant backgrounds"
  }
}
```

---

## Gemini Prompt Template

Use this prompt when asking Gemini to analyze a video:

```
Analyze this video and produce a JSON blueprint following the video-inspiration-blueprint-v1.0 schema.

RULES:
1. Only describe what you can SEE - no code, no library names, no implementation details
2. Use seconds for all timestamps (not frames)
3. Use descriptive easing names like "ease-out-elastic" (not bezier coordinates)
4. For continuous motion (wiggling, floating, spinning), use the "motion" format with amplitude/period
5. For start-to-end motion, use "keyframes" format
6. Include sourceType for each asset: "user-provided" for images/illustrations the user must supply, "generate" for shapes/text Claude can create
7. Position all elements with explicit anchor + x/y offsets in pixels
8. Write snapshots as if describing the frame to someone who can't see it

OUTPUT FORMAT:
- Valid JSON only
- No markdown code fences
- No explanatory text before or after

ANALYZE THIS VIDEO:
[video attachment or URL]
```

---

## Validation Checklist

Before using a blueprint, verify:

- [ ] All `time` values are in seconds
- [ ] All `position` objects have `anchor` field
- [ ] All assets have `sourceType`
- [ ] No JavaScript code anywhere
- [ ] No library names (remotion, three, react, etc.)
- [ ] No angle brackets `<` `>`
- [ ] No parentheses `()` except in descriptions
- [ ] Easing values are from the allowed list
- [ ] Every asset referenced in `animations` exists in `assets`
- [ ] Every asset in `layers` exists in `assets`
- [ ] `scenes` array covers full duration without gaps
