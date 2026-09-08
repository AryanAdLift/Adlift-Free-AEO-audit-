# AdLift — Free AEO/GEO Audit (static, framework-free)

A clean, hand-written version of the landing page. **No framework, no build
step, no runtime dependency** — just `index.html` + `assets/`. This replaces the
earlier Claude Design export (which needed a custom runtime + CDN React/Babel).

## Files

```
index.html      # Everything: markup, CSS (in <head>), JS (before </body>)
assets/
  forbes.png  yahoo-finance.png  investors-hangout.png   # "In the news"
  logos/*.png|.webp                                       # "Trusted by" marquee
```

## How index.html is organised

The page is split into clearly commented sections, in this order:

1. Announcement bar
2. Header / nav
3. Hero + form
4. Stats bar
5. Trusted by (auto-scrolling logo marquee)
6. What the audit covers (accordion)
7. In the news
8. Final CTA + footer

- **CSS** lives in one `<style>` block in `<head>`, grouped by the same section
  banners, driven by CSS variables (`--orange`, `--navy`, `--gold`, …) at the top.
- **JS** is one small vanilla `<script>` before `</body>`: the form submit
  handler and the accordion. No libraries.

Deployed on Vercel — pushing to `main` triggers a production deploy (static
files, no build). `vercel.json` adds long-cache headers for `/assets`.

## Run it

Any static server (no build):

```bash
python -m http.server 8000      # then open http://localhost:8000
```

Or drop `index.html` + `assets/` straight into your site / theme.

## ⚠️ Wire up the form (the one thing left to do)

The form is a single screen that **collects every field**, but it does not yet
send anything to a server. In `index.html`, find the `audit-form` submit handler
(marked `TODO (backend)`) and POST the `payload`:

```js
var payload = { name, phone, company, email, website };
// TODO: POST payload to your CRM / form endpoint, then show the confirmation.
```

`name`, `email` and `website` are marked `required` and validated by the browser
(`email` uses `type="email"`). On success the form is hidden and the confirmation
message is shown.

## Notes

- Responsive: verified with no horizontal scroll at 375px and 768px. Breakpoints
  at 1000 / 860 / 680px.
- Fonts: Poppins via Google Fonts (the only external request; swap for a
  self-hosted copy if you want zero third-party calls).
- Logos are the respective owners' trademarks, used for credibility.
