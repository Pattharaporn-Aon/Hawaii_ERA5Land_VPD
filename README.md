# Hawaii_ERA5Land_VPD

Hourly meteorological data for Hawaiʻi from 1950 to 2026, based on the ERA5-Land reanalysis, together with derived **Vapor Pressure Deficit (VPD)** and related variables.

## Time Zone

All timestamps are in **UTC**. Hawaiʻi Standard Time (HST) is UTC−10 and has no daylight saving time.

```
local_time (HST) = timestamp (UTC) − 10 hours
```

For example, `2025-01-01 23:00:00` in the files corresponds to 13:00 HST on 1 January 2025. Convert to HST before aggregating to daily values. Otherwise each "day" will run from 14:00 HST to 14:00 HST of the next day.

```python
import pandas as pd
df = pd.read_csv("Processed_Data/Processed_1950_2026_Data.csv", parse_dates=["timestamp"])
df["time_hst"] = df["timestamp"].dt.tz_localize("UTC").dt.tz_convert("Pacific/Honolulu")
```

## Downloading the Data

The CSV files are stored with **Git LFS**. A plain `git clone` without Git LFS installed downloads only small pointer files (about 130 bytes each), not the data.

Use one of the following options.

1. **Web browser.** Open a file on the GitHub page and click **Download raw file**.
2. **Git with LFS.**
   ```bash
   git lfs install
   git clone https://github.com/Pattharaporn-Aon/Hawaii_ERA5Land_VPD.git
   cd Hawaii_ERA5Land_VPD
   git lfs pull
   ```

A downloaded CSV should start with a header line beginning with `timestamp,`. A file beginning with `version https://git-lfs.github.com/spec/v1` is a pointer file and the real data has not been downloaded.

## Files

| File | Size | Rows | Description |
|------|------|------|-------------|
| `Raw_Data/All_Hourly_Data/1950_2026_Data.csv` | 104 MB | 673,259 | Raw hourly ERA5-Land variables, 1950-01-01 01:00 to 2026-10-22 16:00 UTC |
| `Raw_Data/Hourly_Data_By_Year/era5_hawaii_YYYY.csv` | about 1.3 MB each | about 8,760 each | The same raw data split into one file per year, 1950 to 2026 (77 files, same columns) |
| `Processed_Data/Processed_1950_2026_Data.csv` | 132 MB | 673,259 | Same period with unit conversions and derived variables (VPD, relative humidity) |

```
Hawaii_ERA5Land_VPD/
├── Raw_Data/
│   ├── All_Hourly_Data/          # 1950_2026_Data.csv
│   └── Hourly_Data_By_Year/      # era5_hawaii_1950.csv … era5_hawaii_2026.csv
├── Processed_Data/               # Processed_1950_2026_Data.csv
└── README.md
```

## Data Source

| Property | Value |
|----------|-------|
| Dataset | ERA5-Land Hourly (ECMWF) |
| Access | Google Earth Engine (`ECMWF/ERA5_LAND/HOURLY`) |
| Native pixel size | 11,132 m (about 0.1°) |
| Temporal resolution | Hourly |
| Period | 1950-01-01 01:00 to 2026-10-22 16:00 UTC |

Values are grid-cell averages from a reanalysis model, not direct station observations. They may differ from local point observations affected by terrain, vegetation and buildings.

## Raw Data Columns

Applies to `1950_2026_Data.csv` and every `era5_hawaii_YYYY.csv`.

