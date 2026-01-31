# Video to Remotion Reframer

Automated pipeline that reverse-engineers videos into editable Remotion composition scaffolds. Processes video frames through Gemini Flash multimodal API to generate structured JSON templates for React-based video editing.

## What This Does

Takes any video and outputs a Remotion project scaffold with:
- Scene-by-scene breakdowns with timing data
- Asset specifications (text, images, animations)
- Structured JSON templates ready for editing
- React component scaffolds for each scene

Enables non-technical users to convert existing videos into editable, programmatic video projects.

## Architecture
```
Video Input
    ↓
Frame Extraction
    ↓
Gemini Flash API (multimodal analysis)
    ↓
Scene Detection + Timing Analysis
    ↓
JSON Template Generation
    ↓
Remotion Composition Scaffold
```

## Features

- **Automated scene detection**: Identifies transitions, text overlays, and visual elements
- **Timing extraction**: Generates precise frame-level timing data for each scene
- **Asset cataloging**: Lists all text, images, colors, and animations present
- **Remotion scaffolding**: Outputs React components ready for customization
- **JSON templates**: Structured data enabling programmatic video editing workflows

## Use Cases

- Agency workflows: Convert client videos into editable templates
- Video remixing: Adapt existing content with new branding/copy
- Template generation: Build reusable video structures from references
- A/B testing: Programmatically generate video variations

---

**Status**: Ongoing project
