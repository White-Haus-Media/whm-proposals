# WHM Proposals — Project Instructions

This repo houses White Haus Media's client proposal system. Static HTML proposals deployed on Vercel at whm-proposals.vercel.app. Every proposal is generated from a master template — never built from scratch.

## STRUCTURE
```
_template/
  proposal-template.html   ← MASTER TEMPLATE — source of truth for all proposals
  generate.js              ← Manual batch script for specific verticals (e.g. daycare)
proposals/
  company-slug/
    index.html             ← Generated proposal, filled from master template
    preview.html           ← Site concept built for each proposal (required)
```

## PROPOSAL URL FORMAT
`https://whitehausmedia.com/proposals/[company-slug]/index.html`

Slug rules: lowercase, hyphens only, match company name closely.
Example: "Aversboro Road Child Care Center" → `aversboro-road-child-care-center`

## FILE ARCHITECTURE
Every proposal ships two files:
- `proposals/[slug]/index.html` — the proposal itself (from master template)
- `proposals/[slug]/preview.html` — a fully built site concept for the client

The preview.html is a real single-page HTML site showing what we'd build — not a mockup image or placeholder. It loads inside the proposal iframe. Build it with the client's actual brand palette, services, testimonials, and content. It should feel like a working site, not a wireframe.

## MASTER TEMPLATE SYSTEM
The template at `_template/proposal-template.html` uses `{{PLACEHOLDER}}` variables. **Always fetch this file via GitHub API and fill placeholders — never generate proposal HTML from scratch.**

### Key Placeholders
| Placeholder | What it is |
|---|---|
| `{{COMPANY_NAME}}` | Client company name |
| `{{CONTACT_NAME}}` | Primary contact first name |
| `{{INDUSTRY}}` | Industry label |
| `{{AUDIENCE}}` | Industry-specific audience term (see below) |
| `{{CITY}}` | City |
| `{{HEADLINE}}` | Hero headline |
| `{{SUBHEAD}}` | Hero subheadline |
| `{{OPP_INTRO}}` | Opportunity section intro paragraph |
| `{{OPP_1_TITLE}}` through `{{OPP_4_TITLE}}` | Opportunity card titles |
| `{{OPP_1_BODY}}` through `{{OPP_4_BODY}}` | Opportunity card body copy |
| `{{DEL_1}}` through `{{DEL_8}}` | Deliverable names (solution section + pricing list) |
| `{{DEL_1_DESC}}` through `{{DEL_8_DESC}}` | Deliverable descriptions |
| `{{PREVIEW_URL}}` | URL used in the preview iframe |
| `{{BUILD_FEE}}` | One-time build price (e.g. "1,997") |
| `{{CARE_PLAN}}` | Care plan name (e.g. "Growth Plan") |
| `{{CARE_MONTHLY}}` | Monthly care plan price (e.g. "197") |
| `{{BOOKING_URL}}` | Calendly or booking link (defaults to #contact if none) |
| `{{LETTER_BODY}}` | Personalized letter body copy |

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
| retail | customers |
| nonprofit | supporters |
| education | students and families |
| default | clients |

## PROPOSAL COPY TONE — NON-NEGOTIABLE

**The cardinal rule: We are a partner excited about the client's potential, never a critic who found their flaws.**

Every word of copy should pass this test: *would a prospect feel seen and understood, or would they feel judged?*

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
| "No lead capture form" | "Capturing homeowner inquiries around the clock" |
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
- Section headline: aspiration-forward ("Four opportunities worth capturing" not "Four things you're leaving on the table")
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
1. Fetches the master template via GitHub API
2. Fills placeholders using prospect/company data
3. Pushes `proposals/[slug]/index.html` to this repo
4. Builds and pushes `proposals/[slug]/preview.html` as a real site concept
5. Creates a Supabase proposal record
6. Updates prospect status to 'Proposal Generated'
7. Auto-sends if build_fee < $800, otherwise queues 'Ready for Review' in The Haus

## DESIGN REFERENCE
- Background: #080808 (near-black)
- Gold: #C6A44C
- Cream: #F4EFE6
- Fonts: Cormorant Garamond (display) + Plus Jakarta Sans (body)
- Icons: thin-stroke gold SVGs — never emoji
