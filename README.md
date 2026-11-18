# Kenya Maize Production by County (2020-2024)

I grew up around farmers, and maize is the crop everyone talks about. Some years the harvest is good and prices drop. Other years it fails and flour gets expensive. I wanted to look at the actual numbers behind that: which counties grow most of Kenya's maize, which ones get the most out of each hectare, and what the national figures say about supply and prices.

This repo is that analysis, done in Python with pandas, NumPy, Matplotlib, Seaborn and a small SQLite database.

## Key questions

- Which counties produce the most maize?
- Which counties have the highest average yield?
- How does the area planted relate to how much is harvested?
- How much do county yields change from year to year?
- What do the national supply, use and price figures show?

## The data

Everything comes from one Excel file, "Kenya Maize Production 2019 to 2025, with County Area and Production", put together by Stats Kenya from two official sources
i. KNBS Economic Survey 2026 
ii. AFA Yearbook of Statistics 2025.

The sheets don't cover the same years:

- County data (area and production for all 47 counties): 2020-2024
- National AFA data (area, production, yield, farm-gate price, imports): 2019-2024
- National KNBS data (production, farmer and retail prices): 2021-2025
- Supply and use (where the maize went): 2024 and 2025 only

The 2020-2024 in the title refers to the county analysis, which is the core of the project.

Datas limitations:

- **KNBS and AFA don't give the same production numbers.** For 2021 they're about 11,000 tonnes apart, which is more than rounding. So I never join the two into one line. I use AFA as the main national series because it covers more years and the county figures come from it: the 47 counties add up to within 0.25% of AFA's national production in every year from 2020 to 2024. KNBS is shown only for comparison.
- **The 2025 figures are provisional.**
- **Prices are national and not adjusted for inflation**
- **Very small maize areas give unreliable yields.** For each county I take the average area planted over 2020-2024. Counties averaging under 1,000 ha are flagged in the cleaning notebook. They stay in everything to do with production (totals, county comparisons, the area vs production analysis), but they're left out of the yield rankings, the yield charts and the area-and-yield grouping. On a few hundred hectares, a small change in area can make yield jump several times over from one year to the next, and I didn't want that noise deciding the rankings. Five counties fall under the line: Wajir, Garissa, Isiolo, Nairobi and Mombasa, which leaves 42 counties in the yield rankings. The 1,000 ha cut-off is my own judgement and is set in one line of the cleaning notebook, so it's easy to change.
- **Mombasa reports 0 tonnes for 2021** on a planted area that isn't zero. I couldn't tell whether that's a real failed season or a missing figure so I kept it as published. Mombasa is one of the small counties so it doesn't affect the yield rankings.
- **AFA's reported yield for 2023 doesn't match its own figures.** AFA gives yield in 90-kg bags per hectare, so I converted its production the same way (tonnes x 1,000 / 90). Using the 2023 production and area from the same row of the same AFA table, 4,285,206 t on 2,552,931 ha works out to about 18.65 bags/ha, not the reported 22.8. For every other year the recalculated yield is within about half a bag of the reported one. Using the county area total for 2023 instead (2,430,013 ha) gives about 19.6 bags/ha, which still doesn't explain it. Stats Kenya's own review notes on the dataset page point out the same problem. I don't know which number is wrong, so I flag it rather than pick one, and my yield analysis uses yield calculated from production and area.

The data says nothing about rainfall, soil, fertiliser or seed. It can show which counties do better but not why.

## What's in the repo

```
data/raw/            original Excel file (untouched)
data/processed/      cleaned CSVs
notebooks/           the analysis
outputs/figures/     charts
database/            SQLite database (built by the last notebook)
reports/             written report
```

The notebooks, in the order I'd read them:

- `data_understanding.ipynb` - first look at each sheet, units, and what the data can and can't answer
- `data_cleaning.ipynb` - quality checks, reshaping the county table, adding yield, saving clean CSVs
- `eda_visualisation.ipynb` - the analysis and charts
- `sqlite_database.ipynb` - the same data in a relational database, queried with SQL

## What I did

In the county sheet years had been added in the column names ("Area (ha) 2020", "Production (t) 2020"), so i reshaped it to one row per county per year. After that i calculated yeilds.

Before any analysis, the cleaning notebook checks:

- missing values, duplicate rows and repeated years
- that all 47 counties are there, each exactly once, with no spaces in the names
- whether the county totals add up to the AFA national totals, comparing the same years (2020-2024) in the same units (tonnes and hectares)
- whether the supply and use table balances: (production + imports) - (exports and stock change) = total of all the uses

