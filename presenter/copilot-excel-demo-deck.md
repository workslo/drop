---
**Audience**: Tax AI Global Workshop attendees
**Presenter**: Shane Slosar, Tax Operations Sr. Associate
**Duration**: ~20 minutes (10 min live demo + 10 min discussion)
**Data**: 100% synthetic — fictional fund "MERIDIAN FLEX FUNDS", account `123456789`
**Figures**: recomputed from source data 2026-08-24 (realistic split-adjusted prices)
---

# Copilot in Excel — ABC Missing-Cost Reconciliation Demo

## Presentation Flow

```
 ┌───────────────────────────────────────────────────────────────────┐
 │  0. SETUP  (pre-room, not shown)                                  │
 │     Load SPL + WSR as named Excel Tables; RECON stays in a        │
 │     separate CLOSED workbook                                      │
 ├───────────────────────────────────────────────────────────────────┤
 │  1. CONTEXT SETTING  (~3 min)                                     │
 │     What is a missing-cost reconciliation?                        │
 │     What does the manual process look like?                       │
 │     What are we about to ask Copilot to do?                       │
 ├───────────────────────────────────────────────────────────────────┤
 │  2. LIVE DEMO  (~10 min, 5 escalating prompts)                    │
 │     Step 1: Aggregation         — "How big is the book?"          │
 │     Step 2: Unit comparison     — "Where do the numbers differ?"  │
 │     Step 3: Population diff     — "What's missing from each?"     │
 │     Step 4: Basis completeness  — "Which lots have no cost?"      │
 │     Step 5: Classification      — "What kind of break is each?"   │
 ├───────────────────────────────────────────────────────────────────┤
 │  3. WHAT COPILOT DID vs. WHAT A HUMAN STILL MUST DO  (~3 min)     │
 │     Strong use cases vs. weak/risky use cases                     │
 │     "Junior analyst" framing                                      │
 ├───────────────────────────────────────────────────────────────────┤
 │  4. OPEN DISCUSSION  (~5–10 min)                                  │
 │     "What would you try?"                                         │
 └───────────────────────────────────────────────────────────────────┘
```

---

## Section 0 — Setup (Before the Room Sees the Screen)

**Environment:** Excel on the web, Copilot pane. Primary model **Opus (4.8/5)**; backup
**GPT 5.6 extra think**. Web has no import wizard to declare column types — build the
workbook **in advance** (desktop import, or paste into columns pre-formatted as Text, or
leading zeros die silently), save it to OneDrive, and just open it on demo day.

**What to prepare:**

1. Open a blank Excel workbook.
2. Import [data/SPL_123456789.csv](data/SPL_123456789.csv) → format as Table → name it `tblSPL`.
3. Import [data/WSR_123456789.csv](data/WSR_123456789.csv) → format as Table → name it `tblWSR`.
4. *(Safety net)* Keep [data/RECON_123456789.csv](data/RECON_123456789.csv) in a **separate,
   closed workbook** — NOT a hidden tab. Copilot can read hidden sheets and named tables in
   the open workbook, which means a hidden answer key can be cheated from or cited aloud
   mid-demo. Only open the recon workbook if Copilot stalls live. Give that workbook a
   second tab from [data/RECON_LEGEND.csv](data/RECON_LEGEND.csv) — acronym glossary and
   column provenance, one tab away if the grid ever goes on screen.
5. Format the `Security ID` column (tblWSR) and `Product` column (tblSPL) as **Text** — prevents
   CUSIPs with leading zeros from being silently converted to numbers.
6. Open the Copilot pane. Close every other workbook (on web: every other workbook tab).
7. Set the model picker to **Opus (4.8/5)** and confirm **GPT 5.6 extra think** is
   available on the demo machine — model availability is a tenant/machine setting, not a
   personal one. Rehearse the full run on **both** engines and log variance per engine;
   Steps 2 and 5 are where they diverge most.

---

## Section 1 — Context Setting

### Slide: What Is a Missing-Cost Reconciliation?

**Presenter says (suggested script):**

> "When securities transfer between entities — custodians, prime brokers, internal fund
> restructures — the share positions and the cost basis sometimes travel separately.
>
> The shares show up in one system we call the SPL — the Share Position Ledger. That's
> custody. It knows *what you hold* and *how many*.
>
> The cost basis shows up in a different system we call the WSR — the Worksheet Report from
> GainsKeeper, the tax lot accounting engine. It knows *what you paid* for each lot.
>
> A missing-cost reconciliation answers one question: **for every position the fund holds,
> does the tax system have a cost basis for it?** If not, why not — and what do we do about it?"

