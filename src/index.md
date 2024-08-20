---
theme: air
toc: true
---

<!-- Get current weather data from the NWS -->
```js
const longitude = -120.410440;
const latitude = 37.355171;

async function json(url) {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`fetch failed: ${response.status}`);
  return await response.json();
}

const station = await json(`https://api.weather.gov/points/${latitude.toFixed(2)},${longitude.toFixed(2)}`);
const forecast = await json(station.properties.forecastHourly);

// display(forecast);
```

<!-- Load and transform the data from the remote and from a local CSV (for testing) -->
```js
// Load output Data
// var output = await FileAttachment("data/toby_ml_data.csv").csv({typed: true, array: true});

// // Reformat output Data from CSV to JS Object
// const header_rows = output.slice(1,2);
// const data_rows = output.slice(3, output.length);
// // output = data_rows.map((row) => Object.fromEntries(row.map((d, i) => [header_rows[0][i], d])));

// Load Toby's ML Data
var csv_output = await FileAttachment("data/toby_ml_data.csv").csv({typed: true});
console.log("CSV Row:");
console.log(csv_output[0]);

// Load data from server at "localhost:3000/table/wofost_2023" using `fetch`
// const response = await fetch("http://localhost:3000/table/toby_ml_data");
// let output_remote = await response.json();
let output_remote = csv_output;  // For demo, use CSV data instead of remote data

// Reformat the data from the server for proper dates
output_remote = output_remote.map(d => {
  d.datetime = new Date(d.datetime);
  return ({
    ...d, // Spread operator to keep all other columns
    // Optionally, you can remove the original date and time columns:
    // date: undefined,
    // time: undefined
  })
});
console.log("Remote Row:");
console.log(output_remote[0]);

let output = output_remote;

// Sort the data
output = output.sort((a, b) => new Date(a.datetime) - new Date(b.datetime));

// Filter the data to date range
/*output = output.filter(d => {
  return d["datetime"] >= new Date("2020-01-01T00:00:00Z") && d["datetime"] <= new Date("2024-11-30T23:59:59Z");
});*/

// Set a "simulated" data cutoff threshold
const simulated_data_threshold = new Date("2023-11-05T00:00:00Z");

