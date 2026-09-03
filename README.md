# AdLift — Free AEO/GEO Audit landing page

A single-page marketing site that invites visitors to request a free AEO/GEO
(Answer Engine / Generative Engine Optimization) audit. Visitors enter their
domain and contact details; an analyst then runs the audit and emails the report.

- **Live:** https://adlift-free-aeo-audit.vercel.app
- **Reference design:** https://www.adlift.com/free-aeo-geo-audit/
- **Hosting:** Vercel (auto-deploys on push to `main`)

---

## ⚠️ Read this first (for the dev team)

1. **The form does not submit anywhere yet.** The two-step form is fully
   functional on the *front end* (validation-free state transitions only). The
   domain field is bound to component state, but the **Name / Work email /
   Company / Phone fields are not wired to anything**, and pressing **"Send my
   audit"** simply advances the UI to the success screen. **You must connect the
   form to a backend / CRM / email service** (see [Wiring up the form](#wiring-up-the-form)).

2. **This page is a [Claude Design](https://claude.ai/design) canvas export.**
   It is *not* hand-written HTML. `index.html` contains custom `<x-dc>`,
   `<sc-if>`, `<sc-for>` and `{{ ... }}` template tags that only render because
   of **`support.js`** (the "dc-runtime"). See [How it works](#how-it-works).

3. **`support.js` loads React, ReactDOM and Babel from a CDN at runtime**
   (unpkg). The page needs a network connection to render, and Babel compiles
   the component in the browser on every load. This is fine for a marketing
   page but see [Production hardening](#production-hardening) before scaling.

---

## Project structure

```
.
├── index.html          # The page: markup + inline styles + the form component
├── support.js          # Claude Design "dc-runtime" that renders index.html
├── vercel.json         # Static hosting config (long-cache headers for assets)
├── assets/
│   ├── martechseries.png       # "In the news" logos
│   ├── yahoo-finance.png
│   ├── investors-hangout.png
│   ├── free-aeo-geo-banner.png # NOT currently used on the page (see Assets)
│   └── logos/                  # "Trusted by" marquee logos (10)
│       ├── marcus.png  mercer.png  schneider-electric.png  airbnb.png
│       ├── blackstone.png  bausch-lomb.png  emirates-nbd.webp
│       └── national-debt-relief.png  grand-canyon.png  guardian-protection.png
├── docs/
│   └── AdLift Free AEO-GEO Audit - Page Copy.docx  # source marketing copy
└── README.md
```

That's the whole site — there is **no build step**. Deploying = serving these
static files with `index.html` at the root.

---

## Local development

Serve the folder with any static file server and open it in a browser. Opening
`index.html` via `file://` will **not** work (the runtime and its CDN scripts
need an `http://` origin).

```bash
# Python 3
python -m http.server 8000

# Node
npx serve .

# PHP
php -S localhost:8000
```

Then visit http://localhost:8000. There is no watch/compile step — edit
`index.html` and refresh.

---

## How it works

`index.html` is a Claude Design canvas file:

- The visible page lives inside `<x-dc>…</x-dc>` as templated markup.
- `<script src="./support.js">` boots the **dc-runtime**, which:
  1. Injects the stylesheet/fonts from the `<helmet>` block into `<head>`.
  2. Loads **React 18.3.1**, **ReactDOM 18.3.1** and **@babel/standalone 7.29.0**
     from `unpkg.com`.
  3. Compiles the component in the trailing `<script type="text/x-dc">` with
     Babel and renders the `<x-dc>` tree with React, resolving `{{ ... }}`
     bindings, `<sc-if>` (conditionals) and `<sc-for>` (loops).

### The form component

At the bottom of `index.html`, `class Component extends DCLogic { … }` holds all
interactivity. It's a small state machine:

| `state.phase` | Flag        | What shows                                   |
|---------------|-------------|----------------------------------------------|
| `idle`        | `isIdle`    | Step 1 — domain input + "Continue"           |
| `score`       | `isScore`   | Step 2 — name / email / company / phone       |
| `done`        | `isDone`    | Success screen ("Your audit is queued")       |

Handlers: `startScan` (idle→score), `submit` (score→done), `reset` (→idle),
`onDomain` (binds the domain field).

### Configurable prop

`data-props` on the component script exposes one editable value:

- **`turnaround`** — integer hours, default `48`. Drives the "48h" stat in the
  hero stats bar. *(Note: the step-2 copy was changed to read "2 business days"
  and no longer uses this value.)*

---

## Wiring up the form

To actually capture leads, edit the `Component` class in `index.html`:

- Give the Name / Email / Company / Phone inputs `value` + `onChange` bindings
  (mirror how the domain input uses `domainValue` / `onDomain`).
- In `submit`, `POST` the collected fields to your endpoint (CRM webhook, form
  API, serverless function, etc.) before setting `phase: 'done'`.
- Add validation (required fields, email format) and error/loading states.

If you'd rather not extend the dc-runtime component, the alternative is to
**eject** to plain React/HTML (see below) and build the form conventionally.

---

## Deployment

Connected to Vercel; **pushing to `main` triggers a production deploy**. No
build command and no output directory — it's served as static files.

To deploy elsewhere (Netlify, S3+CloudFront, Nginx, GitHub Pages, …): upload the
repo contents as-is and make sure `index.html` is served at `/`. `vercel.json`
only adds long-cache headers for `/assets/*`; other hosts can ignore it.

---

## Assets

All logos are the client's/publishers' own trademarks, used for
"trusted by" / "in the news" credibility.

- `assets/logos/*` — the 10 "Trusted by" logos in the auto-scrolling marquee.
- `assets/martechseries.png`, `yahoo-finance.png`, `investors-hangout.png` —
  the three "In the news" cards.
- `assets/free-aeo-geo-banner.png` — **not referenced on the page right now.**
  It was previously a full-width banner at the top of the hero and was removed
  on request. Kept here in case you want to reinstate it.

---

## Responsive / browser support

- Layout is responsive down to ~360px. Breakpoints at 1000px (hero collapses to
  one column), 860px and 680px (nav hidden, grids collapse). Verified with no
  horizontal scroll at 375px and 768px.
- Targets evergreen browsers (Chrome, Edge, Safari, Firefox). React 18 + the
  CSS features used (grid, `mask-image`) require modern browsers.

---

## Production hardening (recommended, optional)

The dc-runtime approach is great for a design handoff but has two trade-offs for
a high-traffic production page:

1. **External CDN dependency** — React/ReactDOM/Babel load from `unpkg.com`. If
   unpkg is slow or unreachable, the page won't render. Consider self-hosting
   pinned copies of those scripts, or ejecting.
2. **In-browser compilation** — Babel transpiles the component on every page
   load, adding a small startup cost.

For a hardened build you can **eject** the design to a conventional stack
(plain React app or static HTML + a little vanilla JS for the form), which
removes the runtime, the CDN dependency and the client-side Babel step. The
current files remain the source of truth for markup, copy and styling.