### Slide: The Manual Process Today

**Presenter says:**

> "Today, a Tax Associate doing this review would:
>
> 1. Pull the SPL for the account — that's the custody position file.
> 2. Pull the WSR for the same account — that's the tax lot detail.
> 3. Build a VLOOKUP or pivot to match CUSIPs across both.
> 4. Eyeball every CUSIP where the numbers don't agree.
> 5. Investigate each break — is it a corporate action? A settlement timing issue? A
>    genuine missing cost?
> 6. Write up the findings and send them to a reviewer.
>
> On a small account that's an hour. On 12 accounts with 3,700 rows, that's a week.
>
> We're going to show Copilot doing steps 1 through 5 in about ten minutes."

### Slide: What We're Working With

| File | What it is | Rows | Key columns |
|------|-----------|------|-------------|
| `tblSPL` | Share Position Ledger — custody holdings | 58 | Product (CUSIP), TD (units), Product Desc |
| `tblWSR` | Worksheet Report — tax lot detail | 126 | Security ID (CUSIP), Units Utilized, Book Cost, Status |

> "These two tabs are already loaded. No formulas, no pivots — just raw data. Everything
> Copilot does from here, it builds live."

---

## Section 2 — Live Demo: The Five Prompts

---

### Step 1 — Simple Aggregation: "How Big Is the Book?"

#### What the presenter types into the Copilot pane:

```
In tblWSR, total the Book Cost by Security ID and show me the grand total.
```

#### What Copilot should return (pre-run result):

> **Copilot:**
>
> I totaled Book Cost by Security ID in tblWSR. Here's the summary:
>
> | Security ID | Security | Total Book Cost |
> |---|---|---:|
> | 369604301 | GE AEROSPACE (GE) | $19,897,920.00 |
> | 594918104 | MICROSOFT (MSFT) | $63,165.73 |
> | 78409V104 | S&P GLOBAL (SPGI) | $54,162.55 |
> | 037833100 | APPLE (AAPL) | $45,992.00 |
> | 64110L106 | NETFLIX (NFLX) | $38,837.40 |
> | ... | *(52 more rows)* | ... |
>
> **Grand Total: $20,782,173.27**
>
> One position — GE Aerospace — accounts for approximately 96% of the total book cost.

#### What the presenter says to the room:

> "One prompt. No pivot table, no SUMIF formula. Copilot read 126 rows across 57 CUSIPs,
> grouped them, summed them, and immediately flagged the concentration risk — GE Aerospace
> is 96% of the book. That's the thread we'll pull on."

#### Talking point for the room:

> *This is the same aggregation you'd build by hand in a pivot table. Copilot does it
> conversationally. Notice it also surfaced an insight — the concentration — without being
> asked. That's useful, but it's also where you start watching it carefully.*

---

### Step 2 — Unit Comparison: "Where Do the Numbers Differ?"

#### What the presenter types:

```
Compare total units per CUSIP between tblSPL (column "Product" = CUSIP, column "TD" = units)
and tblWSR (column "Security ID" = CUSIP, column "Units Utilized" = units). Show me every
CUSIP where the totals don't match.
```

#### What Copilot should return (pre-run result):

> **Copilot:**
>
> I compared unit totals per CUSIP across both tables. **13 CUSIPs** have mismatched totals:
>
> | CUSIP | Name | SPL Units | WSR Units | Difference |
> |---|---|---:|---:|---:|
> | 369604301 | GE Aerospace | 100,000 | 120,000 | −20,000 |
> | 654106103 | Nike Inc. Class B | 250 | 140 | +110 |
> | 458140100 | Intel Corporation | 2 | 3 | −1 |
> | 549498103 | Lucid Group Inc. | 3 | 1 | +2 |
> | 067901108 | Barrick Gold Corp | 200 | 0 | +200 |
> | 585055106 | Medtronic Inc. | 150 | 0 | +150 |
> | 68389X105 | Oracle Corporation | 240 | 0 | +240 |
> | 70450Y103 | PayPal Holdings Inc. | 180 | 0 | +180 |
> | 90353T100 | Uber Technologies | 320 | 0 | +320 |
> | 06849F108 | Barrick Mining Corp | 0 | 200 | −200 |
> | 125523100 | Cigna Group | 0 | 95 | −95 |
> | 78409V104 | S&P Global Inc. | 0 | 130 | −130 |
> | G5960L103 | Medtronic plc | 0 | 150 | −150 |
>
> The remaining **49 CUSIPs** match exactly between both tables.

