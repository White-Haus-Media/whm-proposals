# WHM Proposals — Project Instructions

This repo houses White Haus Media's client proposal system. Static HTML proposals deployed on Vercel. Every proposal is generated from a master template — never built from scratch.

## REPO

GitHub: `White-Haus-Media/whm-proposals` (case-sensitive). The legacy lowercase `whitehausmedia/whm-proposals` redirects on reads but throws 307 on API writes — always use the canonical name.

## STRUCTURE
```
_template/
  proposal-template.html   ← MASTER TEMPLATE — source of truth for all proposals
  generate.js              ← Manual batch script for specific verticals (e.g. daycare)
proposals/
  company-slug/
    index.html             ← Generated proposal, filled from master template (only file)
```

## DEPLOYMENT URLS

Vercel project: `whm-proposals` → deploys to `whm-proposals.vercel.app`
Custom domain: `proposals.whitehausmedia.com`

Working URLs after auto-deploy:
- `https://whm-proposals.vercel.app/proposals/[slug]/`
- `https://proposals.whitehausmedia.com/proposals/[slug]/`

The legacy pattern `https://whitehausmedia.com/proposals/[slug]/` requires a rewrite rule on the apex domain that is not currently in place. Always send prospects the `proposals.whitehausmedia.com` URL or the `whm-proposals.vercel.app` URL until the apex rewrite is confirmed.

When writing the `url` field on the Supabase `proposals` record, use `https://proposals.whitehausmedia.com/proposals/[slug]/` as the canonical.

Slug rules: lowercase, hyphens only, match company name closely.
Example: "Aversboro Road Child Care Center" → `aversboro-road-child-care-center`

## FILE ARCHITECTURE — ONE FILE ONLY

Every proposal ships exactly one file: `proposals/[slug]/index.html`. No `preview.html`. No mockup file. No assets folder per client.

The current master template contains an iframe that references `preview.html`, but the iframe gracefully falls back to a "Preview loading after deployment" message when no preview file is present. This is intentional — the one-file architecture is the canonical pattern.

If a future strategy requires a separate preview page, that decision needs an explicit update to this doc and the template before being implemented.

## LOGO FETCHING

When a proposal needs the client's actual logo (for any embedded brand reference, not for a preview file):
1. While scraping the prospect's site, find the logo `<img>` src (usually in `<header>` or `<nav>`)
2. Fetch the image via curl: `curl -s -L -A "Mozilla/5.0" [logo-url] -o /tmp/logo.png`
3. Resize to 2x retina nav height (typically 585px wide max) using Pillow
4. Base64-encode and embed as a data URI: `<img src="data:image/png;base64,...">`
5. Never hotlink — the proposal must not depend on the client's server

If no logo is found:
- Check `/images/`, `/assets/`, `/img/`, `/wp-content/uploads/` common paths
- Fall back to styled text with the company name in the nav brand font
- Never use a generic SVG icon as a logo substitute

## MASTER TEMPLATE PLACEHOLDERS

Always fetch the template at `_template/proposal-template.html` via the GitHub API and fill placeholders. Never generate proposal HTML from scratch.

The current template uses these placeholders (verified by grep):

| Placeholder | What it is |
|---|---|
| `{{COMPANY_NAME}}` | Client company name |
| `{{SLUG}}` | URL slug (lowercase-hyphenated) |
| `{{DATE}}` | Proposal date (e.g. "April 24, 2026") |
| `{{PREPARED_FOR}}` | Full string in hero meta (e.g. "Jenn Bernat, Local Charm") |
| `{{AUDIENCE}}` | Industry-specific audience term (see table below) |
| `{{HERO_HEADLINE}}` | Big hero h1 |
| `{{HERO_BODY}}` | Hero subheadline / supporting copy |
| `{{LETTER_GREETING}}` | Letter opener (e.g. "Hi Jenn,") |
| `{{LETTER_P1}}` | Letter paragraph 1 — acknowledge what they've built |
| `{{LETTER_P2}}` | Letter paragraph 2 — name the opportunity |
| `{{LETTER_P3}}` | Letter paragraph 3 — frame the path forward and close |
| `{{OPP_HEADLINE}}` | Opportunity section h2 |
| `{{OPP_SUB}}` | Opportunity section subhead |
| `{{OPP_1_TITLE}}` through `{{OPP_4_TITLE}}` | Opportunity card titles |
| `{{OPP_1_BODY}}` through `{{OPP_4_BODY}}` | Opportunity card body copy |
| `{{DEL_1}}` through `{{DEL_8}}` | Deliverable items — short single-line phrases (used in both deliverables grid and pricing list) |
| `{{PREVIEW_DOMAIN}}` | URL string shown in the fake browser bar above the preview iframe |
| `{{PRICE}}` | Build fee number only, no `$` (e.g. "797") |
| `{{BOOKING_URL}}` | Calendly link, defaults to `https://calendly.com/whitehausmedia` |

