# BetterLinks MCP Launch Email

A production-ready, dark-mode-safe HTML email built with table-based layout, inline CSS and Outlook conditional fallbacks. This repository documents the build as a working experiment in modern HTML email design, where "modern" still means shipping markup that a 2007 rendering engine can parse.

---

## Table of contents

- [What this is](#what-this-is)
- [Why HTML email is its own discipline](#why-html-email-is-its-own-discipline)
- [Repository structure](#repository-structure)
- [Quick start](#quick-start)
- [Anatomy of the email](#anatomy-of-the-email)
- [Core techniques](#core-techniques)
- [CSS class reference](#css-class-reference)
- [Assets](#assets)
- [ESP integration](#esp-integration)
- [Testing checklist](#testing-checklist)
- [Known limitations](#known-limitations)
- [Changelog](#changelog)
- [Contributing](#contributing)
- [License](#license)

---

## What this is

`update-email.html` is a single-file responsive HTML email announcing BetterLinks MCP support. It is a self-contained deliverable: one file, no build step, no dependencies. Paste it into an ESP, swap the merge tags and asset URLs, and send.

The experiment tracks three questions:

1. Can a visually rich email (gradients, rounded cards, a video poster overlay) survive Outlook, Gmail and Apple Mail without degrading into an unreadable mess?
2. Can dark mode be handled deliberately rather than left to each client's auto-inversion algorithm?
3. How much of the layout can be expressed once and reused, given that email CSS has no variables, no flexbox and no reliable cascade?

**Constraints accepted up front**

| Constraint | Consequence |
|---|---|
| No external stylesheets | All critical styling is inline on the element |
| No flexbox or grid | Layout is nested `<table>` elements |
| No `<style>` block in some clients | The `<style>` block only holds progressive enhancement |
| No JavaScript | Every interaction is a link |
| No relative asset paths | Every image must resolve to an absolute hosted URL |

---

## Why HTML email is its own discipline

Web CSS assumes one rendering engine per browser. Email assumes dozens, several of which predate CSS2. Outlook 2007 through 2019 on Windows renders through **Microsoft Word**, not a browser engine. Gmail strips `<style>` blocks in some contexts. Apple Mail aggressively re-colors your palette in dark mode unless you explicitly opt out.

The practical rule this file follows throughout: **inline styles are the source of truth, the `<style>` block is enhancement only.** If a client drops the entire `<style>` block, the email still renders correctly at a fixed 600px width. Responsive behavior and dark mode are the things you lose, not legibility.

---

## Repository structure

```
.
├── update-email.html                    # The email. Single file, no build step.
├── email-assets/
│   ├── betterlinks-logo-white.png       # 340x74 transparent PNG (2x retina)
│   └── mcp-logo.png                     # Model Context Protocol mark
└── README.md
```

---

## Quick start

**Preview locally**

```bash
git clone <repo-url>
cd <repo>
python3 -m http.server 8000
# open http://localhost:8000/update-email.html
```

Browser preview shows you layout and dark mode, but it does **not** show you Outlook. A browser renders the CSS your recipients may never see. Treat local preview as a rough check only and confirm in a real client testing service before sending.

**Ship it**

1. Upload everything in `email-assets/` to your ESP or CDN.
2. Replace each relative `src` with the absolute hosted URL (see [Assets](#assets)).
3. Replace `{{ mirror }}` and `{{ unsubscribe }}` with your ESP's merge tags.
4. Paste the full file into your ESP as a raw HTML template.
5. Send a seed test to Gmail, Outlook and Apple Mail before the real send.

---

## Anatomy of the email

Sections are marked in the source with banner comments so you can jump between them. Top to bottom:

| # | Section | Background | Notes |
|---|---|---|---|
| 1 | Preview text | Hidden | Inbox preview snippet, visually hidden |
| 2 | Preheader bar | `#161a2b` | "Having trouble viewing this email?" plus mirror link |
| 3 | Brand logo header | `#ffffff` | WPDeveloper mark on a white plate |
| 4 | Header and hero | Gradient | BetterLinks logo, NEW FEATURE badge, headline |
| 5 | Intro | `#ffffff` | Body copy |
| 6 | Feature highlights | `#ffffff` | Bulleted list with square dot markers |
| 7 | Video CTA | Poster image | Play button over a YouTube thumbnail |
| 8 | Primary CTA | `#ffffff` | Gradient pill button |
| 9 | Sign off | `#ffffff` | Divider rule and signature |
| 10 | Footer | `#161a2b` | Permission reminder, unsubscribe, copyright |

Everything from section 2 to section 10 sits inside a single `.wrapper` table with `border-radius:18px; overflow:hidden`. That is what gives the dark preheader rounded top corners and the dark footer rounded bottom corners without either needing its own radius.

### The two dark strips

Sections 2 and 10 bookend the email in the same `#161a2b`. Neither carries a `dm-` class. This is deliberate: they are dark by design in both light and dark mode, so letting the dark-mode rules touch them would only introduce drift. The hero gradient follows the same principle.

```html
<!-- Preheader: locked dark, no dm- class -->
<tr>
  <td align="center" bgcolor="#161a2b" style="padding:10px 20px; background-color:#161a2b; font-size:11px; line-height:16px; color:#a9aec2; mso-line-height-rule:exactly;">
    Having trouble viewing this email?
    <a href="{{ mirror }}" target="_blank" class="link-hover" style="color:#d9d3ef; text-decoration:underline;">View it in your browser</a>
  </td>
</tr>
```

### The footer

Each `<p>` repeats its own `font-family`, `font-size`, `line-height` and `color` rather than inheriting from the parent `<td>`. Outlook drops inherited typography on paragraphs, so the repetition is not redundancy, it is the fix.

```html
<tr>
  <td align="center" class="px" bgcolor="#161a2b" style="padding:26px 28px 30px 28px; background-color:#161a2b; ...">
    <p style="margin:0 0 10px 0; font-size:11px; line-height:18px; color:#9e96b3; ...">
      You are receiving this email because you use or subscribed to a WPDeveloper product.
    </p>
    <p style="margin:0 0 10px 0; ...">
      <a href="{{ unsubscribe }}" target="_blank" class="link-hover" style="color:#cbc4dc; text-decoration:underline;">Unsubscribe</a>
    </p>
    <p style="margin:0; ... color:#746d87;">&copy; 2026 WPDeveloper. All rights reserved.</p>
  </td>
</tr>
```

Explicit `margin` on every paragraph replaces default paragraph spacing, which varies enough between clients to visibly break vertical rhythm.

---

## Core techniques

### 1. Color scheme opt-in

Without this declaration, macOS and iOS Mail auto-invert your palette and produce results you did not design. Two meta tags plus a `:root` rule tell the client you have handled dark mode yourself.

```html
<meta name="color-scheme" content="light dark">
<meta name="supported-color-schemes" content="light dark">
```

```css
:root {
  color-scheme: light dark;
  supported-color-schemes: light dark;
}
```

### 2. Two dark-mode syntaxes, not one

Apple Mail and modern clients read the media query. Outlook.com and Windows Mail read the `[data-ogsc]` attribute selector, which they inject themselves. Both blocks are required, and both must be kept in sync when you change a color.

```css
@media (prefers-color-scheme: dark) {
  .dm-card { background-color:#171a23 !important; }
  .dm-text { color:#e9ebf5 !important; }
}

[data-ogsc] .dm-card { background-color:#171a23 !important; }
[data-ogsc] .dm-text { color:#e9ebf5 !important; }
```

### 3. The `keep-light` escape hatch

The dark-mode rules repaint white surfaces. Any element holding a **dark-ink transparent PNG** must opt out or the artwork vanishes against a dark plate. `keep-light` forces the plate to stay white, and `keep-dark-ink` keeps the adjacent text readable on it.

```css
.keep-light    { background-color:#ffffff !important; }
.keep-dark-ink { color:#2a2140 !important; }
```

Used on the WPDeveloper logo plate, the MCP badge and the four connector chips.

The inverse case matters too. The BetterLinks header logo is a **white transparent PNG**, so it needs no plate at all and sits directly on the gradient. Applying `keep-light` there would produce white artwork on a white background. Match the treatment to the artwork, not to habit.

### 4. Auto-link recolor guards

Gmail and Apple Mail detect phone numbers, dates and addresses and restyle them in their own blue. These four rules suppress that, alongside a `format-detection` meta tag.

```css
.im { color:inherit !important; }
a[x-apple-data-detectors] { color:inherit !important; text-decoration:none !important; font-size:inherit !important; ... }
u + #body a { color:inherit; text-decoration:none; }
#MessageViewBody a { color:inherit; text-decoration:none; }
```

### 5. Outlook conditionals and VML

Outlook ignores `max-width`, so an MSO-only fixed-width table wraps the card:

```html
<!--[if mso]>
<table role="presentation" width="600" cellpadding="0" cellspacing="0" border="0"><tr><td>
<![endif]-->
```

Outlook also ignores CSS `background-image`. The video poster is restored with a VML `<v:rect>` fallback, which is the only reliable way to get a background image behind text in Word-rendered Outlook.

```html
<!--[if gte mso 9]>
<v:rect fill="true" stroke="false" style="width:520px; height:250px;">
  <v:fill type="frame" src="https://img.youtube.com/vi/EfYUMTKc9OM/maxresdefault.jpg" color="#161a2b" />
  <v:textbox inset="0,0,0,0"><div>
<![endif]-->
```

The `xmlns:v` and `xmlns:o` namespaces on `<html>` plus the `OfficeDocumentSettings` block in `<head>` are what make this parse. Removing them silently breaks the fallback.

### 6. Responsive without media query support

The layout is fluid by default (`width:100%; max-width:600px`) so it degrades gracefully even where media queries are stripped. The breakpoints are enhancement:

| Breakpoint | Behavior |
|---|---|
| `<= 620px` | Header stacks, padding tightens, headings scale down, CTA goes full width |
| `<= 400px` | Headline and chip text scale down again |

The header stacking is worth noting. It uses `display:inline-block` columns rather than floats, with `.hdr-col { width:100% !important; }` forcing them onto separate lines on mobile. Floats are unreliable in email, inline-block is not.

### 7. Line height discipline

`mso-line-height-rule:exactly` appears on effectively every text cell. Without it, Outlook applies its own line spacing and vertical rhythm drifts across the whole email. It is verbose, and it is not optional.

---

## CSS class reference

**Layout**

| Class | Purpose |
|---|---|
| `.wrapper` | 600px card, full width on mobile |
| `.px` | Horizontal padding, tightens to 20px on mobile |
| `.py-hero` | Hero vertical padding |
| `.h1`, `.h2` | Responsive heading sizes |
| `.hdr-col` | Inline-block header column, stacks on mobile |
| `.hdr-tbl-left`, `.hdr-tbl-right` | Header alignment, centered on mobile |
| `.hdr-badge` | Adds top spacing when the badge stacks |
| `.logo-img` | Logo sizing, 148x32 on mobile |
| `.chip-cell` | Connector chips wrap instead of overflowing |
| `.btn-primary`, `.btn-wrap` | CTA button, full width on mobile |
| `.video-cell`, `.video-inner` | Video poster height, 190px on mobile |
| `.hero-img` | Hero image corner radius on mobile |

**Dark mode**

| Class | Repaints |
|---|---|
| `.dm-bg` | Page background |
| `.dm-card` | Card surface |
| `.dm-panel` | Inset panels and borders |
| `.dm-text` | Body copy |
| `.dm-strong` | Emphasized copy |
| `.dm-muted` | Secondary copy |
| `.dm-border` | Divider rules |
| `.dm-fade` | Gradient fade |
| `.keep-light` | Opts **out**, plate stays white |
| `.keep-dark-ink` | Opts **out**, ink stays dark |

**Interaction**

| Class | Purpose |
|---|---|
| `.btn-primary a:hover` | Button hover state |
| `.link-hover` | Underline on hover |

Hover states are desktop-webmail only. They are a nicety, never a requirement for comprehension.

---

## Assets

| File | Dimensions | Rendered at | Notes |
|---|---|---|---|
| `betterlinks-logo-white.png` | 340x74 | 170x37 | Transparent, white mark, 2x retina |
| `mcp-logo.png` | 40x40 | 20x20 | Model Context Protocol mark |

Logos are exported at **2x the display size** and constrained down in HTML. This is the standard retina approach: the browser or client downsamples cleanly, and the file cost is modest at these dimensions (the BetterLinks logo is 9.2 KB).

Both the `width` and `height` HTML attributes **and** the matching inline CSS are set on every image. Outlook reads the attributes, everything else reads the CSS. Setting only one produces layout shift in whichever client reads the other.

**Absolute URLs are mandatory.** Relative paths like `email-assets/logo.png` work in local preview and fail in every inbox. Replace them before sending:

```html
<!-- Local preview -->
<img src="email-assets/betterlinks-logo-white.png" ...>

<!-- Production -->
<img src="https://your-cdn.com/email-assets/betterlinks-logo-white.png" ...>
```

Always set meaningful `alt` text. Many clients block images by default, so `alt` is what a meaningful share of recipients actually read.

---

## ESP integration

Two merge tags need replacing. Syntax varies by platform, so check your ESP's documentation rather than assuming.

| Placeholder | Purpose |
|---|---|
| `{{ mirror }}` | Hosted browser version of the email |
| `{{ unsubscribe }}` | Per-recipient unsubscribe URL |

A working unsubscribe link is a legal requirement under CAN-SPAM in the US and comparable rules elsewhere, not a design choice. Never ship this file with the placeholder unresolved.

**Before every send**

- [ ] Both merge tags replaced with real ESP syntax
- [ ] All image `src` values switched to absolute hosted URLs
- [ ] Unsubscribe link verified on a real recipient record
- [ ] Mirror link resolves to the hosted version
- [ ] All outbound links carry correct UTM parameters

---

## Testing checklist

**Clients that matter most**

| Client | Watch for |
|---|---|
| Outlook Windows (Word engine) | 600px width, VML poster, line heights, no rounded corners |
| Outlook.com and Windows Mail | `[data-ogsc]` dark mode |
| Gmail webmail | `<style>` block retention |
| Gmail app, iOS and Android | Responsive breakpoints |
| Apple Mail macOS | Dark mode, auto-inversion suppressed |
| Apple Mail iOS | Dark mode plus mobile stacking together |
| Yahoo and AOL | General layout integrity |

**Every send**

- [ ] Renders correctly in both light and dark mode
- [ ] No white logo on a white surface, no dark logo on a dark surface
- [ ] Header stacks cleanly below 620px
- [ ] CTA button is tappable and full width on mobile
- [ ] Images blocked: layout holds and `alt` text is meaningful
- [ ] `<style>` block stripped: layout still legible at 600px
- [ ] Every link resolves, no placeholder URLs remain
- [ ] Preview text reads well in the inbox list
- [ ] Spam score checked before the real send

Use Litmus, Email on Acid or your ESP's built-in preview. There is no substitute for seeing the Word engine render your markup.

---

## Known limitations

| Limitation | Affected | Mitigation |
|---|---|---|
| No `border-radius` in Outlook Windows | Card, buttons, chips | Square corners, colors and layout intact |
| No CSS gradients in Outlook Windows | Hero, CTA button | `bgcolor` solid fallback on every gradient element |
| `<style>` block stripped in some contexts | Dark mode, responsive | Fluid base layout stays legible |
| Hover states unsupported on mobile and in Outlook | Buttons, links | Purely decorative |
| `rgba()` unsupported in older clients | Video overlay | Solid `bgcolor` fallback beneath |
| Dark mode is per-client and inconsistent | Whole email | Two syntaxes plus explicit opt-outs |

The pattern across all of these: **every enhancement has a solid fallback underneath it.** A gradient always sits on a `bgcolor`. A background image always sits on a solid color. Nothing that carries meaning depends on a feature that might not render.

---

## Changelog

**Unreleased**

- Added preheader bar with "Having trouble viewing this email?" and mirror link
- Added WPDeveloper brand logo header above the product hero
- Added footer with permission reminder, unsubscribe link and copyright
- Replaced the dark BetterLinks header logo with a white transparent PNG and removed the white plate it no longer needs
- Exported the white logo at 340x74 for retina, alpha channel preserved

---

## Contributing

If you extend this template, three rules keep it from degrading:

1. **Inline the critical styles.** Anything a recipient must see to understand the email belongs on the element. The `<style>` block is for enhancement only.
2. **Update both dark-mode blocks.** A change to the `@media (prefers-color-scheme: dark)` rules that is not mirrored into `[data-ogsc]` will look correct in Apple Mail and wrong in Outlook.com.
3. **Give every enhancement a fallback.** New gradient? Add the `bgcolor`. New background image? Add the VML and the solid color. Test with the `<style>` block deleted.

Test in a real client suite before opening a PR. Browser preview is not evidence.

---

## License

Add your license here. The BetterLinks and WPDeveloper marks are trademarks of their respective owners and are not covered by any license applied to this code.