#### What the presenter says:

> "One prompt just did what would normally be a VLOOKUP-and-eyeball exercise across two
> exports. We now have 13 CUSIPs that don't agree — but 'different totals' isn't the same
> as 'a real break.' That's Step 3."

---

### Step 3 — Population Diff: "What's Missing from Each Side?"

#### What the presenter types:

```
Of those 13 mismatches, which CUSIPs appear only in tblSPL with zero in tblWSR,
and which appear only in tblWSR with zero in tblSPL?
```

#### What Copilot should return (pre-run result):

> **Copilot:**
>
> **SPL Only** (5 CUSIPs — position in custody, no tax lots):
>
> | CUSIP | Name | SPL Units |
> |---|---|---:|
> | 067901108 | Barrick Gold Corp | 200 |
> | 585055106 | Medtronic Inc. | 150 |
> | 68389X105 | Oracle Corporation | 240 |
> | 70450Y103 | PayPal Holdings Inc. | 180 |
> | 90353T100 | Uber Technologies Inc. | 320 |
>
> **WSR Only** (4 CUSIPs — tax lots exist, no custody position):
>
> | CUSIP | Name | WSR Units | Book Cost |
> |---|---|---:|---:|
> | 06849F108 | Barrick Mining Corp | 200 | $4,035.00 |
> | 125523100 | Cigna Group | 95 | $15,884.00 |
> | 78409V104 | S&P Global Inc. | 130 | $54,162.55 |
> | G5960L103 | Medtronic plc | 150 | $11,527.50 |
>
> The remaining **4 mismatches** (GE, Nike, Intel, Lucid) appear in both tables but with
> different unit counts — those are quantity breaks, not population breaks.

#### What the presenter says:

> "Now here's the moment I want the room to see. Look at those SPL-only and WSR-only
> lists side by side. Does anyone notice anything?"
>
> *(Pause for the room. Someone may spot it.)*
>
> "Barrick Gold Corp — 200 units, SPL only. Barrick *Mining* Corp — 200 units, WSR only.
> Same number, similar name, different CUSIP. That's not a break — it's a corporate action.
> The company changed its name and CUSIP. The position is the same.
>
> Same story with Medtronic Inc. and Medtronic plc — a redomiciliation.
>
> Copilot surfaced the data. A human caught the pattern. That's the partnership."

---

### Step 4 — Basis Completeness: "Which Lots Have No Cost?"

#### What the presenter types:

```
In tblWSR, flag every row where Book Cost is blank or empty. Group by Security ID
and show me the total units affected per CUSIP.
```

#### What Copilot should return (pre-run result):

> **Copilot:**
>
> I found **8 tax lots** with blank Book Cost, across **8 CUSIPs**:
>
> | Security ID | Security | Units | Status |
> |---|---|---:|---|
> | 456788108 | Infosys Limited ADR (INFY) | 500 | UT |
> | 539439109 | Lloyds Banking Group ADR (LYG) | 340 | UT |
> | 783513203 | Ryanair Holdings ADR (RYAAY) | 150 | UT |
> | G0403H108 | Aon plc (AON) | 279 | UT |
> | G0450A105 | Arch Capital Group Ltd. (ACGL) | 122 | UT |
> | G7997R103 | Seagate Technology (STX) | 88 | UT |
> | N07059210 | ASML Holding N.V. (ASML) | 64 | UT |
> | N82405106 | Stellantis N.V. (STLA) | 410 | UT |
>
> **Total units with missing cost: 1,953**
>
> All 8 lots have Status = "UT" (Unsettled Transfer) and no Buy Date, Book Cost, or Tax
> Cost populated.

#### What the presenter says:

> "This is the actual missing-cost population. Eight securities, 1,953 units, all showing
> Status 'UT' — Unsettled Transfer. No buy date, no cost, no tax cost. Every single one.
>
> That pattern — UT with blanks across every cost field — tells us these lots arrived via a
> transfer where the delivering entity never sent cost basis.
>
> Point out that this list ties exactly to a file already sitting in the data folder —
> `MISSING_COST_123456789.csv`. Copilot just reproduced, live, the same exception list it
> took a Python script to build the first time."

