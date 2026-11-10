# Carbon Emissions and Cost

The energy since the start is turned into grams of CO2 equivalent, for the machine and for the followed program:

```
carbon (g CO2e) = energy (kWh) × carbon intensity (g CO2e per kWh)
```

The carbon intensity is chosen in the Setup tab, under *Electricity*:

- **Country**: the yearly average of the country's electricity, from [Ember's yearly electricity data](https://ember-energy.org/data/yearly-electricity-data/), over the life cycle of the power plants. The first time, the country of the time zone is chosen (on Windows, the one of the region setting), or the world's average. After that, the one chosen last time.
- **Country, this month**: the average of the current calendar month, from the latest year [Ember's monthly electricity data](https://ember-energy.org/data/monthly-electricity-data/) has it, as electricity is cleaner in some seasons. Ember has monthly figures for fewer countries (88 and the world). Elsewhere, the yearly average is used, and the note under it says so.
- **Energy source**: electricity from a single source, such as your own solar panels.
- **Own value**: your own figure in g CO2e per kWh, such as your supplier's.

The energy sources use the life-cycle medians of the IPCC (AR5, WG3, Annex III), which Ember uses too, in g CO2e per kWh:

| Source | Wind | Nuclear | Hydro | Geothermal | Solar | Bioenergy | Gas | Oil | Coal |
|---|---|---|---|---|---|---|---|---|---|
| g CO2e per kWh | 11 | 12 | 24 | 38 | 48 | 230 | 490 | 700 | 820 |

Oil is Ember's figure for oil and other fossil fuels.

Changing the carbon intensity works out the emissions again from the last measurement, including after Stop.
The emissions are not written to the CSV files, which keep PowerJoular's columns.

## Cost

With a price per kWh (a comma or a point for the decimals) and a currency, the cost of the machine's energy since the start is shown, with the followed program's part of it.
The currency is written as text (EUR, $, £, CHF...), as the font has no euro sign.
An empty price means no cost.

## Everyday equivalents

Under the cost, the machine's energy and emissions are given in everyday terms, from the US EPA's [greenhouse gas equivalencies](https://www.epa.gov/energy/greenhouse-gas-equivalencies-calculator-calculations-and-references):
- A smartphone charge takes 0.019 kWh.
- An average gasoline car emits 3.93 × 10<sup>-4</sup> t CO2e per mile, so 244 g per km.
- Below one phone charge, the energy is given as the time a 10 W LED bulb would take.

These are orders of magnitude for people, not results to report.

## The data files

The values of the countries are read from ```carbon-intensity.csv``` and ```carbon-intensity-monthly.csv```, beside the program (in *Contents/Resources* of the application on macOS), or in ```../share/joularenergymeter/``` from it, as the Linux and FreeBSD packages install them.
Without these files, the same values built into the program are used.

An edited file works without compiling again:
- ```carbon-intensity.csv```: one row per country, ```ISO 3 code,Area,Year,grams per kWh```. The world has an empty code.
- ```carbon-intensity-monthly.csv```: one row per country and month, ```ISO 3 code,Year,Month,grams per kWh```. A country needs all twelve months, or it has none.

To update them, run from the source folder:

```
python3 data/update.py
```

It downloads Ember's latest yearly and monthly data, and rewrites both files and their built-in copy, ```src/carbon_data.c```.
```python3 data/update.py YEARLY.csv MONTHLY.csv``` uses files already downloaded.
Ember adds a new year once a year, and new months each month.
