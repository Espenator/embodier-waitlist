# Waitlist — public page with live numbers (system 7 of 7)

Public bilingual (ES primary / EN toggle) waitlist page for the Embodier token
program: live counter, tier explainer, $100 refundable-reservation mechanic,
allocation windows, signup → intake → confirmation, referral hook.

## Live counter — the real sheet + cell

| | |
|---|---|
| Sheet | **Token Buyer Prospects** `1ft2aTXGkg-PXvLRm_9gfMyetbMe5EcDwNnC77NXGRyU` |
| Tab | `Waitlist` |
| Cell | **`Waitlist!B1`** |
| Formula | `=COUNTA(C3:C)` — counts non-empty **Email** cells from row 3 down (row 1 = live-count label, row 2 = headers) |
| Capital-route tab | Capital Partners sheet `1bEPUJEGlGji9KNiUJZeEFasFhNDi7KUMVpkrMlKKZZw`, tab `Waitlist`, cell `Waitlist!B1` (same formula) — tier $50K+ signups land here |

**How the number reaches the page:** `sync-count.py` reads `Waitlist!B1` via
`hatch_gws_cli` and writes `waitlist-count.json` (same directory as the page).
The page fetches that JSON every 5 minutes. **No number is hardcoded in
`index.html`** — before the JSON loads the counter shows "—", never a fake
number. Run the sync on a 15-minute cron:

```
*/15 * * * * /usr/bin/python3 ~/workspace/goals/latam-resort-tokenization-acquisition/hidden_files/embodier-automation/waitlist/sync-count.py
```

(After the repo is Pages-enabled, the cron must also push the refreshed
`waitlist-count.json` — see "Going live".)

## Page — `index.html`

Single self-contained file (inline CSS/JS). Photography is real Pexels-licensed
imagery (free commercial use — never AI imagery): `index.html` references the
Pexels CDN URLs directly because the repo commit path available to the machine
is text-only and cannot carry binary JPEGs. The canonical verified JPEG copies
are kept in the workspace at `waitlist/assets/` (hero-machu-picchu.jpg,
sacred-valley.jpg, beach-resort.jpg, visually verified). When a binary-capable
push path exists, swap the three `images.pexels.com` URLs in `index.html` for
relative `assets/*.jpg` paths and commit the JPEGs alongside.

- **Language toggle top-right** — ES default (Spanish primary for LATAM), EN
  toggle; all 76 strings in both languages; choice persists in localStorage.
- **Brand bar** — hot link `www.embodier.ai`.
- **Contact strip** — hot links only: `https://www.embodier.ai`,
  `sms:+12397775813`, `mailto:espen@embodier.ai`. **Never WhatsApp** (capital-lane rule).
- **Endline** — "Stay. Own. Return."
- **Compliance block** — whitelist/soft commitments only; no live token sale;
  no investment contract; no promised returns; DASP gates public sale; KYC/AML
  + jurisdiction eligibility. No yield projections, no fee/carry economics.
- **$100 mechanic** — fully refundable Stripe reservation, credited toward the
  $500 minimum at the offering; never framed as an investment or return-bearing.
- **Config block (top of `<script>`)** —
  `INTAKE_WEBHOOK_URL` (empty until the intake is deployed; the form then shows
  an honest manual-fallback panel with a prefilled mailto — it never fakes a signup),
  `STRIPE_DEPOSIT_URL` (empty until Espen provides the payment link; the button
  then shows a "coming soon" note instead of a dead link).
- **Referral hook** — reads `?ref=` into a hidden field, passes it to intake;
  on success shows the sharer's own link `?ref=CODE` + copy + email/SMS share
  (WhatsApp share only for token tiers — never for the $50K+ capital tier).

## Intake — `waitlist-intake/`

`index.js` — Google Cloud Function (Node 20), adapted from the standing
`intake-webhook-template`. `waitlist-intake.gs` — Apps Script twin for fast
deploy bound to the Token sheet (recommended first deploy).

