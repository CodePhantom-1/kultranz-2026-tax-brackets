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
| Flat-rate states | 16 (AZ, CO, GA, IA, ID, IL, IN, KY, LA, MI, MO, MS, NC, OH, PA, UT) |
| Progressive | 25 states + DC |
| Federal brackets | 7 brackets × 2 filing statuses |
| FICA | Social Security 6.2% (wage base $184,500), Medicare 1.45%, Additional Medicare 0.9% over $200k single / $250k MFJ |
| 2026 federal standard deduction | $16,100 single / $32,200 MFJ |

**Row counts**

- `csv/federal_tax_2026.csv` — **14 rows** (7 single + 7 MFJ federal brackets, with standard deduction and FICA parameters on every row)
- `csv/states_tax_2026.csv` — **311 rows** in long format: 261 progressive bracket rows (25 states + DC), 32 flat-rate rows (16 states × 2 filing statuses), 18 rows for the 9 no-tax states
- `data/tax_2026.json` — the **canonical source**; the CSVs are derived from it, value-for-value

## File map

```
kultranz-2026-tax-brackets/
├── data/
│   └── tax_2026.json          # Canonical JSON: _meta, fica, federal, states{51}
├── csv/
│   ├── federal_tax_2026.csv   # 14 rows — federal brackets × filing status + FICA + std deduction
│   └── states_tax_2026.csv    # 311 rows — long format, one row per state × status × bracket
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
- See the full kultranz [methodology page](https://kultranz.com/pages/methodology/).

### Caveats (from the source data)

- This dataset is an **estimate, not tax advice**. It excludes local/city income taxes (e.g. NYC, Philadelphia), credits, and itemized deductions.
- Washington and New Hampshire have no tax on wage income (modeled as `none`).
- A few states' MFJ standard deduction was set to 2× the single value where the exact figure was not in the source (KS, ME, MT, NE, NM, ND, OK, SC, VA, WI); the effect is small and disclaimed.
- Flat-rate states are represented as a single bracket starting at $0 at the statutory rate.
- Rates are decimals (multiply by 100 for percent). Bracket floors are USD of taxable income (after the standard deduction).

## License

[CC BY 4.0](./LICENSE). You are free to share and adapt for any purpose, including commercially, with attribution:

> **kultranz.com** — source: [kultranz 2026 tax brackets dataset](https://github.com/CodePhantom-1/kultranz-2026-tax-brackets), links to https://kultranz.com

## More from kultranz.com

- 📖 [Methodology](https://kultranz.com/pages/methodology/) — how every number in this dataset was sourced and checked
- 💵 [Paycheck Calculator](https://kultranz.com/tools/paycheck-calculator/) — this dataset in action, federal + state + FICA
- 🔌 [API](https://kultranz.com/pages/api-docs/) — hosted JSON endpoints, including a **$5 lifetime tier**
- 🏙️ Sibling dataset: [kultranz-cost-of-living-index](https://github.com/CodePhantom-1/kultranz-cost-of-living-index) — US metro cost-of-living (BEA RPP) + rent/home value/income