---

### Step 5 — Classification and Materiality: "What Kind of Break Is Each?"

#### What the presenter types:

```
For every CUSIP that isn't a clean match between tblSPL and tblWSR, classify it as one of:
- Corporate Action (appears only on one side but the same position exists under a
  different CUSIP on the other side)
- Missing Basis (units match but Book Cost is blank)
- Quantity Break (appears on both sides but unit totals differ)
- Population Break (appears only on one side with no corresponding CUSIP on the other)

For Quantity Breaks and Population Breaks, estimate the dollar impact using the average
Book Cost per unit from tblWSR. Mark it "Material" if the estimate exceeds $10,000.
```

#### What Copilot should return (pre-run result):

> **Copilot:**
>
> Here's the break classification:
>
> | CUSIP | Ticker | Category | SPL Units | WSR Units | Est. Impact | Material? |
> |---|---|---|---:|---:|---:|---|
> | 067901108 | GOLD | Population Break — SPL Only | 200 | 0 | — | *(see note)* |
> | 06849F108 | B | Population Break — WSR Only | 0 | 200 | $4,035 | No |
> | 585055106 | MDT | Population Break — SPL Only | 150 | 0 | — | *(see note)* |
> | G5960L103 | MDT | Population Break — WSR Only | 0 | 150 | $11,528 | Yes |
> | 369604301 | GE | Quantity Break | 100,000 | 120,000 | $3,316,320 | Yes |
> | 654106103 | NKE | Quantity Break | 250 | 140 | $7,785 | No |
> | 458140100 | INTC | Quantity Break | 2 | 3 | $40 | No |
> | 549498103 | LCID | Quantity Break | 3 | 1 | $43 | No |
> | 68389X105 | ORCL | Population Break — SPL Only | 240 | 0 | — | — |
> | 70450Y103 | PYPL | Population Break — SPL Only | 180 | 0 | — | — |
> | 90353T100 | UBER | Population Break — SPL Only | 320 | 0 | — | — |
> | 125523100 | CI | Population Break — WSR Only | 0 | 95 | $15,884 | Yes |
> | 78409V104 | SPGI | Population Break — WSR Only | 0 | 130 | $54,163 | Yes |
> | 456788108 | INFY | Missing Basis | 500 | 500 | — | — |
> | 539439109 | LYG | Missing Basis | 340 | 340 | — | — |
> | 783513203 | RYAAY | Missing Basis | 150 | 150 | — | — |
> | G0403H108 | AON | Missing Basis | 279 | 279 | — | — |
> | G0450A105 | ACGL | Missing Basis | 122 | 122 | — | — |
> | G7997R103 | STX | Missing Basis | 88 | 88 | — | — |
> | N07059210 | ASML | Missing Basis | 64 | 64 | — | — |
> | N82405106 | STLA | Missing Basis | 410 | 410 | — | — |
>
> **4 material breaks**, led by GE Aerospace at an estimated $3.32M impact.
> The 8 Missing Basis items have matching units but no cost data to estimate impact.

#### What the presenter says (this is the payoff):

> "Copilot didn't quite get the corporate-action pairs. It classified Barrick Gold and
> Barrick Mining as separate population breaks — which is technically correct from the data,
> but a human reviewer knows those are the same 200 shares.
>
> So let's steer it."

#### Follow-up prompt (the "domain context" nudge):

```
Barrick Gold Corp (067901108) and Barrick Mining Corp (06849F108) are the same position
before and after a corporate action — treat them as a reclassification, not two separate
breaks. Same for Medtronic Inc. (585055106) and Medtronic plc (G5960L103). Reclassify
those four rows as "Corporate Action — Waive" and update the table.
```

#### What Copilot returns after the nudge:

> **Copilot:**
>
> Updated. Reclassified those two pairs as Corporate Action — Waive:
>
> | CUSIP | Ticker | Category | Disposition |
> |---|---|---|---|
> | 067901108 | GOLD | Corporate Action | Waive — paired with 06849F108 |
> | 06849F108 | B | Corporate Action | Waive — paired with 067901108 |
> | 585055106 | MDT | Corporate Action | Waive — paired with G5960L103 |
> | G5960L103 | MDT | Corporate Action | Waive — paired with 585055106 |
>
> Remaining material breaks: **3** (GE $3.32M, S&P Global $54.2K, Cigna $15.9K).

#### What the presenter says:

