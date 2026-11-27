# Pyroprocess Carbon Console

A single-file, browser-based console for tracking the CO₂ footprint of a cement kiln system (pyroprocess) month by month, built on a White Cement plant's own production data.

> **Purpose.** This console is a **process-evaluation tool**. It is meant for comparing carbon performance across months and for seeing how process shifts (fuel mix, raw-meal chemistry, clinker output, excess air, dust handling) move the carbon numbers. It is **not** a certified GHG inventory, an EPD, or a verified product carbon footprint.

---

## What it does

- **Compares months.** Pick any month or year-to-date (YTD) and every card recalculates from the same engine, so months can be compared on a like-for-like basis.
- **Shows process-shift effects.** Edit an input (fuel mass, carbon content, CaO/MgO, O₂ in flue gas, bypass dust, etc.) and see the effect on calcination CO₂, combustion CO₂, intensity per tonne of clinker and the mass balance immediately.
- **Cross-checks calcination two ways:**
  - **Method I** (headline): clinker CaO/MgO stoichiometry, following IPCC 2006 Tier 2 (Eq. 2.2) and GCCA V4.2 output method B2.
  - **Method II** (check): the raw-meal mass-balance difference, which is independent of the clinker XRF.
  - The console flags the gap between the two as Converged, Check assumptions, or Diverging.
- **Separates emission scopes:**
  - Scope 1: fossil CO₂.
  - Biogenic CO₂: reported as a memo and never subtracted.
  - Scope 2: grid electricity.
  - Scope 3: upstream fuel emissions.
- **Allocates to products.** Allocates to SKUs in two mass-allocation steps: first per tonne of clinker, then × the clinker-to-cement factor.
- **Checks mass-balance closure** of the kiln system (inputs vs outputs, with a closure %).
- **Shows data provenance.** Each emission factor carries a source badge (Strict Audit Mode): Sourced, Custom, Unverified, or Excluded.

## Scope boundary

| Included | Not included |
|---|---|
| Calcination CO₂ (CaCO₃, MgCO₃) | CH₄ and N₂O from combustion (so results are **CO₂ only**, not full GHG) |
| Fossil combustion CO₂ | CO₂ from organic carbon (TOC) in raw materials |
| Biogenic CO₂ (memo item) | Raw material extraction, transport, cement grinding, packaging |
| Scope 2 electricity (kiln and quench) | Cradle-to-gate stages outside the kiln boundary |
| Scope 3 upstream of purchased fuels | |

## Running it

There is no build step, server or database.

1. Open `console.html` in a modern browser.
2. All calculations run client-side; no data leaves the browser.
3. The page uses two external resources:
   - **SheetJS** (`xlsx` 0.18.5, from cdnjs), used only for Excel import.
   - **Google Fonts** (IBM Plex).
4. Offline, the calculations still work, but Excel import doesn't and the fonts fall back.

Edits are **not persisted**. Reloading the page restores the built-in baseline data.

## Repository contents

| File | What it is |
|---|---|
| `console.html` | The whole application: HTML, CSS and JS in one file |

---

## ⚠️ Flags and caveats

### 1. Excel upload / "Get the .xlsx template" is a demo

- **Upload Excel file (.xlsx)** reads only one fixed sheet layout (sheet name and row/column positions shown in the console's *Template layout reference*, visible when **Show how it's calculated** is on). Files in any other layout will not import correctly.
- **Get the .xlsx template** does **not** download anything. **No template file is included in this repository yet.** The button only opens an on-page *layout reference*.
- Treat both as a **demonstration of the import path**, not a supported data-entry workflow. A proper template and a validated importer are future work.

### 2. Air ratio is a theoretical value calculated from fuel composition

- The air ratio is the **theoretical (stoichiometric) air demand** for each fuel. It is calculated from the fuel's elemental composition:
  ```
  air ratio (kg air / kg fuel, as-fired) = 11.51·C + 34.34·(H − O/8) + 4.31·S
  combustion air = as-fed mass × air ratio × excess-air factor
  excess-air factor = 20.95 / (20.95 − O₂ in flue gas)
  ```
