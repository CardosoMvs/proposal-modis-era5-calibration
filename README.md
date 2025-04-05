# 🌡️ LST(modis)-to-Air-Temperature Calibration(era5)

## 📝 Description  
This Google Earth Engine script processes MODIS Land Surface Temperature (LST) data to estimate air temperature for Brazil's Pantanal biome, using ERA5 reanalysis data for calibration. The tool:

- Processes both day and night MODIS LST data (Terra satellite)
- Applies quality filters to remove low-quality pixels
- Calibrates LST to air temperature using ERA5 data
- Generates mean, minimum and maximum temperature estimates
- Produces visualizations and statistical outputs

## 🛠️ Script Header

```javascript
/**
 * MODIS LST to Air Temperature Calibration Tool
 * Converts MODIS Land Surface Temperature to air temperature estimates
 * Uses ERA5 reanalysis data for calibration (2010-01-01 to 2010-01-31)
 * Processes data for Brazil's Pantanal biome
 * 
 * Inputs: 
 *   - MODIS/006/MOD11A1 (LST data)
 *   - ECMWF/ERA5/DAILY (calibration data)
 * Outputs: 
 *   - Calibrated air temperature (mean, min, max)
 *   - Time series charts
 *   - Statistical summaries
 *   - GeoTIFF exports
 * 
 * Version: 1.0
 */
```

## 🌟 Key Features
- **Dual-sensor processing**: Combines Terra day/night LST data
- **Quality control**: Implements MODIS QC filters
- **Multi-temperature outputs**: Generates mean, min and max estimates
- **Visual analytics**: Creates interactive charts and maps
- **Validation metrics**: Computes regional statistics

## 🧩 Dependencies
- Google Earth Engine account
- MODIS LST data access
- ERA5 reanalysis data access

## 📊 Outputs
1. Map visualizations of:
   - Mean air temperature
   - Maximum air temperature  
   - Minimum air temperature
2. Time series charts of daily variations
3. Statistical summaries (mean, stdDev, min/max)
4. Exported GeoTIFF files

## 🚀 Usage
1. Copy script into Google Earth Engine Code Editor
2. Run script (may require asset permissions)
3. View outputs in:
   - Console (stats and charts)
   - Map panel (visualizations)
   - Google Drive (exported files)

## 📄 Sample Visualization Code
```javascript
// Example visualization for mean temperature
Map.addLayer(meanAirTemp, {
  palette: ['blue', 'green', 'yellow', 'red'],
  min: 25,
  max: 35
}, 'Temperatura Média do Ar (°C)');
```

## 🤝 Contributing
Suggestions welcome for:
- Additional calibration methods
- Expanded time periods
- Other biome applications
- Improved visualization options
