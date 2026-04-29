# Studio Player — Case Study

> Multi-mode media player for WordPress. Native Bricks, Gutenberg, Divi, and Elementor integrations with **true cross-page continuous playback**.

🚧 **Pre-release.** Source code lives in a private repo. This page documents the architecture, engineering decisions, and screenshots.

---

## The problem

WordPress has plenty of audio player plugins. Almost all of them have one of three limitations:

- **Single-mode.** Most are audio-only. If a band wants their site to play audio *and* embed Spotify *and* host a podcast feed *and* show a YouTube reel, that's four plugins, four UIs, four configurations.
- **Single-builder.** Sonaar covers Elementor and Gutenberg. FilesJ does Bricks. Nobody covers all four major page builders with native integrations.
- **Hard reload kills the audio.** Click a link to another page → music stops. The "continuous playback" features other plugins advertise only work if the active theme already implements SPA-style navigation (barba.js, jquery.pjax) — almost no Bricks or Divi sites do.

Studio Player solves all three.

## What I built

### Six unified modes
One element, six playback contexts:

```
🎵 Audio playlist   📻 Live radio (Icecast)   🎙 Podcast (RSS)
🟢 Spotify embed    🎶 Apple Music embed     🎬 Video (YouTube/Vimeo/MP4)
```

### Eight position layouts
```
↓ Sticky bottom (full width)    ↘ Sticky bottom-right
↙ Sticky bottom-left            ↗ Sticky top-right
↖ Sticky top-left               ▭ Inline (flow with content)
🎨 Album (large hero layout)    ▫ Mini (play/pause only)
```

### Native integrations for all four major editors

| Editor | Surface |
|--------|---------|
| **Bricks** | Native element under Media category, all 6 modes, all 8 positions, full Style tab |
| **Gutenberg** | Native block, mode-conditional InspectorControls, native ColorPalette pickers |
| **Divi** | Native module, settings modal toggles per mode, color-alpha pickers |
| **Elementor** | Native widget under "Studio Grid" category, Style tab with live-preview color updates |
| **Shortcode** | `[studio_player]` works in any context (classic editor, Oxygen, etc.) |

### True cross-page continuous playback
The differentiating feature. The plugin ships its own AJAX navigation layer — internal links are intercepted, the destination is fetched, the page content is swapped, and the player node lives outside the swap region. Audio actually survives navigation. No theme requirements, no SPA setup.

This is what no other Bricks-targeting audio player does.

### Listener-side polish
- Playback speed (0.5×–2×)
- Shuffle + 3-state repeat (none / all / one)
- Per-track progress memory (resume where you stopped)
- Tracklist search (appears at >5 tracks)
- Adaptive accent color extracted from cover art
- Optional waveform visualization
- Per-track download button
- Per-track share menu (Web Share API on mobile, popover on desktop)
- Per-track Buy bridge (works with WooCommerce, EDD, Gumroad, Bandcamp — anything URL-based)
- Podcast Subscribe row (Apple Podcasts, Spotify, YouTube, RSS)

---

## Engineering decisions worth talking about

### Single source of truth across four editors

All four editor integrations delegate server-side rendering to one place: the plugin's shortcode renderer. The Bricks element doesn't have its own render path. The Gutenberg block doesn't either. They each have their own attribute schema, their own UI controls — but when it's time to produce HTML, they all map their attributes onto the canonical shortcode shape and call into one function.

This means:

- Adding a new option once propagates to every editor automatically
- Bug fixes in the render layer land everywhere simultaneously
- The output is byte-identical regardless of which editor placed the player

The cost: each editor integration has its own attribute-mapping translation layer. The benefit: the rendering surface stays small, well-tested, and impossible to drift out of sync.

### True cross-page playback without a SPA theme

Most "continuous playback" features in the WordPress plugin ecosystem rely on the active theme already implementing SPA-style navigation. If the theme uses barba.js or jquery.pjax, audio survives. If it doesn't, audio dies on every link click.

Studio Player ships its own AJAX navigation layer. Internal `<a>` clicks are intercepted, the destination is fetched, and a configurable content-area selector is replaced in-place. The player wrapper lives in a footer-injected persistent host that lives outside that swap region — so it's never destroyed, and the audio element never reloads.

The layer is conservative. External origins, fragments, modifier-key clicks, downloads, forms, and a configurable URL-exclusion list all defer to native browser navigation. Failures fall back to a full page reload — never silent breakage.

### No build step

The plugin ships ~2,200 lines of vanilla JavaScript with no bundler, no transpiler, no Node dependency. A WordPress plugin shouldn't require Node.js to clone and edit. Browser support for everything used (Web Audio, MediaElement, IntersectionObserver, MutationObserver, Web Share, FileReader, `fetch`) is broad. A build step would be dead weight.

### In-house ID3 reader

Visitor-uploaded MP3s get their title, artist, album, and cover art automatically extracted. This is bundled — no `jsmediatags` CDN dependency, no fingerprintable third-party request. Roughly 100 lines of ID3v2.3/v2.4 parsing in the engine, defensive about encoding (Latin-1, UTF-16 with BOM, UTF-8) and about malformed files.

### Lazy-loaded heavy dependencies

Waveform visualization uses `wavesurfer.js` (~70KB). It loads only when the toggle is on, only on the page where it's needed. Players without waveform mode pay zero cost. Self-hosters can override the URL to keep the dependency same-origin.

---

## Tech stack

- **PHP 8.0+ / WordPress 6.4+**
- **Vanilla JavaScript, no build step** — ~2,200 lines, single file
- **CSS custom properties** for theming
- **REST API + admin-ajax** hybrid
- **Lazy-loaded** wavesurfer.js for waveform mode
- **Localized via WordPress i18n** — 70+ strings translated

---

## Architecture

For the high-level engineering deep-dive — the single-source-of-truth pattern, the continuous playback flow, the security and performance considerations — see [architecture.md](architecture.md).

For where the project is going, see [roadmap.md](roadmap.md).

---

## Status

**v1.6.0.** All four editor integrations shipped. Continuous playback layer shipped. The 1.7 polish pass focuses on production QA across real Bricks, Gutenberg, Divi, and Elementor sites.

---

## Screenshots

> Screenshots TK — capturing during the v1.7 polish pass.

```
./screenshots/01-album-hero.png         # Album hero layout
./screenshots/02-sticky-bottom.png      # Sticky bottom bar collapsed + expanded
./screenshots/03-bricks-element.png     # Bricks element in the canvas
./screenshots/04-gutenberg-block.png    # Gutenberg block with InspectorControls
./screenshots/05-elementor-widget.png   # Elementor widget panel
./screenshots/06-podcast-subscribe.png  # Podcast mode with subscribe row
./screenshots/07-waveform.png           # Waveform visualization
./screenshots/08-continuous-playback.png # Cross-page playback in action (GIF)
```

---

## About

Built by **Mark Carranza** ([Studio Grid](https://github.com/studiogrid)). Part of the Studio Grid family alongside:

- **StudioDarkroom** — premium WordPress media library with folders, smart filters, and focal points
- **Studio Media** — Bricks-native gallery, video, Lottie, and filter elements
- **StudioDashboard** — modern WordPress admin dashboard with built-in analytics

This is a private commercial plugin. Source code is not public. This page exists to document the engineering for prospective collaborators and employers.
