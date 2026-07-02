# PanelWave JSON Schema

**Official JSON Schema definitions for the PanelWave dynamic graphic novel format.**

[![Schema Version](https://img.shields.io/badge/schema-1.1.0-blue.svg)](./1.0/panelwave.schema.json)
[![JSON Schema](https://img.shields.io/badge/json--schema-2020--12-green.svg)](https://json-schema.org/draft/2020-12/schema)
[![License](https://img.shields.io/badge/license-CC%20BY%204.0-lightgrey.svg)](LICENSE)

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

### Current Version: 1.1.0

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

### 1.1.0 (Current)

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

- **Player**: [bitbucket.org/jenshoppe/panelwave-player](https://bitbucket.org/jenshoppe/panelwave-player) - Open-source Angular player
- **Types**: [`@panelwave/types`](https://bitbucket.org/jenshoppe/panelwave-packages/src/master/packages/types) — Full TypeScript interfaces for type-safe development
- **CLI**: [`@panelwave/cli`](https://bitbucket.org/jenshoppe/panelwave-packages/src/master/packages/cli) — Validate, bundle, diff, and upgrade manifests from the command line
- **Examples**: [tbd](tbd) - Sample manifests

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
- Player implementations (the official player is MIT/Apache-2.0)

## Support

- **Documentation**: [docs.panelwave.org](https://docs.panelwave.org)
- **Issues**: [GitHub Issues](https://github.com/panelwave/schema/issues)
- **Email**: schema@panelwave.org
- **Discord**: [discord.gg/panelwave](https://discord.gg/panelwave)

---

**PanelWave Schema** - Open format for dynamic graphic novels  
Version 1.0.0 | [panelwave.org](https://panelwave.org) | Made for creators