Notes:
- There are NO separate `{{DEL_X_DESC}}` fields. Deliverables are single-line phrases only.
- There is NO `{{CARE_PLAN}}` or `{{CARE_MONTHLY}}` placeholder in the current template. Pricing is a single `{{PRICE}}` field. If a care plan needs to appear in the proposal, the template needs to be extended first.
- There are NO standalone `{{CITY}}`, `{{INDUSTRY}}`, or `{{CONTACT_NAME}}` placeholders. Bake those into other fields (e.g. `{{PREPARED_FOR}}`, `{{LETTER_P1}}`) as needed.
- The `{{LETTER_BODY}}` placeholder referenced in older docs has been split into `{{LETTER_P1}}`, `{{LETTER_P2}}`, `{{LETTER_P3}}`.

### {{AUDIENCE}} by Industry
| Industry | Audience Term |
|---|---|
| childcare / daycare | families |
| healthcare | patients |
| restaurant / food | guests |
| church / faith | members |
| medspa / aesthetics | clients |
| legal / law | clients |
| fitness / gym | members |
| real estate | buyers and sellers |
| retail / boutique | shoppers |
| nonprofit | supporters |
| education | students and families |
| default | clients |

## SUPABASE SCHEMA REFERENCE

The Haus runs on Supabase. Foreign-key chain when generating a proposal:
**companies → contacts** (separate, optional for proposal flow)
**companies → prospects → deals → proposals**

Insert in this order: companies, contacts, prospects, deals, proposals.

### `companies` table
Required: `name`. Optional: `domain`, `website`, `phone`, `city`, `state`, `zip`, `industry`, `google_rating`, `google_reviews`, `incumbent_vendor`, `incumbent_tech`, `notes`, `whm_notes`, `hubspot_id`.

`industry` is a strict enum. Allowed values: `Dental`, `Legal`, `Other`, `Real Estate`, `Restaurant`. Anything else is rejected with `22P02 invalid input value for enum industry_type`. For boutique, retail, medspa, fitness, etc., use `Other` and note the real industry in `whm_notes` until the enum is expanded.

### `contacts` table
Fields: `company_id` (FK), `first_name`, `last_name`, `email`, `phone`, `job_title`, `is_primary`, `referral_source`, `referral_type`, `referral_payout`, `hubspot_id`.

### `prospects` table
Fields: `company_id` (FK), `opp_score`, `score_label`, `tier`, `status`, `site_score`, `mobile_score`, `seo_score`, `design_score`, `content_score`, `digital_gaps`, `strengths`, `originated_by`, `assigned_to`, `notes`.

Strict enums:
- `tier`: `Starter`, `Growth`, `Premium`
- `status`: `New`, `In Pipeline`, `Outreached` (no `Proposal Generated` value exists — use `In Pipeline` after pushing the proposal)
- `score_label`: `Cold`, `Warm`, `Hot`

`digital_gaps` and `strengths` are stored as comma-separated strings, not arrays.

### `deals` table
Fields: `company_id` (FK), `prospect_id` (FK), `deal_name`, `stage`, `build_fee` (numeric), `care_plan`, `care_plan_monthly` (numeric), `proposal_url`, `est_revenue`, `payment_split`, `close_date`, `owner`, `notes`.

Strict enums:
- `stage`: `Ready for Review` (others may exist but this is the verified working value for newly generated proposals — no `Proposal Sent` value)
- `care_plan`: `None` (others may exist for active care plans, but `None` is the verified default when no plan is locked in)