// Filter out data after the simulated data threshold in a separate column
output = output.map(d => {
  if (new Date(d.datetime) > simulated_data_threshold) {
    return ({
      ...d,
      "Sim Actual ET": null,
    })
  } else {
    return ({
      ...d,
      "Sim Actual ET": d["Actual ET"],
    })
  }
});
```

# Forecast Current Conditions
Forecast for ${new Date(forecast.properties.periods[0].startTime).toLocaleTimeString()}–${new Date(forecast.properties.periods[0].endTime).toLocaleTimeString()} at the UCM Smart Farm.
<div class="grid grid-cols-4">
  <div class="card">
    <h2>Forecast Temperature 🌡️ <span class="muted">via NWS</span></h2>
    <span class="big">${forecast.properties.periods[0].temperature} °F</span>
  </div>
  <div class="card">
    <h2>Relative Humidity 💧 <span class="muted">via NWS</span></h2>
    <span class="big">${forecast.properties.periods[0].relativeHumidity.value}%</span>
  </div>
  <div class="card">
    <h2>Wind Speed 🍃 <span class="muted">via NWS</span></h2>
    <span class="big">${forecast.properties.periods[0].windSpeed}</span>
  </div>
  <div class="card">
    <h2>Wind Direction 🧭 <span class="muted">via NWS</span></h2>
    <span class="big">${forecast.properties.periods[0].windDirection}</span>
  </div>
</div>

# Smart Farm Data

```js
Plot.plot({
  marks: [
    Plot.tickX(output, {
      // y: "air_temperature",
      x: "datetime",
      // stroke: "air_temperature",
      strokeWidth: 0.2,
      tip: true
    })
  ],
  color: {
    legend: true,
    scheme: "warm"
  },
  width: Math.max(width, 1200),
  height: 200,
  marginLeft: 10,
  marginBottom: 50,
  x: { reverse: false },
  title: "Data Availability Dates",
  caption: "Dates for which we have data from the smart farm."
})
```

<div class="muted">Taking the analysis model data and implementing it into a more readable format. Below are graphs that are serveral different forms of data displayed!</div>

<div class="grid grid-cols-2">
  <div class="card">${resize((width) => h2oLine(width))}</div>
  <div class="card">${resize((width) => ho2Scatter(width))}</div>
  <div class="card">${resize((width) => airLine(width))}</div>
  <div class="card">${resize((width) => lolipopH2oChart(width))}</div>
</div>

# ML Model Predictions
<div class="muted">The following show how our ML model's predictions compare to actual ET values.</div>
<div class="grid grid-cols-2">
<div class="card">${resize((width) => PEScatter(width))}</div>
  <div class="card">${resize((width) => AcPETScatter(width))}</div>
</div>

<div class="grid grid-cols-1">
<div class="card">${resize((width) => overlayETChart(width))}</div>
<div class="card">${resize((width) => dotET(width))}</div>
</div>

```js
function h2oLine(width){
  return Plot.plot({
    title: "Changes in H2O data",
    caption: "Visual of H2O flux data changing daily in a line plot.",
    grid: true,
    x: { label: "Date Time" },
    y: { label: "H2O Flux" },
    style: {
      fontSize: "17px",
    },
    marks: [
      Plot.lineY(output, {x: "datetime", y: "h2o_flux", strokeWidth: 2, stroke: "#4b5a9d", tip: true})
    ],
    marginBottom: 70,
  })
}
```

```js
function airLine(width){
  return Plot.plot({
    title: "Date Time vs. Air Temperature",
    caption: "Demonstrates that the air temperature stayed constant daily because the sensor is broken.",
    grid: true,
    x: { label: "Date Time" },
    y: { label: "Air Temperature (K)" },
    style: {
      fontSize: "17px",
    },
    marginBottom: 70,
    marks: [
        Plot.lineY(output, {x: "datetime", y: "air_temperature", stroke: "#a4bf8a", strokeWidth: 5, tip: true})
    ]
  })
}
```

```js
function ho2Scatter(width){
  return Plot.plot({
    title: "Changes in H2O Data",
    caption: "Visual of H2O flux data changing daily in a scatter plot.",
    grid: true,
    style: {
      fontSize: "17px",
    },
    marginBottom: 70,
    marks: [
      Plot.dot(output, {x: "datetime", y: "h2o_flux", stroke: "#2ca25f", strokeWidth: 1, tip: true})
    ]
  })
}
```

```js
function overlayETChart(width){
  return Plot.plot({
    width: 1000,
    height: width * 0.6,
    title: "Date Time vs. H2O Flux compared with Predicted ET",
    label: 150,
    caption: "Demonstration of H2O Flux and Predicited ET over time",
    x: { label: "Date Time" },
    y: { label: "Flux Value" },
    symbol: {legend: true},
    style: {
      fontSize: "17px",
    },
    grid: true,
    marginBottom: 70,
    marks: [
      Plot.ruleY([0]),
      Plot.lineY(output, {x: "datetime", y: "h2o_flux", stroke: "#e3856b", strokeWidth: 3, tip: true}),
      Plot.lineY(output, {x: "datetime", y: "Predicted ET", stroke: "#bababa",strokeWidth: 3, tip: true})
    ]
  })
}
```

```js
function lolipopH2oChart(width){
  return Plot.plot({
    x: { label: "CO2 Flux", tickPadding: 6, tickSize: 0 },
    y: { label: "H2O Flux", percent: true },
    title: "CO2 vs H2O flux",
    caption: "Comparison of CO2 and H2O in a lollipop chart.",
    style: {
      fontSize: "17px",
    },
    grid: true,
    marginLeft: 70,
    marginBottom: 70,
    marks: [
      Plot.ruleX(output, {x: "co2_flux", y: "h2o_flux", strokeWidth: 2}),
      Plot.dot(output, {x: "co2_flux", y: "h2o_flux", r: 2, strokeWidth: 3, tip: true}),
    ]
  })
}
```

```js
function AcPETScatter(width){
  return Plot.plot({
    color: {
      legend: true,
      scheme: "Viridis"
    },
    title: "Actual ET vs Predicted ET",
    caption: "Figure 4.2 Comparsion of Actual and Predicted during the course of data collection. Take note that the color of the diamond represents the Delta",
    marginBottom: 70,
    grid: true,
    style: {
      fontSize: "17px",
    },
    marks: [
      Plot.dot(output, {x: "Actual ET", y: "Predicted ET", stroke: "Delta", r: 14, strokeWidth: 3, tip: true, symbol: "diamond2"})
    ]
  })
}
```

```js
// Filter out problematic percent error values
const perc_err_output = output.map((row) => {
  return ({...row, "Filtered Percent Error": row["Actual ET"] < 0.02 && row["Actual ET"] > -0.02 ? null : row["Percent Error"]})
});
```

```js
function PEScatter(width){
  return Plot.plot({
    color: {
      legend: true,
      scheme: "Inferno"
    },
    title: "Actual ET vs Percent Error",
    caption: "Figure 4.1 The amount of Actual ET and the Percent Error from the model. Take note that the color of the dot represents the Delta. Also note that all values with ETs of close to zero have been filtered out.",
    marginLeft: 80,
    marginBottom: 70,
    grid: true,
    style: {
      fontSize: "17px",
    },
    marks: [
      Plot.dot(perc_err_output, {x: "Actual ET", y: "Filtered Percent Error", r: "Delta", stroke: "Delta", strokeWidth: 3, r: 7, tip: true})
    ]
  })
}
```

<!-- ```js
function dotET(width){
    return Plot.plot({
        width: 1000,
        height: width * 0.6,
        title: "Actual, Predicted ET vs date & time",
        caption: "Figure 5.2 Demonstration of Actual ET and Predicited ET over time shown in a scatter plot. The line across the graph is the average of both actual and predicted ET.",
        grid: true,
        marginBottom: 70,
        style: {
            fontSize: "20px",
            },
        marks: [
            Plot.ruleY([0]),
            Plot.dot(output, {x: "datetime", y: "Actual ET", stroke: "#5ab4ac", strokeWidth: 5, tip: true}),
            Plot.dot(output, {x: "datetime", y: "Predicted ET", stroke: "#6F256F", strokeWidth: 4, tip: true}),
            Plot.line(output, Plot.windowY(300, {x: "datetime", y: "Actual ET", stroke: "#d8b365", strokeWidth: 10})),
            Plot.line(output, Plot.windowY(300, {x: "datetime", y: "Predicted ET", stroke: "#7fbf7b", strokeWidth: 5, tip: true})),
            ]
        })
    }
