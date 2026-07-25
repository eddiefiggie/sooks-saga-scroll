# Ideation — Simple usage telemetry for Sook's Saga Scroll

**Date:** 2026-07-20 · **Category:** Personal · **Mode:** repo-grounded ideation
**Focus:** How to see how much usage the public site gets, at
<https://eddiefiggie.github.io/sooks-saga-scroll/> — favoring simple, low-effort,
privacy-friendly options, with an eye on *where the data actually gets reported*.

---

## Grounding context (what constrains the choice)

Pulled from the actual repo, not assumed:

- **Single self-contained ~1.4 MB `index.html`.** No build framework, no module
  bundler — telemetry is a `<script>` (or `<img>`) tag pasted into the source build.
- **Offline-capable by design**, with `no-store, no-cache` headers. → Any beacon
  must be **async / non-blocking** and **fail silently** when there's no network,
  or it undermines the offline promise. This is the load-bearing constraint.
- **GitHub Pages hosting.** No backend, no server access, **no server logs you can
  read**, and the `*.github.io` domain **cannot be reverse-proxied**. → Proxy-log
  analytics (the kind that reads real server logs) is simply not available here.
- **Deploy path:** `deploy.yml` copies the *parked build* → `index.html`. → The
  snippet must live in the **parked source `.html`** so it carries forward through
  every future build (edit the build file, not `index.html` directly).
- **Personal D&D tracker, small known audience.** → Free + no cookie-consent
  banner is strongly preferred; enterprise dashboards are overkill.
- **No telemetry exists today** (confirmed — the only "analytics"/"Fathom" hits in
  the file are D&D quest content).

**Topic axes:** (1) where the data lives / who holds it · (2) effort to add & to
maintain · (3) privacy & consent-banner burden · (4) cost · (5) what you can
actually see (a count vs. referrers/geo vs. behavior).

---

## Survivors (ranked)

Each row answers the question you led with — **where the data gets reported** —
plus the trade-offs.

### 1. GoatCounter  ⭐ Recommended

- **Where data is reported:** GoatCounter's servers → your own dashboard at
  `https://<you>.goatcounter.com` (pageviews, referrers, country, browser, screen size).
- **Effort:** One `<script>` tag before `</body>`. ~5 min including signup.
- **Privacy / banner:** Cookieless, no personal data stored → **no consent banner needed.**
- **Cost:** Free for personal / non-commercial use.
- **Fit:** Purpose-built for exactly this — a small static personal site. Beacon is
  `async`, so offline loads just don't report. Best effort-to-value ratio here.

```html
<script data-goatcounter="https://YOURNAME.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>
```

### 2. Cloudflare Web Analytics

- **Where data is reported:** Cloudflare's servers → the Web Analytics tab in your
  Cloudflare dashboard (visits, page views, top referrers, countries, core web vitals).
- **Effort:** One beacon `<script>`. Works via JS on `github.io` **without** needing
  to move DNS/proxy the domain to Cloudflare (that's the key — you can't proxy `github.io`).
- **Privacy / banner:** Cookieless, no fingerprinting → no banner.
- **Cost:** Free.
- **Fit:** Great free alt to GoatCounter, especially if you already have a Cloudflare
  account. Dashboard is a touch less granular for tiny sites but rock-solid.

### 3. Umami (Umami Cloud or self-hosted)

- **Where data is reported:** Umami's servers (Cloud free tier) **or a host you run**
  → a clean self-serve dashboard.
- **Effort:** One `<script>` for Cloud. Self-hosting is real infra work — only worth it
  if data ownership matters more than convenience.
- **Privacy / banner:** Open-source, cookieless, GDPR-friendly → no banner.
- **Cost:** Cloud has a free hobby tier; self-host is "free" but you run it.
- **Fit:** The pick if you want an **open-source** stack and *maybe* own the data later,
  without writing code today.

### 4. Self-owned counter — Cloudflare Worker + KV (you hold the data)

- **Where data is reported:** **Your** Cloudflare Worker + KV store — the data lives in
  *your* account, nobody else's. Read it via a tiny JSON endpoint, or surface a live
  count on the page itself.
- **Effort:** Highest of the survivors — write + deploy a small Worker, add a `fetch()`
  beacon to the page. An afternoon, not 5 minutes.
- **Privacy / banner:** Whatever you choose to log (can be just an incrementing count) → no banner.
- **Cost:** Effectively free at this traffic (Workers/KV free tier).
- **Fit:** For "I want to *own* the numbers and not depend on a third party." Overkill if
  you just want to glance at usage — but the most durable and private.

### 5. Microsoft Clarity (different value: *behavior*, not just counts)

- **Where data is reported:** Microsoft's servers → Clarity dashboard with **heatmaps and
  session recordings** — you can literally watch how players scroll the saga.
- **Effort:** One `<script>`.
- **Privacy / banner:** Free but **uses cookies** and records sessions → in some regions
  you'd want a consent banner; more invasive than the cookieless options.
- **Cost:** Free.
- **Fit:** Only if the real question shifts from *"how many"* to *"how are people using the
  scroll."* Heavier and more privacy-sensitive; not the default for a headcount.

### 6. Google Analytics 4 (only if you already live in Google)

- **Where data is reported:** Google's servers → the GA4 console.
- **Effort:** One `<script>` / gtag snippet.
- **Privacy / banner:** Cookie-based; wants a consent banner in the EU/UK; sends data to
  Google. Heaviest footprint of the list.
- **Cost:** Free.
- **Fit:** Powerful and familiar, but overkill and the least privacy-friendly for a personal
  toy. Listed for completeness, ranked last on purpose.

---

## Rejected (with reasons)

- **CountAPI-style public hit counters** — zero-signup and tempting, but these free shared
  services have a history of going down or being abandoned. Too fragile to build a habit on;
  the Worker+KV counter (#4) is the durable version of the same idea.
- **GitHub repo Insights → Traffic tab** — measures visits to the *repo page on github.com*,
  **not** loads of the published `github.io` site. It does not answer "how much usage does the
  site get," so it's a trap, not a solution.
- **Plausible / Fathom / Simple Analytics (hosted)** — excellent, cookieless, same one-line
  install as GoatCounter — but **~$9/mo**. Poor cost fit for a personal project when
  GoatCounter and Cloudflare give you the same answer free.
- **Google Apps Script / Google Form webhook** — a "free and owned" hack, but fragile, rate-limited,
  and gives you a spreadsheet, not a dashboard. Worker+KV (#4) is strictly better for the same goal.
- **Reverse-proxy / server-log analytics (GoAccess, Netlify Analytics, etc.)** — depends on
  reading server logs or proxying the domain. **Impossible on `*.github.io`.** Out on hosting grounds.

---

## Recommendation & next step

For a free, banner-free, 5-minute answer to "how much usage is this getting," **GoatCounter (#1)**
is the clear fit; **Cloudflare Web Analytics (#2)** is the equally-free fallback if you'd rather
keep it in a Cloudflare account. Reach for **Worker+KV (#4)** only if owning the data matters, and
**Clarity (#5)** only if the question becomes *how* people use it.

Whichever you pick: the snippet goes into the **parked source build** (not `index.html` directly),
placed just before `</body>`, so `deploy.yml` carries it into every future publish.

→ To turn one of these into an actual change, run **ce-brainstorm** on the chosen option (it'll pin
down account setup, exact placement, and offline-fail behavior), or just say "wire in GoatCounter"
and I'll add the snippet to the current build and republish.
