



# Simple code for water detection  Using Google Earth Engine 

```javascript
/*******************************************************************************************
 *  Automatic Water detection and surface water estimation time series
 *  Modou MBAYE 
 *  my.gandhy@gmail.com 
 *  +221 779393741 
 * ****************************************************************************************/ 
 
 
var region=ee.Image().paint(RBDS,1,1)
Map.centerObject(RBDS,12)
Map.setOptions('satellite')
Map.addLayer(region,{}, 'dakar')



var date1='2022-11-01'
var date2='2022-11-30'


      
var S2=S2
          .filterDate(date1, date2)
          .filterBounds(RBDS)
          .filterMetadata('CLOUD_COVERAGE_ASSESSMENT','less_than',10)   
          
          
          
var waterThreshold = 0; 

print('check cloud granule over time : ',S2.aggregate_array('CLOUDY_PIXEL_PERCENTAGE'))

var waterfunction = function(image){
  var ndvi = image.clip(RBDS).normalizedDifference(['B8', 'B4']).rename('NDVI');
  var water01 = ndwi.lt(waterThreshold);
  image = image.updateMask(water01).addBands(ndvi);

  var area = ee.Image.pixelArea();
  var waterArea = water01.multiply(area).rename('waterArea');
  image = image.addBands(waterArea);

  var stats = waterArea.reduceRegion({
    reducer: ee.Reducer.sum(), 
    geometry: RBDS, 
    scale: 30,
  });

  return image.set(stats);
};
var collection = S2.map(waterfunction);


print(collection);


var chart = ui.Chart.image.series({
  imageCollection: collection.select('waterArea'), 
  region: RBDS, 
  reducer: ee.Reducer.sum(), 
  scale: 30,
});
print(chart);

var idx1 =collection.select('NDVI').min().lt(waterThreshold)
Map.addLayer(idx1.mask(idx1),{min:1,max:1,palette:'blue'},'mask')

```





![](/Users/modoumbaye/Dev/Earth-Observation-Lectures/Figures/RBDS.png)