``` -->
```js
function dotET(width){
  return Plot.plot({
    width: 1000,
    height: width * 0.6,
    title: "Date & Time vs. Actual, Predicted ET",
    caption: "Figure 5.2 Demonstration of Actual ET and Predicted ET over time shown in a scatter plot. The line across the graph is the average of both actual and predicted ET.",
    grid: true,
    style: {
        fontSize: "1.2em",
    },
    marginTop: 40,
    marginBottom: 80,
    // marginRight: 10,
    marks: [
      Plot.ruleY([0]),
      Plot.ruleX([simulated_data_threshold], {stroke: "#000000", strokeWidth: 2}),
      Plot.tip(
        [`Actual ET Data Cutoff`],
        {x: simulated_data_threshold, y: 0.8, dy: -3, anchor: "right"}
      ),
      Plot.dot(output, {x: "datetime", y: "Sim Actual ET", stroke: "#5ab4ac", strokeWidth: 4, symbol: "circle", tip: true}),
      Plot.dot(output, {x: "datetime", y: "Predicted ET", stroke: "#6F256F", strokeWidth: 1, symbol: "diamond2"}),
      Plot.line(output, Plot.windowY(300, {x: "datetime", y: "Sim Actual ET", stroke: "#d8b365", strokeWidth: 10})),
      Plot.line(output, Plot.windowY(300, {x: "datetime", y: "Predicted ET", stroke: "#7fbf7b", strokeWidth: 5, tip: true})),
      // Plot.text(output, Plot.selectFirst({x: "datetime", y: "Sim Actual ET", text: ["FOO BAR"], fillOpacity: 0.5, lineAnchor: "bottom", dy: -6})),
      // Plot.text(output, Plot.selectFirst({x: "datetime", y: "Predicted ET", text: ["BAZ BIN"], lineAnchor: "top", dy: 6})),
      // Place dot at (2024-12-01, 1.0)
      Plot.dot(output, {dx: new Date("2023-12-01"), dy: 1.0, stroke: "#5ab4ac", strokeWidth: 4, symbol: "circle", tip: true}),
      // Plot.dot([], {x: new Date("2023-12-01"), y: 1.0, stroke: "#5ab4ac", r: 4, symbol: "circle", tip: false}),

      // custLegend({frameAnchor: "top-right"}),
      // legendSpike([0.5, 1.0, 1.5], {frameAnchor: "top-right", stroke: "red"}),
      custom_legend([
        {text: "Actual ET", type: "circle", color: "#5ab4ac", length: 0.5},
        {text: "Predicted ET", type: "diamond2", color: "#6F256F", length: 1.0},
        {text: "Act ET (SMA)", type: "line", color: "#d8b365", length: 1.5},
        {text: "Pred ET (SMA)", type: "line", color: "#7fbf7b", length: 1.5}
      ], {frameAnchor: "top-right", stroke: "red"}),
    ]
  })
}
```

```js
function custom_legend(values, {frameAnchor = "bottom-right", format = "~s", stroke} = {}) {
  if (!Array.isArray(values)) values = Array.from(values);

  if (typeof format !== "function") format = d3.format(format);

  const end_date = new Date("2024-01-01");
  const start_date = new Date(end_date) - 50 * (24 * 60 * 60 * 1000); // Start date is 50 days before end date

  return Plot.marks([
    values.map((entry, i) => {
      const dy = -100;
      const dx = -50;
      const y_val = 1.0;
      const distance = 30;
      const x_val = end_date - ((i * distance) * 86400000);

      let marker = null;
      if (entry.type === "line") {
        marker = Plot.line([
          [x_val - 3 * 86400000, y_val],
          [x_val + 3 * 86400000, y_val]
        ], {
          dx: dx,
          dy: dy,
          stroke: entry.color,
          strokeWidth: 3
        });
      } else {
        marker = Plot.dot([0], {x: x_val, y: y_val, dx: dx, dy: dy, frameAnchor, stroke: entry.color, r: 5, symbol: entry.type, tip: false});
      }

      return [
        marker,

        // Text
        Plot.text([entry.text], {
          // text: [format(length)],
          text: [entry.text],
          fontSize: "0.8em",
          x: x_val,
          y: y_val,
          dy: dy + 18,
          dx: dx,
          frameAnchor,
          textAnchor: "middle"
        }),
      ];
    })
  ]);
}
```

# Smart Farm Map

```js
import * as L from "npm:leaflet";
// import { defaultName } from "npm:proj4";
import parseGeoraster from "npm:georaster";
import GeoRasterLayer, { GeoRaster } from "npm:georaster-layer-for-leaflet@3.5.0";

