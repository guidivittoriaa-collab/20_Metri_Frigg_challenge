# 20_Metri_Frigg_challenge
20_Metri — Training Data README

This archive contains all data files used to train and validate both the
short-term (XGBoost quantile regression) and long-term (linear regression +
climatology shaping) forecasting models.

Folder structure:
short_term/
  electricity_prices_delu/   Day-ahead prices + generation mix (DE-LU, 2024–2026)
  electricity_prices_es/     Day-ahead prices + generation mix (ES, 2024–2026)
  gas/                       TTF natural gas daily prices (2024–2026)
  tso_forecast/              ENTSO-E wind+solar day-ahead generation forecasts (2024–2026)
  weather/                   Open-Meteo weather observations and NWP forecast

long_term/
  electricity_prices_delu/   Day-ahead prices + generation mix (DE-LU, 2018–2026)
  electricity_prices_es/     Day-ahead prices + generation mix (ES, 2017–2026)
  scenarios/                 Future scenario inputs (renewable share, carbon, gas)


SHORT-TERM DATA 

- short_term/electricity_prices_delu

Source : energy-charts.info (Fraunhofer ISE)
URL    : https://energy-charts.info/charts/price_spot_market/chart.htm
Format : CSV, 1 row per hour, timezone CET/CEST (GMT+1)

Files:
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2024.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2025.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2026.csv
    Date range : 2024-01-01 to 2026-05-07 (hourly)
    Variables  : Date (CET/CEST), Non-Renewable generation (MW),
                 Renewable generation (MW), Day Ahead Auction EXAA price (EUR/MWh)
    Used for   : target variable (price), renewable generation share feature

  energy_charts_delu_2025_patch.csv
    Date range : Subset of 2025 (UTC timestamps, hourly)
    Variables  : timestamp_utc, price_eur_mwh
    Purpose    : Fills gaps in the 2025 energy-charts file caused by
                 data availability issues around the CET/CEST transition.
    Used for   : target variable gap-fill

  recent_prices_delu.csv
    Date range : ~2026-04-23 to 2026-05-07 (15-minute resolution, UTC)
    Variables  : timestamp (UTC), price_eur_mwh
    Source     : energy-charts.info API (fetched by update_data.py)
    Used for   : same_hour_level feature (recent price regime anchor)

- short_term/electricity_prices_es/ 

Source : energy-charts.info (Fraunhofer ISE)
Format : CSV, 1 row per hour, timezone CET/CEST (GMT+1)

Files:
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2024.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2025.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2026.csv
    Date range : 2024-01-01 to 2026-05-07 (hourly)
    Variables  : Date (CET/CEST), Non-Renewable generation (MW),
                 Renewable generation (MW), Day Ahead Auction (ES) price (EUR/MWh)
    Used for   : target variable (price), renewable generation share feature

  recent_prices_es.csv
    Date range : ~2026-04-23 to 2026-05-07 (15-minute resolution, UTC)
    Variables  : timestamp (UTC), price_eur_mwh
    Source     : energy-charts.info API (fetched by update_data.py)
    Used for   : same_hour_level feature (recent price regime anchor)

- short_term/gas/ 

  ttf_daily.csv
    Source     : Yahoo Finance (TTF=F front-month futures contract)
    Date range : 2024-01-02 to 2026-05-07 (daily, business days only)
    Variables  : date, ttf_eur_mwh
    Used for   : gas_price feature — marginal cost anchor for both zones.
                 Forward-filled from daily to hourly for model input.
    Note       : TTF (Title Transfer Facility) is the Continental European
                 reference gas hub, used by both DE-LU and ES as pricing benchmark.

─ short_term/tso_forecast/ 

Source : ENTSO-E Transparency Platform — Wind and Solar Day-Ahead Generation Forecast
URL    : https://transparency.entsoe.eu
Format : CSV, 15-minute resolution, CET/CEST

Files:
  DELU2024.csv, DELU2025.csv, DELU2026.csv   (bidding zone: BZN|DE-LU)
  ES2024.csv,   ES2025.csv,   ES2026.csv     (bidding zone: BZN|ES)
    Date range : 2024-01-01 to 2026-05-07 (15-minute intervals)
    Variables  : MTU (CET/CEST), Area, Day-ahead (MW), Intraday (MW),
                 Current (MW), Actual (MW)
    Used for   : tso_forecast_mw feature — the day-ahead scheduled
                 wind+solar generation, the same signal used by market
                 participants when submitting bids.
    Aggregation: 15-minute data averaged to hourly before use.

─ short_term/weather/ 