`POST { name, email, phone, tier, ref, lang, source }`

| tier | lane | sheet |
|---|---|---|
| `tier-50k` | `capital` | Capital Partners sheet |
| `tier-5k` | `token-buyer` | Token Buyer sheet |
| `tier-500` | `token-buyer` | Token Buyer sheet |

Pipeline: validate → **Suppression List check** (`SUPPRESSION_SHEET_ID` env /
Script Property, tab `Suppression List`; match on normalized email/phone;
rejects with 409; fails open but flags the row `suppression-unverified`) →
dedupe by email (idempotent, returns existing ref code) → append row →
`{ ok, lane, ref_code }`. **Never sends email.**

Env: `TOKEN_SHEET_ID`, `CAPITAL_SHEET_ID`, `WAITLIST_TAB` (default `Waitlist`),
`SUPPRESSION_SHEET_ID`, `SUPPRESSION_TAB` (default `Suppression List`).
No secrets in code — ADC auth.

## Confirmation emails — `confirmation-email/`

`build_confirmation.py` builds 4 staged `.eml` files
(`staged/waitlist-confirm-{token,capital}-{ES,EN}.eml`) via the standing
`email_kit`: full Espen signature, CID-inline hero image (720px q82, never
external URLs), lane-correct contact strip (capital = call/SMS, **no
WhatsApp**), compliance disclaimer, `{{REF_CODE}}`/`{{NAME}}`/`{{TIER_LABEL}}`
placeholders. **Staged only — nothing is sent.** Wire into the nurture
sequencer when the intake goes live.

Rebuild: `python3 confirmation-email/build_confirmation.py`
(uses `STRIPE_DEPOSIT_URL` / `WAITLIST_PAGE_URL` env when set).

## Going live — wiring checklist

1. **Deploy intake** — paste `waitlist-intake.gs` into the Token sheet's bound
   Apps Script, set Script Properties (`CAPITAL_SHEET_ID`,
   `SUPPRESSION_SHEET_ID`), Deploy → Web app (Execute as Me, Anyone).
   _Or_ deploy `index.js` as a Cloud Function with the env vars above.
2. **Set `INTAKE_WEBHOOK_URL`** in `index.html` to the web app URL.
3. **[BLOCKED] Stripe payment link** — Espen creates the **$100 USD refundable
   reservation** payment link in Stripe and provides the URL → set
   `STRIPE_DEPOSIT_URL` in `index.html` (and rebuild the confirmation emails).
   No invented links, ever.
4. **Publish the page** — enable GitHub Pages on `Espenator/machupicchu-cloud`
   (Settings → Pages → Deploy from branch `main`, root) → page serves at
   `https://espenator.github.io/machupicchu-cloud/embodier-automation/waitlist/`.
   Then point the count-sync cron at committing the refreshed JSON.
5. **Verify per the publish rule** — URL opens, counter shows the real number
   (read back `Waitlist!B1` to confirm), form submits end-to-end, confirmation
   email staged. Never report "live" without this read-back.

## Files

```
waitlist/
├── index.html                  # the page (ES primary, EN toggle)
├── assets/                     # canonical local copies (Pexels license, verified;
│   ├── hero-machu-picchu.jpg  #  workspace-only: commit path is text-only, so
│   ├── sacred-valley.jpg      #  index.html uses the Pexels CDN URLs directly)
│   └── beach-resort.jpg
├── waitlist-count.json          # synced live counter (never hand-edited)
├── sync-count.py               # sheet cell -> JSON (cron every 15 min)
├── waitlist-intake/
│   ├── index.js                # Cloud Function (Node 20)
│   ├── package.json
│   └── waitlist-intake.gs      # Apps Script twin (fast deploy)
├── confirmation-email/
│   ├── build_confirmation.py
│   └── staged/                 # 4 staged .eml drafts — NEVER auto-sent
└── README.md
```

_Standing rules honored: lane firewall, PachaMind firewall, no secrets in code,
Spanish-primary, brand hot-links, real photography only, publish verification._