// Load Leaflet plugins
import MiniMap from 'npm:leaflet-minimap';
```

```js
const div = display(document.createElement("div"));
div.style = "height: 600px;";

// Add base map(s)
const minimap_tiles = L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
});

const tileLayer = L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
});

const cycl_tiles = L.tileLayer("https://{s}.tile-cyclosm.openstreetmap.fr/cyclosm/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
});

const open_topo_tiles = L.tileLayer("https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://opentopomap.org/credits">OpenTopoMap</a>'
});

const esri_tiles = L.tileLayer("http://services.arcgisonline.com/ArcGIS/rest/services/Elevation/World_Hillshade/MapServer/tile/{z}/{y}/{x}", {
  attribution: '&copy; <a href="https://www.esri.com/en-us/home">ESRI</a>'
});

// DEBUG: Add a circle
const circle = L.circle([37.355867, -120.408328], {
  color: 'red',
  fillColor: '#f03',
  fillOpacity: 0.5,
  radius: 50
}).bindPopup("Near the smart farm");

// Add SE2 marker
const se2_marker = L.marker([37.36590127985323, -120.42155263885883])
  .bindPopup("UCM SE2")
  .openPopup();

// Test GeoJSON
const geojsonFeature = {
    "type": "Feature",
    "properties": {
        "name": "UCM ESF",
        "amenity": "Farm",
        "popupContent": "This is where the experiments are conducted!"
    },
    "geometry": {
        "type": "Point",
        "coordinates": [-120.408328, 37.355867]
    }
};

