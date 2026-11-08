# ArmaFlyMap — V2 (Altis)

Helicopter flight planner for Arma 3. Place the **base**, the **landing zone (LZ)** and the **anti-air batteries**. The app computes a route that stays hidden behind the terrain.

## Getting started
Double-click `index.html` (Chrome, Edge or Firefox). Keep the `assets/` folder next to it.

## How to use it
The left column has two tabs: **AO SET UP** (terrain, base & landing zone, air defence, flight settings) and **SQUADS PATH** (squad insertion, flight plan, exports and missions). The selected tab is remembered.

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
1. Place the **AO** (AO mode). It is the objective. Enemies can be anywhere inside it, **but never higher than the ground at the AO marker**: AO positions on higher ground are ignored. AA pits can also fire if the option is checked.
2. Set the LZ distance ring around the AO centre (min / max), the enemy effective fire range, the max LZ slope and the squad posture (prone 0.5 m, crouched 1 m, standing 1.7 m).
   **Building cover** (0 = off, 10 = max): buildings detected on the topographic HD map make walking along houses cheaper and reduce how much exposure counts there (walls give cover), so the paths are drawn towards built-up areas. Each card shows the metres walked along buildings. Tick *Detected buildings* in the map panel (SQUAD mode) to see what was detected. Buildings also **block enemy fire** where the line of fire passes less than 2 m above the ground (always on, whatever the slider): shooters higher up (towers, summits) can still fire over them. Along each path, **HC** squares mark **hard cover** spots (buildings right next to the path, at most one every 100 m), and each card gives their number.
3. Click **Find LZs & covered paths**. The app:
   - computes where the squad can be shot from (purple zone), with the terrain as cover;
   - looks for landing zones in the ring that are flat, dry and out of small-arms fire, and flags those where air defence could see a helicopter landing;
   - finds the most covered foot path from an LZ to an assault position on the edge of the AO (slopes slow the squad down, water and cliffs are impassable);
   - then blocks that path and a corridor of about 100 m on each side, and searches again: **up to 5 routes (A to E) that never cross**, A being the best. Each has its own LZ (at least 500 m apart) and its own assault position. A dotted line marks the final assault to the AO centre.
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
- `assets/buildings.js`: building map at 12.5 m (1 bit per cell), detected once from the grey squares of the topographic HD map (used by *Building cover*, fire occlusion below 2 m and hard-cover spots).
- `v1/`: frozen copy of V1.
- `assets/dem.js`: the app's only terrain dataset. HD Altis terrain exported from Arma 3 (`getTerrainHeightASL`), 1228 × 1228 points every 25 m (exported with `afm_export_relief_25m.sqf`), accurate to 0.1 m, on the Arma frame (0 to 30720 m, origin bottom-left).
- `assets/hd/`: **Topographic HD** and **Aerial HD** base maps extracted from the PLANOPS Atlas (zoom 5, about 2.5 m per pixel), as 4 × 4 mosaics of 3048 px per layer. They work offline. When you're online and zoom in further, the app also loads the finer tiles (topo up to ~1.3 m/px, aerial up to ~0.6 m/px). Alignment checked against the HD terrain (99% land/sea match). Content © Bohemia Interactive: for personal use only, don't redistribute.
- `assets/satellite.webp`, `assets/carte.webp`: the original layer2 and layer1, still available.
- `afm_export_relief.sqf`: the export script that produced the terrain dataset.
- `afm_export_relief_25m.sqf`: 25 m terrain export in two parts (set `_part = 1`, then `2`; save as `relief25_part1.txt` / `relief25_part2.txt`).

## Working resolution

The squad planner runs on its own 12.5 m grid in a square around the AO (LZ max distance + 300 m on each side): terrain interpolated from the 25 m HD terrain, buildings from the 12.5 m map, line of sight blocked by terrain and by buildings below 2 m. The helicopter planner works directly on the 25 m terrain grid.

## V1 limitations
- Vegetation and buildings don't hide the helicopter. Only the terrain counts.
- AA ranges are approximate: adjust them to your mission.
- Arma 3's low-altitude radar ground clutter is not modeled, so the model is conservative against radars.

## V4 / V5 additions

- **Terrain at 25 m** (1228 × 1228) for the helicopter; squad grid at 12.5 m around the AO with a 12.5 m building map.
- **Number of infantry paths**: slider, 1 to 5 non-crossing routes.
- **AO shooter height (towers, rooftops)**: 2 to 10 m (default 10 m) above the ground for AO shooters in the squad sight calculation. The altitude cap still applies to the ground they stand on.
- **Placed heli LZ → infantry path**: when the helicopter LZ is placed by hand, a white covered foot path is computed from it straight to the objective (or the AO edge) — "HELI LZ (placed)" card. It ignores the LZ distance/slope rules and the other routes.
- **Detected buildings** are drawn as black hatched footprints.
- **Calculation mode** (Terrain section, also in Squad insertion): Fast = heli 100 m / squad 50 m, Standard (default) = heli 50 m / squad 25 m, Precise = heli 25 m / squad 12.5 m. Coarser grids are averaged from the 25 m terrain.
- **Objective marker** (★ Obj on the map toolbar, one only, drag to move, right-click to delete): every infantry path ends exactly there, including the white path from a hand-placed heli LZ. The paths stay apart until they reach the AO (or the last 300 m around the objective), then converge. Without an objective, paths stop at the AO edge as before. LZ distances are still measured from the AO centre (from the objective if there is no AO).
- **Squad terrain detail**: the squad planner always reads the full 25 m HD terrain (whatever the calculation mode), its grid is aligned on the terrain samples, and the last 40 m of each line of sight is checked every 4 m, so ditches, banks and sunken roads hide the squad when they are deep enough in the data. Narrow ditches caught as chains of single low terrain samples are joined and carved into the squad grid so the channel stays continuous.
- **Ditch cover** slider (0–10, default 5): prefers paths in ditches and low ground (cells below the median ground 25 m around them), like *Building cover* does for houses.
- **Squad insertion switch** (top of the Squad insertion section): OFF stops all squad calculations and hides LZs, paths, fire zones and the objective; the SQUAD / ALL reading modes are disabled. The setting is remembered.
- **LZ SAFE**: when "Fly heli here", the automatic best LZ, or an LZ placed by hand points to a spot that air defence can see, the helicopter LZ is moved to the closest safe landing spot (outside the red air-defence areas at flight height and at least *LZ SAFE margin* away from them — slider, default 200 m — flat, dry, no building within 25 m, outside the AO). The map shows "LZ SAFE" with a dashed line to the deferred LZ, and the squad gets its own covered path from LZ SAFE to the objective ("LZ SAFE · for LZ x" card).
- **Base maps**: Topographic HD, Aerial HD and Shaded relief (the satellite and paper maps were removed; `assets/satellite.webp` and `assets/carte.webp` are no longer used).
- **Contour lines** layer (display options): isolines computed from the 25 m terrain, every 5 / 10 / 20 / 50 m or *auto* (interval follows the zoom). Every 5th line is a thicker index contour with its altitude when zoomed in; colours switch to light lines on the aerial map.
