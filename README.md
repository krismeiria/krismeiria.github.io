# FAERS 2023–2026 — FDA Adverse Event Reporting System

> Data engineering pipeline: API extraction → enrichment → classification → analytics-ready dataset.

## Overview

This repository hosts a curated, analytics-ready sample of **3,000 adverse event reports** from the **FDA Adverse Event Reporting System (FAERS)**, spanning **January 2023 through July 2026**.

The dataset has been enriched with:
- **Temporal dimensions** (`Period`, `Quarter`) for trend analysis
- **Medical taxonomy classification** (`Reaction_SOC`, `Reaction_Group`) using the **MedDRA System Organ Class** hierarchy

Raw data sourced via the [openFDA API](https://open.fda.gov/apis/drug/event/).

---

## 📁 Dataset: `faers_2023_2026.csv`

### 13 Columns

| # | Column | Type | Description | Example |
|---|--------|------|-------------|---------|
| 1 | `ReportID` | String | Unique FAERS safety report ID | `21803134` |
| 2 | `Age` | Integer | Patient age at onset | `59` |
| 3 | `Age_Unit` | Integer | Unit code (801=Years, 802=Months, 803=Weeks) | `801` |
| 4 | `Sex` | String | Patient sex | `Male`, `Female` |
| 5 | `Drug` | String | Drug (medicinal product) | `DUPIXENT`, `RINVOQ` |
| 6 | `Reaction` | String | Adverse reaction (MedDRA PT) | `Headache`, `Fatigue` |
| 7 | `Country` | String | Reporter country (ISO-2) | `US`, `ZA`, `GB` |
| 8 | `Serious` | Integer | 1=Serious, 2=Not serious | `1`, `2` |
| 9 | `Seriousness_Death` | Integer | 1=Death, 2=No death | `1`, `2` |
| 10 | `Period` | String | **Enriched:** Year-Month of report | `2023-01` |
| 11 | `Quarter` | String | **Enriched:** Year-Quarter of report | `2023-Q1` |
| 12 | `Reaction_SOC` | String | **Enriched:** System Organ Class (26 categories) | `Nervous System Disorders` |
| 13 | `Reaction_Group` | String | **Enriched:** Aggregated group (12 super-categories) | `Neurological` |

---

## 🏗️ Data Engineering Pipeline

```
openFDA API (4.3M records) → Stratified sampling → Flat CSV
                                                          ↓
                                              Temporal enrichment
                                              (Period, Quarter)
                                                          ↓
                                              MedDRA SOC classification
                                              (Reaction_SOC, Reaction_Group)
                                                          ↓
                                              analytics-ready CSV
```

### 1. Extraction

- **Source:** [openFDA drug/event endpoint](https://api.fda.gov/drug/event.json)
- **Filter:** `receivedate:[20230101 TO 20261231]`
- **Total available:** 4,305,106 records (as of April 2026)
- **Sample:** 3,000 records (~750 per year) via stratified sampling across 2023–2026
- **Method:** Batched pagination (100 records/request) with rate limiting

### 2. Temporal Enrichment

Two derived columns extracted from the `receivedate` field:

| Column | Format | Example | Use Case |
|--------|--------|---------|----------|
| `Period` | `YYYY-MM` | `2023-01` | Monthly trend analysis |
| `Quarter` | `YYYY-Q#` | `2023-Q1` | Quarterly aggregation |

### 3. Reaction Classification (MedDRA SOC)

**956 unique reaction terms** mapped to **26 System Organ Classes** using keyword-based classification against the MedDRA hierarchy.

**Coverage:** 99.5% (only 16 of 3,000 records unclassified)

**Two-level taxonomy:**

```
Reaction_Group (12)          Reaction_SOC (26)
────────────────────────     ────────────────────────
Administration               Product & Medication Issues
                             Administration Site Reactions
Cardiovascular               Cardiac Disorders
                             Vascular Disorders
Dermatological               Skin & Subcutaneous Disorders
Gastrointestinal             Gastrointestinal Disorders
                             Dental & Oral Disorders
General                      General Disorders
                             Injury & Trauma
                             Death
Haematological               Blood & Lymphatic Disorders
Hepatobiliary                Hepatobiliary Disorders
Immunological                Immune System Disorders
Infectious                   Infections
Investigations               Lab & Investigation Findings
Metabolic                    Metabolic & Endocrine Disorders
Neurological                 Nervous System Disorders
Oncology                     Neoplasms
Ophthalmological             Eye Disorders
Otological                   Ear & Labyrinth Disorders
Other                        Other / Unclassified
Procedural                   Surgical & Procedural
Psychiatric                  Psychiatric Disorders
Renal                        Renal & Urinary Disorders
Reproductive                 Reproductive & Pregnancy Disorders
Respiratory                  Respiratory Disorders
Serious Outcome              Death (severe cases)
```

#### Top 10 Reaction_SOC Distribution

| SOC | Count | % |
|-----|-------|---|
| Product & Medication Issues | 544 | 18.1% |
| General Disorders | 338 | 11.3% |
| Nervous System Disorders | 226 | 7.5% |
| Skin & Subcutaneous Disorders | 220 | 7.3% |
| Gastrointestinal Disorders | 185 | 6.2% |
| Respiratory Disorders | 178 | 5.9% |
| Death | 154 | 5.1% |
| Infections | 125 | 4.2% |
| Musculoskeletal Disorders | 125 | 4.2% |
| Blood & Lymphatic Disorders | 114 | 3.8% |

---

## 🔬 Example Use Cases

**In Tableau / BI tools:**

- **Drug Safety Profile:** `Drug` vs `Reaction_SOC` heatmap
- **Trend Analysis:** `Period` / `Quarter` on timeline → reaction counts over time
- **Demographic Breakdown:** `Age` by `Sex` by `Reaction_SOC`
- **Geographic Map:** `Country` bubble map coloured by `Reaction_Group`
- **Seriousness Analysis:** `Serious` × `Seriousness_Death` by `Drug`

---

## 📊 Technical Notes

- **Age codes:** 801 = Years, 802 = Months, 803 = Weeks
- **Country codes:** ISO 3166-1 alpha-2
- **Serious:** 1 = serious (life-threatening, hospitalisation, disability, congenital anomaly, or other serious event), 2 = not serious
- **Seriousness_Death:** 1 = death reported, 2 = no death reported
- **Disclaimer:** This dataset is sourced from the FDA's public API and should not be used for medical decision-making.

---

## 🧪 Reproducibility

The full data engineering pipeline is reproducible via Python:

```python
# Extraction
openFDA API → paginated GET requests → CSV

# Temporal enrichment
receivedate[:4] + '-' + receivedate[4:6] → Period
month_to_quarter(Period) → Quarter

# SOC classification
keyword match against 26 SOC rule sets → Reaction_SOC
SOC-to-group mapping → Reaction_Group
```

Pipeline scripts available on request.

---

*Built with openFDA public API, Python, and the MedDRA hierarchical terminology system.*