### `proposals` table
Fields: `company_id` (FK), `deal_id` (FK), `title`, `status`, `url`, `version`, `sent_date`, `notes`.

`status` for newly created proposals: `Draft`. Set `sent_date` only when actually sent.

`build_fee` and care plan info live on the `deals` table, not on the proposals table. Reference them in the proposals `notes` field for human readability.

## PROPOSAL COPY TONE — NON-NEGOTIABLE

**The cardinal rule: We are a partner excited about the client's potential, never a critic who found their flaws.**

Every word of copy should pass this test: *would a prospect feel seen and understood, or would they feel judged?*

Additional Rico-voice rules:
- No em dashes anywhere in client-facing copy. Use periods, commas, or restructure the sentence.
- Direct, fast-moving, psychologically aware. No filler.
- Native vocabulary: "Find/Win/Keep," "speed-to-lead," "anchor pricing," "economic value transfer."

### What to NEVER write
- "Your site is outdated / dated / old"
- "Your site doesn't match your work"
- "You're missing / losing leads"
- "Your site is costing you clients"
- "No way to capture leads" (accusatory framing)
- "Working against you"
- Any language that implies the prospect has been negligent or behind

### How to frame opportunity
Always lead with what's *possible*, not what's *broken*. Translate every weakness into an aspiration:

| Instead of... | Write... |
|---|---|
| "No lead capture form" | "Capturing inquiries around the clock" |
| "Outdated design" | "A first impression worth the craft behind it" |
| "Bad SEO" | "A modern foundation built to get found" |
| "Reviews buried" | "Turning your reviews into your best sales tool" |
| "Site doesn't match your work" | "A digital presence that reflects what you've built" |

### Letter tone
- Open by acknowledging what they've built — be specific (mention the blog, testimonials, service area, tenure)
- Frame the website opportunity as "there's more you could be doing" not "you're falling short"
- Position WHM as excited collaborators, not consultants diagnosing a problem
- Never include phrases like "hasn't kept pace," "makes it harder," or "working against you"

### Opportunity section
- Section headline: aspiration-forward
- Section sub: acknowledge their strength first, then name the opportunity
- OPP card titles: always aspirational, never accusatory
- OPP card body: explain the benefit of fixing something, not how broken it currently is

### Hero headline
- Lead with what's great about the prospect, then invite them into what's possible
- "The craftsmanship is world-class. Let's make sure the world sees it." ✓
- "Your restoration work speaks for itself. Your website should too." ✗ (implies it doesn't)

## INTERNAL DATA RULES
- **Scores (opp_score, site_score, etc.) are NEVER shown to clients** — use them internally to write qualitative copy only
- No scorecard sections in delivered proposals
- No percentages, grades, or numerical ratings visible to prospects

## GENERATE.JS USAGE
The manual batch script is for bulk-generating proposals for a specific vertical. To use:
```bash
node _template/generate.js
```
Edit `DAYCARE_DEFAULTS` and the company array at the top of the file to match the target companies. Always push the generated files to GitHub — Vercel auto-deploys.

## AUTO-GENERATOR (Scheduled Task)
The `auto-proposal-generator` Cowork task runs 3x/day and handles proposals automatically for prospects with opp_score ≥ 60. It:
1. Fetches the master template via GitHub API from `White-Haus-Media/whm-proposals`
2. Fills placeholders using prospect/company data (per the placeholder table above)
3. Pushes `proposals/[slug]/index.html` to this repo (one file only)
4. Inserts records in order: companies (if new) → contacts → prospects → deals → proposals
5. Updates prospect status to `In Pipeline` (the `Proposal Generated` value referenced in older docs does not exist in the enum)
6. Auto-sends if build_fee < $800, otherwise sets deal stage to `Ready for Review` in The Haus

## DESIGN REFERENCE
- Background: #080808 (near-black)
- Gold: #C6A44C
- Cream: #F4EFE6
- Fonts: Cormorant Garamond (display) + Plus Jakarta Sans (body)
- Icons: thin-stroke gold SVGs — never emoji
