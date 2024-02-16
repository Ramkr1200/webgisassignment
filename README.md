<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src='https://unpkg.com/@turf/turf@6/turf.min.js'></script>
  <script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet-ajax/2.1.0/leaflet.ajax.min.js"></script>
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css" />
  <title>Web Mapping Application</title>
  <style>
    body {
      padding: 20px;
    }
    #map {
      height: 400px;
    }
    .btn-container {
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1 class="mt-4 mb-4">Spatial analysis Application</h1>
    <div id="map"></div>
    <div class="btn-container">
      <button class="btn btn-primary" onclick="togglePopupLayer()">Toggle Popup Layer</button>
      <button class="btn btn-success" onclick="applyBuffer()">Apply Buffer</button>
      <button class="btn btn-info" onclick="applyClip()">Apply Clip</button>
    </div>
  </div>

  <script>
    var map = L.map('map').setView([0, 0], 2);
    var geojsonLayer1, geojsonLayer2;

    // Base Layers from GeoServer

    var District_Layer  = L.tileLayer.wms('http://localhost:8080/geoserver/spatial_Analysis/wms?', {
      layers: 'spatial_Analysis:district_dataset',
      format: 'image/png',
      transparent: true
    });

    var Provinence_Layer  = L.tileLayer.wms('http://localhost:8080/geoserver/spatial_Analysis/wms?', {
      layers: 'spatial_Analysis:province_dataset',
      format: 'image/png',
      transparent: true
    });

    var Road_Layer  = L.tileLayer.wms('http://localhost:8080/geoserver/spatial_Analysis/wms?', {
      layers: 'spatial_Analysis:road_network',
      format: 'image/png',
      transparent: true
    });

    var Substation_Layer  = L.tileLayer.wms('http://localhost:8080/geoserver/spatial_Analysis/wms?', {
      layers: 'spatial_Analysis:substation',
      format: 'image/png',
      transparent: true
      
    });

    var Transmision_Grid_Layer  = L.tileLayer.wms('http://localhost:8080/geoserver/spatial_Analysis/wms?', {
      layers: 'spatial_Analysis:transmission_grid',
      format: 'image/png',
      transparent: true
    });

  
    // Additional Base Layers (non-GeoServer)
var osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
});
osm.addTo(map);
    var singleMarker = L.marker([28.25255,83.97669]);
singleMarker.addTo(map);
var popup = singleMarker.bindPopup('This is a popup')

// Google Map Layer

googleStreets = L.tileLayer('https://{s}.google.com/vt/lyrs=m&x={x}&y={y}&z={z}',{
    maxZoom: 20,
    subdomains:['mt0','mt1','mt2','mt3']
 });
 googleStreets.addTo(map);

 // Satelite Layer
googleSat = L.tileLayer('https://{s}.google.com/vt/lyrs=s&x={x}&y={y}&z={z}',{
   maxZoom: 20,
   subdomains:['mt0','mt1','mt2','mt3']
 });
googleSat.addTo(map);


    // Add base layers to the map
    var baseLayers = {
      'OpenStreetMap': osm,
      'Satellite':googleSat,
      'Google Map':googleStreets,
       
    };

    var overlays = {


'Provinence_Layer': Provinence_Layer,
'District_Layer': District_Layer,
'Road_Layer': Road_Layer, 
'Substation_Layer' : Substation_Layer,
'Transmision_Grid_Layer' : Transmision_Grid_Layer,
'Marker': singleMarker,    
    };

L.control.layers(baseLayers,overlays).addTo(map);

//SEARCH BUTTON   
L.Control.geocoder().addTo(map);

function togglePopupLayer() {
      if (!geojsonLayer1 && !geojsonLayer2) {
        geojsonLayer1 = L.geoJson.ajax('http://localhost:8080/geoserver/spatial_Analysis/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=spatial_Analysis%3Atransmission_grid&maxFeatures=50&outputFormat=application%2Fjson', {
          onEachFeature: function (feature, layer) {
            layer.bindPopup(feature.properties.name);
          }
        }).addTo(map);

        geojsonLayer2 = L.geoJson.ajax('http://localhost:8080/geoserver/spatial_Analysis/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=spatial_Analysis%3Asubstation&maxFeatures=50&outputFormat=application%2Fjson', {
          onEachFeature: function (feature, layer) {
            layer.bindPopup(feature.properties.name);
          }
        }).addTo(map);
      } else {
        map.removeLayer(geojsonLayer1);
        map.removeLayer(geojsonLayer2);
        geojsonLayer1 = null;
        geojsonLayer2 = null;
      }
    }

    function applyBuffer() {
      if (geojsonLayer2) {
        geojsonLayer2.on('data:loaded', function () {
          geojsonLayer2.eachLayer(function (layer) {
            var buffered = turf.buffer(layer.toGeoJSON(), 500, { units: 'meters' });
            L.geoJson(buffered).addTo(map);
          });
        });
      } else {
        console.error('GeoJSON Layer 2 is not defined or has no data.');
      }
    }

    function applyClip() {
      if (geojsonLayer1) {
        geojsonLayer1.on('data:loaded', function () {
          geojsonLayer1.eachLayer(function (layer) {
            var bbox = [-83.2, -26.23, 80.3, 27.31]; // Example bounding box
            var clipped = turf.bboxClip(layer.toGeoJSON(), bbox);
            L.geoJson(clipped).addTo(map);
          });
        });
      } else {
        console.error('GeoJSON Layer 1 is not defined or has no data.');
      }
    }
  </script>
</body>
</html>
