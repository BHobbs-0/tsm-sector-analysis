# What drives tenant satisfaction in social housing?

Every large social landlord in England now publishes the same tenant satisfaction survey results, the Regulator of Social Housing's **Tenant Satisfaction Measures (TSMs)**. That makes it possible, for the first time, to compare landlords fairly and ask what separates the ones tenants are happy with from the ones they aren't.

This project uses Python to answer that question with public data only: the TSMs for 2023/24 and 2024/25, the Statistical Data Return (stock data) and the English Indices of Deprivation 2025.

## Key findings

Satisfaction scores are the percentage of tenants who are satisfied. Differences between scores are given in **percentage points**, so a rise from 70% to 74% is a rise of 4 points.

1. **[What goes with satisfaction?](#1-what-goes-with-satisfaction)** Stage 2 complaints (complaints escalated after the first response) have the strongest link with low satisfaction, followed by repairs done on time. How the survey is run matters too: postal surveys score about 10 points higher than online ones.
2. **[How much does a simple model explain?](#2-how-much-does-a-simple-model-explain)** Almost 60% of the differences between housing associations, using four operational measures plus survey method, size and region.
3. **[What else might explain the differences?](#3-what-else-might-explain-the-differences)** Not deprivation, the survey company or the type of homes. The one thing that does is who did the interviews: landlords whose own staff did them score about 4 points higher than expected.
4. **[How did things change over a year?](#4-how-did-things-change-over-a-year)** Very little. Low scorers tend to improve by chance alone, and getting better at repairs goes with a small rise in satisfaction.
5. **[Does machine learning do better?](#5-does-machine-learning-do-better)** No. With fewer than 200 housing associations, the simple regression predicts better than the more complex models.
6. **[What matters most to tenants?](#6-what-matters-most-to-tenants)** A well-maintained home, being listened to and fair treatment. Being listened to is the biggest opportunity: it matters a lot, yet only about two in three tenants are satisfied with it at a typical housing association.

Section 7 sets out [what this means](#7-what-this-means) for landlords and anyone comparing them. Also: [the notebooks](#the-notebooks), [method](#method), [limitations](#limitations), [terms used](#terms-used) and [how to run it](#data-and-how-to-run).

---

## 1. What goes with satisfaction?

*Notebooks [01](notebooks/01_load_and_explore.ipynb) and [02](notebooks/02_join_and_explore.ipynb)*

- The data covers **353 landlords**: 195 housing associations and 158 councils.
- **Complaints handling has by far the lowest scores.** At a typical landlord only 35.5% of tenants who complained were satisfied with how it was handled. **Being listened to** is the measure that varies most from one landlord to another.
- **Stage 2 complaints have the strongest link with low satisfaction.** Housing associations with more of them per 1,000 homes tend to have clearly lower overall satisfaction. **Repairs done on time** goes with higher satisfaction, especially with how long repairs take.
- **The survey method makes a big difference.** Landlords that surveyed mainly by post have a typical overall satisfaction of 77.6%, against 67.9% for those surveying online. So every comparison in this project is made like for like.

![Satisfaction by survey method](charts/1_survey_method.png#gh-light-mode-only)
![Satisfaction by survey method](charts/dark/1_survey_method.png#gh-dark-mode-only)

![What moves with satisfaction](charts/2_drivers.png#gh-light-mode-only)
![What moves with satisfaction](charts/dark/2_drivers.png#gh-dark-mode-only)

<details>
<summary>Technical detail</summary>

- Correlations are Spearman (rank) correlations across 195 housing associations, because several measures are skewed. Stage 2 complaints per 1,000 homes: −0.65 with overall satisfaction. Repairs on time: +0.39 with satisfaction with repair time.
- "Typical" means the median. The survey method medians are across all 353 landlords.

</details>

## 2. How much does a simple model explain?

*Notebooks [03](notebooks/03_regression_model.ipynb) and [04](notebooks/04_charts.ipynb)*

- **A regression model explains almost 60% of the differences** in overall satisfaction between housing associations. Context on its own (survey method, size and region) explains about a third, so repairs and complaints add a lot on top.
- **It holds up on landlords it hasn't seen.** Tested on landlords left out while it was built, it still explains just under half the differences, against under a fifth for context alone.
- **The size of the effects:** going from a typical low performer to a typical high performer is worth about 3.9 points of satisfaction on stage 2 complaints and 2.3 points on repairs on time.
- **The model gets 57% of housing associations to within 5 points.** Those further away, above or below, are where something the operational figures don't show is going on. Small landlords are harder to predict because they survey fewer tenants.

![Regression results](charts/3_model_effects.png#gh-light-mode-only)
![Regression results](charts/dark/3_model_effects.png#gh-dark-mode-only)

![Actual against predicted](charts/4_actual_vs_predicted.png#gh-light-mode-only)
![Actual against predicted](charts/dark/4_actual_vs_predicted.png#gh-dark-mode-only)

<details>
<summary>Technical detail</summary>

- Ordinary least squares with robust (HC3) standard errors. Predictors: stage 2 complaints per 1,000 homes, repairs completed on time, non-decent homes and stage 1 complaints answered on time, plus survey method, size band and region.
- R² 0.59 (context alone 0.32). Cross-validated R² 0.49 over 5-fold cross-validation repeated 10 times (notebook 8), against 0.17 for context alone. A single 5-fold run in notebook 3 gives 0.42.
- "Typical low to typical high" means moving from the 25th to the 75th percentile.
- Checked for overlapping predictors (all VIFs close to 1) and for influential landlords (Cook's distance).

</details>

## 3. What else might explain the differences?

*Notebooks [05](notebooks/05_deprivation.ipynb) and [06](notebooks/06_survey_and_stock_checks.ipynb)*

I checked five other possible explanations. Most were added to the model to see whether it then predicts unseen landlords better.

| Explanation | How it was checked | Result |
| --- | --- | --- |
| **Chance** (small survey samples) | Funnel plot of 148 housing associations that surveyed by telephone | Three in four landlords are further from average than chance alone would explain. **The differences are real** |
| **Area deprivation** | A deprivation score for each landlord, based on where its homes are | **Barely linked to satisfaction**, and no type of deprivation has a clear effect in the model |
| **Survey company** | Whether any company's clients score higher or lower than predicted | **No clear difference** between companies |
| **Who did the interviews** | Landlord's own staff against an external company | **About 4 points higher than predicted** with own staff (26 landlords). The only explanation that improves the model |
| **Type of homes** | Older people's and supported housing, building age, houses or flats, energy ratings | **Adds nothing**, and makes predictions worse |

The interviewing result fits with tenants answering more positively when their own landlord's staff ask the questions. It's worth bearing in mind when comparing any landlord's TSMs.

![Everything tested](charts/9_explanations_tested.png#gh-light-mode-only)
![Everything tested](charts/dark/9_explanations_tested.png#gh-dark-mode-only)

![Funnel plot](charts/7_funnel_plot.png#gh-light-mode-only)
![Funnel plot](charts/dark/7_funnel_plot.png#gh-dark-mode-only)

<details>
<summary>Technical detail and more charts</summary>

- Funnel plot: 74% of landlords sit outside the 99.8% control limits, where chance alone would put 0.2%.
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

## 4. How did things change over a year?

*Notebook [07](notebooks/07_year_on_year.ipynb), 192 housing associations with results for both 2023/24 and 2024/25*

- **Satisfaction barely moves from one year to the next.** Landlords that scored well in 2023/24 nearly always scored well again. All 12 measures rose slightly, by a median of 1.3 points for overall satisfaction and 2.1 for complaints handling.
- **Low scorers tend to improve by chance, and high scorers tend to slip.** A landlord on 60% would be expected to rise about 3 points, and one on 90% to fall about 1, with no change in service at all. This is **regression to the mean**, and any year-on-year comparison needs to allow for it.
- **Getting better at repairs goes with a small rise in satisfaction.** A housing association that improved repairs on time by 10 percentage points would expect about 0.7 points more satisfaction. Changes in complaint numbers show no clear link, possibly because the Complaint Handling Code became law on 1 April 2024 and changed what landlords record.

![Sector change on each measure](charts/10_sector_change.png#gh-light-mode-only)
![Sector change on each measure](charts/dark/10_sector_change.png#gh-dark-mode-only)

![Regression to the mean](charts/11_regression_to_mean.png#gh-light-mode-only)
![Regression to the mean](charts/dark/11_regression_to_mean.png#gh-dark-mode-only)

<details>
<summary>Technical detail and more charts</summary>

- Correlation of overall satisfaction between the two years: 0.91.
- Change models are first-difference regressions (change in satisfaction against change in each operational measure), with robustness checks. Repairs on time: about +0.07 points per point, half the single-year estimate (0.17). Noise in both years pulls change estimates towards zero (attenuation).

![Single-year link against change link](charts/12_single_year_vs_change.png#gh-light-mode-only)
![Single-year link against change link](charts/dark/12_single_year_vs_change.png#gh-dark-mode-only)

</details>

## 5. Does machine learning do better?

*Notebook [08](notebooks/08_machine_learning.ipynb)*

- **No, the simple regression wins.** It predicts unseen landlords better than a random forest or gradient boosting. With fewer than 200 housing associations, the extra flexibility mostly fits noise.
- **The machine learning models back up the main finding.** The random forest relies almost entirely on stage 2 complaints, and hints that repairs on time matters most once a landlord is above about 80%.
- **Struggling landlords can be flagged, but half are missed.** A model trained to spot housing associations in the bottom quarter for satisfaction works well overall, but misses 23 of the 49. Their repairs and complaints figures look fairly ordinary, so their problem doesn't show up in the operational data.

![Model comparison](charts/13_model_comparison.png#gh-light-mode-only)
![Model comparison](charts/dark/13_model_comparison.png#gh-dark-mode-only)

![Predicted risk against actual satisfaction](charts/16_risk_vs_actual.png#gh-light-mode-only)
![Predicted risk against actual satisfaction](charts/dark/16_risk_vs_actual.png#gh-dark-mode-only)

<details>
<summary>Technical detail and more charts</summary>

- scikit-learn pipelines, 5-fold cross-validation repeated 10 times. Cross-validated R²: regression 0.49, random forest and gradient boosting about 0.43. Mean absolute error 5.3 points, against 7.8 for a baseline that always predicts the average.
- Importance from permutation importance; shape from partial dependence curves.
- Bottom-quartile classifier: logistic regression, cross-validated AUC about 0.85.

![Feature importance](charts/14_feature_importance.png#gh-light-mode-only)
![Feature importance](charts/dark/14_feature_importance.png#gh-dark-mode-only)

![Partial dependence](charts/15_partial_dependence.png#gh-light-mode-only)
![Partial dependence](charts/dark/15_partial_dependence.png#gh-dark-mode-only)

</details>

## 6. What matters most to tenants?

*Notebook [09](notebooks/09_key_drivers.ipynb)*

- **The 11 specific survey questions explain almost all of a landlord's overall satisfaction score.** They overlap so much that an ordinary regression can't separate them, so I used **dominance analysis**, which shares the credit out fairly between them.
- **The three biggest drivers are a well-maintained home (12.9% of what's explained), being listened to (12.4%) and being treated fairly and with respect (11.4%).** The top two stay in the top three however the data is resampled.
- **Being listened to is the clearest opportunity.** It's the second biggest driver, and only 65.5% of tenants are satisfied with it at a typical housing association. Complaints handling has the most room to improve (38.7% at a typical housing association). A well-maintained home matters most but already scores fairly well (74.7%).

![Key drivers](charts/17_key_drivers.png#gh-light-mode-only)
![Key drivers](charts/dark/17_key_drivers.png#gh-dark-mode-only)

![Priority matrix](charts/18_priority_matrix.png#gh-light-mode-only)
![Priority matrix](charts/dark/18_priority_matrix.png#gh-dark-mode-only)

<details>
<summary>Technical detail</summary>

- 193 housing associations. The 11 measures explain 95% of the variation (R²), with VIFs up to 26.
- Dominance analysis fits all 2,048 combinations of the 11 measures. It's the same idea as Shapley values in machine learning.
- Stability checked with 200 bootstrap resamples.
- Medians here are for housing associations only, so they differ slightly from the all-landlord figures in section 1.

</details>

## 7. What this means

**Escalated complaints are an early warning.** A stage 2 complaint means a tenant's problem wasn't put right the first time. Housing associations with more of them have lower satisfaction, so the number reaching stage 2 is worth watching closely, alongside repairs on time. The link could run both ways, though: unhappy tenants may also be more likely to escalate.

**Landlords need to be compared like with like.** A landlord that switched from a postal survey to an online one could see its score fall by several points without the service changing at all. So a league table of every landlord partly ranks how they ran their surveys. The fairest comparison is with landlords using the same survey method, and a change of method should be treated as a break in the trend.

**Context doesn't explain away a low score.** Deprivation, the survey company and the type of homes make no clear difference once repairs and complaints are allowed for. A low score can't simply be put down to where a landlord's homes are or what kind of homes they are. The differences that remain are more likely to be about the service itself.

**Year-on-year changes need care.** Low scorers tend to improve by chance, so a rise after a bad year isn't necessarily proof that something worked. In the same way, a fall after a very good year isn't necessarily a sign of decline.

**Simple models are the right tool here.** With fewer than 200 housing associations, a well-checked regression was more accurate than machine learning, and it's much easier to explain to a manager or a board. Complex models are worth trying again once more years of data are available.

**Not everything shows up in the operational figures.** About half of the struggling housing associations have ordinary-looking repairs and complaints figures. Two of the things tenants care about most, being listened to and being treated fairly, are about how tenants are dealt with day to day rather than anything the operational measures count. A landlord's own data, such as individual survey responses, complaint themes and tenants' comments, is the place to look for those.

**Next for this project:** add the 2025/26 TSMs when they're published, to give a third year for the change analysis.

---

## The notebooks

The notebooks are numbered as steps 1 to 9. The README sections above group them by question.

| Notebook | README section | What it does |
| --- | --- | --- |
| [01 Load and explore](notebooks/01_load_and_explore.ipynb) | 1 | Loads the survey results, tidies column names, profiles the 353 landlords, checks data quality and looks at which measures vary most |
| [02 Join and explore](notebooks/02_join_and_explore.ipynb) | 1 | Adds the repairs, complaints and other operational data, sets out its caveats, compares landlords by size and checks correlations |
| [03 Regression model](notebooks/03_regression_model.ipynb) | 2 | Builds and checks the regression model of overall satisfaction, then looks at which landlords it predicts well or badly |
| [04 Charts](notebooks/04_charts.ipynb) | 2 | Builds charts 1 to 4 |
| [05 Deprivation](notebooks/05_deprivation.ipynb) | 3 | Gives each housing association a deprivation score from where its homes are, and tests it in the model. Charts 5 and 6 |
| [06 Survey and stock checks](notebooks/06_survey_and_stock_checks.ipynb) | 3 | Tests chance, survey company, who did the interviews and type of homes. Charts 7 to 9 |
| [07 Year on year](notebooks/07_year_on_year.ipynb) | 4 | Matches landlords across two years, measures regression to the mean and tests whether changes in service go with changes in satisfaction. Charts 10 to 12 |
| [08 Machine learning](notebooks/08_machine_learning.ipynb) | 5 | Compares the regression with a random forest and gradient boosting, and tests a model to flag struggling landlords. Charts 13 to 16 |
| [09 Key drivers](notebooks/09_key_drivers.ipynb) | 6 | Works out which survey questions matter most to overall satisfaction, and combines that with how the sector scores. Charts 17 and 18 |

Each notebook explains what I did and why as it goes, in notes between the steps and comments in the code, and ends with a summary of what I found.

## Method

- **Like-for-like comparisons.** Results are compared within the same survey method where it matters, and the models allow for survey method, size and region.
- **Housing associations only** for the models, because councils differ in ways the data doesn't capture.
- **Rank (Spearman) correlations**, because several measures are skewed by a few extreme values.
- **Predictors chosen so they don't overlap.** Stage 1 and stage 2 complaint numbers are closely linked, so only stage 2 is used.
- **Operational measures used to explain overall satisfaction**, because the 12 survey measures are all closely linked to each other. Notebook 9 deals with that overlap directly.
- **Every model tested on landlords it hasn't seen**, with checks for unusual landlords throughout.

## Limitations

- **Linked doesn't mean caused.** Unhappy tenants may complain more, as well as complaints making tenants unhappy. The year-on-year analysis gets closer to cause and effect, but only has one change to work with.
- **Landlords report their own operational figures.** "Repairs on time" is measured against each landlord's own target, which ranges from 5 to 260 days. Non-decent homes figures depend on how recently homes were surveyed, and complaints recording changed when the Complaint Handling Code became law.
- **Small numbers.** Fewer than 200 housing associations means a few unusual landlords can sway the results, and some groups are small (26 landlords used their own staff for interviews).
- **Many tests.** Eight deprivation scores were tested, so a borderline result or two would be expected by chance.
- **Areas, not households.** Deprivation is measured by local authority, which probably understates how deprived social tenants' own neighbourhoods are.
- **Landlord averages, not individual tenants.** The key driver analysis shows which measures move together across landlords. Individual survey responses would give a stronger answer.
- **The gap between a landlord's actual and predicted score isn't a measure of anything specific.** It's simply what the model's measures don't account for.

## Terms used

<details>
<summary>Plain-English definitions</summary>

- **Housing association:** a social landlord that isn't a council (registered providers in the regulator's data).
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

## Data and how to run

1. Download the source files listed in [`data/README.md`](data/README.md) and save them in `data/` with the names shown there.
2. Install the requirements: `pip install -r requirements.txt`
3. Run the notebooks in order. Each one also runs in Google Colab, where it asks you to upload the files it needs.

The charts are saved in `charts/`. Dark versions in `charts/dark/` use the same data with a dark colour palette, and GitHub shows them automatically if you use dark mode.

Contains public sector information licensed under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/). Sources: Regulator of Social Housing, Tenant Satisfaction Measures 2023/24 and 2024/25 and Statistical Data Return 2025; Ministry of Housing, Communities and Local Government, English Indices of Deprivation 2025.

## About

Ben Hobbs, data analyst in social housing. Built with Python (pandas, statsmodels, scikit-learn, matplotlib). [LinkedIn](https://www.linkedin.com/in/ben-hobbs-608347361/)
