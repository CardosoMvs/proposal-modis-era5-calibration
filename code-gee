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
 * contact: cardoso.mvs@gmail.com
 */

// Define ROI: Pantanal biome
var roi = ee.FeatureCollection('projects/mapbiomas-workspace/AUXILIAR/biomas_IBGE_250mil')
            .filter(ee.Filter.eq('Bioma', 'Pantanal'));

// Draw boundary
var empty = ee.Image().byte(); 
var contorno = empty.paint({
  featureCollection: roi, 
  color: 1,
  width: 2
});

Map.addLayer(contorno, {palette:["red"]}, 'Pantanal Boundary');
Map.centerObject(roi, 6);

// Filter quality: day
var filterDay = function(image) { 
  var qa = image.select('QC_Day');
  var bitMask2 = 1 << 2;
  var mask = qa.bitwiseAnd(bitMask2).eq(0);
  return image.updateMask(mask);
};

// Filter quality: night
var filterNight = function(image) { 
  var qa = image.select('QC_Night');
  var bitMask2 = 1 << 2;
  var mask = qa.bitwiseAnd(bitMask2).eq(0);
  return image.updateMask(mask);
};

// Load ERA5 daily air temperature for calibration
var era5_mean = ee.ImageCollection("ECMWF/ERA5/DAILY")
           .filterDate('2010-01-01', '2010-01-31')
           .select('mean_2m_air_temperature');

var era5_min = ee.ImageCollection("ECMWF/ERA5/DAILY")
           .filterDate('2010-01-01', '2010-01-31')
           .select('minimum_2m_air_temperature');

var era5_max = ee.ImageCollection("ECMWF/ERA5/DAILY")
           .filterDate('2010-01-01', '2010-01-31')
           .select('maximum_2m_air_temperature');

// Calibrate LST to air temperature
function calibrateLSTtoTAmean(image, alpha) {
  var era5SameDaymean = era5_mean.filter(ee.Filter.date(image.date()))
                      .first()
                      .subtract(273.15);
  
  return image.add(era5SameDaymean.subtract(image).multiply(alpha))
              .rename('air_temperature_mean')
              .copyProperties(image, ['system:time_start', 'system:time_end']);
}

function calibrateLSTtoTAmin(image, alpha) {
  var era5SameDaymin = era5_min.filter(ee.Filter.date(image.date()))
                      .first()
                      .subtract(273.15);
  
  return image.add(era5SameDaymin.subtract(image).multiply(alpha))
              .rename('air_temperature_min')
              .copyProperties(image, ['system:time_start', 'system:time_end']);
}

function calibrateLSTtoTAmax(image, alpha) {
  var era5SameDaymax = era5_max.filter(ee.Filter.date(image.date()))
                      .first()
                      .subtract(273.15);
  
  return image.add(era5SameDaymax.subtract(image).multiply(alpha))
              .rename('air_temperature_max')
              .copyProperties(image, ['system:time_start', 'system:time_end']);
}

// MODIS Terra: Daytime LST
var day_terra = ee.ImageCollection("MODIS/006/MOD11A1")
                  .filterDate('2010-01-01', '2010-01-31')
                  .select(['LST_Day_1km', 'QC_Day'])
                  .map(filterDay)
                  .map(function(image) {
                    return image.select('LST_Day_1km')
                               .multiply(0.02)
                               .subtract(273.15)
                               .rename('LST')
                               .copyProperties(image, ['system:time_start', 'system:time_end']);
                  });

// MODIS Terra: Nighttime LST
var night_terra = ee.ImageCollection("MODIS/006/MOD11A1")
                    .filterDate('2010-01-01', '2010-01-31')
                    .select(['LST_Night_1km', 'QC_Night'])
                    .map(filterNight)
                    .map(function(image) {
                      return image.select('LST_Night_1km')
                                 .multiply(0.02)
                                 .subtract(273.15)
                                 .rename('LST')
                                 .copyProperties(image, ['system:time_start', 'system:time_end']);
                    });

// Merge day and night collections
var combined = day_terra.merge(night_terra);

// Clip images to Pantanal region
var combined_pantanal = combined.map(function(image) {
  return image.clip(roi);
});

// Apply calibration (alpha = 0.6)
var alpha = 0.6;

var airTemp_pantanal_mean = combined_pantanal.map(function(image) {
  return calibrateLSTtoTAmean(image, alpha);
});

var airTemp_pantanal_min = combined_pantanal.map(function(image) {
  return calibrateLSTtoTAmin(image, alpha);
});

var airTemp_pantanal_max = combined_pantanal.map(function(image) {
  return calibrateLSTtoTAmax(image, alpha);
});

