# VIN Report Workflow (v2)

Standard procedure for processing a new VIN. Run in order.

---

## Phase 0: Collect Inputs

**Required from user:**
1. **VIN** (17 chars, no I/O/Q)
2. **Current mileage** (actual odometer reading)
3. **Asking price**
4. **Location** (city/state)
5. **Intended use** (daily driver, weekend, project, etc.)

**Optional but helpful:**
6. **Carfax URL** if available — `https://www.carfax.com/vehicle/{VIN}`
7. **Listing URL** (AutoTrader, Cars.com, FB Marketplace, etc.)
8. **Annual mileage plan** (if different from 12k/yr default)

If the user gives only a VIN, ask for the rest before running subagents. Assumptions about mileage/price tank the analysis.

---

## Phase 1: VIN Decode (NHTSA, free, no auth)

```
GET https://vpic.nhtsa.dot.gov/api/vehicles/decodevin/{VIN}?format=json
```

Extract: ModelYear, Make, Model, Trim, Engine, Transmission Style, Drive Type, Body Class, Plant Country/City.

If `Make` or `Model` is empty, abort.

---

## Phase 2: Recall Lookup (NHTSA, free)

```
GET https://api.nhtsa.gov/recalls/recallsByVehicle?make={MAKE}&model={MODEL}&modelYear={YEAR}
```

Note: Mercedes-Benz often returns 0 results when queried as "MERCEDES-BENZ" but works with "Mercedes-Benz" + the specific model code (e.g. "C300" instead of "C-Class").

For Takata recalls, note PA/Northeast = Zone B = elevated humidity-driven rupture risk.

---

## Phase 3: Carfax Fetch (when URL provided)

```
GET https://www.carfax.com/vehicle/{VIN}
```

Extract:
- Accident history
- Owner count
- Ownership timeline + states
- Service record count
- Annual mileage average
- Last service date/location/mileage
- Carfax Value (their internal benchmark — distinct from KBB)
- Disclosed issues from seller description
- Recent price drops
- Dealer name + location (cross-check against user-stated location)

**Why this matters:** Carfax data flips verdicts. A 1-owner / no-accident / 86-service-record car at 127k miles is fundamentally different from a 4-owner, lapsed-service car at the same mileage — even though the static specs are identical.

---

## Phase 4: 5 Parallel Research Subagents

Each subagent receives the full context (year, make, model, trim, engine, drivetrain, mileage, asking, location, intended use, Carfax findings).

### Subagent A — Reliability & known issues
Be specific to year+engine combo, not generic model. Search CarComplaints, RepairPal, NHTSA complaints, model-specific forums (SwedeSpeed, MBWorld, BimmerForums, etc.), Reddit, TSBs.

### Subagent B — Safety & recalls
IIHS ratings, NHTSA NCAP (note pre-2011 used old system, no overall stars), standard safety equipment, Takata zone classification, recall remedy status warnings.

### Subagent C — Market pricing (ENHANCED)
- Filter listings to ±30k miles primary, ±50k miles secondary
- Apply -$0.08/mile adjustment for mid-mileage (100k-200k)
- Distinguish dealer asking vs. private-party fair value
- If Carfax Value provided, anchor analysis to it (Carfax weights owner count, accidents, service history)
- Prioritize same-region listings
- Find 1-2 stale listings as negotiation leverage
- Confidence: High (8+ comps ±30k), Medium (5-7 ±50k), Low (<5)

### Subagent D — 5-year ownership cost
- Use Fuelly real-world MPG (NOT EPA)
- Regional insurance rates
- Premium fuel adjustment if applicable
- Realistic repair budget ($6k-$12k for German luxury at 100k+, $4.5k-$9.5k for AWD Volvo, etc.)
- Tire size + replacement cost (note staggered setups cost ~20% more)
- Depreciation curve at this age/mileage
- Note when annual cost approaches/exceeds vehicle value

