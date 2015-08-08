# Intro to Web Mapping
August 8, 2015 at the WorkPlace

## Housekeeping
- Introductions
- A bit about Maptime
- MapLift

## please install:
- Chrome or Firefox
- Any text editor; you could use Notepad/TextEdit, but it'll be nicer to use:
  - [Atom](https://atom.io/)
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Sublime Text](http://www.sublimetext.com/)
  - [Brackets](http://brackets.io/)

## What to map?
For this workshop, we'll need data in the [GeoJSON format](http://www.macwright.org/2015/03/23/geojson-second-bite.html), which is the preferred format for sending and receiving geospatial data on the web.

 `/data/parks.geojson` is the data file I'll be using today. These are polygons of Detroit parks with some extra attributes.

I found this data on Detroit's [open data portal](https://data.detroitmi.gov/); it came as a shapefile, which I converted to a GeoJSON file using `ogr2ogr`: a command-line 'swiss army knife' for converting geospatial data between different formats.

You can find/create suitable GeoJSON data a few ways:
- Make your own and edit existing data using [geojson.io](http://geojson.io)
- Convert a shapefile using [ogr2ogr web client](http://ogre.adc4gis.com/) (watch for projections!)
- Use [overpass turbo](http://overpass-turbo.eu/) to extract features from OpenStreetMap
  - Explore OSM tags for use in the wizard: [OSM Taginfo](https://taginfo.openstreetmap.org/)

## Creating a web page
Edit `index.html` so it looks like this:

```html
<!DOCTYPE html>
<html>

    <head>
      <title>
        Maptime Detroit
      </title>
    </head>

    <body>
        Hello world!
    </body>

</html>
```

## Serve up the web page
OS X/Ubuntu/other Linux:
- open Terminal.app
- `cd` to the project directory
- `python -m SimpleHTTPServer 8080`

Windows:
- Open a command prompt (cmd)
- `chdir` to the project directory
- Try: `python -m SimpleHTTPServer 8080`
- else: `python -m http.server 8080`
- still not working? download [Mongoose](https://code.google.com/p/mongoose/) and run that in the project directory

## Load Leaflet & create a map on the page
- Load Leaflet: copy these two lines into your `<head>` tag:

```html
<link rel="stylesheet" href="http://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.3/leaflet.css" />

<script src="http://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.3/leaflet.js"></script>
```

- Add a `<div>` tag with the id `map` to the **body** of your webpage:

```html
<div id="map"> </div>
```

- Add a styling rule to `<div id="map">` in the **head** of your page:

```html
<style>
    #map {
        height: 400px;
    }
</style>
```

## Add a base map: loading a TileLayer
- Add this code to your main `<script>`:

```html
L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}', {
    attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, <a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="http://mapbox.com">Mapbox</a>',
    maxZoom: 18,
    id: 'mapbox.streets',
    accessToken: 'pk.eyJ1Ijoiam1jYnJvb20iLCJhIjoianRuR3B1NCJ9.cePohSx5Od4SJhMVjFuCQA'
}).addTo(map);
```

- Let's also change the coordinates to Detroit:
  - `42.331, -83.045`

## Bring GeoJSON data into the map
- Add JQuery to the **head** of your webpage:

```html
<script src="http://code.jquery.com/jquery-2.1.0.min.js"></script>
```

- Get the GeoJSON data and display it on the map:

```html
$.getJSON("../data/parks.geojson", function(data) {
  var parks = L.geoJson(data);
  parks.addTo(map);
});
```

## Style the GeoJSON
- Create an object with some styling options:

```html
var parkStyle = {
    "color": "#304D1C",
    "weight": 1.5,
    "opacity": 0.75
};
```

- Add this style as a option to your GeoJSON layer:

```html
$.getJSON("../data/parks.geojson", function(data) {

  var parks = L.geoJson(data, {
      style: parkStyle
  });

  parks.addTo(map);

});
```

More about GeoJSON layers and their options in the [Leaflet docs](http://leafletjs.com/reference.html#geojson).

## Add a pop-up to your GeoJSON data
- Add a function to your main `<script>`:

```html
function onEachPark(feature, layer){
    layer.bindPopup(feature.properties.NAME);
}
```

- Reference this function in the `onEachFeature` option for your GeoJSON layer:

```html
$.getJSON("../data/parks.geojson", function(data) {
  var parks = L.geoJson(data, {
      style: parkStyle,
      onEachFeature: onEachPark
  });
  parks.addTo(map);
});
```

## Filter your GeoJSON data
:warning: There are some 'parks' in the data that are placeholders for future parks! :warning:

Let's filter those out with an inline function:

```html
$.getJSON("../data/parks.geojson", function(data) {
    var parks = L.geoJson(data, {
        style: parkStyle,
        onEachFeature: onEachPark,

        filter: function(feature) {
            if (feature.properties.NAME.startsWith("Future")) {
                return false;
            } else return true;
        }

    });
    parks.addTo(map);
});
```

## Host your map on the internet:
- [GitHub](https://pages.github.com/)
- [Neocities](https://neocities.org/)
