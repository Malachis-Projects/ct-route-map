# Connecticut Route Map

An interactive route optimization tool for the state of Connecticut. Add addresses, let it work out the best order to visit them, and see what the trip will cost in time and gas given traffic, weather and terrain.

It is a single web page with no build step, no server and no API keys.

## What it does

### Map
- Live street map of Connecticut with four base layers: Streets, Light, Terrain and Satellite.
- All 169 Connecticut towns outlined. Hover a town for its name and area; click to section it off (highlight it). Any number of towns can be selected at once.
- Searchable town list in the side panel. Clicking a town in the list selects it and zooms to it.

### Addresses and stops
- **Address lookup with type-ahead.** Suggestions appear as you type and are limited to Connecticut. Arrow keys and Enter work.
- **Paste a list.** One address per line; every match is added as a stop, and anything not found is left in the box to fix.
- **Drop a stop on the map** by switching the click mode.
- Each stop is tagged with the town it falls in.
- Each stop has a checkbox, so it can be left out of a route without deleting it.
- Stops are remembered in the browser between visits.

### Route optimization
- Stop 1 is the starting point. The remaining stops are reordered to minimise either **drive time** or **distance (least gas)**.
- Round trip (return to stop 1) or one way.
- "As listed" draws the route in the order entered, for comparison.
- The route is drawn along real roads, and the panel reports what reordering saved in minutes, miles and dollars.
- **Stick to main roads** favours highways and penalises local and service streets.

### Trip planning
- **Leaving time.** Pick a date and time; every stop shows its arrival time, allowing for a set number of minutes at each stop.
- **Traffic.** Drive time is adjusted for typical congestion by time of day and day of week.
- **Weather.** The hourly forecast at each stop for the time you would arrive. Snow, heavy rain, fog and strong gusts raise a warning and add to drive time.
- **Terrain.** Elevation is sampled along the route to estimate total climbing.
- **Gas.** Enter the car's MPG and the gas price to get gallons and cost. Congestion and climbing both add to the estimate.
- **Best time to leave.** A bar chart shows total driving time for every half hour from 5 AM to 9 PM. Click a bar to switch to that time.

## How the optimization works

1. A road drive-time and distance matrix between all stops is fetched from OSRM.
2. A first tour is built with the **nearest-neighbour** heuristic, starting from stop 1.
3. The tour is improved with **2-opt** (reverse a segment) and **or-opt** (move one stop) until neither finds a better tour.
4. The final order is sent to the router for the road geometry and per-leg times.

This is a heuristic for the travelling salesman problem. It gives a very good order quickly, but not a guaranteed optimum.

The trip is then simulated leg by leg from the leaving time, in 10-minute slices, so a long leg that crosses into rush hour or a rain band is slowed for only that part.

Fuel is `miles / MPG`, scaled up for stop-and-go driving, plus the energy to lift the car up the total climb (`m·g·h` at about 25% engine efficiency).

## Limits

- **Traffic is not live.** It is a typical rush-hour curve, not a real-time feed. Live traffic needs a paid key from a provider such as Google or TomTom.
- **Climbing is approximate.** Elevation is sampled at up to 300 points along the route.
- **The stop order is solved on clear-road times.** Traffic and weather change the arrival times and totals, not the order.
- **"Best time to leave" only minimises driving time**, so it can suggest a late-evening departure.
- **Weather covers about 10 days ahead.** Beyond that it is not counted.
- The free routing and lookup services are meant for light use and have no uptime guarantee.

## Run it

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8765
```

Then go to `http://localhost:8765`.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole app: layout, styles and logic |
| `data/ct-towns.js` | Boundaries of the 169 Connecticut towns |

## Data and services

| Used for | Source |
| --- | --- |
| Town boundaries | US Census Bureau TIGERweb (county subdivisions) |
| Street, light and terrain maps | OpenStreetMap, CARTO, OpenTopoMap |
| Satellite imagery | Esri World Imagery |
| Map display | Leaflet |
| Drive-time matrix and routes | OSRM public server |
| Main-roads routing | Valhalla (FOSSGIS public server) |
| Address lookup | Photon (Komoot), on OpenStreetMap data |
| Weather forecast and elevation | Open-Meteo |