// Air temperature stats
var meanAirTemp = airTemp_pantanal_mean.mean();
var minAirTemp = airTemp_pantanal_min.min();
var maxAirTemp = airTemp_pantanal_max.max();

// Visualization
Map.addLayer(meanAirTemp, {
  palette: ['blue', 'green', 'yellow', 'red'],
  min: 25,
  max: 35
}, 'Mean Air Temperature (°C)');

Map.addLayer(maxAirTemp, {
  palette: ['blue', 'green', 'yellow', 'red'],
  min: 30,
  max: 40
}, 'Max Air Temperature (°C)');

Map.addLayer(minAirTemp, {
  palette: ['blue', 'green', 'yellow', 'red'],
  min: 15,
  max: 25
}, 'Min Air Temperature (°C)');

// Time series charts
var chartMean = ui.Chart.image.series({
  imageCollection: airTemp_pantanal_mean,
  region: roi,
  reducer: ee.Reducer.mean(),
  scale: 1000,
  xProperty: 'system:time_start'
}).setOptions({
  title: 'Daily Mean Air Temperature - Pantanal (Jan 2020)',
  vAxis: {title: 'Temperature (°C)'},
  hAxis: {title: 'Date', format: 'dd/MM'},
  lineWidth: 2,
  colors: ['red'],
  curveType: 'function'
});

// Temporal statistics
var meanAirTempImage = airTemp_pantanal_mean.mean();

var stats = meanAirTempImage.reduceRegion({
  reducer: ee.Reducer.mean().combine({
    reducer2: ee.Reducer.stdDev(),
    sharedInputs: true
  }).combine({
    reducer2: ee.Reducer.minMax(),
    sharedInputs: true
  }),
  geometry: roi,
  scale: 1000,
  maxPixels: 1e13
});
print('Air Temperature Statistics - Pantanal (Jan 2020):', stats);

// Time series as FeatureCollection
var timeStats = airTemp_pantanal_mean.map(function(image) {
  var stats = image.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: roi,
    scale: 1000,
    maxPixels: 1e13
  });
  return ee.Feature(null, stats).set('date', image.date());
});

var statsFC = ee.FeatureCollection(timeStats);
print('Temporal Statistics:', statsFC);

var timeChart = ui.Chart.feature.byFeature({
  features: statsFC,
  xProperty: 'date',
  yProperties: ['air_temperature_mean']
}).setOptions({
  title: 'Daily Mean Air Temperature Variation',
  vAxis: {title: 'Temperature (°C)'},
  hAxis: {title: 'Date'},
  lineWidth: 2,
  colors: ['red']
});
print(timeChart);

var chartMax = ui.Chart.image.series({
  imageCollection: airTemp_pantanal_max,
  region: roi,
  reducer: ee.Reducer.max(),
  scale: 1000,
  xProperty: 'system:time_start'
}).setOptions({
  title: 'Daily Max Air Temperature - Pantanal (Jan 2020)',
  vAxis: {title: 'Temperature (°C)'},
  hAxis: {title: 'Date', format: 'dd/MM'},
  lineWidth: 2,
  colors: ['darkred'],
  curveType: 'function'
});

var chartMin = ui.Chart.image.series({
  imageCollection: airTemp_pantanal_min,
  region: roi,
  reducer: ee.Reducer.min(),
  scale: 1000,
  xProperty: 'system:time_start'
}).setOptions({
  title: 'Daily Min Air Temperature - Pantanal (Jan 2020)',
  vAxis: {title: 'Temperature (°C)'},
  hAxis: {title: 'Date', format: 'dd/MM'},
  lineWidth: 2,
  colors: ['blue'],
  curveType: 'function'
});

// Display charts
print(chartMean);
print(chartMax);
print(chartMin);

// Export images to Google Drive
Export.image.toDrive({
  image: meanAirTemp.round(),
  description: 'MeanAirTemp_Pantanal_Jan2020',
  folder: 'GEE',
  region: roi.geometry(),
  scale: 1000,
  crs: 'EPSG:4326',
  maxPixels: 1e13
});

Export.image.toDrive({
  image: maxAirTemp.round(),
  description: 'MaxAirTemp_Pantanal_Jan2020',
  folder: 'GEE',
  region: roi.geometry(),
  scale: 1000,
  crs: 'EPSG:4326',
  maxPixels: 1e13
});

Export.image.toDrive({
  image: minAirTemp.round(),
  description: 'MinAirTemp_Pantanal_Jan2020',
  folder: 'GEE',
  region: roi.geometry(),
  scale: 1000,
  crs: 'EPSG:4326',
  maxPixels: 1e13
});
