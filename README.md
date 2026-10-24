# What drives tenant satisfaction in social housing? Explaining and comparing England's Tenant Satisfaction Measures

Every social landlord in England with 1,000 or more homes now reports the same tenant satisfaction survey results to the Regulator of Social Housing: the **Tenant Satisfaction Measures (TSMs)**. For the first time, landlords can be compared on the same measures, but a raw league table hides a lot: how each survey was run, the type of landlord, and chance.

This project uses the TSMs for **353 landlords** (195 housing associations and 158 councils), with the Statistical Data Return (stock data) and the English Indices of Deprivation 2025, to answer five questions:

1. **Drivers:** which operational measures go with satisfaction, and how much of the difference between housing associations do they explain?
2. **Other explanations:** do deprivation, survey practice or the type of homes explain the rest?
3. **Change:** how did satisfaction change between 2023/24 and 2024/25, and did better performance go with higher satisfaction?
4. **Machine learning:** does it predict better than a simple regression?
5. **Priorities:** which parts of the service matter most to tenants, and where are the sector's biggest opportunities?

Satisfaction scores are the percentage of tenants who are satisfied. Differences between scores are given in **percentage points**, so a rise from 70% to 74% is a rise of 4 points.

## At a glance

- **Drivers:** stage 2 complaints (complaints escalated after the first response) have the strongest link with low satisfaction, and repairs done on time goes with higher satisfaction. Four operational measures, with survey method, size and region, explain **almost 60%** of the differences between housing associations.
- **Comparing landlords:** scores differ by survey method (post about 10 points higher than online across all landlords), and landlords whose own staff did the interviews score **about 4 points higher** than expected. Landlords need to be compared like for like.
- **Other explanations:** area deprivation, the survey company and the type of homes make no clear difference once repairs and complaints are allowed for.
- **Change:** satisfaction barely moves from one year to the next. Low scorers tend to improve by chance alone, and getting better at repairs goes with a small rise in satisfaction.
- **Machine learning:** with fewer than 200 housing associations, the simple regression predicts better than a random forest or gradient boosting.
- **Priorities:** a well-maintained home, being listened to and fair treatment matter most. **Being listened to is the biggest opportunity**: it matters a lot, yet only about two in three tenants are satisfied with it at a typical housing association.

## Explaining satisfaction

**At the typical (median) landlord, 71.8% of tenants are satisfied overall, but only 35.5% of tenants who complained are satisfied with how their complaint was handled.**

### What goes with satisfaction?

