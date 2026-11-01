# ArmaFlyMap — V2 (Altis)

Helicopter flight planner for Arma 3. Place the **base**, the **landing zone (LZ)** and the **anti-air batteries**. The app computes a route that stays hidden behind the terrain.

## Getting started
Double-click `index.html` (Chrome, Edge or Firefox). Keep the `assets/` folder next to it.

## How to use it
1. **Base / LZ**: click *Base* or *LZ*, then click on the map. You can also type a grid (`036130`, `03601300` or `0360013000`). Markers can be dragged.
2. **AA**: pick a type, then click *+ AA* and click on the map (or type a grid). Right-click an AA to delete it. Range and sensor height can be edited.
3. **Settings**: height above ground, speed, caution, valley floor preference, climb penalty, **avoidance margin**. The avoidance margin makes the route keep its distance from batteries and from the edges of exposed areas, so it skirts threats tangentially (0 = off).
4. **AO** (Air defense section): choose *AO* mode, click to place the AO centre, drag to set its radius (or edit it in the list). Right-click the centre marker to delete it. Two modes switch automatically:
   - **AA pits placed**: threats come only from your AA pits. The AO is shown for reference.
   - **No AA pits**: worst case. AA is assumed anywhere in the AO at full range, including its high ground (local summits are marked with small red triangles). The route avoids everything that could be seen from there.
5. **Threat layers** (map panel):
   - **Danger heatmap**: continuous danger level. Yellow = near danger, orange = close, red = seen by an AA, magenta = seen by several.
   - **Route tolerance corridor**: around the route, how far you can drift left and right and stay hidden (up to 600 m). The colour blends continuously between path points, from red (exposed) through orange and yellow to green (450 m+ margin), and fades from the path out to the corridor edge. The same colours appear as a strip under the flight profile, and hovering the profile shows the exact left/right margins.
6. The plan updates automatically. Red areas = where the helicopter would be seen at the chosen height. Red parts of the route = unavoidable exposure.
7. **Exports**: a text briefing, SQF code that draws the route on the in-game map, and SQF code that makes an AI helicopter (named `heli1`) fly the route.

## Squad insertion (V2)
Plan where to drop a squad and how it reaches the objective under cover.
1. Place the **AO** (AO mode). It is the objective. Enemies can be anywhere inside it, including its high ground. AA pits can also fire if the option is checked.
2. Set the LZ distance ring around the AO centre (min / max), the enemy effective fire range, the max LZ slope and the squad posture (prone 0.5 m, crouched 1 m, standing 1.7 m).
3. Click **Find LZs & covered paths**. The app:
   - computes where the squad can be shot from the AO (purple zone), with the terrain as cover;
   - looks for landing zones in the ring that are flat, dry and out of small-arms fire, and flags those where air defence could see a helicopter landing;
   - finds the most covered foot path from each LZ to an assault position on the edge of the AO (slopes slow the squad down, water and cliffs are impassable);
   - shows the **best LZ (A)** and **3 alternatives (B, C, D)** from the other directions around the AO (one per 90° sector), each with its own assault position. A dotted line marks the final assault to the AO centre.
4. Each card gives the walk distance, estimated time, exposed metres (red on the map) and climb. **Fly heli here** sets that LZ as the helicopter destination, so the flight plan is computed to it.
The results update automatically when you move the AO, the AA pits or change a setting.

## Map reading modes
The selector at the top of the map switches what is drawn, so the map stays readable:
- **✈ HELI**: air danger heatmap, AA range circles, helicopter route and its tolerance corridor.
- **⛶ SQUAD**: enemy small-arms fire zone (purple), squad LZs A–D, covered foot paths, assault positions. Air layers are hidden.
- **ALL**: everything together.
The legend, the layer checkboxes and the hover info follow the selected mode. The app switches automatically: SQUAD after *Find LZs & covered paths*, HELI after *Compute flight plan*, ALL after *Fly heli here*.

## How it works
- **Visibility**: for each 60 m cell within an AA's range, the app traces a ray from the sensor to the helicopter (ground + flight height). If the terrain cuts the ray, the cell is hidden.
- **Route**: A* search on the grid. The cost combines distance, a very heavy penalty in exposed areas, height above the surroundings (ridges are penalized) and climbing. The path is then smoothed into straight legs without adding exposure.

## Data
- `v1/`: frozen copy of V1.
- `assets/dem.js`: the app's only terrain dataset. HD Altis terrain exported from Arma 3 (`getTerrainHeightASL`), 614 × 614 points every 50 m, accurate to 0.1 m, on the Arma frame (0 to 30720 m, origin bottom-left).
- `assets/hd/`: **Topographic HD** and **Aerial HD** base maps extracted from the PLANOPS Atlas (zoom 5, about 2.5 m per pixel), as 4 × 4 mosaics of 3048 px per layer. They work offline. When you're online and zoom in further, the app also loads the finer tiles (topo up to ~1.3 m/px, aerial up to ~0.6 m/px). Alignment checked against the HD terrain (99% land/sea match). Content © Bohemia Interactive: for personal use only, don't redistribute.
- `assets/satellite.webp`, `assets/carte.webp`: the original layer2 and layer1, still available.
- `afm_export_relief.sqf`: the export script that produced the terrain dataset.

## V1 limitations
- Vegetation and buildings don't hide the helicopter. Only the terrain counts.
- AA ranges are approximate: adjust them to your mission.
- Arma 3's low-altitude radar ground clutter is not modeled, so the model is conservative against radars.
