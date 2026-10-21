# Weather Station Archive

Temperature observations, official daily highs and lows, and weather-model forecasts for **24 major US
weather stations**. It is collected continuously and published here so anyone can use it: for research,
forecast verification, teaching, or simply looking back at what the weather did.

The archive is updated automatically about every 6 hours. Each update adds the complete UTC days recorded
since the last one. Files are append-only: rows are added, never removed.

## What's here

| Folder | What it is | Period | One file per |
|---|---|---|---|
| `daily_climate/` | Official NWS daily high and low (from the NWS climate reports, "CLI") | 2024-01-01 on | station |
| `forecasts/historical/` | Model forecasts of each day's high and low, 1–3 days ahead, reconstructed from archived model runs | 2024 on | station |
| `forecasts/recorded/` | Model forecasts of each day's high and low, as they were published, recorded live | 2026-10-14 on | UTC day |
| `forecasts/ensemble/` | Ensemble forecasts (every member's high and low), recorded live | 2026-10-14 on | UTC day |
| `observations/metar/` | Every METAR and SPECI report seen for the stations (full report text included) | 2026-10-17 on | UTC day |
| `observations/nws_api/` | Latest-observation temperatures from api.weather.gov, polled every few minutes | 2026-10-16 on | UTC day |
| `observations/madis_hfmetar/` | 5-minute ASOS temperatures from NOAA MADIS (high-frequency METAR) | 2026-10-19 on | UTC day |
| `stations.csv` | Station list: ICAO id, city, latitude, longitude, UTC offset (standard time) | | |

Day files live under `YYYY/MM/YYYY-MM-DD.csv`. All files are plain CSV (UTF-8, comma-separated, one header row)
and open in Excel, Google Sheets, Python (pandas), R and anything else that reads CSV.

## Stations

KATL Atlanta · KAUS Austin · KBOS Boston · KDCA Washington DC · KDEN Denver · KDFW Dallas/Fort Worth ·
KEWR Newark · KHOU Houston (Hobby) · KLAS Las Vegas · KLAX Los Angeles · KMDW Chicago (Midway) · KMIA Miami ·
KMSP Minneapolis · KMSY New Orleans · KNYC New York (Central Park) · KOKC Oklahoma City · KPHL Philadelphia ·
KPHX Phoenix · KSAN San Diego · KSAT San Antonio · KSDF Louisville · KSEA Seattle · KSFO San Francisco · KTTN Trenton

## Time and units

- **Timestamps** ending in `Z` are UTC (ISO 8601).
- **`date` columns** (the day a high or low belongs to) follow the NWS climate day: midnight to midnight
  **local standard time**, all year. During daylight saving time that is 1 AM to 1 AM on the clock.
  `utc_offset_standard_hours` in `stations.csv` gives each station's offset.
- **Units** are in the column name: `_c` is degrees Celsius, `_f` is degrees Fahrenheit.
- **Empty cells** mean the value wasn't reported.

## Columns

### daily_climate/STATION.csv
- `date`: the climate day.
- `high_f`, `low_f`: the official high and low (°F), from the final climate report issued after that day ended.
  Afternoon "so far today" reports are excluded.
- `high_time_local`, `low_time_local`: the time of the high and low as printed in the report.
- `nws_product_id`: the NWS product the values came from (issue time UTC, office, product code).

### forecasts/historical/STATION.csv
- `date`: the day being forecast.
- `model`: the Open-Meteo model name, for example `gfs_seamless`, `ecmwf_ifs025`, `icon_seamless`,
  `gem_seamless`, `jma_seamless`, `ukmo_seamless`, `ncep_nbm_conus`, `ncep_hrrr_conus`, `ecmwf_aifs025_single`
  or `best_match`.
- `lead_days`: how many days before `date` the model run was made (1 = the day before).
- `high_f`, `low_f`: the highest and lowest hourly 2 m temperature that run forecast within the climate day.
- `hours`: how many hourly values that day had. 24 is a full day; fewer means part of the day was missing.

These come from Open-Meteo's archive of previous model runs ("previous runs" API), not from live recording.

### forecasts/recorded/YYYY/MM/DAY.csv
- `recorded_utc`: when the forecast was downloaded (the file's day is this time's UTC date).
- `station`, `date`, `model`: the station, the day being forecast, and the model.
- `high_f`, `low_f`, `hours`: as in the historical files.

Forecasts are downloaded every few minutes, but a row is written only when a value differs from the previous
download for the same station, day and model. To get "the forecast as of time T", take the latest row with
`recorded_utc` at or before T.

### forecasts/ensemble/YYYY/MM/DAY.csv
- `model`, `model_id`: the ensemble, for example `gfs` / `gfs025` (31 members) or `ecmwf` / `ecmwf_ifs025` (51 members).
- `members`: how many members there are.
- `high_mean_f`, `high_sd_f`, `low_mean_f`, `low_sd_f`: the mean and standard deviation across members.
- `member_highs_f`, `member_lows_f`: every member's value, separated by semicolons.

As with recorded forecasts, a row is written only when the members change.

### observations/metar/YYYY/MM/DAY.csv
- `report_time_utc`: the observation time in the report. The file's day is this time's UTC date.
- `type`: `METAR` (routine) or `SPECI` (special). It is blank when the report was only seen on the NWS text server.
- `temp_c`, `dewpoint_c`: taken from the remarks T-group (0.1 °C) when present, otherwise from the
  whole-degree temperature/dew point group.
- `max6h_c`, `min6h_c`: the 6-hour maximum and minimum (remarks groups `1snnn` / `2snnn`) when the report has them.
- `first_received_utc`: when the report first reached our recorder.
- `sources`: `awc` is aviationweather.gov; `tg` is the NWS text server (tgftp.nws.noaa.gov).
- `raw`: the full report text. Wind, visibility, weather, clouds, pressure and every remark are in here.

Corrected reports (`COR`) are kept as their own rows.

### observations/nws_api/YYYY/MM/DAY.csv
- `obs_time_utc`: the observation time.
- `temp_c`: the temperature.
- `max24h_c`, `min24h_c`: the 24-hour max and min when reported.
- `first_received_utc`: when we first saw the observation.

### observations/madis_hfmetar/YYYY/MM/DAY.csv
- `obs_time_utc`: the observation time.
- `temp_c`, `temp_f`: the temperature.
- `qc_flag`: MADIS's quality-control flag (`V` verified, `S` screened, `Q` questioned, `X` failed, …).
- `madis_file`: the MADIS file the value came from.
- `first_seen_utc`: when we first saw the value.

## Caveats

- **Recorded live.** If a recorder was offline, that stretch is missing. Gaps are not filled in later.
- **Values are as published.** Nothing is corrected or quality-controlled beyond the sources' own flags.
- **Temperature focus.** The METAR `raw` text carries the full observation, but the other observation tables
  hold temperature only.
- **Not official.** The official record for any station is the NWS's own. This is a convenience copy.

## Sources and credits

- **NOAA / National Weather Service:** api.weather.gov, the NWS text server, and the daily climate reports
  (CLI). US government data, public domain.
- **aviationweather.gov** (NOAA Aviation Weather Center): METAR/SPECI. Public domain.
- **NOAA MADIS** (madis-data.ncep.noaa.gov, public feed): high-frequency METAR. Public domain.
- **Iowa Environmental Mesonet** (Iowa State University), which archives the NWS climate reports these daily
  values were read from.
- **Weather data by [Open-Meteo.com](https://open-meteo.com/)**, licensed under
  [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). This covers all forecast files. Open-Meteo combines
  forecasts from national weather services (NOAA, ECMWF, DWD, Environment Canada, JMA, UK Met Office and others).

## License

The compilation is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/): use it for
anything, including commercially, with credit to this archive. Forecast data must also credit Open-Meteo as
above. NOAA/NWS data is public domain. See `LICENSE`.