> "That's the demo. Five prompts. We went from two raw data files to a fully classified,
> materiality-tiered break list with dispositions — in about ten minutes.
>
> But notice what just happened in that last exchange. Copilot needed a human to tell it
> that Barrick Gold and Barrick Mining are the same company. That's domain knowledge. An
> AI can match numbers; it can't know your corporate action history unless you tell it.
>
> That's the partnership model. Copilot handles the data grunt work — the joins, the
> aggregations, the comparisons. You bring the judgment."

---

## Section 3 — What Copilot Did vs. What a Human Still Must Do

### Slide: Strong Use Cases (What Copilot Just Demonstrated)

| Capability | What we saw |
|---|---|
| Exception aggregation | Grand total, per-CUSIP subtotals, concentration flag |
| Cross-table comparison | CUSIP-level unit matching across SPL and WSR |
| Population identification | SPL-only vs WSR-only bucketing |
| Data profiling | Blank-cost detection, Status-field pattern recognition |
| Break classification | Category assignment + materiality tiering |
| Steering with domain context | Corporate-action reclassification on follow-up prompt |

### Slide: Weak / Risky Use Cases (Where Humans Stay in the Loop)

| Risk area | Why |
|---|---|
| Autonomous root-cause determination | "No cost received from delivering entity" is a *conclusion*, not a data point. Copilot can surface the evidence (blank fields, UT status), but the conclusion requires a reviewer who understands the transfer mechanism. |
| Final accounting or regulatory conclusions | SOX 404, PCAOB expectations, and EUC controls all require human sign-off on any figure that feeds a financial statement. |
| Deterministic reconciliations without review | Copilot outputs are non-deterministic — the same prompt can produce slightly different formulas or groupings on successive runs. Any number that matters must be independently verified. |
| Corporate-action identification | Copilot matched numbers but couldn't link Barrick Gold → Barrick Mining without explicit instruction. Matching across name/CUSIP changes requires domain knowledge or a reference table. |
| Hardcoded values replacing formulas | A known Copilot behavior: sometimes it returns a static number instead of a formula. If the underlying data changes, the static number won't update. Always verify that Copilot wrote a formula, not a value. |

### Slide: The One-Liner

> **Think of Copilot as a junior analyst who is very fast at reading tables and writing
> formulas — but who needs you to check every number, catch every corporate action, and
> sign off on every conclusion.**

---

## Section 4 — Open Discussion

### Suggested prompts for the room:

1. "What reconciliation in your workflow has the most manual VLOOKUP/pivot/eyeball time?
   Could Copilot take a first pass?"

2. "Where would you *not* trust this output without a second check? What's the control?"

3. "If you could teach Copilot one thing about your process — like we just taught it about
   the Barrick corporate action — what would it be?"

---

## Appendix A — Data Files Reference

| File | Rows | Schema | Purpose |
|------|------|--------|---------|
| `SPL_123456789.csv` | 58 | Company, Product (CUSIP), Product Type, Account, Balance Type, Date, TD, SD, Acct Desc, Product Desc | Custody position snapshot |
| `WSR_123456789.csv` | 126 | Security ID (CUSIP), Security, Buy Date, Units Utilized, Price Per Unit, Book Cost, Tax Cost, GL Term, Status | Tax lot detail |
| `MISSING_COST_123456789.csv` | 8 | fund, account, cusip, symbol, long_short, units, cost_received_from_contra, basis_at_inception | Pre-generated exception list (safety net) |
| `RECON_123456789.csv` | 62 | CUSIP, Ticker, Security Name, SPL Units, WSR Lot Count, WSR Units Total, Unit Difference, Match Status, WSR Book Cost Total, Lots Missing Basis, Basis Status, Break Category, Materiality, Reviewer Note | Pre-generated recon grid (separate closed workbook) |
| `master-roster_taxai-demo.csv` | 62 | CUSIP, Ticker, Issuer Name, BREAK_FLAG, SPL_UNITS, WSR_UNITS, LOT_COUNT, NOTES | Source-of-truth roster (not shown to audience) |
| `RECON_LEGEND.csv` | 28 | Type, Item, Source, Explanation | Glossary + column provenance; second tab of the recon workbook |

All files regenerated 2026-08-24 from the same source data; every figure ties.

**RECON column provenance:** CUSIP through Basis Status are derived mechanically from
SPL + WSR — the columns Copilot rebuilds live in Steps 1–4. Break Category, Materiality,
and Reviewer Note are analyst-authored — the corporate-action linkage, the dispositions,
the sign-off. No prompt fills those in; that is the point of the demo.

