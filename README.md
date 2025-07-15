# Flood Area Analysis Tool

A Python tool for calculating flood areas by administrative units (villages/districts) with customizable depth classifications. This tool processes flood depth rasters and village/district shapefiles to generate detailed flood impact statistics.

## Features

- **Dynamic Flood Classification**: Classify flood depths into customizable categories
- **Multiple Area Units**: Calculate areas in square meters, hectares, square kilometers, or rai (Thai unit)
- **Village-level Analysis**: Generate flood statistics for each administrative unit
- **Flexible Thresholds**: Define custom depth thresholds for flood severity classification
- **Spatial Analysis**: Raster-vector overlay analysis with proper CRS handling

## Requirements

### Dependencies

```bash
pip install rasterio geopandas numpy pandas
```

### Required Libraries

```python
import rasterio
import geopandas as gpd
import numpy as np
import pandas as pd
from rasterio.mask import mask
```

## Data Requirements

### Input Data

1. **Flood Depth Raster**: 
   - Format: GeoTIFF (.tif)
   - Contains flood depth values in meters
   - Projected coordinate system (UTM recommended)

2. **Administrative Boundaries Shapefile**:
   - Format: Shapefile (.shp)
   - Contains village/district polygons
   - Should have ID and name fields
   - Same or compatible coordinate system as raster

### Expected Shapefile Fields

The code expects these field names (adjust in code if different):
- `ADM3_EN`: Administrative unit ID (English)
- `ADM3_TH`: Administrative unit name (Thai/local language)

## Usage

### Basic Usage

```python
from flood_analysis import calculate_flood_area_by_village, classify_flood_depth

# Using default thresholds [0.5, 1.0, 2.0, 3.0] meters
results_df = calculate_flood_area_by_village(
    flood_raster_path='flood_depth.tif',
    villages_shapefile_path='admin_boundaries.shp'
)

# Save results
results_df.to_csv('flood_analysis_results.csv', index=False)
```

### Custom Thresholds

```python
# Define custom depth thresholds (in meters)
custom_thresholds = [0.1, 0.25, 0.5, 0.75, 1.0, 1.5, 2.0]

results_df = calculate_flood_area_by_village(
    flood_raster_path='flood_depth.tif',
    villages_shapefile_path='admin_boundaries.shp',
    thresholds=custom_thresholds
)
```

### Different Area Units

```python
# Calculate areas in rai (Thai unit: 1 rai = 1,600 m²)
results_rai = calculate_flood_area_by_village(
    flood_raster_path='flood_depth.tif',
    villages_shapefile_path='admin_boundaries.shp',
    thresholds=[0.1, 0.5, 1.0, 2.0],
    unit='rai'
)

# Available units: 'm2', 'km2', 'hectare', 'rai'
```

### Multiple Units Analysis

```python
# Generate results in all units
units = ['m2', 'rai', 'hectare', 'km2']
thresholds = [0.1, 0.25, 0.5, 0.75, 1.0, 1.5, 2.0]

for unit in units:
    results = calculate_flood_area_by_village(
        'flood_depth.tif', 
        'admin_boundaries.shp', 
        thresholds, 
        unit
    )
    results.to_csv(f'flood_analysis_{unit}.csv', index=False)
```

## Function Parameters

### `calculate_flood_area_by_village()`

**Parameters:**
- `flood_raster_path` (str): Path to flood depth raster file
- `villages_shapefile_path` (str): Path to administrative boundaries shapefile
- `thresholds` (list, optional): List of depth thresholds in meters. Default: [0.5, 1.0, 2.0, 3.0]
- `unit` (str, optional): Output area unit. Options: 'm2', 'km2', 'hectare', 'rai'. Default: 'm2'

**Returns:**
- pandas.DataFrame with flood statistics for each administrative unit

### `classify_flood_depth()`

**Parameters:**
- `depth_array` (numpy.array): Array of flood depth values
- `thresholds` (list, optional): List of depth thresholds in meters

**Returns:**
- numpy.array with classified flood categories (0 = no flood, 1+ = flood categories)

## Output Format

The output DataFrame contains the following columns:

| Column | Description |
|--------|-------------|
| `village_id` | Administrative unit ID |
| `village_name` | Administrative unit name |
| `total_area` | Total area of the administrative unit |
| `no_flood_area` | Area with no flooding |
| `0-{threshold} m` | Area with flood depth 0 to first threshold |
| `{prev}-{curr}m` | Area with flood depth between thresholds |
| `>{last_threshold} m` | Area with flood depth above last threshold |
| `total_flooded_area` | Total area affected by flooding |

*Note: All area columns include unit suffix (e.g., "(rai)", "(m²)", etc.)*

## Example Output

```
village_id  village_name     total_area (rai)  no_flood_area (rai)  0-0.1 m (rai)  ...
1          Village A         1000.5            850.2                45.3           ...
2          Village B         1250.8            1100.1               38.7           ...
```

## Troubleshooting

### Common Issues

1. **CRS Mismatch Error**: 
   ```
   ValueError: Input shapes do not overlap raster
   ```
   **Solution**: Ensure both raster and shapefile are in the same coordinate system

2. **Field Name Errors**:
   **Solution**: Check shapefile field names and update in code:
   ```python
   # Update these lines in the code
   'village_id': village.get('YOUR_ID_FIELD', idx),
   'village_name': village.get('YOUR_NAME_FIELD', f'Village_{idx}'),
   ```

3. **Memory Issues with Large Files**:
   **Solution**: Process subsets of villages or use chunked processing

### Checking Data Compatibility

```python
import rasterio
import geopandas as gpd

# Check raster properties
with rasterio.open('flood_depth.tif') as src:
    print(f"Raster CRS: {src.crs}")
    print(f"Raster bounds: {src.bounds}")

# Check shapefile properties
villages = gpd.read_file('admin_boundaries.shp')
print(f"Shapefile CRS: {villages.crs}")
print(f"Shapefile fields: {villages.columns.tolist()}")
print(f"Shapefile bounds: {villages.total_bounds}")
```

## Performance Tips

1. **Use projected coordinate systems** (UTM) for accurate area calculations
2. **Ensure data overlap** before processing
3. **For large datasets**, consider processing in batches
4. **Optimize raster resolution** for your analysis needs
