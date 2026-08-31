# Security Policy — GrowIndie Digital site

Adapted from the Claude Skills Library security policy
(https://github.com/alirezarezvani/claude-skills/blob/main/SECURITY.md) for this project.

## Reporting a vulnerability

Do **not** open a public issue. Report privately to growindiedigital@gmail.com
with: vulnerability type, full details, reproduction steps, potential impact,
suggested fix, and your contact details.

Response targets: initial reply 48h · assessment 1 week · fix by severity.

| Severity | Examples | Fix window |
| --- | --- | --- |
| Critical | RCE, unauthorised access to lead data, privilege escalation | 24–48h |
| High | Data exposure, auth bypass | 1 week |
| Medium | XSS, information disclosure, misconfiguration | 2 weeks |
| Low | Minor leaks, best-practice violations | 1 month |

Public disclosure only after the fix is deployed. Responsible reporters are
credited if they wish.

---

## Vulnerability audit — 31 Aug 2026 (re-run after multi-page split)

Scope: all 8 site pages (Home, About, Services, Industries, Pricing,
Testimonials, FAQ, Contact) plus shared `Nav.dc.html`, `Footer.dc.html`,
`content.js` (bilingual copy + `postForm` helper, now a single source instead
of duplicated per page), `image-slot.js`, `support.js`. Checked for injection
sinks, unsafe evaluation, data leakage, storage misuse, third-party risk,
form/transport handling, and secrets. No new findings — all items below still
hold across every page.

### No vulnerabilities found in our own code

| Check | Result |
| --- | --- |
| XSS sinks (`innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`) | None in `Home.dc.html`. All copy — including both Tamil and English translation tables — renders as text nodes. |
| Unsafe evaluation (`eval`, `new Function`, string `setTimeout`) | None in our code. |
| `javascript:` URLs, inline `onerror`/`onload` handlers | None. |
| Network calls (`fetch`, `XMLHttpRequest`) | None in our code. No analytics, pixels, or tag managers. |
| Client-side storage (`localStorage`, `sessionStorage`, cookies, IndexedDB) | Nothing written. No consent banner needed as-is. |
| Reverse tabnabbing | All 4 `target="_blank"` links (`wa.me` ×2, `instagram.com` ×2) carry `rel="noopener"`. ✅ |
| Secrets / API keys / tokens in source | None. |
| Open redirects, URL params read into the DOM | None — the page reads no query string. |
| Mixed content | All external references are HTTPS. |
| Third-party subresources | Only Google Fonts (`fonts.googleapis.com`, `fonts.gstatic.com`), with `crossorigin` on the preconnect. |
| Prototype pollution / unvalidated object keys | `industryMult[roiIndustry]` is keyed only by three hardcoded setter callbacks — not reachable from user input. |

### Findings

**F-1 · High — FIXED 29 Aug 2026 (needs your access key).**
Every field was missing a `name` attribute and `submitForm` only flipped a
success flag — submissions were discarded while the user was told they were
received. Now: every field has `name` + `autocomplete` + a `maxlength` cap, the
form POSTs to Web3Forms over HTTPS, the success state appears only after the
API confirms `success: true`, and a failure shows an error with the WhatsApp
number as fallback. The footer newsletter is a real form too.
**Action required:** verify growindiedigital@gmail.com at web3forms.com and
paste the access key into `FORM_ACCESS_KEY` in `Home.dc.html`. Until then
submits fail loudly instead of pretending success.

**F-2 · Medium — PARTLY FIXED 29 Aug 2026.**
Both forms now carry an off-screen `botcheck` honeypot; a filled honeypot is
dropped client-side and also rejected by Web3Forms. Still open: add Cloudflare
Turnstile if volume warrants it, and rely on Web3Forms' own rate limiting.

**F-3 · Medium — no server-side validation planned.**
`required` and `type="email"` are UX hints only; a bot posts directly to the
endpoint. Validate and length-cap every field on the server, and escape values
before they land in any email body, spreadsheet, or CRM (formula injection: a
value starting `=`, `+`, `-`, or `@` becomes a live formula in Excel/Sheets —
prefix with `'`).

**F-4 · Medium — FIXED 29 Aug 2026.**
Shipped as `_headers` (Netlify / Cloudflare Pages read it natively; translate to
`.htaccess` or nginx if you host elsewhere):
```
Content-Security-Policy: default-src 'self'; img-src 'self' data:; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; form-action https://<your-form-endpoint>; frame-ancestors 'none'; base-uri 'none'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=()
```
`frame-ancestors 'none'` also covers clickjacking (no `X-Frame-Options` needed).

**F-5 · Medium — FIXED 29 Aug 2026.**
`Privacy.dc.html` now states what is collected, why, who processes it, retention
per data type, and how to exercise DPDP rights (7-day reply, 30-day completion).
`Terms.dc.html` covers the free audit, retainers, ad-spend billing, ownership,
and liability. Both are linked from the footer, and the audit form carries a
privacy line linking to the policy. Still yours to do: keep leads in one
access-restricted inbox, and don't forward lead lists around WhatsApp.

**F-6 · Low — build artefacts must not ship.**
`support.js` is the design-tool runtime: it uses `new Function(...)` to evaluate
the logic class and `window.parent.postMessage(..., '*')` with a wildcard
target origin. That is correct inside the editor but has no business on a public
domain. `image-slot.js` likewise is an editor upload widget that `fetch`es a
sidecar state file. **Before launch, export a static build**: replace the three
`<image-slot>` badge slots with plain `<img>` tags and drop both scripts.
Shipping them as-is widens the page's attack surface for zero user benefit.

**F-7 · Low — FIXED 29 Aug 2026.**
`setRoiSpend` now clamps to ₹10,000–₹10,00,000 and falls back to ₹1,00,000 on a
non-finite value, so no `NaN` can reach the displayed lead count or ROAS range.

**F-8 · Informational — contact details are public and unobfuscated.**
A privacy line now sits under the form ("we use your details only to prepare
and send your audit"). `growindiedigital@gmail.com` and `+91 83447 35921` still
appear in plain `mailto:` /
`tel:` links in two places each. That is a deliberate trade-off for a lead-gen
site, but expect scraping — enable Gmail's spam filtering aggressively, and
consider moving to a domain email you can filter and rotate.

**F-9 · Informational — Google Fonts is a third-party request.**
Every visitor's IP reaches Google. Self-hosting the three families (Inter,
Inter Tight, Noto Sans Tamil) as `@font-face` + local `.woff2` removes the
third party, tightens the CSP, and loads faster on Indian mobile networks.

**F-10 · Informational — FIXED 29 Aug 2026.**
`Home.dc.html` now carries a JSON-LD `@graph` with ProfessionalService/
LocalBusiness (Thoothukudi geo, service areas, retainer offers), WebSite, and
FAQPage. Verify with Google's Rich Results Test after the domain is live.

### Ongoing practice
- Review any script before adding it: check its network calls, DOM writes, and
  what data it reads. Pin versions — never `@latest` from a CDN.
- Add Subresource Integrity (`integrity` + `crossorigin`) to any third-party
  script you do add.
- Keep dependencies minimal — the page currently has none beyond fonts.
- Never commit credentials; keep the form endpoint in host env vars.
- Re-run this audit whenever a form, embed, tracking pixel, or CMS is added.

References: [OWASP Top 10](https://owasp.org/www-project-top-ten/) ·
[MDN CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