- It is **not** a measured air flow.
- For the plant's three current fuels (petcoke, rice husk, cracker bottom), there are no H/O/S lab results. The console therefore uses fixed theoretical values for these fuels.
- Air demand affects **only the mass balance**, not any CO₂ figure.

### 3. Oxidation factor (OF = 1.0) is a guideline default, not a plant measurement

- OF = 1.0 means all fuel carbon is assumed to be oxidised to CO₂.
- The value comes from the **IPCC 2006 Guidelines, Vol. 2 (Energy), Ch. 1**, default.
- It is consistent with the **WBCSD-CSI CO₂ Protocol** and the **GCCA Cement CO₂ and Energy Protocol V4.2**, which treat kiln-fuel carbon as fully oxidised.
- Unburnt carbon in clinker or dust has **not** been measured at this plant. OF can be edited in the assumptions panel if plant data becomes available.

### Other caveats

- **Fuel basis.** All fuel properties are entered on an **as-fired** basis. A dry → as-fired converter is provided. The as-fired basis of the current lab values still needs confirmation from the lab.
- **Method II monthly swings.**
  - Method II is the difference of two large numbers, so it amplifies raw-meal weighing error by about 2.8×.
  - It also swings with silo stock changes, so read it on a YTD or rolling basis.
  - Example: May is +23 % against Method I, but YTD agrees within +1.5 %.
- **Emission factors still to confirm.**
  - Rice husk upstream EF is 0 and marked *Unverified*.
  - Cracker bottom (CKB) upstream EF uses a heavy-fuel-oil proxy.
  - The ecoinvent electricity dataset includes upstream grid emissions.
- **Clinker-to-cement factor.** It is 0.92 for every SKU until mix designs are available.
- **Gray OPC data.** The `grayOPC` product line contains **proxy demo data**, not measured plant data.
- **Agreement threshold.** The 5 % tolerance between Method I and Method II is a project-set threshold. GCCA cites 5 % only as an example materiality level.

## Methods and references (summary)

| Calculation | Reference |
|---|---|
| Calcination, Method I | IPCC 2006 Vol. 3 Ch. 2, Tier 2 Eq. 2.2; GCCA V4.2 output method B2 |
| CKD correction (CFckd = 1.000, closed loop confirmed) | IPCC 2006 Vol. 3 Eq. 2.5 |
| Calcination, Method II | Mass-balance difference; comparable to GCCA input method A1 |
| Combustion CO₂ | IPCC 2006 Vol. 2 Ch. 1, carbon-content method |
| Biogenic CO₂ reported separately | GHG Protocol; ISO 14067 |
| Scope 2 | GHG Protocol, location-based; SimaPro (ecoinvent) TH medium-voltage grid |
| Scope 3 (fuels) | GHG Protocol Scope 3, Category 3; SimaPro exports |
| Product footprint | Two-step mass allocation (ISO 14067 approach) |
| Combustion air | Standard combustion stoichiometry |
| Dry → as-fired converter | ISO 18125 |

Full formulas are shown in the console itself: turn on **Show how it's calculated** and open *Methodology & formulas*.

## Known technical limitations

- No input range validation; for example, negative values or moisture above 100 % are accepted.
- Number fields don't accept thousands separators.
- There is no save function and no audit log.
- SheetJS 0.18.5 has known high-severity advisories (GHSA-4r6h-8v6p-xvw6, GHSA-5pgg-2g8v-p4x9). Upgrade before any production use.

## Roadmap

- Extend the boundary to cradle-to-gate, **or** export console results to LCA tools (e.g. SimaPro) for a full cradle-to-gate assessment.
- Add CH₄ and N₂O for a full GHG (CO₂e) result.
- Ship a real `.xlsx` template and a validated importer.
- Measure the oxidation factor and H/O/S fuel composition at plant level.
