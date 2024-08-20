---
theme: dashboard
title: WOFOST Simulation Results
toc: true
---

<!-- Load and transform the data -->
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
let output_remote = csv_output;  // For demo, use the CSV instead of backend

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

# WOFOST Simulation Results

<!-- Load the WOFOST 2023 data from the server -->
```js
// Load data from server at "localhost:3000/table/wofost_2023" using `fetch`
// const response_wofost_2023 = await fetch("http://localhost:3000/table/wofost_2023");
// let wofost_2023 = await response_wofost_2023.json();
let wofost_2023 = await FileAttachment("data/wofost_2023.csv").csv({typed: true}); // For demo, use the CSV instead of backend

// Reformat the data from the server
wofost_2023 = wofost_2023.map(d => {
  const date = d.day;
  // const time = d.time;

  // Format the date into ISO format
  const iso_date = new Date(date).toISOString().split('T')[0];

  // Format the date and time into a single datetime column
  // You can use the `new Date()` constructor to create a new Date object
  // by passing in a string in the format "YYYY-MM-DDTHH:MM:SS"
  const datetime = new Date(`${iso_date}T00:00:00`);

  return ({
    datetime: datetime, // Create a new Date object
    ...d, // Spread operator to keep all other columns
    // Optionally, you can remove the original date and time columns:
    // date: undefined,
    // time: undefined
  })
});
```

<!-- Load the WOFOST 2024 data from the server -->
```js
// const response_wofost_2024 = await fetch("http://localhost:3000/table/wofost_2024");
// let wofost_2024 = await response_wofost_2024.json();
let wofost_2024 = await FileAttachment("data/wofost_2024.csv").csv({typed: true}); // For demo, use the CSV instead of backend

// Reformat the data from the server
wofost_2024 = wofost_2024.map(d => {
  const date = d.day;
  // const time = d.time;

  // Format the date into ISO format
  const iso_date = new Date(date).toISOString().split('T')[0];

  // Format the date and time into a single datetime column
  // You can use the `new Date()` constructor to create a new Date object
  // by passing in a string in the format "YYYY-MM-DDTHH:MM:SS"
  const datetime = new Date(`${iso_date}T00:00:00`);

  return ({
    datetime: datetime, // Create a new Date object
    ...d, // Spread operator to keep all other columns
    // Optionally, you can remove the original date and time columns:
    // date: undefined,
    // time: undefined
  })
});
```

<!-- Timeline Showing data availability from simulations -->
```js
// Combine wofoost_2023 and wofoost_2024 with extra column for year
const wofost_combined = wofost_2023.map(d => ({...d, year: "2023"})).concat(wofost_2024.map(d => ({...d, year: "2024"})));
console.log(wofost_combined);
```

```js
Plot.plot({
  marks: [
    Plot.tickX(wofost_combined, {
      y: "year",
      x: "datetime",
      strokeWidth: 0.2,
      tip: true,
    })
  ],
  color: {
    legend: true,
    scheme: "warm"
  },
  width: Math.max(width, 1000),
  height: 200,
  marginLeft: 100,
  y: {
    type: "band", // Needed to squelch warning about y-axis values looking like numbers but actually being strings
    reverse: true
  },
  title: "Simulation Dates",
  caption: "Dates for which we have simulated results. Each y-axis category represents a growing season simulation."
})
```
---
## Compare LAI

