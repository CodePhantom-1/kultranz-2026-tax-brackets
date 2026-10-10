# 2026 US Income Tax Brackets & Standard Deductions — Federal + All 50 States + DC

![License: CC BY 4.0](https://img.shields.io/badge/license-CC%20BY%204.0-blue.svg)
![Tax year](https://img.shields.io/badge/tax%20year-2026-green.svg)
![Formats](https://img.shields.io/badge/formats-JSON%20%7C%20CSV-orange.svg)

A free, open, machine-readable dataset of **2026 US income tax parameters**: federal brackets and standard deductions, FICA rates and wage bases, and the income tax structure of **all 50 states + the District of Columbia** — for both **single** and **married filing jointly (MFJ)** filing statuses. Maintained by [kultranz.com](https://kultranz.com).

## What's inside

| | |
|---|---|
| Jurisdictions | **51** — all 50 states + DC |
| Filing statuses | single, married filing jointly |
| No wage income tax | 9 states (AK, FL, NV, NH, SD, TN, TX, WA, WY) |
| Flat-rate states | 13 (AZ, CO, GA, IA, ID, IL, IN, KY, LA, MI, NC, PA, UT) |
| Progressive | 28 states + DC |
| Federal brackets | 7 brackets × 2 filing statuses |
| FICA | Social Security 6.2% (wage base $184,500), Medicare 1.45%, Additional Medicare 0.9% over $200k single / $250k MFJ |
| 2026 federal standard deduction | $16,100 single / $32,200 MFJ |

**Row counts**

- `csv/federal_tax_2026.csv` — **14 rows** (7 single + 7 MFJ federal brackets, with standard deduction and FICA parameters on every row)
- `csv/states_tax_2026.csv` — **329 rows** in long format: 285 progressive bracket rows (28 states + DC), 26 flat-rate rows (13 states × 2 filing statuses), 18 rows for the 9 no-tax states
- `data/tax_2026.json` — the **canonical source**; the CSVs are derived from it, value-for-value

## File map

```
kultranz-2026-tax-brackets/
├── data/
│   └── tax_2026.json          # Canonical JSON: _meta, fica, federal, states{51}
├── csv/
│   ├── federal_tax_2026.csv   # 14 rows — federal brackets × filing status + FICA + std deduction
│   └── states_tax_2026.csv    # 329 rows — long format, one row per state × status × bracket
├── LICENSE                    # CC BY 4.0
└── README.md
```

## Quick start

### Python — marginal bracket lookup

```python
import json

with open("data/tax_2026.json") as f:
    tax = json.load(f)

def marginal_rate(jurisdiction, filing_status, taxable_income):
    """Marginal rate applied to the next dollar of income."""
    if jurisdiction == "federal":
        brackets = tax["federal"]["brackets"][filing_status]
    else:
        state = tax["states"][jurisdiction]
        if state["type"] == "none":
            return 0.0
        if state["type"] == "flat":
            return state["rate"]
        brackets = state["brackets"][filing_status]
    rate = brackets[0][1]
    for floor, r in brackets:
        if taxable_income >= floor:
            rate = r
    return rate

marginal_rate("federal", "single", 60_000)  # 0.22
marginal_rate("CA", "mfj", 300_000)         # 0.093
marginal_rate("TX", "single", 120_000)      # 0.0  (no wage income tax)
```

### JavaScript (Node) — same lookup

```js
const fs = require("fs");
const tax = JSON.parse(fs.readFileSync("data/tax_2026.json", "utf8"));

function marginalRate(jurisdiction, filingStatus, taxableIncome) {
  let brackets;
  if (jurisdiction === "federal") {
    brackets = tax.federal.brackets[filingStatus];
  } else {
    const state = tax.states[jurisdiction];
    if (state.type === "none") return 0;
    if (state.type === "flat") return state.rate;
    brackets = state.brackets[filingStatus];
  }
  let rate = brackets[0][1];
  for (const [floor, r] of brackets) if (taxableIncome >= floor) rate = r;
  return rate;
}

marginalRate("NY", "single", 100_000); // 0.059
```

### CSV (long format)

```python
import csv

with open("csv/states_tax_2026.csv") as f:
    rows = [r for r in csv.DictReader(f) if r["state"] == "CA" and r["filing_status"] == "single"]
# 10 California single bracket rows, ordered by income_floor_usd
```

## Data dictionary

### `csv/federal_tax_2026.csv`

| Column | Meaning |
|---|---|
| `tax_year` | 2026 |
| `filing_status` | `single` or `mfj` |
| `income_floor_usd` | Lower bound of the bracket (USD of taxable income) |
| `marginal_rate` | Marginal rate for that bracket, as a decimal (e.g. `0.22`) |
| `standard_deduction_usd` | 2026 federal standard deduction for that filing status |
| `fica_social_security_rate` / `fica_social_security_wage_base_usd` | 6.2% / $184,500 |
| `fica_medicare_rate` | 1.45% (no wage base) |
| `fica_medicare_additional_rate` | 0.9% Additional Medicare |
| `fica_medicare_additional_threshold_single_usd` / `..._mfj_usd` | $200,000 / $250,000 |

### `csv/states_tax_2026.csv`

| Column | Meaning |
|---|---|
| `state` | USPS two-letter code (`DC` = District of Columbia) |
| `state_name` | Full state name |
| `tax_type` | `none`, `flat`, or `progressive` |
| `filing_status` | `single` or `mfj` (one pair of rows per state) |
| `income_floor_usd` | Lower bound of the bracket; `$0` for flat states |
| `marginal_rate` | Rate as a decimal; empty for `none` states |
| `standard_deduction_usd` | State standard deduction for that filing status; empty when the state has none in this dataset |

## Sources & methodology

- **Federal brackets & standard deduction:** IRS Rev. Proc. 2025-32 (tax year 2026).
- **FICA wage base:** Social Security Administration, 2026 ($184,500). FICA rates (6.2% / 1.45% / 0.9%) are statutory.
- **State brackets & deductions:** Tax Foundation, *2026 State Individual Income Tax Rates and Brackets* (state revenue departments via kultranz methodology).
- See the full kultranz [methodology page](https://kultranz.com/pages/methodology/?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct).

### Caveats (from the source data)

- This dataset is an **estimate, not tax advice**. It excludes local/city income taxes (e.g. NYC, Philadelphia), credits, and itemized deductions.
- Washington and New Hampshire have no tax on wage income (modeled as `none`).
- A few states' MFJ standard deduction was set to 2× the single value where the exact figure was not in the source (KS, ME, MT, NE, NM, ND, OK, SC, VA, WI); the effect is small and disclaimed.
- Flat-rate states are represented as a single bracket starting at $0 at the statutory rate.
- Rates are decimals (multiply by 100 for percent). Bracket floors are USD of taxable income (after the standard deduction).

## Corrections

- **2026-10-09** — Corrected the 2026 state income tax structure for three states and republished all files. **Missouri** was previously modeled as a flat 2.0% tax; the real 2026 schedule is graduated — 0% on the first $1,348 of taxable income, then 2.0/2.5/3.0/3.5/4.0/4.5%, top 4.7% above $9,436 (same thresholds for single and MFJ; Missouri DOR 2026 schedule / Tax Foundation 2026). **Mississippi** was previously flat 4.0% from dollar zero; the real 2026 law (H.B. 1) exempts the first $10,000 (0% bracket) and applies 4.0% above. **Ohio** was previously flat 2.75% from dollar zero; the real 2026 law (H.B. 96) applies 2.75% only to nonbusiness income over $26,050 (0% below). `csv/states_tax_2026.csv` grew from 311 to 329 rows (261→285 progressive rows, 32→26 flat rows) and the flat-rate state count changed from 16 to 13.

## License

[CC BY 4.0](./LICENSE). You are free to share and adapt for any purpose, including commercially, with attribution:

> **kultranz.com** — source: [kultranz 2026 tax brackets dataset](https://github.com/CodePhantom-1/kultranz-2026-tax-brackets?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct), links to https://kultranz.com

## Commercial use & API access

The CC BY 4.0 license covers commercial use at no cost: attribution to kultranz.com is the only requirement, so you can ship this dataset inside commercial products, reports, and internal tools for free.

If your product needs live computed values rather than static files, use the hosted API instead. It powers the paycheck engine (take-home pay for any salary, state, and filing status), city-to-city comparisons, and salary percentiles, so you do not have to reimplement the tax engine yourself:

- **Free tier:** 25 requests/day
- **$5 lifetime key:** https://kultranz.com/pages/api-docs/?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct
- **RapidAPI tiers** (Pro, Ultra, Mega): https://rapidapi.com/cianot978/api/kultranz?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct
- **MCP server** for LLM and agent integration: https://api.kultranz.com/mcp

## More from kultranz.com

- 📖 [Methodology](https://kultranz.com/pages/methodology/?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct) — how every number in this dataset was sourced and checked
- 💵 [Paycheck Calculator](https://kultranz.com/tools/paycheck-calculator/?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct) — this dataset in action, federal + state + FICA
- 🔌 [API](https://kultranz.com/pages/api-docs/?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct) — hosted JSON endpoints, including a **$5 lifetime tier**
- 🏙️ Sibling dataset: [kultranz-cost-of-living-index](https://github.com/CodePhantom-1/kultranz-cost-of-living-index?utm_source=github&utm_medium=organic&utm_campaign=revtest-oct) — US metro cost-of-living (BEA RPP) + rent/home value/income