Source : Open-Meteo (https://open-meteo.com)
         Historical Weather API + Weather Forecast API
Format : CSV, hourly, UTC

Files:
  Historical_DELU.csv
  Historical_ES.csv
    Date range : 2024-01-01 to ~2026-05-06 (hourly, UTC)
    Variables  : time, temperature_2m (°C), wind_speed_100m (km/h),
                 cloud_cover (%), precipitation (mm),
                 shortwave_radiation (W/m²)
    Used for   : weather features in short-term training data
    Location   : Representative grid point for each bidding zone
                 (DE-LU: Frankfurt area ~50.09°N 8.65°E;
                  ES: Madrid area ~40.42°N -3.70°E)

  Forecast_DELU.csv
  Forecast_ES.csv
    Date range : ~2026-05-06 to ~2026-05-22 (hourly, UTC)
    Variables  : same as historical files
    Used for   : weather features for the evaluation window prediction
                 (the model uses forecast weather for future hours)


LONG-TERM DATA (used by linear regression long-term model, horizon > 14 days)

─ long_term/electricity_prices_delu/ 

Source : energy-charts.info (Fraunhofer ISE) — full annual downloads
Format : CSV, 1 row per hour, CET/CEST. Contains full generation mix breakdown
         and CO2 emission price (not available in the short-term 4-column files).

Files:
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2018.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2019.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2020.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2021.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2022.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2023.csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2024 (1).csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2025 (1).csv
  energy-charts_Electricity_production_and_spot_prices_in_Germany_in_2026 (1).csv
    Date range : 2018-01-01 to 2026-05-07 (hourly)
    Key variables used:
      Renewable generation (MW)        — to compute renewable_share_pct
      Non-Renewable generation (MW)    — denominator for share calculation
      Day Ahead Auction price (EUR/MWh) — target variable for monthly aggregation
      CO2 Emission Allowances (EUR/t)  — carbon_eur_t feature
    Note       : Gas price column is NOT present in these files.
                 Gas prices for DE-LU are sourced from the scenario file
                 (actual TTF values for 2018–2026, IEA projections thereafter).

─ long_term/electricity_prices_es/ 

Source : energy-charts.info (Fraunhofer ISE) — full annual downloads
Format : CSV, 1 row per hour, CET/CEST.

Files:
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2017.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2018.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2019.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2020.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2021.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2022.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2023.csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2024 (1).csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2025 (1).csv
  energy-charts_Electricity_production_and_spot_prices_in_Spain_in_2026 (1).csv
    Date range : 2017-01-01 to 2026-05-07 (hourly)
    Key variables used:
      Renewable generation (MW)        — to compute renewable_share_pct
      Non-Renewable generation (MW)    — denominator for share calculation
      Day Ahead Auction (ES) price (EUR/MWh) — target variable
      CO2 Emission Allowances (EUR/t)  — carbon_eur_t feature
    Note       : ES files carry no gas price column. Gas prices for ES are
                 filled from DE-LU TTF data (both zones reference TTF as the
                 Continental European gas benchmark).

─ long_term/scenarios/DE_ES_scenarios.csv 

  Source     : Compiled from public policy targets and market data
  Date range : 2018–2045 (annual rows, one per zone)
  Zones      : DE-LU and ES
  Variables  :
    year                 — calendar year
    zone                 — DE-LU or ES
    renewable_share_pct  — annual renewable share (%)
                           2018–2026: actual from energy-charts data
                           2027–2045: interpolated from national targets
                             DE: Energiewende — 80% by 2030, 100% by 2035
                             ES: PNIEC — 81% by 2030, 98% by 2045
    carbon_eur_t         — EU ETS CO2 allowance price (EUR/t)
                           2018–2026: actual market data
                           2027–2045: EU ETS trajectory (125 by 2030, 200 by 2040)
    gas_eur_mwh          — TTF gas price (EUR/MWh)
                           2018–2026: actual TTF annual averages
                           2027–2045: IEA moderate scenario (gradual decline to 15 by 2045)
    source               — 'actual' or 'scenario'
  Used for   : Future feature inputs to the long-term linear regression model
               (replaces energy-charts data for years beyond training set)



NOTES:
Timezone handling
  All data is converted to UTC before use. energy-charts files are in CET/CEST
  (UTC+1/+2); ENTSO-E TSO files are in CET/CEST; weather files from Open-Meteo
  are already in UTC.

Data splits
  Short-term validation : last 60 days of available price data (held out)
  Long-term validation  : leave-one-year-out (LOYO) cross-validation

Missing data handling
  Price gaps are forward-filled (max 3 hours). The 2025 DELU patch file fills
  a ~2-week gap in the energy-charts 2025 annual download.
  Weather gaps: linear interpolation up to 6 hours.

