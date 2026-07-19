# Methodology and Data Audit

How the used vehicle advertised price benchmark was framed, audited, built and tested.

This document is for reviewers, hiring managers and future maintainers checking the work. The business recommendation lives in [README.md](README.md). The principle throughout was simple: a claim did not enter the report until the data had earned it.

The project followed six sequential stages:

1. Frame the decision.
2. Structure the analytical problem.
3. Establish honest data.
4. Analyse using transparent statistical methods.
5. Translate evidence into a business consequence.
6. Communicate the recommendation and its boundaries.

## 1. Provenance: why the first dataset was rejected

The project initially considered a different Kaggle car sales dataset containing 50,000 rows. Its source name identified it as a mock second hand car dataset, and its structure showed several warning signs of synthetic data:

* Only five manufacturers and fifteen models.
* Suspiciously complete and regular fields.
* Implausible combinations of manufacturer, model and year, including a 1988 Toyota RAV4.
* Price patterns that appeared to be generated from the same observable attributes a regression would use.

That created a serious risk. Strong predictive accuracy could simply reflect recovery of a synthetic generation formula, while residuals would represent injected noise rather than meaningful price variation. The dataset was rejected before modelling.

The final source is the [Kaggle 100,000 UK Used Car Data Set](https://www.kaggle.com/datasets/adityadesai13/used-car-dataset-ford-and-mercedes), described by the source as used car listings scraped from the British market. These are advertised listings, not completed transactions.

Rejecting the first dataset was not a cosmetic cleaning decision. It prevented the project from producing polished but commercially meaningless claims about a synthetic market.

## 2. Scope: which files were used

Thirteen files were supplied. Nine manufacturer files were included.

| File group | Decision | Reason |
|---|---|---|
| `audi.csv`, `bmw.csv`, `ford.csv`, `hyundi.csv`, `merc.csv`, `skoda.csv`, `toyota.csv`, `vauxhall.csv`, `vw.csv` | Used | Full manufacturer files with the common analytical fields |
| `focus.csv` | Excluded | Substantially duplicates Ford Focus records already represented in `ford.csv` |
| `cclass.csv` | Excluded | Substantially duplicates Mercedes-Benz C Class records already represented in `merc.csv` |
| `unclean focus.csv`, `unclean cclass.csv` | Excluded | Deliberately malformed source versions retained only as source documentation |

Across the shared columns:

* `focus.csv` contains 5,454 rows; **4,324 distinct row patterns** overlap with `ford.csv`.
* `cclass.csv` contains 3,899 rows; **3,533 distinct row patterns** overlap with `merc.csv`.

Including these files would have counted those model families twice and given them too much weight in manufacturer and model comparisons.

Manufacturer is encoded by filename rather than a field in the source data. During the union, two traceability fields were added:

* `manufacturer`
* `source_file`

Two structural inconsistencies were standardized:

* `hyundi.csv` names its tax column `tax(£)` while the others use `tax`.
* Model values contain leading whitespace, so model names were trimmed before grouping.

## 3. Data audit: from 99,187 rows to 97,673

| Audit stage | Rows removed | Rows remaining |
|---|---:|---:|
| Starting rows across nine manufacturer files | Not applicable | 99,187 |
| Excess exact duplicate rows | 1,475 | 97,712 |
| Impossible model years: two at 1970 and one at 2060 | 3 | 97,709 |
| Unreliable Mercedes-Benz A Class model labels | 32 | 97,677 |
| Corrupt price records | 4 | **97,673** |

### Unreliable model labels

The 32 excluded A Class records carried 4.0 litre engines and prices from approximately £58,000 to £140,000, sharply inconsistent with the surrounding A Class observations. They were treated as unreliable model labels rather than as valid A Class price evidence.

This is a narrow exclusion affecting approximately 0.03% of the data after exact duplicates were removed. It does not materially determine the portfolio results, but leaving the labels unchanged would contaminate interpretation by model.

### Corrupt price records

| Vehicle | Listed price | Audit evidence |
|---|---:|---|
| BMW 2 Series, 2015 | £123,456 | Placeholder digits; the next comparable observation is below £19,000 |
| Hyundai i10, 2017 | £92,000 | Approximately 12.7× its median for the same model and year |
| Vauxhall Mokka, 2016 | £52,489 | Price exactly equals the recorded mileage and is approximately 5.6× the median for the same model and year |
| Skoda Karoq, 2019 | £91,874 | Approximately 4.3× the median for the same model and year |

Four genuinely inexpensive cars were reviewed and retained:

* 2003 Ford Focus: £495, 177,644 miles.
* 2002 Vauxhall Corsa: £495, 99,842 miles.
* 2001 Vauxhall Astra: £450, 159,000 miles.
* 2003 Vauxhall Agila: £450, 90,000 miles.

This illustrates why a blanket outlier rule would be unsafe. It could delete credible budget cars with high mileage while retaining obvious field errors such as £123,456.

### Field decisions

* **Engine size:** 267 final analytical records contain an engine size of zero. Zero is treated as unavailable, not as a real engine displacement. The record remains usable for analyses that do not require engine size.
* **Transmission:** nine final records have unavailable transmission and are excluded from the regression population, not from the descriptive population.
* **MPG:** 33 invalid MPG values below the audit floor were set unavailable for audit reporting. MPG was not used in the selected regression.
* **Mileage and price extremes:** unusual but plausible values were retained and flagged. Median summaries, modelling on log price and segment diagnostics limit their influence without deleting credible vehicles merely because they are unusual.

The final analytical population contains:

* **97,673 records** representing 99,148 source rows after duplicate weights are considered.
* **9 manufacturers**.
* **195 manufacturer and model combinations**.
* Model years from **1996 to 2020**.
* Prices from **£450 to £159,999**.

## 4. Record independence and leakage control

The source provides no listing ID or vehicle ID, so record independence cannot be assumed.

On the final analytical population:

* The 97,673 records collapse to **95,945 exact fingerprints based on all recorded fields except price**.
* **1,315 fingerprints** contain more than one distinct price.
* Those fingerprints contain 3,043 unique records and represent 3,678 source rows.
* They contribute **1,728 additional unique price records** beyond one record per fingerprint.

The fingerprint fields are:

* Manufacturer.
* Model.
* Model year.
* Transmission.
* Mileage.
* Fuel type.
* Annual tax.
* MPG.
* Engine size.

A random row split could place the same recorded specification on both sides of the build/test boundary, producing flattering validation results. The final split therefore assigns whole fingerprints to either training or test.

| Population | Records | Fingerprints |
|---|---:|---:|
| Full training split | 78,173 | 76,755 |
| Complete training population used for regression | **77,965** | **76,555** |
| Full test split | 19,500 | 19,190 |
| Complete test population used for regression | **19,433** | **19,125** |
| Fingerprints shared across train and test | Not applicable | **0** |

Comparable medians, category pooling, model transformations and smearing corrections were estimated from training data only. The test population was used for final model comparison and diagnostics, not for choosing record level parameters.

## 5. Reference year decision

Vehicle age requires a reference date, but the source contains no listing date, registration date or manufacture month.

In the final analytical population:

* Vehicles from model year 2020: **4,039 listings**.
* Vehicles from model year 2019: **26,149 listings**, the clear peak.

This pattern is consistent with a scrape conducted during 2020, when 2020 models had only recently entered the used market, but it does not prove the timing. The analysis therefore defines:

```text
approximate_vehicle_age = 2020 - model_year
```

The reference year is an explicit assumption. `model_year` is retained so that the transformation is reversible. Because age is a linear recode of year, using one rather than the other changes interpretation rather than the fitted information, provided nonlinear terms are mapped consistently.

Age should be interpreted with approximately one year of timing uncertainty.

## 6. Target and analytical specification

The target is:

> **Expected observed advertised price given the recorded vehicle attributes.**

It is not called market value, fair value, transaction price or optimal price.

The selected model uses log advertised price because prices are skewed to the right and the main relationships are proportional. Its explanatory fields are:

* Manufacturer and model group.
* Approximate vehicle age.
* Mileage.
* Reported engine size.
* Fuel type.
* Transmission.

The regression excludes records with unavailable engine size or transmission but retains them in the broader descriptive and routing population.

Prices are transformed back to pounds using a Duan smearing correction. For the selected model:

* Smearing factor: **1.0081**.
* Adjusted R squared on the training population: **0.9441**.
* Training records: **77,965**.
* Regression model groups: **147**, after pooling manufacturer and model groups with limited support.

The smearing correction is applied, although its practical effect is small.

## 7. The baseline the model had to beat

The operational baseline is a hierarchical lookup of comparable medians, built only from training fingerprints and requiring at least 25 fingerprints before a segment median is used.

Fallback hierarchy:

1. Model × age band × mileage band.
2. Model × age band.
3. Model.
4. Manufacturer × age band.
5. Manufacturer.
6. Overall median if required.

Usage of the strengthened hierarchy on the test population:

| Comparable level | Test records | Share |
|---|---:|---:|
| Model × age × mileage | 16,517 | 84.99% |
| Model × age | 2,486 | 12.79% |
| Model | 330 | 1.70% |
| Manufacturer × age | 99 | 0.51% |
| Manufacturer | 1 | 0.01% |

All 19,433 test records used for regression received a comparable prediction.

Adding mileage to the comparable hierarchy reduced MdAPE from **11.54% to 9.61%**. The regression was required to beat this strengthened version.

## 8. Model selection and complexity control

| Specification | Test MdAPE | Test MAE | Decision |
|---|---:|---:|---|
| Hierarchical comparable median | 9.61% | £2,355.01 | Operational fallback |
| A1: simple regression on log price | 7.92% | £1,639.99 | Strong transparent challenger |
| **A2: piecewise regression on log price** | **7.77%** | **£1,613.11** | **Selected** |
| A3: excess mileage for age | 7.92% | £1,643.48 | Rejected: no performance gain |
| B1: piecewise regression on raw price | 10.49% | £2,162.80 | Rejected on performance and diagnostics |

### Why A2 was selected

A2 improves MdAPE by only **0.15 percentage points** over A1, so the project does not present this as a large accuracy gain. It was selected because its piecewise age and mileage terms better represent the steep early decline that later flattens, while remaining a transparent regression.

A2 also modestly improves MAE and the percentage of predictions within ±15% and ±20% of observed listed price.

### Relationship between age and mileage

Age and direct mileage are related:

* Pearson correlation: 0.745.
* Spearman correlation: 0.809.
* VIF for age and direct mileage: approximately 2.26 to 2.28.

A3 expresses mileage as the amount above or below the typical level for a vehicle’s age. This reduces the correlation between age and excess mileage to 0.040 and VIF to approximately 1.0, making the story easier to communicate. It does not improve test error, so it remains an interpretive diagnostic rather than the selected specification.

### Why regression on raw price was rejected

B1 performs materially worse and produces **129 negative price predictions** on the test population, which are operationally invalid. Log price better matches the proportional error objective and guarantees positive predictions after conversion back to pounds.

No tree ensemble or other black box model was introduced. The project objective is transparent pricing analysis, and additional complexity was required to earn an operational role.

## 9. Test performance and uncertainty

### Overall performance

| Measure | Strengthened comparables | Selected A2 benchmark |
|---|---:|---:|
| Test records | 19,433 | 19,433 |
| MAE | £2,355.01 | **£1,613.11** |
| Median absolute error | £1,335.00 | **£1,057.47** |
| MdAPE | 9.61% | **7.77%** |
| Within ±10% | 51.38% | **61.28%** |
| Within ±15% | 67.65% | **80.04%** |
| Within ±20% | 78.60% | **90.35%** |
| Median signed percentage error | 0.07% | 1.03% |

The £741.90 reduction in MAE is rounded to **£742 of benchmark error per listing**. It is not interpreted as money saved.

### Segment diagnostics

| Test segment | A2 MdAPE | Within ±20% | Interpretation |
|---|---:|---:|---|
| Price below £10k | 8.42% | 86.67% | Moderate positive bias remains in the budget segment |
| £10k to £20k | 7.45% | 92.19% | Strong core performance |
| £20k to £30k | 7.17% | 92.13% | Strong core performance |
| £30k to £50k | 9.16% | 87.76% | Weaker segment in absolute price terms |
| £50k and above | 11.03% | 80.60% | Review required for high value vehicles |
| Age 0 to 2 years | 7.82% | 90.93% | Strong |
| Age 3 to 5 years | 7.33% | 91.59% | Strong |
| Age 6 to 10 years | 9.60% | 84.10% | Weaker |
| Age 11+ years | 16.85% | 57.63% | Not suitable for standard guidance |

These price band diagnostics use observed listed price and therefore cannot be used directly as routing rules before a vehicle is listed.

### Empirical uncertainty calibration

The complete test population was split again by fingerprint into calibration and validation samples:

| Sample | Records | Fingerprints |
|---|---:|---:|
| Calibration | 9,746 | 9,610 |
| Validation | 9,687 | 9,515 |
| Fingerprints in both | Not applicable | 0 |

| Empirical range | Multipliers | Validation coverage | Nominal target |
|---|---:|---:|---:|
| 90% | 0.82× to 1.21× | 89.28% | 90% |
| 95% | 0.79× to 1.26× | 94.19% | 95% |

Coverage varies materially by segment:

| Validation segment | 90% range coverage | Assessment |
|---|---:|---|
| Age 0 to 2 | 89.83% | Reliable |
| Age 3 to 5 | 90.91% | Reliable |
| Under 10,000 miles | 89.23% | Reliable |
| Age 6 to 10 | 81.43% | Weaker |
| 60,000 to 99,999 miles | 80.19% | Weaker |
| Age 11+ | 55.17% | Not credible for standard guidance |
| 100,000+ miles | 65.33% | Not credible for standard guidance |

Observed listed price bands also showed weaker coverage above £30,000, but observed price is unavailable when guiding a new listing. Deployment was therefore checked using predicted price bands:

| Predicted benchmark band | Validation records | MdAPE | Within ±20% | 90% range coverage |
|---|---:|---:|---:|---:|
| Below £30k | 8,942 | 7.77% | 90.24% | 89.54% |
| £30k to £49,999 | 648 | 8.38% | 90.12% | 87.65% |
| £50k+ | 97 | 11.98% | 79.38% | 76.29% |

The £30,000 review trigger was rejected because vehicles with predicted prices from £30,000 to £49,999 remained reasonably accurate. The predicted £50,000 threshold was retained.

## 10. Guidance tiers

The operating tiers follow the validation evidence rather than intuition.

| Tier | Listings | Share of listings | Share of aggregate listed price value |
|---|---:|---:|---:|
| Standard guidance | 77,166 | 79.00% | 80.32% |
| Cautious review | 17,152 | 17.56% | 14.43% |
| Manual appraisal | 3,355 | 3.43% | 5.25% |
| **Total** | **97,673** | **100.00%** | **100.00%** |

Routing logic:

* **Tier 1, standard guidance:** age up to 5 years, mileage below 60,000, supported model and fuel category, complete required inputs and predicted benchmark below £50,000.
* **Tier 2, cautious review:** age from 6 to 10 years, mileage from 60,000 to 99,999, or thinner comparable support, provided no Tier 3 condition applies.
* **Tier 3, manual appraisal:** age 11 years or older, mileage of at least 100,000, missing or unclear required inputs, unsupported electric records, or a predicted benchmark of at least £50,000.

All **1,020** records with predicted benchmark prices of £50,000 or more route to Tier 3.

Operating ranges:

* Tier 1 uses the pooled 90% empirical range, from 0.82× to 1.21×.
* Tier 2 uses the wider pooled 95% range, from 0.79× to 1.26×, as a conservative review aid.
* Tier 3 receives no calibrated benchmark range.

These are global ranges. They do not guarantee nominal coverage within every tier or segment.

### Support for the age and mileage price book

The supported reference is a vehicle that is one year old with 10,000 miles, set to an index of 100. It is supported by **4,562 training fingerprints**.

Price book publication rules:

* At least 100 fingerprints: strong support.
* From 25 to 99 fingerprints: directional only.
* Fewer than 25 fingerprints: no published index.

Across the displayed grid:

* 9 cells have strong support.
* 6 cells are directional.
* 5 cells are blank.

The joint index is the multiplication of the separate nonlinear age and mileage factors on the price scale. The selected regression does not include an age × mileage interaction.

## 11. What the analysis cannot establish

These are source limitations, not unfinished calculations:

* **Advertised prices only:** no transaction price, sale status, time on lot or price change history.
* **No margin economics:** no acquisition cost, preparation cost or margin; therefore no price that maximises profit and no ROI estimate.
* **No demand signal:** no elasticity, enquiry conversion or estimate of sale probability.
* **Missing vehicle detail:** no trim, condition, options, accident or service history, dealer or geography. Genuine price differences remain in the residual.
* **No validated mispricing outcome:** a deviation from the benchmark is a review prompt, never proof of overpricing or underpricing.
* **Electric vehicles out of scope:** only six records are labelled electric.
* **No causal claims:** age, mileage and feature relationships are conditional associations in this historical dataset.
* **No monetisation of error reduction:** the £742 MAE improvement is benchmark error, not money saved.
* **Reference year uncertainty:** exact listing dates are unavailable; age uses 2020 as an explicit assumption.

## 12. Published project files

The published package contains the business report, this methodology document and the five final visuals:

```text
.
├── README.md
├── METHODOLOGY_AND_DATA_AUDIT.md
└── images/
    ├── age_mileage_price_index.png
    ├── benchmark_vs_comparables.png
    ├── diesel_reversal.png
    ├── guidance_tiers.png
    └── model_vs_brand_positioning.png
```

The source data and working analysis environment are not included. The source CSVs remain available from Kaggle under the dataset owner’s access and licensing terms.

### Techniques demonstrated

* Hierarchical comparable medians with minimum support fallbacks.
* Transparent hedonic regression on log price.
* Piecewise nonlinear age and mileage terms.
* Duan smearing when converting predictions back to pounds.
* Training and test splitting that keeps exact fingerprints together.
* Segment diagnostics by manufacturer, age, mileage and price band.
* Empirical uncertainty calibration and validation using separate fingerprints.
* Guidance tiers that keep a person in control and follow documented reliability gates.

## Data source note

Data: [Kaggle 100,000 UK Used Car Data Set](https://www.kaggle.com/datasets/adityadesai13/used-car-dataset-ford-and-mercedes).

This is an independent portfolio case study. It is not affiliated with or endorsed by Kaggle, a dealer, marketplace, lender or insurer. The benchmark is a methodology demonstration and is not recommended for live pricing without validation on the intended source and commercial outcome data.
