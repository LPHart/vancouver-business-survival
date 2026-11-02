# Vancouver Business Survival: Building a Business-Level Dataset from 30 Years of Licences

This notebook turns the City of Vancouver's business licence records (1997 to 2026, about 1.95 million rows) into a clean dataset that follows each business from the year it opened to the year it closed. The output feeds a Tableau analysis of which kinds of businesses last in Vancouver, and in which neighbourhoods.

It was built for the group project "Where Should You Start a Business in Vancouver?" in BAIT 518 Data Visualization (UBC Sauder School of Business, Master of Business Analytics), October 2026.

## The problem

The City publishes one row per **licence per year**, with **no business identifier**. A business renewed for 20 years shows up as 20 unrelated rows, chains share a name across dozens of addresses, and the City replaced its entire list of business types in 2024. You cannot count openings, closures, or survival until you know which licences belong to the same business.

## What the notebook does

| Step | What happens |
|---|---|
| Load | Reads three semicolon-separated files (all columns as text, 15 of 25 kept) |
| Deduplicate | Removes the 2024 overlap between files on `LicenceRSN`, keeping the newest status |
| Filter | Drops pending applications, invalid records, and one-off administrative permits |
| Categorize | Maps about 250 old and new business types into 16 consistent categories |
| Build a business ID | One business = one legal name at one address, after standardizing names and addresses |
| Handle chains | Each chain location counts as its own business; names at 5 or more addresses are flagged |
| Collapse | Merges multiple licences held by one business in the same year (employees: max, never sum) |
| Add neighbourhoods | Fills older records with a point-in-polygon spatial join to the City's boundary file |
| Derive fields | Lifespan, closure, five-year survival, size group, opening period, 2024 transition flag |
| Save | Two tables for Tableau, joined on `business_id` |
| Quality checks | Tests neighbourhood coverage, the 2024 closure spike, and the chain flag |

Every step in the notebook states the decision, the reason for it, and the trade-off.

## Outputs

| File | One row per | Rows (approx.) |
|---|---|---|
| `output/businesses_by_year.csv` | business per year | 1.5 million |
| `output/businesses_summary.csv` | business | 273,000 |
| `output/unmapped_business_types.csv` | business type not yet categorized | for review |

In Tableau, relate the two main files on `business_id`.

## Key findings from the data work

- **The 2024 closure spike is mostly an artifact.** Construction and Trades closures rose from 843 in 2023 to 2,091 in 2024, about 1,250 of roughly 3,000 extra closures. This lines up with the City's 2024 licence system change. Those closures are flagged with `closed_in_2024_transition`.
- **The business ID holds up across the 2024 system change.** The share of 2024 closures that "reopened" at the same address in 2025 (16%) was in line with earlier years (16 to 19%).
- **About half of licences have no address** (mostly home-based businesses), so neighbourhood maps cover storefront and office businesses only.

## How to run it

1. Install Python 3.10 or later, then install the packages:
   ```
   pip install -r requirements.txt
   ```
2. Download these datasets from the [City of Vancouver Open Data Portal](https://opendata.vancouver.ca/) and save them in the `data/` folder with these exact names:
   - `business-licences-1997-to-2012.csv`
   - `business-licences-2013-to-2024.csv`
   - `business-licences.csv`
   - `local-area-boundary.geojson` (optional; needed for the neighbourhood lookup)

   Download the CSVs with **semicolon** separators (the portal's default export).
3. Open `build_business_dataset.ipynb` in Jupyter or VS Code and run all cells from the top folder of this repository.

The raw data (about 500 MB) is not included in this repository.

## Limitations

- Survival is licence-based. A business that moves or changes its legal name looks like one business closing and another opening, so survival rates are conservative.
- Businesses with no address are matched on name alone.
- Older records are less consistent than recent ones.
- The category mapping involves judgment calls (for example, gas stations are counted as Retail); they are listed in the notebook.

## Data source and licence

Contains information licensed under the Open Government Licence - Vancouver. Data: City of Vancouver Open Data Portal, *Business licences* and *Local area boundary*.

## Credits and AI use

Data pipeline by Liam Hart. I set the data decisions (such as how to define a business and how to treat chains) and validated the results. The Python code was written with Claude (Anthropic) as a coding assistant; I ran it, debugged it, and checked the outputs against the source data.

Project team (Group 5): Taylor Wei, LAN LAN, Vertika Chaudhary, Landy Feng, Shallovern Srodah, and Liam Hart.
