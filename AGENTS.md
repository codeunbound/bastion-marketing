# AGENTS.md — read this before editing the Bastion marketing site

This file is written for an AI agent (or any new developer) about to make changes.
Read it fully first. The site looks simple but has a **persona system** and a set of
**truth/legal constraints** that are easy to break by accident.

---

## 1. What this is

Bastion is **managed, DNS-based content filtering** for three audiences: **families,
businesses, and schools**. It is sold as a managed service ("we run it for you"), not a
self-install tool. This repo is the **static marketing site** (4 pages). The actual product
dashboard lives separately at `dash.bastion.codeunbound.dev` — it is NOT in this repo.

- **No build step.** Plain HTML + one CSS file + one JS icon file. Open `index.html` and it works.
- **No framework** in production. (A React-based "Tweaks" design panel exists in the working
  source but was stripped from this deploy build — see §8.)

### Files
```
index.html      → home: persona gate, 3 hero variants, trust strip, "Why Bastion"
                  spotlight, How-it-works, Features, Platforms, Straight talk,
                  pricing teaser, FAQ, CTA band, footer
pricing.html    → price hero + monthly/annual toggle, 3 tiers, pricing FAQ, CTA
terms.html      → Terms & Conditions
privacy.html    → Privacy Policy
bastion-site.css→ ALL styles for ALL pages (shared)
icons.js        → hydrates <i class="ph ... ph-NAME"> markup into inline SVG.
                  No icon webfont — icons render in screenshots/PDF/offline too.
                  If you use a new ph-NAME, you MUST add its SVG path to icons.js.
img/            → the four phone screenshots used in heroes/spotlight
CNAME, .nojekyll→ GitHub Pages config (custom domain + disable Jekyll)
```
Fonts: **Figtree** (UI) + **DM Mono** (mono labels), loaded from Google Fonts in each `<head>`.

---

## 2. THE PERSONA SYSTEM (most important thing to understand)

The whole site pivots between three personas: **`family` · `business` · `school`**.

- The active persona is stored as `document.documentElement` attribute `data-persona`
  and persisted to `localStorage['bastion_persona']`.
- On first visit, a **gate** ("What do you do?") asks the visitor to pick; the choice is
  remembered and the gate never shows again. The hero switcher ("Show me Bastion for")
  lets them change at any time.
- `pricing.html`, `terms.html`, `privacy.html` **read** the stored persona on load and
  mirror it (so the visitor stays in their world across pages).

### How a section pivots — two mechanisms
1. **Whole-section variants** — three sibling blocks, one per persona:
   ```html
   <div class="pv" data-for="family">…</div>
   <div class="pv" data-for="business">…</div>
   <div class="pv" data-for="school">…</div>
   ```
   CSS shows only the one matching `html[data-persona]`. **`.pv` blocks are display:none by default.**
2. **Inline word swaps** — for a single differing word/phrase inside shared copy:
   ```html
   <span class="p-only p-family">homework hours</span><span class="p-only p-business">work hours</span><span class="p-only p-school">the timetable</span>
   ```

### ⚠️ THE GOLDEN RULE
**If you edit one persona's variant, you MUST keep all three in sync.** Every `.pv`/`p-only`
group needs a `family`, `business`, AND `school` version, or a persona will see a blank gap.
When adding any new persona-tailored section, add all three variants up front.

### ⚠️ DON'T PLASTER THE SAME PITCH EVERYWHERE
The three personas buy for **genuinely different reasons**. This was a deliberate content
decision — do not "rephrase the same sentence" across them. The axis each leans on:

| Persona  | Leans on            | Why they buy                                            |
|----------|---------------------|---------------------------------------------------------|
| family   | **Time** (schedules)| "Schedule the internet, not just block it." Set & forget|
| business | **People** (groups) | A flat DNS block dies because one rule breaks marketing's job. Per-group rules + a security floor (scam/phishing for everyone) + reports. NOT "lost productive hours." |
| school   | **Proof** (record)  | Safeguarding: when a parent/board asks, you have a documented record. Grade-wise rules, exam lockdown, your own isolated instance. Positive hero, defensive spine. |

Mechanically, groups and schedules are **one engine** — a per-group category rule is just a
schedule set to "allow/block all day." Say that truthfully; don't imply separate features.

