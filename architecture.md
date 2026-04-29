# Roadmap

A high-level look at where Studio Player is and where it's headed.

> Specific feature breakdowns and release timing for upcoming versions live in the private repo. This page covers the trajectory.

---

## Now — v1.7 (production polish)

The polish pass before public release. Focus areas:

- Production QA across real Bricks, Gutenberg, Divi, and Elementor sites
- Edge cases in the AJAX navigation layer — third-party plugin compatibility, lazy-loaded image rebinding, analytics re-initialization
- Compatibility matrix across popular Bricks themes and Divi child themes
- Performance pass — first-paint cost, lazy-load tuning, bundle size audit
- Documentation, screenshots, GIF demos
- Public release checklist

This is the version that ships first. Everything below depends on it landing solid.

---

## Soon — v2

The next major version moves Studio Player from **"the best Bricks-native audio player"** into territory that competes with category leaders on their own ground. Themes the v2 release will cover:

- **Beat marketplace tooling** — features that turn the player into a sales surface, not just a playback surface
- **Anti-piracy** — protections that make the player suitable for selling original audio, not just streaming free media
- **Conversion tooling** — features that nudge listeners toward a purchase decision while they're listening

Each theme is its own discrete piece of work. Specific feature names, ordering, and implementation details aren't being shared publicly until each piece ships. The shape will become visible at release.

This is the version that justifies a higher price tier.

---

## Later — beyond v2

Possibilities being scoped for future major versions, in rough order of interest:

- **Statistics and analytics** — usage signals, top tracks, listener geography
- **Bricks query-loop integration** — auto-build playlists from a CPT, tag, or category. Same pattern for Divi loops and Gutenberg query blocks.
- **Dynamic-data bridges** — pull track data from custom fields (ACF, Metabox, JetEngine)
- **Pre-built editor templates** — drop-in album page, podcast show page, radio station page templates for Bricks, Gutenberg, Divi, and Elementor
- **Service Worker pre-caching** — pre-fetch the next track in a playlist for instant skip
- **Multi-radio and multi-podcast** — composite players that aggregate multiple feeds in one element

These are the conversations happening internally. None are committed; some won't ship; one or two might land sooner than others depending on how customers actually use v1 and v2.

---

## Won't ship — explicit non-goals

Worth saying out loud so the conversation doesn't keep coming up:

- **Native video player features** (chapters, captions, video gallery) — Studio Player stays focused on audio. The WordPress video space is dominated by Presto Player; competing there would dilute the "best Bricks audio player" wedge. The existing video mode covers basic embed needs and stays scoped to that.
- **Hosted SaaS** — the plugin runs on the customer's WordPress site. There's no central server, no cloud media library, no multi-site analytics service. Not now, not later.
- **Mobile app** — out of scope.
- **Page builder as the management surface** — native integrations ship today for Bricks, Gutenberg, Divi, and Elementor (and that surface will keep growing). What stays out of scope: moving the configuration / preset / track-management workflow into the page-builder canvas as a primary surface. The shortcode + element/block/module/widget pattern is the right abstraction.

---

## Long-term commitment

Studio Player is a multi-year project. The plan is to keep shipping incrementally — v1 establishes the foundation across four editors, v2 extends the surface for commercial audio sales, future versions sharpen the niche as customer needs become legible.

The Studio Grid family is built on the same long-arc thinking: each plugin is its own focused tool, designed to ship for years rather than as a one-time release.

For licensing inquiries, evaluation copies, or technical conversations: contact via the [Studio Grid GitHub](https://github.com/studiogrid).