| Column | Unit | Description |
|--------|------|-------------|
| `timestamp` | datetime (UTC) | Hourly timestamp (`YYYY-MM-DD HH:MM:SS`) |
| `temperature_2m` | K | Air temperature at 2 m above the surface |
| `dewpoint_temperature_2m` | K | Temperature to which the air at 2 m would have to be cooled for saturation to occur. A measure of air humidity |
| `surface_solar_radiation_downwards_hourly` | J/m² | Solar (shortwave) radiation reaching the surface, direct and diffuse, accumulated over the hour. Divide by 3,600 s to obtain W/m² |
| `surface_pressure` | Pa | Atmospheric pressure at the surface (1 hPa = 100 Pa) |
| `total_precipitation_hourly` | m | Total precipitation over the hour, expressed as water depth over the grid cell. Multiply by 1,000 to obtain mm |
| `u_component_of_wind_10m` | m/s | Eastward wind component at 10 m (positive = toward the east) |
| `v_component_of_wind_10m` | m/s | Northward wind component at 10 m (positive = toward the north) |
| `volumetric_soil_water_layer_1` | m³/m³ | Volumetric soil water content in layer 1 (0–7 cm) |

Wind speed is `sqrt(u² + v²)`.

## Processed Data Columns

Applies to `Processed_1950_2026_Data.csv`.

| Column | Unit | Description |
|--------|------|-------------|
| `timestamp` | datetime (UTC) | Hourly timestamp |
| `temperature_2m_c` | °C | `temperature_2m` − 273.15 |
| `dewpoint_temperature_2m_c` | °C | `dewpoint_temperature_2m` − 273.15 |
| `relative_humidity` | % | Relative humidity from T and Td, capped at 100 |
| `vpd_kpa` | kPa | Air vapor pressure deficit at 2 m (air VPD, not leaf-to-air VPD) |
| `light_intensity_wm2` | W/m² | `surface_solar_radiation_downwards_hourly` ÷ 3,600, negative values set to 0 |
| `surface_pressure` | Pa | Same values as the raw data |
| `total_precipitation_hourly` | m | Same values as the raw data |
| `u_component_of_wind_10m` | m/s | Same values as the raw data |
| `v_component_of_wind_10m` | m/s | Same values as the raw data |
| `volumetric_soil_water_layer_1` | m³/m³ | Same values as the raw data |

### Derived Variables

Saturation vapor pressure (kPa), Tetens equation, with T in °C:

```
es(T) = 0.6108 × exp(17.27 × T / (T + 237.3))
```

Relative humidity and VPD:

```
RH  = 100 × es(Td) / es(T)      [%]
VPD = es(T) − es(Td)            [kPa]
```

where `T` is `temperature_2m_c` and `Td` is `dewpoint_temperature_2m_c`.

The VPD in this dataset is the **air VPD** at 2 m above the surface. It is computed from air temperature and is not the leaf-to-air VPD. Leaf-to-air VPD uses leaf temperature in place of air temperature. Under strong sunlight, leaves can be warmer than the surrounding air, and the leaf-to-air VPD can then be higher than the values in this dataset.

VPD is computed for each hour before any aggregation. Daily or monthly VPD should be obtained by averaging the hourly `vpd_kpa` values, not by applying the formula to daily mean temperature and dew point.

## Data Summary (1950–2026)

| Variable | Mean | Min | Max |
|----------|------|-----|-----|
| Air temperature (°C) | 20.5 | 11.3 | 29.1 |
| Dew point (°C) | 17.7 | 6.0 | 25.3 |
| Relative humidity (%) | 84.6 | 36.3 | 100.0 |
| VPD (kPa) | 0.39 | 0.00 | 2.13 |
| Annual precipitation (mm) | 2,356 | 1,283 | 3,456 |

## Data Quality Notes

- No missing values (NaN) and no duplicated timestamps.
- The raw and processed files contain the same timestamps, row for row.
- The `vpd_kpa` column reproduces the formula above to floating-point precision.
- **Missing hours.** 29 hourly records between 2026-10-21 03:00 and 2026-10-22 15:00 UTC are absent. All other hours from 1950-01-01 01:00 onward are present.
- **One negative VPD value.** At 1994-03-02 14:00 UTC the dew point is slightly above the air temperature, which gives a VPD of about −0.000 kPa. Set this value to 0 before analysis, consistent with the relative humidity cap at 100%.
- ERA5-Land is released with a delay of a few days. The most recent records may be revised in later updates.