const feat = L.geoJSON(geojsonFeature, {
  style: {
    "color": "#2fcc70",
    "weight": 5,
    "opacity": 0.65
  }
});

// Create the map
const map = L.map(div, {
    layers: [tileLayer, circle, se2_marker, feat],
    center: [37.355867, -120.408328],
    zoom: 14
  });

// Add minimap
new MiniMap(minimap_tiles, {
  zoomLevelFixed: 5,
  autoToggleDisplay: true
}).addTo(map);

// Add layers control
const layerControl = L.control.layers({
  "Tiles": tileLayer,
  "Cyclosm": cycl_tiles,
  "OpenTopoMap": open_topo_tiles,
  "ESRI": esri_tiles
}, {
  "Circle": circle,
  "SE2": se2_marker,
  "ESF": feat
}, { collapsed: false }).addTo(map);

// Data structure to keep track of layers
let layers = [];
```

```js
// Button to load a new GeoTIFF
const go_home = view(Inputs.button("Go Home"));
const load_test_button = view(Inputs.button("Load Test Raster"));
```

```js
go_home; // run this block when the button is clicked

// Go back to the home location
map.setView([37.355867, -120.408328], 14);
```

```js
load_test_button; // run this block when the button is clicked
const load_test_raster = (function* () {

  // Only load the raster if the button has been pressed, not on page load
  if (load_test_button > 0) {
    console.log("Will load a test raster (button pressed)");

    // Version that gets GeoTIFFs over the network
    var url_to_geotiff_file = "https://geotiff.github.io/georaster-layer-for-leaflet-example/example_4326.tif";
    fetch(url_to_geotiff_file)
      .then(response => response.arrayBuffer())
      .then(arrayBuffer => {
        parseGeoraster(arrayBuffer).then(georaster => {
          console.log("georaster:", georaster);

          var layer = new GeoRasterLayer({
              georaster: georaster,
              opacity: 0.7
          });
          layer.addTo(map);

          // Add to the layers data structure
          layers.push(layer);

          map.fitBounds(layer.getBounds());
      });
    });
  }
})();
```

# Tables

<div class="muted">ESF Data ranges to air temperature, co2 flux, h2o flux,
predicted ET, actual ET, Percent Error, and date & time that the data was presented!</div>

<br>
<h2>ML Data Table</h2>
<div class="grid grid-cols-1">
  <div class="card">${resize((width) => (table))}</div>
</div>

<!-- Display the data in a table -->
```js
// Show Table
const table = Inputs.table(output, {});

view(table);
```
