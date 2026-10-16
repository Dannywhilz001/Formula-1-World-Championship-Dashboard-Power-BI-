Formula 1 World Championship Dashboard (Power BI)

An interactive four-page Power BI report covering every Formula 1 season in the dataset, from 1950 to 2024: **75 seasons, 1,125 Grands Prix, 861 drivers and 211 constructors**. I built it with Power Query, a DAX measures layer, and a custom dark F1 theme, and designed it so someone who has never opened Power BI can explore it with a few clicks.

[Overview page](https://ibb.co/hRMjTYVs)

> Unofficial fan project. Not affiliated with, endorsed by, or connected to Formula 1, the FIA, or any team or driver. See [Disclaimer](#disclaimer).

## Pages

| Page | What it answers |
|---|---|
| **Overview** | How big is the sport's history? Top 10 drivers and constructors by wins, how the calendar grew, and how reliability changed over time |
| **Drivers** | Career profile: podiums, poles, fastest laps, win rate, finishing position distribution, wins and points by season, and the teams a driver raced for |
| **Circuits and Races** | Does pole position win races? Pole conversion, winner's grid position, DNF rate, a circuit table, and the top winning drivers at a chosen circuit |
| **Season Review** | Hall of fame view: most successful driver and team, wins by driver, constructor points, and race winners |

Slicers for season, driver, circuit, and country are synced across pages.

| | |
|---|---|
| [Drivers](https://ibb.co/QFBydGyh) | [Circuits and Races](https://ibb.co/0RqKFHgC) |
| [Season Review](https://ibb.co/vxrG7Jcm) | |

## Headline numbers

| Metric | Value |
|---|---|
| Seasons | 75 (1950 to 2024) |
| Grands Prix held | 1,125 |
| Drivers | 861 |
| Constructors | 211 |
| Total points scored | about 53.2K |
| Overall DNF rate | 43.3% |
| Overall podium rate | 12.7% of starts |
| Overall win rate | 4.2% of starts |
| Pole conversion (pole starts that became wins) | 42.3% |

## What the data shows

- **Lewis Hamilton has the most race wins** of any driver in the dataset, ahead of Michael Schumacher and Max Verstappen.
- **Ferrari leads all constructors with 249 wins**, followed by McLaren (185), Mercedes (129), Red Bull (122), and Williams (114).
- **The calendar has more than tripled:** from fewer than 10 races in the early 1950s to 24 in 2024.
- **Cars are far more reliable.** The share of starts that ended in a retirement peaked at roughly 70% in the late 1980s and is around 10% in recent seasons.
- **Pole position matters, but it is not decisive.** About 42% of poles turned into a win, and roughly 480 race winners started from the front of the grid.

## Data

- **Source:** [Formula 1 World Championship dataset on Kaggle](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020), published by Rohan Rao. Check the license on the Kaggle page before reusing the data.
- **Tables used:** races, results, drivers, constructors, circuits, status, seasons, driver_standings, constructor_standings, qualifying.
- **Not loaded:** lap_times, pit_stops, sprint_results, constructor_results. They were left out to keep the model simple and fast.
- The raw CSV files are not stored in this repository. Download them from Kaggle (see [Run it locally](#run-it-locally)).

## Tools

- **Power BI Desktop:** data model, DAX, report design
- **Power Query (M):** loading, cleaning, and reshaping the CSV files
- **DAX:** all measures live in a dedicated `_Measures` table
- **Deneb (Vega-Lite):** custom rounded bar charts
- **Custom theme (JSON):** consistent colors and fonts across pages

## Data model

A multi-fact star schema: `results`, `driver_standings`, `constructor_standings`, and `qualifying` are the fact tables, and they share the `races`, `drivers`, `constructors`, `circuits`, `status`, and `seasons` dimensions. All 13 relationships are many-to-one with single-direction filtering. Key columns are hidden in the report view. Full details in [docs/data-model.md](docs/data-model.md).

## Data preparation

All cleaning was done in Power Query so the raw files are never edited and the process is repeatable. The main steps:

1. A reusable `fnLoad` function loads each CSV, promotes headers, and replaces the `\N` missing-value marker with real nulls before any data types are set.
2. Data types set, columns renamed to readable names, and URL and internal reference columns removed.
3. A `Driver Name` column created from forename and surname.
4. A `Finish Type` column added to the `status` table (Finished or DNF) so retirements can be counted.

Full walkthrough with code in [docs/data-preparation.md](docs/data-preparation.md).

## Key DAX measures

All measures are in [dax/measures.dax](dax/measures.dax). A few examples:

```dax
Wins = CALCULATE(COUNTROWS(results), results[Finish Position] = 1)

Win Rate = DIVIDE([Wins], COUNTROWS(results))

DNF Rate = DIVIDE([DNFs], COUNTROWS(results))

Pole-to-Win % = DIVIDE([Pole Wins], [Poles])
```

## Design

- **Palette:** near-black background (`#15151E`), F1 red (`#E10600`), white and light grey text
- **Fonts:** Cosmic for headline numbers and titles, Segoe UI for labels
- **Navigation:** page navigator buttons across the top of every page
- **Theme file:** [theme/F1_Dark_Theme.json](theme/F1_Dark_Theme.json). Import it with View > Themes > Browse for themes.

## Challenges and lessons learned

- **Missing values were text, not nulls.** The CSVs use `\N` for missing data. If you set a number type first, Power Query returns errors. The fix is to replace `\N` with null *before* changing the type, which is why the loader function does it in that order.
- **Rates need the right denominator.** My first DNF rate divided by the number of races, which gave a value of about 5 to 30 instead of a percentage. The correct denominator is the number of race entries (one row per car per race).
- **Standings tables are running totals.** Championship points are cumulative within a season, so summing them double-counts. Totals are calculated from the results table instead.
- **Top N includes ties.** A "Top 10" chart showed 11 bars when two drivers tied for tenth. A tie-breaker measure fixed it.
- **Wins (1,128) are slightly higher than races (1,125).** A few early-era races had a shared drive, where two drivers are credited with the win.
- **Synced slicers affect every page.** I chose measures that work for any season selection instead of single-season measures, so the report stays smooth.

## Limitations and data notes

- The points scoring system changed several times, so points are not directly comparable across eras. The jump in the points-by-season chart around 2010 reflects a scoring change, not a data error.
- Fastest-lap rankings only exist in the dataset from 2004 onward.
- **DNF** means anything other than a classified finish or a lapped finish, which includes disqualifications.
- **Poles** are counted as starts from first on the grid, which can differ slightly from official pole position records when grid penalties apply.
- Average finish position includes retirements, so it is higher than it would be for classified finishers only.

## Run it locally

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020) and unzip it into a folder on your computer.
2. Open `F1_Dashboard.pbix` in Power BI Desktop (Windows).
3. Go to **Transform data > fnLoad > Advanced Editor** and change the folder path to where you unzipped the CSV files.
4. Click **Close and Apply**, then explore the pages.

If you only want to look at the report, the screenshots above show every page.

## Repository structure

```
f1-power-bi-dashboard/
├── README.md
├── LICENSE
├── .gitignore
├── F1_Dashboard.pbix          # add your report file here (or link to it)
├── dax/
│   └── measures.dax
├── theme/
│   └── F1_Dark_Theme.json
├── docs/
│   ├── data-model.md
│   └── data-preparation.md
└── images/
    ├── overview
    ├── drivers
    ├── circuits-and-races
    └── season-review
```

## Disclaimer

This is an independent, non-commercial portfolio project. Formula 1, F1, and related names and marks are trademarks of their respective owners. Data comes from a public Kaggle dataset, and any images shown belong to their owners.

## Author

**Oladotun Olawale**, aspiring Business and Data Analyst, Nigeria

- [LinkedIn](https://www.linkedin.com/in/oladotun-olawale)
- [Email](oladotunolawale29@yahoo.com)

If you found this useful, a star on the repo is appreciated. ⭐

