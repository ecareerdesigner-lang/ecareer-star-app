# eCareer STAR Response Assistant

A real, deployable version of the USPS eCareer application-drafting app. This
moves AI generation and payment behind a proper backend, which is the piece
that couldn't work reliably inside the Claude.ai artifact preview.

Not an official USPS product — internal drafting tool only.

## What changed from the artifact prototype

- All calls to the Anthropic API now happen server-side (`pages/api/*.js`),
  using a real `ANTHROPIC_API_KEY` from environment variables — never exposed
  to the browser.
- Requirement extraction from pasted text tries a local, no-API numbered-list
  parser first (`lib/parse.js`) and only falls back to the AI extraction
  endpoint for unstructured text.
- Payment goes through real Stripe Checkout (`pages/api/checkout.js`) instead
  of a simulated button.

## Setup

```bash
npm install
cp .env.local.example .env.local
# then edit .env.local and add your real ANTHROPIC_API_KEY and STRIPE_SECRET_KEY
npm run dev
```

Visit `http://localhost:3000`.

## Deploying

This is a standard Next.js app — it deploys as-is to Vercel, or any host
that runs Next.js. Set `ANTHROPIC_API_KEY` and `STRIPE_SECRET_KEY` as
environment variables in your hosting provider's dashboard (don't rely on
`.env.local`, which isn't committed to git).

## Making adjustments

**Price and promo codes** — everything lives in `lib/config.js`:

```js
export const FLAT_FEE_CENTS = 5000; // change the flat fee here
export const PROMO_CODES = {
  LAUNCH50: { type: "percent_off", value: 50 },
  SAVE10: { type: "amount_off", value: 1000 }, // $10 off, in cents
};
```

Uncomment/add entries and they immediately apply at checkout — no other code
changes needed.

**Name and branding** — also in `lib/config.js` (`APP_NAME`, `APP_TAGLINE`),
plus the color tokens at the top of the `Home` component in `pages/index.js`
(`navy`, `red`, `paper`, `ink`, `brass`) if you want a different palette.

**Job title library** — `JOB_LIBRARY` in `lib/parse.js`. Add more entries as
`"job title (lowercase)": { title: "...", requirements: [...] }`.

**Character budget** — `TOTAL_CHARACTER_BUDGET` in `lib/config.js` (currently
6000, matching the eCareer combined KSA limit).

## Before this handles real applicant data

- Add a Stripe webhook (`pages/api/webhook.js` listening for
  `checkout.session.completed`) so payment confirmation happens server-side
  rather than trusting the `?paid=1` redirect alone — the current setup is
  fine for a v1 but that redirect could technically be visited manually.
- Add a real database (e.g. Postgres via Supabase) to persist applications
  and reusable background profiles across sessions — right now all state
  lives in memory in the browser tab and is lost on refresh.
- Add authentication so background profiles are tied to a specific user.
- Encrypt background-profile data at rest, since it contains career/PII
  details, and give users a way to delete their data.
- This app assists in *drafting* — add a visible reminder that the applicant
  should review responses for accuracy before submitting, since eCareer
  responses are attested by the applicant.

## Project structure

```
pages/
  index.js          # main UI — all 6 steps
  api/
    extract.js      # AI fallback for parsing pasted qualifications
    generate.js      # per-requirement STAR response generation
    summary.js       # Special Skills & Associations summary
    checkout.js      # Stripe Checkout session creation
lib/
  config.js         # pricing, promo codes, branding, character budget
  parse.js          # local numbered-list parser + demo job library
  claude.js         # server-side Anthropic API client
```