### Subagent E — Sentiment, theft, parts
- Owner forums + Reddit + Edmunds reviews (3+ year owners)
- HLDI/NICB theft data
- Parts availability across RockAuto, FCP Euro, Pelican Parts
- DIY-vs-shop pricing arbitrage (especially relevant for German cars)
- Long-term recommendation

---

## Phase 5: Synthesize Verdict

Based on all inputs, choose ONE verdict headline:
- **BUY** (clear good deal with clean history)
- **BUY with conditions** (good base case, specific service items to verify)
- **Negotiate** (overpriced but acceptable car)
- **Proceed with caution** (mixed signals)
- **Walk away** (red flags outweigh positives)

Include:
- Carfax-anchored price comparison (if available)
- Open offer / target / walk-away dollar amounts
- Top 5-7 pros (specific, evidence-backed)
- Top 5-7 cons
- 5-year cost summary with monthly figure
- Year/engine-specific known issues with severity + cost ranges
- Pre-purchase inspection checklist tailored to known issues
- Specific service-record questions for the seller
- Walk-away signals (3-5 deal-killers)

---

## Phase 6: Render HTML

Load `assets/report-template.html`. Replace tokens. Add custom callout blocks where appropriate:
- `.overpriced` (red) — asking > Carfax/fair value
- `.fair-deal` (yellow) — asking ≈ fair value
- `.good-deal` / `.key-insight` with green border — Carfax GREAT VALUE
- `.esl-callout` (red border box) — model-specific killer issue (e.g., MB ESL motor)
- `.walk-away` (red) — deal breakers section

Write to `reports/{VIN}.html`.

---

## Phase 7: Update Index + Commit

Prepend a row to `index.html` `<tbody id="report-list">`:
```html
<tr>
 <td>{YYYY-MM-DD}</td>
 <td><a href="reports/{VIN}.html">{YEAR} {MAKE} {MODEL} {TRIM} ({MILES}k mi)</a></td>
 <td class="vin">{VIN}</td>
 <td>{VERDICT_HEADLINE}</td>
</tr>
```

Commit + push:
```bash
git add reports/{VIN}.html index.html
git commit -m "Add report for {VIN}: {YEAR} {MAKE} {MODEL}"
git push origin main
```

GitHub Pages rebuilds in ~1 min. Report is live at:
`https://stevempoole.github.io/vin-reports/reports/{VIN}.html`

---

## Quality Rules

1. **Don't fake precision.** Confidence labels are mandatory and reflect comp count.
2. **Stay specific to year+engine.** Issues vary dramatically by generation/powertrain.
3. **Cite real sources.** Every claim should trace to a URL.
4. **Tailor checklists.** Generic "check the engine" advice is useless — be specific to known failure modes.
5. **Date-stamp everything.** Pricing goes stale within weeks.
6. **Distinguish dealer asking from private-party fair.** This was a major mistake in early reports.
7. **Use Carfax Value as primary benchmark when available.** It already factors owner/accident/service history premium.
8. **Cross-check stated location vs. dealer location** in Carfax/listing.

---

## Limits the agent must respect

The report cannot tell the buyer:
- Accident or title history (requires Carfax/AutoCheck — fetch when URL provided)
- Whether recalls have been performed on this specific VIN (dealer lookup only)
- Service history beyond what Carfax records (request from seller)
- Current mechanical condition (PPI by indie specialist, $150-$200)
- Odometer accuracy

These limits are called out in every report's "Things This Report Cannot Tell You" section.

---

## Subagent Concurrency Note

OpenClaw caps subagent children at 8 per session. With 5 subagents per VIN, you can run two VINs in parallel by staggering — launch all 5 for VIN-1, then immediately launch 3 for VIN-2 (8 active), and queue the remaining 2 for VIN-2 to launch as slots free up.

For best throughput, use Opus 4.7 (`opus47` alias) for all subagents — significantly better at finding nuanced details (Cardone master cylinder caveat, balance shaft serial cutoffs, dealer vs. private-party pricing distinction, regional Takata zones).