---

## 3. PRICING — "Your plan" highlight

Personas map to tiers: **family → Home · business → Team · school → Enterprise**.
`pricing.html` reads `localStorage['bastion_persona']` and:
- adds `.tier--yours` (teal ring) to the matching tier,
- injects a **"Your plan"** badge on it,
- hides the generic **"Most popular"** badge.

With no stored persona it falls back to the default (Team = "Most popular").
Note: the **Enterprise** card has a permanent dark background — that's styling, not a
"highlight." The badge + ring is the real signal of which plan is the visitor's.

Personas are **self-identification** (Families/Business/Schools); tiers are **needs**
(Home/Team/Enterprise). The Enterprise card copy is intentionally generic ("privacy-first
orgs" — schools, coaching institutes, hostels), NOT school-specific.

---

## 4. TRUTH & LEGAL CONSTRAINTS (do not break these)

Bastion's brand voice is **honest about limits**. Claims must stay technically true:

- It is **DNS-level filtering only**. Never imply antivirus, packet inspection, or reading
  of page content / messages / keystrokes. Reports are **domain-level** ("which sites",
  not "what was typed").
- It is **bypassable on an adversarial BYOD device** (anyone who can change DNS or run a
  VPN can tunnel past). The "Straight talk" section says this on purpose — keep it.
- **Data residency:** all activity data is processed and stored **in India (AWS Mumbai)**,
  inside the production environment, and does not leave it. Shared cloud = **logical**
  per-device isolation; Enterprise dedicated VPS = **physical** isolation.
- **Retention:** Home keeps a rolling **7 days**, then genuinely deleted. Enterprise 30/90
  days on its own instance. Extending retention only adds data going forward.
- **Fair use:** Home = single household, up to 10 devices, personal use (commercial use →
  Team). Team = 10 included then per-device, **no hard cap**. Enterprise = dedicated instance.

### Outstanding TODOs (don't ship to real customers without these)
- `terms.html` / `privacy.html` contain placeholders: **`[LEGAL ENTITY NAME]`,
  `[REGISTERED ADDRESS]`, `[EFFECTIVE DATE]`** — fill before launch.
- A **DPDP-specialist review of children's/minors' data** is still open. Draft legal banners
  stay up until counsel signs off.

---

## 5. Editing conventions & gotchas

- **Canonical HTML:** close every element, double-quote attributes, don't self-close non-void
  tags. The CSS is shared across all 4 pages — a change affects everywhere.
- **`data-screen-label`** attributes on heroes/sections are intentional (review tooling) — keep them.
- **Links between pages** are plain relative names: `index.html`, `pricing.html`, `terms.html`,
  `privacy.html`. (The working source used "Bastion Home.html" etc.; this deploy build renames them.)
- **CTAs:** self-serve buttons go to `https://dash.bastion.codeunbound.dev`; school/enterprise
  uses `mailto:support@bastion.codeunbound.dev`. There is a **single "Get started"** button in the
  nav (the auth screen is one combined SSO page, so a separate "Log in" was intentionally removed).
- **New icons:** add the SVG path to `icons.js` — using an unknown `ph-NAME` renders nothing.
- **Reveal-on-scroll:** elements with class `reveal` start hidden and animate in via
  IntersectionObserver. If content looks invisible after a persona switch, that's why —
  scroll it into view or add `.in`.

## 6. Source of truth for COPY
The canonical content + rationale lives in **`Bastion Content Handoff.md`** (in the working
project, not this deploy folder). It carries every copy block plus dated `‹EDITOR NOTE›`
entries explaining *why* wording is the way it is. Read it before rewriting any copy.

## 7. What was stripped from this build
The working source includes a **Tweaks panel** (React + Babel) for design review — a persona
switcher, brand-palette picker, and a business-headline A/B selector. It is **removed** from
this production build (it renders nothing without the design host and only adds weight).
One consequence: the business hero headline that the Tweaks panel toggled is now **baked in**
as **"The office filter that doesn't get switched off."** To change it, edit the `<h1>` inside
`.hero__variant[data-for="business"]` in `index.html`. (Alternatives previously considered:
"One filter. The right rules for every team." / "A filter the whole office can live with.")