For the analysis, I compared counties on five-year averages. I also looked at how concentrated production is, how strongly planted area relates to harvest, how yields differ between counties and how much they move year to year, plus supply, use and prices.

i also found out that counties with a lot of land under maize but low yields are where small improvements would add the most food.

## Results

**National trend (AFA).** Production fell three years running, from about 3.96 million tonnes in 2019 to 3.09 million in 2022, a drop of 22%. It then jumped 39% to 4.29 million tonnes in 2023, the highest year in the series, and eased to 4.03 million in 2024. Part of the 2023 rise came from more land: the area under maize grew about 17.5% that year. Six years isn't enough to call this a long-term trend, but the vary shows how much production moves from season to season.

**Where the maize comes from.** Uasin Gishu is the biggest producer, averaging about 432,000 tonnes a year which is 11.7% of the total, with Trans Nzoia close behind at about 422,000 tonnes (11.4%). Production is concentrated meaning the top five counties grow 43% of the maize, and just seven counties grow half of it.

**Yield.** Across all counties the average is 1.64 tonnes per hectare which is about 18 bags. Uasin Gishu has the highest yield of the 42 ranked counties at 4.01 t/ha that's about 45 bags per hectare, followed by Trans Nzoia at 3.78. So the top two counties lead both because they plant a lot and because they get far more out of each hectare.

**Area vs production.** Counties that plant more generally harvest more. The correlation is 0.68 measured as a straight line and 0.85 measured on rankings. The gap between those two numbers says the relationship isn't one neat line. Some counties plant a lot and harvest very little, which pulls the straight-line fit down.

**Something I didn't expect.** Makueni and Machakos each have more land under maize than Uasin Gishu or Trans Nzoia (about 144,000 and 142,000 ha on average), but their yields are among the lowest in the country: 0.63 and 0.55 t/ha. Machakos's yield also varies a lot from year to year. This is also the same with Kitui Kilifi and Murang'a, they fall in the "large area low yield" group which is where better yields would add the most maize. The data can't say why. County rainfall and input data would be the first thing to bring in to find out.

**Supply and prices.** Food took 78% of domestic supply in 2024 and 93% in 2025, but most of that jump is because feed went from 15% to zero, which is the figure I'm unsure about. Imports covered 9% of supply in 2024 and 15% in 2025. The price paid to farmers more than doubled in nominal terms from KSh 30 to KSh 70 per kg between 2021 and 2025. In 2024 and 2025 the KNBS farmer price was higher than the KNBS retail prices which looks odd and probably means the two series measure different things.

## Running it

You'll need Python 3.12 or newer, because the version of NumPy in `requirements.txt` doesn't support anything older. I tested everything on Python 3.14.5 with pandas 3.1.1, NumPy 2.5.5, Matplotlib 3.11.2 and Seaborn 0.13.2. The exact versions of every package are pinned in `requirements.txt`.

```cmd
python -m venv .venv
.venv\Scripts\activate
python -m pip install -r requirements.txt
```

Put `kenya_maize_production_2019_2025_v2.xlsx` in `data/raw/`, then run the notebooks in the order listed above. The analysis and database notebooks read the files the cleaning notebook creates, so cleaning has to run first.

## Source and licence

**Dataset:** "Kenya Maize Production 2019 to 2025, with County Area and Production", compiled by Stats Kenya (minor processing: reformatting, renaming, unit conversions).
https://statskenya.co.ke/at-stats-kenya/about/maize-production-in-kenya-by-county-agriculture-statistics/130/

**Original sources:**
- Kenya National Bureau of Statistics (2026). Economic Survey 2026. https://www.knbs.or.ke/reports/economic-survey-2026/
- Agriculture and Food Authority (2025). AFA Yearbook of Statistics 2025. https://www.afa.go.ke/

**Citation (as given in the dataset):** KNBS (2026); AFA (2025), with minor processing by Stats Kenya.

**Licence:** the dataset is published under CC BY-NC 4.0 (https://creativecommons.org/licenses/by-nc/4.0/). You can share and adapt it for non-commercial purposes as long as you credit it, and where a source's own terms are stricter, those apply too. The Economic Survey is also CC BY-NC 4.0, and the AFA tables are public statistics to be credited to the Agriculture and Food Authority. This is a non-commercial personal project.