<!-- Display Plots -->

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2023, { x: "datetime", y: "LAI", stroke: "SM", tip: true })
  ],
  x: {domain: [new Date("2023-01-01"), new Date("2023-12-31")], grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. Leaf Area Index (LAI) and Soil Moisture (SM) for 2023",
  caption: "Leaf Area Index (LAI) and Soil Moisture (SM) simulated over a growing season using the WOFOST simulator. The crop used is Pigeon Pea."
})
```

<!-- Display Plots -->

```js
Plot.plot({
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2024, { x: "datetime", y: "LAI", stroke: "SM", tip: true })
  ],
  x: {domain: [new Date("2024-01-01"), new Date("2024-12-31")], grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. Leaf Area Index (LAI) and Soil Moisture (SM) for 2024",
  caption: "Leaf Area Index (LAI) and Soil Moisture (SM) simulated over a growing season using the WOFOST simulator. The crop used is Pigeon Pea."
})
```

---
## Crop Transpiration

```js
Plot.plot({
  color: {
    legend: true,
    scheme: "Magma",
  },
  y: { label: "cm/day" },
  marks: [
    Plot.dot(wofost_2023, { x: "datetime", y: "TRA", stroke: "SM", tip: true })
  ],
  x: {domain: [new Date("2023-01-01"), new Date("2023-12-31")], grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. Crop Transpiration and Soil Moisture for 2023",
  caption: "Crop Transpiration excluding soil evaporation (TRA) and Soil Moisture (SM) simulated over a growing season using the WOFOST simulator. This indicates the amount of water that the plant obtains from the soil and subsequently transpires. The crop used is Pigeon Pea."
})
```

```js
Plot.plot({
  color: {
    legend: true,
    scheme: "Magma",
  },
  y: { label: "cm/day" },
  marks: [
    Plot.dot(wofost_2024, { x: "datetime", y: "TRA", stroke: "SM", tip: true })
  ],
  x: {domain: [new Date("2024-01-01"), new Date("2024-12-31")], grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. Crop Transpiration and Soil Moisture for 2024",
  caption: "Crop Transpiration excluding soil evaporation (TRA) and Soil Moisture (SM) simulated over a growing season using the WOFOST simulator. This indicates the amount of water that the plant obtains from the soil and subsequently transpires. The crop used is Pigeon Pea."
})
```

---
## Compare Soil Moisture

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2023, { x: "datetime", y: "SM", stroke: "LAI", tip: true }),
  ],
  x: {domain: [new Date("2023-01-01"), new Date("2023-12-31")], grid: true},
  y: {grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. Soil Moisture for 2023",
  caption: "Soil Moisture (SM) along with the Leaf Area Index (LAI) simulated over a growing season in 2023 using the WOFOST simulator. The crop used is Pigeon Pea."
})
```

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2024, { x: "datetime", y: "SM", stroke: "LAI", tip: true }),
  ],
  x: {domain: [new Date("2024-01-01"), new Date("2024-12-31")], grid: true},
  y: {grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. Soil Moisture for 2024",
  caption: "Soil Moisture (SM) along with the Leaf Area Index (LAI) simulated over a growing season in 2024 using the WOFOST simulator. The crop used is Pigeon Pea."
})
```

---
## Compare WWLOW

<!-- Display Plots -->

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2023, { x: "datetime", y: "WWLOW", stroke: "LAI", tip: true }),
  ],
  x: {domain: [new Date("2023-01-01"), new Date("2023-12-31")], grid: true},
  y: {grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. WWLOW and Leaf Area Index (LAI) for 2023",
  caption: "Amount of water available to the plant along with the Leaf Area Index simulated over a growing season in 2023 using the WOFOST simulator. The crop used is Pigeon Pea."
})
```

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2024, { x: "datetime", y: "WWLOW", stroke: "LAI", tip: true }),
  ],
  x: {domain: [new Date("2024-01-01"), new Date("2024-12-31")], grid: true},
  y: {grid: true},
  width: Math.max(width, 1200),
  title: "Date vs. WWLOW and Leaf Area Index (LAI) for 2024",
  caption: "Amount of water available to the plant along with the Leaf Area Index simulated over a growing season in 2024 using the WOFOST simulator. The crop used is Pigeon Pea."
})
```

<br>

### Soil Moisture vs. WWLOW

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2023, { x: "SM", y: "WWLOW", tip: true }),
  ],
  x: {grid: true},
  y: {grid: true},
  width: 600,
  height: 600,
  title: "SM vs. WWLOW for 2023",
})
```

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2024, { x: "SM", y: "WWLOW", tip: true }),
  ],
  x: {grid: true},
  y: {grid: true},
  width: 600,
  height: 600,
  title: "SM vs. WWLOW for 2024",
})
```

<!--
---
## Compare Yield

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2023, { x: "datetime", y: "TWSO", stroke: "LAI", tip: true }),
  ],
  x: {domain: [new Date("2023-01-01"), new Date("2023-12-31")], grid: true},
  y: {grid: true},
  title: "Date vs. Yield (TWSO) for 2023",
  caption: "The total weight in storage organs (i.e., the yield) in the plants simulated over a growing season in 2023 using the WOFOST simulator. The crop used is Pigeon Pea."
})
```

```js
Plot.plot({
  r: {range: [3, 15]}, // generate slightly smaller dots
  color: {
    legend: true,
    scheme: "Magma",
  },
  marks: [
    Plot.dot(wofost_2024, { x: "datetime", y: "TWSO", stroke: "LAI", tip: true }),
  ],
  x: {domain: [new Date("2024-01-01"), new Date("2024-12-31")], grid: true},
  y: {grid: true},
  title: "Date vs. Yield (TWSO) for 2024",
  caption: "The total weight in storage organs (i.e., the yield) in the plants simulated over a growing season in 2024 using the WOFOST simulator. The crop used is Pigeon Pea."
})
```
-->

# Tables
<br>
<h2>WOFOST 2023 Table</h2>
<div class="grid grid-cols-1">
  <div class="card">${resize((width) => (wofost_2023_table))}</div>
</div>

<!-- Display the data in a table -->
```js
// Show Table
const wofost_2023_table = Inputs.table(wofost_2023, {});

view(wofost_2023_table);
```

<br>
<h2>WOFOST 2024 Table</h2>
<div class="grid grid-cols-1">
  <div class="card">${resize((width) => (wofost_2024_table))}</div>
</div>

<!-- Display the data in a table -->
```js
// Show Table
const wofost_2024_table = Inputs.table(wofost_2024, {});

view(wofost_2023_table);
```
