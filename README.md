# scroll-3d

A Claude Code / Claude Cowork plugin that turns a one line brief into a deployed,
scroll driven 3D website. Live URL out, source written back to your connected folder.

## Install

Clone the repo, add the marketplace from its `scroll3d-plugin` folder, then install
the plugin:

```
git clone https://github.com/saloxtiktok/3D-website.git
/plugin marketplace add ./3D-website/scroll3d-plugin
/plugin install scroll-3d@scroll3d-plugin
```

The marketplace manifest lives at `scroll3d-plugin/.claude-plugin/marketplace.json`,
not at the repo root, so adding the repo directly as `owner/repo` fails with
"Marketplace file not found".

## Requirements

| Requirement | Needed for |
|---|---|
| `ffmpeg` on PATH | frame extraction, `video-scroll-effect` only |
| Higgsfield MCP + credits | generated footage, roughly 54 credits per 1080p clip |
| `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` | deploy step |

None of these are hard requirements for every build. A site made only of the
real time WebGL techniques needs no ffmpeg, no Higgsfield and no credits. If the
Cloudflare deploy fails or is unavailable, the source is still written out to the
connected folder.

## How it works

`site-planner` is the entry point and the only skill that makes structural
decisions. It runs intake, either reasons from the product or clones a reference
site's structure, produces a JSON site plan, shows one approval checkpoint
(palette, section list, per section asset plan, total credit estimate), then hands
each section to whichever technique skill owns it.

Seven technique skills, one per rendering approach:

| Skill | Method | Note |
|---|---|---|
| `video-scroll-effect` | canvas frame scrub, roughly 180 JPGs | the classic effect, not WebGL |
| `3d-scene-effect` | Three.js, scroll driven | also the zero cost `image-plane` fallback |
| `pointer-follow-effect` | Three.js, cursor driven | static pose on touch devices |
| `click-navigate` | Three.js hotspots to camera waypoints | works on touch and desktop |
| `physics-play` | Three.js plus cannon es | heaviest, at most one per site |
| `hybrid-2d3d` | editorial 2D layout, inline 3D object | the only non pinned technique |
| `cursor-trail` | canvas 2D particle trail | atmospheric, never load bearing |

Three support skills: `shared-scroll-engine` (the project shell, base stylesheet
and shared JS utilities), `asset-generator` (Higgsfield generation and user asset
ingestion), `scroll-style-helper` (the four taste calls specific to scroll sites).

Three agents: `frame-pipeline` (polls renders, runs ffmpeg), `reference-analyzer`
(fetches and structures a reference site), `build-reviewer` (pre deploy QA, blocks
on hard fails).

## Project scaffold

Every generated project shares one scaffold, owned by `shared-scroll-engine`:

```
index.html        <- canonical shell: all library tags, all section registries
styles.css        <- base.css, then one appended fragment per technique used
choreography.js   <- shared scroll, pointer and color math
<engine>.js       <- one per technique the plan actually uses
frames/           <- video-scroll-effect only
```

Technique skills never carry their own `index.html`. Whichever skill runs first
copies the shell, and later skills append into it. This is deliberate: divergent
per skill shells were how a build could end up missing the Three.js tag entirely,
depending only on which technique happened to be built first.

Three.js is pinned to r160 because that is the last release shipping
`build/three.min.js`; the UMD build was removed in r161. Bumping it means
migrating every engine to ES modules and an import map.

## Honesty guarantees

The plugin is built around not overstating what it produced:

* `asset-generator` must return `generation_failed: true` when video was planned
  but not produced, including when generation was never attempted.
* `video-scroll-effect` stops rather than silently rerouting a failed section to a
  cheaper technique.
* `build-reviewer` returns a `plan_deviations` array for any section whose built
  technique differs from the approved plan.
* `site-planner` must relay every deviation verbatim under a `⚠️ Plan deviations`
  heading before declaring a build done.
* Every build writes a `DESIGN.md` with a plan versus build table. A missing or
  empty one is a hard fail.

## License

Not yet specified.
