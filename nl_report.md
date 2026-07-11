# Netherlands Building Archetypes for SESMG — where things stand

Quick write-up of the Dutch archetype work, mirroring what was done for NRW. The goal was to
take every Dutch residential building (~5.7M of them), give each one a TABULA archetype
(building type × construction period), attach its heat demand, hot water, and PV roof area, and
boil the whole thing down to a table SESMG can actually use. Method follows Yang et al.,
*Applied Energy* 280 (2020) 115953.

It's done and validated — the whole thing runs from one notebook in about 3 minutes (15 more
for the roof clustering).

## The numbers

| | |
|---|---|
| buildings / dwellings | 5,616,654 / 8,510,012 |
| reference floor area | 983 Mm² (115.5 m²/dwelling; CBS says ≈ 120) |
| **space heat (calculated)** | **106 TWh/yr**, mean 108 kWh/(m²·a) |
| DHW (kept separate) | 10.1 TWh/yr |
| usable PV roof area | 557 Mm² gross tilted surface |
| final archetypes | 1,514 |

## The one thing I need you to decide

There are two heat-demand numbers, and they differ by a factor of 2.2. Picking between them is
a modelling call, not something the data settles:

- **~106 TWh (calculated / asset demand)** — what the building physics says the stock needs at a
  20 °C setpoint. This is the one you want for sizing heat pumps and networks.
- **~48 TWh (measured / occupant demand)** — what households actually burn, from CBS gas figures.
  This is the one you want for matching real bills or forecasting.

The gap between them isn't an error — it's the prebound effect. Old, badly-insulated homes just
get heated well below 20 °C. It's well documented for exactly this housing stock (Majcen, Itard
& Visscher 2013, *Energy Policy* 54). My instinct would be to carry both series and use ~0.45×
for occupant-facing scenarios, but that's your call to make.

## Why it's 106 and not 71 TWh

Early on I had the refurbishment share `f` set to 0.77 (roughly the Leiden component-renovation
rate). That turned out to be wrong and was pulling the national total down by a third.

What fixed it was EP-Online, the national energy-label register. Its NTA 8800 `Warmtebehoefte`
field is a *calculated* heat need, same kind of number as TABULA's — so comparing the two cancels
out the prebound effect and leaves just the refurbishment state. Across 945k matched buildings,
the pre-1975 stock lines up with TABULA's *unrefurbished* values (ratio 0.98–0.99). And it's not
a sampling fluke: social housing, which is both the most-renovated stock and the best-covered in
the register, shows no refurbishment against baseline either. So `f = 0`.

## What got checked against real data

- **Building-type classification: 98.2% agreement** on 779k buildings against the register's
  `Gebouwtype`. The party-wall rule I made up actually holds. (Caveat: the register can't tell MFH
  from AB — both are just "Appartement" — so that particular split is still untested.)
- **Reference floor area: ratio 1.00** on 29k fully-labelled buildings. I'd worried apartment
  blocks were understated; they're not.
- **Roof orientation is uniform**, which means excluding north-facing roofs throws away exactly
  25% of the pitched area — a clean number rather than a guess.

## Caveats and loose ends

- TABULA runs 15–30% low for 1975–2005 and 13% high for 2015+. It's not refurbishment (those
  periods are too new), it's some offset between the calculation standards, and it changes sign so
  you can't just subtract it. Still unresolved.
- The period-06 TABULA row is six hand-filled numbers, and the register says they're ~13% too high.
  Worth replacing with EP-Online medians once someone confirms where they came from.
- DHW is ~4% high because BAG counts a few percent more dwellings than CBS does. Heat demand isn't
  affected.
- That 557 Mm² of PV area is gross tilted surface — it needs a 40–60% utilisation factor before it
  means anything real.
- Not built yet: electricity demand (CBS 81528ENG — this is the big one), hourly profiles (SESMG
  needs them), PV yield, geothermal.
- The NRW comparison is blocked on three things: the PV areas are computed differently on each side
  (footprint × a constant vs actual tilted surface), NRW's building-type field isn't defined anywhere
  in the code, and the 4,806 archetype count has three different derivations that don't agree.



**Files:** `nl_archetypes_full.ipynb`, `nl_archetypes_sesmg_v3.csv`, and the per-building parquet.


## Sources

**Papers**
- Yang, X., Hu, M., Tukker, A., Zhang, C., Huo, T., & Steubing, B. (2020). A combined GIS-archetype approach to model residential space heating energy: a case study for the Netherlands including validation. *Applied Energy*, 280, 115953. https://doi.org/10.1016/j.apenergy.2020.115953
- Majcen, D., Itard, L.C.M., & Visscher, H. (2013). Theoretical vs. actual energy consumption of labelled dwellings in the Netherlands: Discrepancies and policy implications. *Energy Policy*, 54, 125–136. https://doi.org/10.1016/j.enpol.2012.11.008

**Data**
- 3DBAG (building geometry, LoD2.2) — https://3dbag.nl/
- BAG, Basisregistratie Adressen en Gebouwen (dwelling areas, use function) — https://www.kadaster.nl/zakelijk/registraties/basisregistraties/bag
- NL-TABULA building typology — https://webtool.building-typology.eu/ (project data: https://episcope.eu/building-typology/)
- EP-Online, national energy-label register (NTA 8800 `Warmtebehoefte`) — API key: https://apikey.ep-online.nl/ · downloads: https://www.ep-online.nl/PublicData
- CBS StatLine, table 81528ENG (energy consumption, for the not-yet-built electricity demand) — https://opendata.cbs.nl/statline/
- NEDU standaardjaarprofielen (hourly gas/electricity profiles, not yet built) — https://www.nedu.nl/documenten/verbruiksprofielen/
- PDOK / Kadaster cadastral parcels (for the geothermal step, not yet built) — https://www.pdok.nl/
