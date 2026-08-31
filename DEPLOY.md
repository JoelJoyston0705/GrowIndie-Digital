# Launch checklist — GrowIndie Digital

Everything in this repo is production-ready **except the items under "Only you
can do these"**. Work top to bottom.

---

## 1. What's already done

| Area | State |
| --- | --- |
| Structure | 8 pages (Home, About, Services, Industries, Pricing, Testimonials, FAQ, Contact) + shared `Nav.dc.html` / `Footer.dc.html`, all copy centralised in `content.js` |
| SEO head | Title, meta description, canonical, robots, theme-color, favicon on every page |
| Social sharing | Open Graph + Twitter card tags on Home; `assets/og-image.png` (1200×630) |
| Structured data | LocalBusiness + WebSite JSON-LD on Home, FAQPage JSON-LD on FAQ, OfferCatalog on Pricing |
| Legal | `Privacy.dc.html` (DPDP-aligned) and `Terms.dc.html`, linked from every footer and the audit form |
| Error page | `404.dc.html`, `noindex` |
| Crawling | `robots.txt` + `sitemap.xml` (all 10 URLs) |
| Security headers | `_headers` (CSP, HSTS, nosniff, Referrer-Policy, Permissions-Policy, frame-ancestors, cache rules) — re-audited 31 Aug 2026, no findings |
| Forms | Contact + newsletter POST to Web3Forms over HTTPS, honeypot, `required`, length caps, real success/error states |
| Accessibility | Skip link, `:focus-visible` rings, FAQ as real buttons with `aria-expanded`, tablist semantics, `prefers-reduced-motion`, `<html lang>` follows the EN/TA toggle, 44px+ touch targets on nav controls |
| Mobile | Card grids clamp with `minmax(min(Npx,100%),1fr)` so nothing overflows below ~340px; nav collapses to a scrollable link row under 1024px |
| Honesty | No fabricated stats or invented client logos; pricing shows "Tailored to you" with no GST/GSTIN claim (site isn't GST-registered) |

---

## 2. Only you can do these — site is not live until they're done

1. **Domain.** Code assumes `https://growindiedigital.in`. If yours differs,
   find-and-replace across `Home.dc.html` (canonical/OG/JSON-LD), the other 7
   pages' `<link rel="canonical">`, `robots.txt`, and `sitemap.xml`.
2. **Form access key.** Already wired (`content.js` → `GI_FORM.KEY`) and
   confirmed delivering to the Web3Forms account — no action needed unless you
   rotate the key.
3. **Google Business Profile.** Create and verify it for Thoothukudi with the
   same name/address/phone as the JSON-LD.
4. **Real testimonials.** The three reviews on `Testimonials.dc.html` /
   `content.js` are placeholders written in a plausible style — swap for real
   quotes with the customer's written permission before launch.
5. **Hosting.** Deploy to Netlify or Cloudflare Pages (both read `_headers`
   natively), point DNS, enable HTTPS + HSTS, force `https://` and one
   canonical host.

---

## 3. Static export before deploy

The `.dc.html` files carry an editor runtime that should not ship. I can do this
in one pass across all 10 pages:

1. Export each as a single self-contained file.
2. Rename: `Home`→`index.html`, `About`→`about.html`, `Services`→`services.html`,
   `Industries`→`industries.html`, `Pricing`→`pricing.html`,
   `Testimonials`→`testimonials.html`, `FAQ`→`faq.html`, `Contact`→`contact.html`,
   `Privacy`→`privacy.html`, `Terms`→`terms.html`, `404`→`404.html`.
3. Update every internal link (`Nav.dc.html`, `Footer.dc.html`, and cross-page
   `<a href>`s) to the new filenames.
4. Confirm `support.js` and `image-slot.js` are not referenced by the deployed
   files.

Ask me to run the export and I'll do all four steps.

---

## 4. Post-launch, first week

- Submit `sitemap.xml` in Google Search Console; request indexing of the home page.
- Run each URL through Google's Rich Results Test (LocalBusiness, FAQPage, OfferCatalog).
- Test the share card by pasting the URL into WhatsApp and LinkedIn.
- Lighthouse on mobile for every page: target 90+ performance, 100 accessibility.
- Test the contact form from a phone on mobile data, in both EN and TA.
- Check Pricing and Testimonials grids at 360px and 320px width.
- Set up an uptime monitor (UptimeRobot free tier).
- Add analytics if you want it — Plausible or GA4. Both need a CSP line added to
  `_headers` and a sentence in the privacy policy, which currently states
  truthfully that the site has no tracking.

---

## 5. Deferred by choice — not blockers

- **Self-hosting fonts.** Removes Google Fonts as a third party, speeds up
  Indian mobile loads. Worth doing; not required to launch.
- **Case studies.** Add two real ones once you have client sign-off.
- **Client logo strip.** Add it back once you have named, consenting clients.
- **Cloudflare Turnstile** on the forms, if spam volume warrants it.
- **GST registration.** The site currently states no GST — update pricing copy
  in `content.js` (`pricingNote`, FAQ) once/if you register.
