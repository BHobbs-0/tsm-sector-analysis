# Data

All the data in this project is public. The source files aren't stored in this repository, so that it always points to the latest official versions. RSH and MHCLG occasionally revise their releases. To run the notebooks, download the files below and save them in this folder under the names shown.

| File | Used by | Source |
| --- | --- | --- |
| `2025_TSM_Full_Data_v1.1_FINAL.xlsx` | Notebooks 1 to 9 | [RSH: Tenant Satisfaction Measures 2024/25](https://www.gov.uk/government/statistics/tenant-satisfaction-measures-202425) |
| `2024_TSM_Full_Data_v2.1_FINAL.xlsx` | Notebook 7 | [RSH: TSM 2024 data release](https://www.gov.uk/government/statistics/data-release-tsm-2024) (covers 2023/24) |
| `PRP_Data_Release_2025_Full_Data_FINAL_V1.1.zip` | Notebooks 5 and 6 | [RSH: Private registered provider social housing stock and rents 2024 to 2025](https://www.gov.uk/government/statistics/private-registered-provider-social-housing-stock-and-rents-in-england-2024-to-2025) (full data release, zip) |
| `File_10_-_IoD2025_Local_Authority_District_Summaries__lower-tier__v2.xlsx` | Notebook 5 | [MHCLG: English Indices of Deprivation 2025](https://www.gov.uk/government/statistics/english-indices-of-deprivation-2025) (File 10, v2) |

If you're using Google Colab, you don't need to save anything here. Each notebook shows an upload button and asks for the files it needs.

## Derived data

`landlord_deprivation.csv` is included here because I created it. Notebook 5 builds it, with one row per housing association. Each row has the association's SDR low cost rental homes, how many local authorities those homes are in, and its stock-weighted average deprivation scores (overall IMD and the seven domains). Notebook 6 reads this file, so upload it with the other files when running notebook 6 in Colab.

## Licence

The source data is published under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).