![Satisfaction by survey method](charts/1_survey_method.png#gh-light-mode-only)
![Satisfaction by survey method](charts/dark/1_survey_method.png#gh-dark-mode-only)

![What moves with satisfaction](charts/2_drivers.png#gh-light-mode-only)
![What moves with satisfaction](charts/dark/2_drivers.png#gh-dark-mode-only)

*Each dot is a landlord. Spearman correlation measures how closely two measures rank landlords in the same order, from −1 to +1.*

- **Stage 2 complaints have the strongest link with low satisfaction.** Housing associations with more of them per 1,000 homes tend to have clearly lower overall satisfaction (Spearman −0.65).
- **Repairs done on time** goes with higher satisfaction, especially with how long repairs take (+0.39).
- **Scores differ a lot by survey method.** Landlords that surveyed mainly by post have a typical overall satisfaction of 77.6%, against 67.9% for those surveying online. Some of that gap comes from other differences between the landlords using each method, but it's why the models allow for survey method.
- **Complaints handling has by far the lowest scores**, and **being listened to** is the measure that varies most from one landlord to another.

<details>
<summary>Technical detail</summary>

- Correlations are Spearman (rank) correlations across 195 housing associations, because several measures are skewed.
- "Typical" means the median. The survey method medians are across all 353 landlords.
- Among housing associations alone the survey method gap is smaller (82.8% by post against 75.2% online, from only 8 and 22 landlords). In the regression below, which allows for operations, size and region, postal and online surveys are not clearly different from telephone.

</details>

### How much does a simple model explain?

![Regression results](charts/3_model_effects.png#gh-light-mode-only)
![Regression results](charts/dark/3_model_effects.png#gh-dark-mode-only)

*Each dot is the change in overall satisfaction from moving a typical low performer to a typical high performer (25th to 75th percentile) on that measure, with everything else held the same. Whiskers are 95% confidence intervals: if one crosses zero, the direction of the effect is uncertain.*

- **A regression model explains almost 60% of the differences** in overall satisfaction between housing associations. Context on its own (survey method, size and region) explains about a third, so repairs and complaints add a lot on top.
- **Stage 2 complaints matter most** (about 3.9 points), then **repairs on time** (2.3 points).
- **Region matters too.** London housing associations score about 7.5 points lower than those in the South, even with operations allowed for.
- **It holds up on landlords it hasn't seen.** Tested on landlords left out while it was built, it still explains around 40 to 50% of the differences, depending on how the landlords are split, against under a fifth for context alone.

![Actual against predicted](charts/4_actual_vs_predicted.png#gh-light-mode-only)
![Actual against predicted](charts/dark/4_actual_vs_predicted.png#gh-dark-mode-only)

The model gets **57% of housing associations to within 5 points** of their actual score. Those further away, above or below, are where something the operational figures don't show is going on. Small landlords are harder to predict because they survey fewer tenants.

<details>
<summary>Technical detail</summary>

- Ordinary least squares with robust (HC3) standard errors. Predictors: stage 2 complaints per 1,000 homes, repairs completed on time, non-decent homes and stage 1 complaints answered on time, plus survey method, size band and region.
- 194 housing associations with complete data. R² 0.59 (context alone 0.32).
- Cross-validated R²: 0.42 on notebook 3's single 5-fold split (context alone 0.17), and 0.49 averaged over 5-fold cross-validation repeated 10 times (notebook 8).
- London: −7.5 points against the South (95% confidence interval −11.7 to −3.3).
- Checked for overlapping predictors (all VIFs close to 1) and for influential landlords (Cook's distance).

</details>

### What else might explain the differences?

Five other possible explanations were checked. Most were added to the model to see whether it then predicts unseen landlords better.

| Explanation | How it was checked | Result |
| --- | --- | --- |
| **Chance** (small survey samples) | Funnel plot of 148 housing associations that surveyed by telephone | Three in four of them are further from average than chance alone would explain. **The differences are real** |
| **Area deprivation** | A deprivation score for each landlord, based on where its homes are | **Barely linked to satisfaction**, and no type of deprivation has a clear effect in the model |
| **Survey company** | Whether any company's clients score higher or lower than predicted | **No clear difference** between companies |
| **Who did the interviews** | Landlord's own staff against everyone else (a survey company, or a mix) | **About 4 points higher than predicted** with own staff (26 landlords). The only explanation that clearly improves the model |
| **Type of homes** | Older people's and supported housing, building age, houses or flats, energy ratings | **No useful effect.** All six measures together make predictions worse (181 landlords with complete data) |

![Everything tested](charts/9_explanations_tested.png#gh-light-mode-only)
![Everything tested](charts/dark/9_explanations_tested.png#gh-dark-mode-only)

The interviewing result fits with tenants answering more positively when their own landlord's staff ask the questions. It's worth bearing in mind when comparing any landlord's TSMs.

![Funnel plot](charts/7_funnel_plot.png#gh-light-mode-only)
![Funnel plot](charts/dark/7_funnel_plot.png#gh-dark-mode-only)

*A funnel plot shows how far each landlord could sit from the average by chance alone, given how many tenants it surveyed. Landlords outside the 99.8% limits are very unlikely to be there by chance.*

<details>
<summary>Technical detail and more charts</summary>

- Funnel plot: 74% of the 148 sit outside the 99.8% control limits, where chance alone would put 0.2%.
- Deprivation: a stock-weighted average of local authority scores from the Indices of Deprivation 2025, using SDR stock by local authority and allowing for group structures. Spearman correlation with overall satisfaction +0.02. All seven domains and the overall score were tested separately.
- Own-staff interviewing: +3.9 points (95% confidence interval +0.3 to +7.4). Survey company and weighting tested with nested F-tests.
- Each explanation is judged by the change in cross-validated R² on the same landlords.

![Deprivation and satisfaction](charts/5_deprivation_scatter.png#gh-light-mode-only)
![Deprivation and satisfaction](charts/dark/5_deprivation_scatter.png#gh-dark-mode-only)

![Effect of each deprivation score](charts/6_deprivation_effects.png#gh-light-mode-only)
![Effect of each deprivation score](charts/dark/6_deprivation_effects.png#gh-dark-mode-only)

![Residuals by survey company](charts/8_survey_company.png#gh-light-mode-only)
![Residuals by survey company](charts/dark/8_survey_company.png#gh-dark-mode-only)

</details>

## Change over a year

**192 housing associations have results for both 2023/24 and 2024/25.**

![Sector change on each measure](charts/10_sector_change.png#gh-light-mode-only)
![Sector change on each measure](charts/dark/10_sector_change.png#gh-dark-mode-only)

- **Satisfaction barely moves from one year to the next.** Landlords that scored well in 2023/24 nearly always scored well again. All 12 measures rose slightly, by a median of 1.3 points for overall satisfaction and 2.1 for complaints handling.
- **Getting better at repairs goes with a small rise in satisfaction.** A housing association that improved repairs on time by 10 percentage points would expect about 0.7 points more satisfaction. Changes in complaint numbers show no clear link, possibly because the Complaint Handling Code became statutory on 1 April 2024 and changed what landlords record.

![Regression to the mean](charts/11_regression_to_mean.png#gh-light-mode-only)
![Regression to the mean](charts/dark/11_regression_to_mean.png#gh-dark-mode-only)

**Low scorers tend to improve by chance, and high scorers tend to slip.** A landlord on 60% would be expected to rise about 3 points, and one on 90% to fall about 1, with no change in service at all. This is **regression to the mean**, and any year-on-year comparison needs to allow for it.

<details>
<summary>Technical detail and more charts</summary>

- Correlation of overall satisfaction between the two years: 0.91.
- Change models are first-difference regressions (change in satisfaction against the changes in the four model measures, 190 housing associations), with robustness checks. Repairs on time: about +0.07 points per point, half the single-year estimate (0.17). Noise in both years pulls change estimates towards zero (attenuation).

![Single-year link against change link](charts/12_single_year_vs_change.png#gh-light-mode-only)
![Single-year link against change link](charts/dark/12_single_year_vs_change.png#gh-dark-mode-only)

</details>

## Machine learning against a simple model

![Model comparison](charts/13_model_comparison.png#gh-light-mode-only)
![Model comparison](charts/dark/13_model_comparison.png#gh-dark-mode-only)

*R² is the share of the differences between landlords a model explains. Scores are for landlords the model did not see while it was trained.*

- **The simple regression wins.** It predicts unseen landlords better than a random forest or gradient boosting. With fewer than 200 housing associations, the extra flexibility mostly fits noise.
- **The random forest backs up the main finding.** It relies almost entirely on stage 2 complaints, and hints that repairs on time matters most once a landlord is above about 80%.

![Predicted risk against actual satisfaction](charts/16_risk_vs_actual.png#gh-light-mode-only)
![Predicted risk against actual satisfaction](charts/dark/16_risk_vs_actual.png#gh-dark-mode-only)

**Struggling landlords can be flagged, but half are missed.** A model trained to spot housing associations in the bottom quarter for satisfaction ranks them well overall, but misses 23 of the 49. Their repairs and complaints figures look fairly ordinary, so their problem doesn't show up in the operational data.

<details>
<summary>Technical detail and more charts</summary>

- scikit-learn pipelines, 5-fold cross-validation repeated 10 times. Cross-validated R²: regression 0.49, random forest and gradient boosting about 0.43. Mean absolute error 5.3 points, against 7.8 for a baseline that always predicts the average.
- Importance from permutation importance; shape from partial dependence curves.
- Bottom-quartile classifier: logistic regression on the four model measures, cross-validated AUC about 0.85 (0.84 for the single set of out-of-fold predictions in the chart). Landlords are flagged at a predicted risk of 50% or more.

![Feature importance](charts/14_feature_importance.png#gh-light-mode-only)
![Feature importance](charts/dark/14_feature_importance.png#gh-dark-mode-only)

![Partial dependence](charts/15_partial_dependence.png#gh-light-mode-only)
![Partial dependence](charts/dark/15_partial_dependence.png#gh-dark-mode-only)

</details>

## What matters most to tenants

![Key drivers](charts/17_key_drivers.png#gh-light-mode-only)
![Key drivers](charts/dark/17_key_drivers.png#gh-dark-mode-only)

*Dominance analysis shares out the credit for overall satisfaction fairly between the 11 specific survey questions, which overlap too much for an ordinary regression to separate. Whiskers are bootstrap 95% intervals.*

- **The 11 specific survey questions explain almost all (95%) of the differences in overall satisfaction between housing associations.**
- **The three biggest drivers are a well-maintained home (12.9% of what's explained), being listened to (12.4%) and being treated fairly and with respect (11.4%).** The top two stay in the top three however the data is resampled.

![Priority matrix](charts/18_priority_matrix.png#gh-light-mode-only)
![Priority matrix](charts/dark/18_priority_matrix.png#gh-dark-mode-only)

**Being listened to is the clearest opportunity.** It's the second biggest driver, and only 65.5% of tenants are satisfied with it at a typical housing association. Complaints handling has the most room to improve (38.7% at a typical housing association). A well-maintained home matters most but already scores fairly well (74.7%).

<details>
<summary>Technical detail</summary>

- 193 housing associations. The 11 measures explain 95% of the variation (R²), with VIFs up to 26.
- Dominance analysis fits all 2,048 combinations of the 11 measures. It's the same idea as Shapley values in machine learning.
- Stability checked with 200 bootstrap resamples.
- Medians here are for housing associations only, so they differ slightly from the all-landlord figures at the top of this page.

</details>

## What the project is for (and what it isn't)

### How a landlord could use it

- **Watch escalations.** A stage 2 complaint means a tenant's problem wasn't put right the first time. Housing associations with more of them have lower satisfaction, so the number reaching stage 2 is worth watching closely, alongside repairs on time.
- **Benchmark like with like.** Landlords whose own staff did the interviews score about 4 points higher than their repairs and complaints figures predict, and raw scores differ by survey method. The fairest comparison is with landlords that ran their surveys the same way, and a change of method or interviewer should be treated as a possible break in the trend.
- **Read year-on-year change carefully.** Low scorers tend to improve by chance, so a rise after a bad year isn't necessarily proof that something worked. In the same way, a fall after a very good year isn't necessarily a sign of decline.
- **Look beyond the operational figures.** About half of the struggling housing associations have ordinary-looking repairs and complaints figures. Being listened to and being treated fairly are about how tenants are dealt with day to day, and a landlord's own data (individual survey responses, complaint themes and tenants' comments) is the place to look for those.
- **Keep the model simple.** With fewer than 200 housing associations, a well-checked regression was more accurate than machine learning, and it's much easier to explain to a manager or a board.

### What it does not do

- **It does not prove cause and effect.** Unhappy tenants may complain more, as well as complaints making tenants unhappy.
- **It is not a league table.** Survey method, who did the interviews and regression to the mean all move scores without any change in service.
- **It does not let context explain away a low score.** Deprivation and the type of homes make no clear difference once repairs and complaints are allowed for, although London housing associations do score lower than the rest even after allowing for everything else.
- **It does not model councils.** The models use housing associations only, because councils differ in ways the data doesn't capture.

## Data

**Sources (all public):**

| File | Used by | Source |
| --- | --- | --- |
| `2025_TSM_Full_Data_v1.1_FINAL.xlsx` | Notebooks 1 to 9 | [RSH: Tenant Satisfaction Measures 2024/25](https://www.gov.uk/government/statistics/tenant-satisfaction-measures-202425) |
| `2024_TSM_Full_Data_v2.1_FINAL.xlsx` | Notebook 7 | [RSH: TSM 2024 data release](https://www.gov.uk/government/statistics/data-release-tsm-2024) (covers 2023/24) |
| `PRP_Data_Release_2025_Full_Data_FINAL_V1.1.zip` | Notebooks 5 and 6 | [RSH: Private registered provider social housing stock and rents 2024 to 2025](https://www.gov.uk/government/statistics/private-registered-provider-social-housing-stock-and-rents-in-england-2024-to-2025) |
| `File_10_-_IoD2025_Local_Authority_District_Summaries__lower-tier__v2.xlsx` | Notebook 5 | [MHCLG: English Indices of Deprivation 2025](https://www.gov.uk/government/statistics/english-indices-of-deprivation-2025) (File 10, v2) |

**Coverage:** every landlord with 1,000 or more homes in the 2024/25 TSMs: 353 landlords, of which 352 join to their management information. The models use the 195 housing associations; the year-on-year analysis uses the 192 with results in both years.

**Reproducing it**

1. Download the four files above and save them in `data/` under the names shown (see [`data/README.md`](data/README.md)).
2. Install the requirements and run the notebooks in order (see [Running it](#running-it)).
3. Or open any notebook in Google Colab, where it shows an upload button and asks for the files it needs.

## Method

| Step | What happens | Where |
| --- | --- | --- |
| Load and join | Survey results and management information joined on landlord code | Notebooks 01, 02 |
| Explore | Spread of each measure, survey method, operational measures by size, Spearman correlations | 01, 02 |
| Model | Regression of overall satisfaction on four operational measures plus survey method, size and region, with robust errors, overlap and influence checks, and cross-validation | 03, 04 |
| Other explanations | Stock-weighted deprivation, funnel plot, survey company, interviewer and weighting, stock profile | 05, 06 |
| Change | Two years matched, regression to the mean, first-difference regressions with robustness checks | 07 |
| Machine learning | Random forest and gradient boosting against the regression, permutation importance, partial dependence, bottom-quartile classifier | 08 |
| Key drivers | Dominance analysis of the 11 survey measures with bootstrap intervals, priority matrix | 09 |

Each notebook explains what is done and why as it goes, in notes between the steps and comments in the code, and ends with a summary of what was found.

### Choices

- **Like-for-like comparisons.** Results are compared within the same survey method where it matters, and the models allow for survey method, size and region.
- **Housing associations only** for the models, because councils differ in ways the data doesn't capture.
- **Rank (Spearman) correlations**, because several measures are skewed by a few extreme values.
- **Predictors chosen so they don't overlap.** Stage 1 and stage 2 complaint numbers are closely linked, so only stage 2 is used.
- **Operational measures used to explain overall satisfaction**, because the 12 survey measures are all closely linked to each other. Notebook 9 deals with that overlap directly.
- **Models compared on landlords they haven't seen** (cross-validation), and the main model checked for overlapping predictors and influential landlords.

## Running it

```bash
python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install -r requirements.txt jupyter
jupyter notebook notebooks/          # run 01 to 09 in order
```

The notebooks save the charts to `charts/`. The dark versions in `charts/dark/` use the same code with a dark palette, so GitHub can show them automatically in dark mode.

## Outputs

| File | Contents |
| --- | --- |
| `charts/1_*` to `charts/4_*` | Survey method, operational links, model effects, actual against predicted (notebook 04) |
| `charts/5_*`, `charts/6_*` | Deprivation (notebook 05) |
| `charts/7_*` to `charts/9_*` | Funnel plot, survey companies, everything tested (notebook 06) |
| `charts/10_*` to `charts/12_*` | Year-on-year change (notebook 07) |
| `charts/13_*` to `charts/16_*` | Machine learning (notebook 08) |
| `charts/17_*`, `charts/18_*` | Key drivers and priority matrix (notebook 09) |
| `charts/dark/` | The same 18 charts for GitHub's dark mode |
| `data/landlord_deprivation.csv` | Stock-weighted deprivation scores for each housing association (built in notebook 05, used by notebook 06) |

## Project structure

```
├── charts/                          # the 18 charts in this README
│   └── dark/                        # the same charts for dark mode
├── data/
│   ├── README.md                    # where to download each source file
│   └── landlord_deprivation.csv     # derived deprivation scores
├── notebooks/
│   ├── 01_load_and_explore.ipynb    # survey results, data quality, spread
│   ├── 02_join_and_explore.ipynb    # management information and correlations
│   ├── 03_regression_model.ipynb    # the main model and its checks
│   ├── 04_charts.ipynb              # charts 1 to 4
│   ├── 05_deprivation.ipynb         # stock-weighted deprivation
│   ├── 06_survey_and_stock_checks.ipynb  # chance, survey practice, type of homes
│   ├── 07_year_on_year.ipynb        # change between 2023/24 and 2024/25
│   ├── 08_machine_learning.ipynb    # machine learning comparison
│   └── 09_key_drivers.ipynb         # key driver analysis
├── requirements.txt
└── LICENSE
```

## Limitations

- **Linked doesn't mean caused.** The year-on-year analysis gets closer to cause and effect, but only has one change to work with.
- **Landlords report their own operational figures.** "Repairs on time" is measured against each landlord's own target, which ranges from 5 to 260 days. Non-decent homes figures depend on how recently homes were surveyed, and complaints recording changed when the Complaint Handling Code became statutory.
- **Small numbers.** Fewer than 200 housing associations means a few unusual landlords can sway the results, and some groups are small (26 landlords used their own staff for interviews).
- **Many tests.** Eight deprivation scores were tested, so a borderline result or two would be expected by chance.
- **Areas, not households.** Deprivation is measured by local authority, which probably understates how deprived social tenants' own neighbourhoods are.
- **Landlord averages, not individual tenants.** The key driver analysis shows which measures move together across landlords. Individual survey responses would give a stronger answer.
- **The gap between a landlord's actual and predicted score isn't a measure of anything specific.** It's simply what the model's measures don't account for.

## Terms used

<details>
<summary>Plain-English definitions</summary>

- **Housing association:** a social landlord that isn't a council (private registered providers, or PRPs, in the regulator's data).
- **Stage 2 complaint:** a complaint escalated because the tenant wasn't happy with the landlord's first response.
- **Points:** percentage points. If satisfaction rises from 70% to 74%, that's a rise of 4 points.
- **Median / typical:** the middle landlord when they're all lined up from lowest to highest.
- **Correlation:** how closely two measures move together, from −1 to +1.
- **Regression:** a model that estimates how much each measure is linked to satisfaction, with the others held the same.
- **R² (R-squared):** the share of the differences between landlords that a model explains, from 0 to 1.
- **Cross-validation:** testing a model on landlords left out while it was built, which is a fairer test than how well it fits the landlords it learnt from.
- **Funnel plot:** a chart showing how far each landlord could be from average by chance alone, given its sample size.
- **Regression to the mean:** unusually high or low scores tend to move back towards average the next time, purely by chance.
- **Dominance analysis:** a way of sharing out credit fairly between measures that overlap.
- **AUC:** how well a model ranks struggling landlords above the rest. 0.5 is no better than guessing and 1 is perfect.

</details>

## Ideas for next steps

- Add the 2025/26 TSMs when they're published, to give a third year for the change analysis.
- Repeat the models for councils, allowing for the ways they differ from housing associations.
- Measure deprivation at neighbourhood (LSOA) level, if stock data by neighbourhood becomes available.
- Run the key driver analysis on individual tenants' survey responses, which each landlord holds internally.

## Licence

Code: MIT (see `LICENSE`). Data: Open Government Licence v3.0 ([details](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/)). Contains public sector information from the Regulator of Social Housing (Tenant Satisfaction Measures 2023/24 and 2024/25, Statistical Data Return 2025) and the Ministry of Housing, Communities and Local Government (English Indices of Deprivation 2025).
