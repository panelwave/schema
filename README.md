# PanelWave JSON Schema

**Official JSON Schema definitions for the PanelWave dynamic graphic novel format.**

[![Schema Version](https://img.shields.io/badge/schema-1.5.0-blue.svg)](./1.0/panelwave.schema.json)
[![JSON Schema](https://img.shields.io/badge/json--schema-2020--12-green.svg)](https://json-schema.org/draft/2020-12/schema)
[![License](https://img.shields.io/badge/license-CC%20BY%204.0-lightgrey.svg)](LICENSE)

> **Documentation:** [docs.panelwave.org/schema/overview](https://docs.panelwave.org/schema/overview)

## Overview

PanelWave is an open JSON format for creating dynamic, interactive graphic novels that combine panels, layers, motion, sound, and branching narratives. This repository contains the official JSON Schema definitions that validate PanelWave manifest files.

### What is PanelWave?

PanelWave enables creators to build modern graphic novels with:

- **Graph-based storytelling** - Panels as nodes, transitions as edges with conditions
- **Layered panels** - Images, audio, video, speech bubbles, and hotspots
- **Multilingual support** - Localized assets, text, and fallback logic
- **Multiple output formats** - Mobile portrait, big-screen landscape, A4/US print, 16:9 video
- **Interactive features** - Decision trees, variables, conditional content
- **Accessibility first** - ARIA labels, keyboard navigation, captions, transcripts
- **Monetization ready** - Flexible paywalls, entitlements, and age gates

## Schema Files

### Current Version: 1.5.0

- **Location**: [`1.0/panelwave.schema.json`](./1.0/panelwave.schema.json) (minor versions are additive and ship in-place within the `1.0/` major-version folder; see [Schema Evolution](#schema-evolution))
- **Schema ID**: `https://panelwave.org/schema/1.0/panelwave.schema.json`
- **Draft**: JSON Schema 2020-12

## Quick Start

### Basic Manifest Structure

A PanelWave manifest requires three top-level sections:

```json
{
  "panelwave": {
    "version": "1.0.0",
    "schema": "https://panelwave.org/schema/1.0/panelwave.schema.json"
  },
  "meta": {
    "id": "my-comic",
    "title": { "en-US": "My Comic" },
    "locales": ["en-US"],
    "default_locale": "en-US"
  },
  "chapters": [
    {
      "id": "ch-01",
      "panels": {
        "panel-01": {
          "layers": [
            {
              "id": "bg",
              "assetId": "img-background"
            }
          ]
        }
      },
      "graph": {
        "entry": "panel-01",
        "edges": []
      }
    }
  ]
}
```

### Validating Your Manifest

#### Using Node.js with AJV

```bash
npm install ajv ajv-formats
```

```javascript
const Ajv = require('ajv');
const addFormats = require('ajv-formats');
const schema = require('./schema/1.0/panelwave.schema.json');
const manifest = require('./my-comic/panelwave.json');

const ajv = new Ajv({ strict: false, allErrors: true });
addFormats(ajv);
const validate = ajv.compile(schema);

if (validate(manifest)) {
  console.log('✓ Manifest is valid');
} else {
  console.error('✗ Validation errors:', validate.errors);
}
```

#### Using the PanelWave CLI (Recommended)

```bash
# Install CLI
npm install -g @panelwave/cli

# Validate a manifest
panelwave validate ./my-comic/panelwave.json

# Validate against a specific schema version
panelwave validate ./my-comic/panelwave.json --schema 1.0.0
```

## Schema Structure

### Top-Level Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `panelwave` | Object | ✓ | Format version and schema reference |
| `meta` | Object | ✓ | Work metadata (title, creators, locales) |
| `chapters` | Array | ✓ | Chapter definitions with panels and flow graph |
| `assets` | Object |  | Asset catalog with images, audio, video |
| `variables` | Object |  | Variable definitions for conditional logic |
| `settings` | Object |  | Global UI/UX defaults and preload settings |
| `localization` | Object |  | Translation workflow state: locales, entries, glossary (since 1.5; rendering consumers may ignore it) |
| `extras` | Object |  | Additional content (covers, character sheets, bonus art) |
| `paywall` | Object |  | Entitlement and paywall rules |
| `tracking` | Object |  | Analytics and event tracking configuration |
| `ui` | Object |  | Branding and UI customization |

### Key Definitions

#### Output Formats

PanelWave supports multiple output targets:

- `flex-landscape` - Responsive landscape (default web)
- `mobile-portrait` - Mobile/phone portrait
- `bigscreen-landscape` - Desktop/TV landscape
- `a4-portrait` / `a4-landscape` - A4 print formats
- `us-portrait` / `us-landscape` - US Letter print formats
- `video-16-9` - Video export (1080p/4K)

#### Localization

All user-facing strings support the `LocalizedString` pattern:

```json
{
  "title": {
    "en-US": "Night Shift",
    "de-DE": "Nachtschicht",
    "fr-FR": "Équipe de nuit"
  }
}
```

Locales follow BCP-47 format (`en-US`, `de-DE`, `ja-JP`, etc.).

#### Graph-Based Flow

Chapters use a graph structure where panels are nodes:

```json
{
  "graph": {
    "entry": "panel-01",
    "edges": [
      {
        "from": "panel-01",
        "to": "panel-02",
        "transition": {
          "type": "slide",
          "dir": "left",
          "durationMs": 500
        }
      },
      {
        "from": "panel-02",
        "to": "panel-03-left",
        "condition": { "==": [{ "var": "choice" }, "left"] }
      },
      {
        "from": "panel-02",
        "to": "panel-03-right",
        "condition": { "==": [{ "var": "choice" }, "right"] }
      }
    ]
  }
}
```

#### Variables and Conditions

Define variables with scope and persistence:

```json
{
  "variables": {
    "definitions": [
      {
        "id": "user.age",
        "type": "integer",
        "scope": "global",
        "default": 18,
        "readOnly": true
      },
      {
        "id": "prefs.speech",
        "type": "boolean",
        "scope": "persistent",
        "default": true
      },
      {
        "id": "chapter.choice",
        "type": "enum",
        "enum": ["door-left", "door-right", "none"],
        "scope": "chapter",
        "default": "none"
      }
    ]
  }
}
```

Conditions use [JSON Logic](https://jsonlogic.com/) syntax:

```json
{
  "condition": {
    "and": [
      { ">=": [{ "var": "user.age" }, 16] },
      { "==": [{ "var": "prefs.explicit" }, true] }
    ]
  }
}
```

#### Panel Structure

Panels support layers, speech bubbles, hotspots, and media:

```json
{
  "panels": {
    "panel-01": {
      "title": { "en-US": "The Alley" },
      "layers": [
        {
          "id": "background",
          "assetId": "img-alley",
          "z": 0
        },
        {
          "id": "character",
          "assetId": "img-detective",
          "z": 10,
          "parallaxDepth": 0.3
        }
      ],
      "speechBubbles": [
        {
          "id": "sb-1",
          "characterId": "detective-mira",
          "text": { "en-US": "Something's not right here." },
          "shape": { "type": "ellipse", "x": 0.6, "y": 0.2, "w": 0.25, "h": 0.15 }
        }
      ],
      "hotspots": [
        {
          "id": "door-left",
          "shape": { "type": "rect", "x": 0.1, "y": 0.4, "w": 0.15, "h": 0.3 },
          "label": { "en-US": "Left door" },
          "action": {
            "type": "goTo",
            "to": "panel-02",
            "mutations": [
              { "op": "set", "var": "chapter.choice", "value": "door-left" }
            ]
          }
        }
      ],
      "audio": [
        {
          "assetId": "sfx-rain",
          "role": "ambient",
          "loop": true,
          "volume": 0.3
        }
      ]
    }
  }
}
```

#### Assets Catalog

Register all assets with variants for different formats:

```json
{
  "assets": {
    "base": {
      "imageBase": "https://cdn.example.com/comic/images/",
      "audioBase": "https://cdn.example.com/comic/audio/"
    },
    "catalog": [
      {
        "id": "img-alley",
        "category": "image",
        "variants": [
          {
            "src": "alley-2048.avif",
            "mime": "image/avif",
            "w": 2048,
            "h": 1536,
            "density": 2
          },
          {
            "src": "alley-1024.jpg",
            "mime": "image/jpeg",
            "w": 1024,
            "h": 768,
            "density": 1
          }
        ],
        "alt": { "en-US": "Dark city alley in the rain" }
      },
      {
        "id": "sfx-rain",
        "category": "audio",
        "role": "sfx",
        "variants": [
          {
            "src": "rain-loop.mp3",
            "mime": "audio/mpeg",
            "loop": true
          }
        ]
      }
    ]
  }
}
```

## Schema Reference

### Common Types

The schema defines several reusable types:

- **`Identifier`** - Alphanumeric IDs with dots, hyphens, colons (1-200 chars)
- **`LocaleCode`** - BCP-47 locale codes (`en-US`, `de-DE`)
- **`Uri`** - Absolute URI strings
- **`ColorHex`** - CSS hex colors (`#FF0000`, `#F00`)
- **`Timestamp`** - ISO 8601 date-time strings
- **`NormalizedNumber`** - Values between 0 and 1 (0%, 100%)
- **`NormalizedRect`** - Rectangle with `{x, y, w, h}` all normalized
- **`JsonLogic`** - JSON Logic expressions for conditions

### Transitions

Seven transition types with optional easing:

```json
{
  "type": "slide",           // none, cut, fade, slide, zoom, push, cover
  "dir": "left",             // left, right, up, down
  "durationMs": 500,
  "easing": "ease-in-out"    // linear, ease, ease-in, ease-out, ease-in-out
}
```

### Extension Fields

Custom properties prefixed with `x-` are allowed throughout:

```json
{
  "x-custom-editor-state": { "collapsed": false },
  "x-plugin-data": { "pluginId": "mega-zoom", "config": {} }
}
```

## Version History

### 1.5.0 (Current)

Additive, backward-compatible with 1.4.0 — existing manifests remain valid unchanged. Adds the authoring metadata a **portable work archive** needs, so a whole work (manifest + assets) can be exported as one zip and re-imported into another team or PanelWave system without losing its asset-library structure or translation state. Both additions are authoring metadata: rendering consumers may ignore them.

- **`assets.folders`** (optional array of `AssetFolder`: `id`, `name`, optional `parentId`, `order`): the asset-library folder tree.
- **`folderIds`** on every asset catalog item (optional, unique `Identifier[]`): the folders an asset is filed under (n:m).
- **`localization`** (optional top-level block): translation workflow state — `locales` (`code`, `isDefault`, `isActive`), `entries` (`key`, `defaultText`, `category`, `context`, `isStale`, per-locale `values` with `text` and a `machine` flag) and `glossary` (`term`, `caseSensitive`, per-locale `translations`).

### 1.4.0

Additive, backward-compatible with 1.3.0 — existing manifests remain valid unchanged. Introduces the **infinite canvas**: a chapter's panels can be placed on one continuous world-space plane and read via an authored camera that travels along the existing graph (Scott McCloud's "infinite canvas"; a single-column layout yields a webtoon-style vertical experience from the same model). Concept & integration study: `_spec_cms/infinite_canvas/` in the umbrella repo.

- **`Chapter.canvas`** (optional `CanvasLayout`): world-space `placements` (map of panel id → `CanvasPlacement`), optional `background` (color / tiled asset), `camera` policy (`fitMode`, `overview.maxZoomOut`, `freeRoam: off | between-moves | always`, `bounds`), and presentational `decorations`. **World units: 1 unit = 1 CSS pixel at zoom 1.0**; coordinates are unbounded, negatives allowed. Panel-internal coordinates (layers, bubbles, hotspots) are untouched — placements only frame the panel on the plane.
- **`CanvasPlacement`**: `x`/`y`/`w`/`h` (world units), optional `z`, `r` (degrees), `origin`, `enterFraming` (panel-relative `NormalizedRect` the camera frames on arrival), `revealMode` (`always` | `on-approach` | `on-visit` — spoiler protection at overview zoom).
- **`Edge.cameraMove`** (optional `CameraMove`): camera travel for canvas view — `path` (`direct` | `arc` | `waypoints` + `WorldPoint[]`), `zoomProfile` (`hold` | `pull-back` | `dive`), `durationMs`, `easing`, `reducedMotionFallback` (a `Transition` used when the reader prefers reduced motion). Inheritance mirrors `defaultTransition`: edge → `settings.outputPresets[format].defaultCameraMove` → built-in direct/hold/800ms/ease-in-out.
- **`FormatPreset.canvasView`** (boolean, default `false`) and **`FormatPreset.defaultCameraMove`**: canvas view renders only when the active format enables it **and** the chapter has a `canvas`; otherwise players fall back to panel view unchanged. Print formats ignore `canvas` — `pages` remain the print model.
- **The graph remains the trail**: `canvas` adds *where panels sit*, never *what comes next*. Branching, conditions, variables, paywalls and analytics are unaffected; screen readers keep using the graph linearization (canvas is presentational).
- Semantic rules (application-level, beyond JSON Schema): every `placements` key must exist in the chapter's `panels` (CLI: error); every panel reachable from `graph.entry` should have a placement when `canvas` is present (CLI: warning); very large canvases (> 150 placed panels) get a performance warning.

### 1.3.0

Additive, backward-compatible with 1.2.0 — existing manifests remain valid unchanged. Introduces **reusable style presets** ("CSS classes" for the manifest): shared styling is defined once under `settings.typography` and referenced by name, so restyling a whole album means editing one preset instead of hundreds of inline blocks.

- **`settings.typography.textStyles`** (map of name → `TextStyle`): named text style presets. `TextLayer` gains **`styleRef`** referencing a key. Resolution cascade: work typography defaults → preset → inline `style` (inline fields win field-by-field).
- **`settings.typography.balloonPresets`** (map of name → `BalloonConfigOverride`): named balloon presets. `SpeechBubble` gains **`styleRef`** referencing a key, slotted into the existing balloon cascade: work `balloon_config` → character `balloonConfig` → **preset** → inline `balloonConfig` (inline fields win).
- `TextLayer.style` is now the shared `$defs/TextStyle` definition (same shape as before; also used by the preset map).
- **`balloonType: "narrator"`**: new tenth balloon type — a sharp-cornered caption box for narrator/caption text (available in `BalloonConfig` and `BalloonConfigOverride` alongside `normal`, `rectangle`, `cutTop`, `cutTopRight`, `cutTopLeft`, `thought`, `shout`, `whisper`, `connector`).
- **Unknown `styleRef`s are ignored** by consumers (render as if unset); authoring tools should warn about dangling references.
- **The speech toggle is now implicit**: every speech bubble is inherently subject to the reader's global speech toggle (initial state: `settings.ui.speechDefault`); the player ANDs the toggle on top of any `visibleIf`. Manifests must no longer carry per-bubble `visibleIf: {"var": "prefs.speech"}` boilerplate — `visibleIf` is reserved for actual story logic (e.g. the clue bubbles in sample 09). Previously, forgetting the boilerplate on one bubble silently made it immune to the toggle.
- `tracking.eventWhitelist`: added `work_complete` — emitted by the player when the reader reaches an end panel (a panel with no outgoing edges); analytics consumers treat it as a completed read-through.

### 1.2.0

Relaxing, backward-compatible with 1.1.0 — existing manifests remain valid unchanged. This release lets exporters write leaner manifests by omitting derivable/default values ("free wins"):

- **`mime` is now optional** on `ImageVariant`, `AudioVariant`, `VideoVariant`, `SubtitleVariant` and `VectorVariant`. When omitted, consumers derive the MIME type from the `src` file extension: `png`/`jpg`/`jpeg`/`webp`/`gif`/`avif`/`svg` → `image/*`, `mp3` → `audio/mpeg`, `m4a` → `audio/mp4`, `ogg`/`wav` → `audio/*`, `mp4`/`webm` → `video/*`, `m3u8` → `application/vnd.apple.mpegurl`, `pdf` → `application/pdf`, `vtt` → `text/vtt`, `srt` → `application/x-subrip`. Query strings/fragments are ignored. `mime` **must** still be written when the `src` has no recognizable extension or the actual type differs from the derivation.
- **`Edge.transition` inheritance clarified**: an edge without a `transition` inherits the active output format's `settings.outputPresets[format].defaultTransition`; if no preset defines one, the player falls back to a plain cut. Exporters should only write transitions that differ from the format default.
- **Export discipline for schema defaults** (no schema change, now the documented convention): exporters should omit values equal to schema defaults — notably `Panel.shareable: true`, `ExtraBlock.shareable: true`, and placement `z: 0` / `r: 0`. The umbrella repo's `normalize-samples.js` applies these rules to existing manifests.

### 1.1.0

Additive, backward-compatible with 1.0.0 — existing manifests remain valid unchanged.

- `VideoLayer`: added `playMode` (`once` | `loop` | `pingpong` | `loop-from`, default `once`), `loopFromMs` (loop re-entry point in ms, only meaningful for `playMode: "loop-from"`), and `startMode` (`on-view` | `on-hover` | `on-click`, default `on-view`).
- `VideoVariant`: added `direction` (`forward` | `reverse`, default `forward`) to mark pre-rendered time-reversed encodes used for smooth ping-pong playback.
- `settings.ui`: added work-level defaults `videoPlayModeDefault`, `videoStartModeDefault`, `videoMutedDefault`, cascading to per-layer overrides (same pattern as balloon config).
- **Legacy field mapping** (`autoplay`/`loop` → `startMode`/`playMode`): the schema 1.0 fields `VideoLayer.autoplay` and `VideoLayer.loop` remain valid and are not deprecated. Precedence when both old and new fields are present:
  - `loop: true` is interpreted as `playMode: "loop"` **only when `playMode` is absent**. If `playMode` is present, it always wins.
  - `autoplay: true` is interpreted as `startMode: "on-view"`; explicit `autoplay: false` is interpreted as `startMode: "on-click"` — **only when `startMode` is absent**. If `startMode` is present, it always wins.
  - Consumers (player, CMS) apply this mapping at read time; the CMS only writes the new fields going forward.
- `loopFromMs` constraint `startAtMs <= loopFromMs < asset.durationMs` is a semantic rule documented in the field description — JSON Schema cannot express this cross-field/cross-asset constraint structurally; validate it in application code.
- `AssetCatalogItemVideo`: added optional `poster` (`VideoPoster`: required `src`, optional `mime` matching `^image\/`, optional `w`/`h`) — a poster/preview frame shown before playback starts, e.g. for click-to-play and reduced-motion presentations.
- `VideoLayer`: added `controls` (boolean, default `false`) to show native video controls.
- `tracking.eventWhitelist`: extended with `videoPlay`, `videoPause`, `videoEnded`, `videoLoop` (camelCase, matching the events the player emits — note the enum's older entries are snake_case; that pre-existing inconsistency is unchanged here).

### 1.0.0

- Initial public release
- Support for panels, chapters, graph-based flow
- Asset catalog with image/audio/video/subtitle variants
- Variable system with JSON Logic conditions
- Multilingual support with locale fallbacks
- Output format presets for web, print, and video
- Paywall and entitlement definitions
- Tracking and analytics configuration
- Plugin system support
- Accessibility hints and features

## Related Resources

### Official Repositories

- **Player**: [github.com/panelwave/player](https://github.com/panelwave/player) - Open-source Angular player
- **Types**: [`@panelwave/types`](https://github.com/panelwave/packages/tree/master/packages/types) — Full TypeScript interfaces for type-safe development
- **CLI**: [`@panelwave/cli`](https://github.com/panelwave/packages/tree/master/packages/cli) — Validate, bundle, diff, and upgrade manifests from the command line
- **Examples**: Sample manifests currently live in the umbrella workspace (`_spec/samples/`, validated by `validate-samples.js`); a public samples repository is tbd

### Documentation

- **Website**: [panelwave.org](https://panelwave.org) tbd
- **Specification**: [docs.panelwave.org/spec](https://docs.panelwave.org/spec) tbd
- **Plugin API**: [docs.panelwave.org/plugins](https://docs.panelwave.org/plugins) tbd
- **Migration Guide**: [docs.panelwave.org/migration](https://docs.panelwave.org/migration) tbd

### Community

- **Discord**: [discord.gg/panelwave](https://discord.gg/panelwave) tbd
- **Forum**: [community.panelwave.org](https://community.panelwave.org) tbd
- **Twitter**: [@panelwave](https://twitter.com/panelwave) tbd

## Contributing

This schema repository is the source of truth for the PanelWave format. We welcome:

- **Bug reports** - Schema validation issues or incorrect definitions
- **Enhancement proposals** - RFC process for new features
- **Clarifications** - Improve descriptions and examples

### Proposing Changes

1. Open an issue describing the problem or enhancement
2. For breaking changes, create an RFC in `/rfcs` directory
3. Discuss with maintainers and community
4. Submit a pull request with schema updates and tests
5. Update version following semantic versioning

### Schema Evolution

- **Patch versions** (1.0.x) - Clarifications, non-functional changes
- **Minor versions** (1.x.0) - Backward-compatible additions
- **Major versions** (x.0.0) - Breaking changes

Deprecated fields are marked with `"deprecated": true` and remain for at least one major version.

## Architecture Principles

PanelWave's schema design follows these principles:

### Simplicity

- Human-readable and writable JSON
- String IDs instead of array indices
- Optional fields only when truly optional

### Extensibility

- `x-` namespace for custom properties
- Plugin system for specialized content
- Forward-compatible with unknown fields

### Robustness

- Fallback mechanisms for locales and assets
- Graceful degradation via conditions
- Memory budgets and performance hints

### Openness

- Public schema in version control
- No proprietary dependencies
- Export-friendly structure

## Use Cases

### Web Comics with Motion

Add parallax layers, zoom animations, and ambient audio to create cinematic web comics that adapt from mobile to desktop.

### Branching Interactive Fiction

Build decision trees with variables and conditions. Readers make choices that shape the narrative path.

### Multilingual Publishing

Author once, publish in multiple languages with localized text, audio, and assets. Automatic fallback handling.

### Print and Video Export

Single source for web, print (A4/US), and video (16:9) outputs with format-specific layouts.

### Educational Content

Interactive training materials with decision scenarios, assessments, and detailed analytics on learner paths.

### Premium Content Distribution

Gate content by chapter, panel, or extras. Support subscriptions, one-time purchases, and age restrictions.

## License

### Schema Files

The JSON Schema files in this repository are licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to:
- **Share** - Copy and redistribute the material
- **Adapt** - Remix, transform, and build upon the material

Under the following terms:
- **Attribution** - Give appropriate credit to PanelWave

### Documentation

All documentation (including this README) is licensed under CC BY 4.0.

### Implementation Notice

This schema does not dictate licensing for:
- Content created using PanelWave format (your work remains yours)
- Software that reads or writes PanelWave manifests
- Player implementations (the official player is MIT-licensed)

## Support

- **Documentation**: [docs.panelwave.org](https://docs.panelwave.org)
- **Issues**: [github.com/panelwave/schema/issues](https://github.com/panelwave/schema/issues)
- **Email**: schema@panelwave.org
- **Discord**: [discord.gg/panelwave](https://discord.gg/panelwave)

---

**PanelWave Schema** - Open format for dynamic graphic novels  
Version 1.5.0 | [panelwave.org](https://panelwave.org) | Made for creators