## Appendix B — Key Numbers for Presenter Reference

| Metric | Value |
|---|---:|
| Total CUSIPs in roster | 62 |
| Clean matches | 41 |
| Total breaks (all categories) | 21 |
| WSR grand total Book Cost | $20,782,173.27 |
| GE Aerospace Book Cost (5 lots × 24,000 units each) | $19,897,920.00 |
| GE Aerospace % of book | ~95.8% |
| Missing-basis CUSIPs | 8 |
| Missing-basis total units | 1,953 |
| Corporate-action pairs | 2 (Barrick Gold/Mining, Medtronic Inc/plc) |
| SPL-only CUSIPs | 5 (3 genuine + 2 corp-action pre-event) |
| WSR-only CUSIPs | 4 (2 genuine + 2 corp-action post-event) |
| Quantity break CUSIPs | 4 (GE, Nike, Intel, Lucid) |
| Largest single break (est.) | GE Aerospace: $3,316,320 (20,000-unit gap) |
| Materiality threshold | $10,000 |
| Material breaks (pre-nudge → post-nudge) | 4 → 3 |

## Appendix C — Recovery Playbook

| Scenario | What to do |
|---|---|
| **Copilot reads CUSIPs as numbers** (drops leading zeros) | Format column as Text before demo. If it happens live: *"Treat the CUSIP column as text, not a number."* |
| **Corporate-action pairs not auto-linked** | Expected. Use the follow-up nudge prompt from Step 5. Frame it as showing how to steer an agent with domain context, not as a failure. |
| **Copilot returns a hardcoded value instead of a formula** | Point it out to the room: *"See how it wrote $20,782,173.27 as a static number instead of =SUM(...)? That's a known behavior. If a row changes, that number won't update. Always check."* |
| **Copilot stalls or gives an odd response** | Three rungs, in order. ① Re-run the same prompt. ② Switch the model picker to **GPT 5.6 extra think** and re-run — narrate it: *"Same prompt, different model — that's the non-determinism point, live."* (Extra think is slower; budget ~a minute.) ③ Open the separate closed RECON workbook and walk the grid: *"This is what the output looks like — let me show you the recon Copilot would have built."* Then the provenance line: *"Everything left of Break Category came from the data — that's what you just watched Copilot rebuild. The last three columns are where the analyst lives, and no prompt fills them in."* |
| **Step 2 balks at the cross-table comparison** | Ask for the mechanism instead of the outcome: *"Add a column to tblSPL that XLOOKUPs total Units Utilized from tblWSR by CUSIP, then show me every row where it differs from TD."* Less magical-looking, nearly unfailable. |
| **Step 5 comes back malformed on a single pass** | Split it: classify first, then a second prompt for impact + materiality. Two prompts instead of one is a rehearsal-day call, not a failure. |
| **Room asks about real data** | *"This is 100% synthetic demo data — a fictional fund called Meridian Flex. The patterns are modeled on a real review I completed, but no real client or account data is shown."* |

## Appendix D — Glossary

Spell out on first spoken use; this table is the safety net (also the recon workbook's
Legend tab, for a global audience where even CUSIP isn't universal).

| Term | Meaning |
|---|---|
| ABC | Fictional client name — the demo's stand-in institution |
| SPL | Share Position Ledger — custody position file: what you hold, how many |
| WSR | Worksheet Report — GainsKeeper tax lot detail: what you paid, lot by lot |
| CUSIP | 9-character North American security identifier; leading zeros are part of the ID |
| UT (Status) | Unsettled Transfer — lot arrived via transfer, cost basis not yet received |
| C (Status) | Complete — settled lot with basis fully populated |
| GL Term: lt / st | Long-term / short-term holding period for gain-loss |
| TD / SD | Trade-date / settle-date units in the SPL |
| Book Cost vs Tax Cost | Book vs tax basis; equal in this data, can diverge in practice |
| Contra | The delivering entity on the other side of a transfer |
| Basis at inception | Whether cost basis was known when the position arrived |
| Corporate action | Issuer event (name/CUSIP change, redomiciliation) that moves a position to a new identifier |
| Materiality | Review threshold — estimated impact above $10,000 |
| Balance Type 6 / Product Type CUS | Feed codes from the custody extract; not used by the recon |
