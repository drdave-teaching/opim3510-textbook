# Chapter 4 — Geospatial Data & Mapping

Now for something that will genuinely differentiate you in the job market. Most business students can wrangle spreadsheets but **can't make a map** — and when your data has a spatial component, that's a real gap. This chapter is a crash course in **spatial analytics with Python**: the data types (vector vs. raster), **making maps** from shapefiles with **GeoPandas**, **geocoding** addresses to points, the **spatial join** (enrich points with a polygon's attributes — no key needed), joining CSVs to shapefiles, pulling **census data** by zip code, and **resampling** high-resolution data to a coarser one. You won't be a GIS expert in two weeks, but you'll ask the right questions and build maps that tell a story.

## 4.1 Geospatial data types

Two kinds of spatial data:
- **Vector** — **points** (airport/Uber-pickup locations), **lines** (roads, town boundaries, power lines), and **polygons** (town/zip/census areas you can aggregate over). We use vector data throughout.
- **Raster** — continuous **gridded** data like satellite imagery (elevation, wildfire, crop health via red/green band math). We won't use it much, but know it exists.

You locate things with **latitude** (horizontal belts; positive North) and **longitude** (pole-to-pole; negative West of England) — Hartford is ~41.7°N, −72.7°W. Because Earth is a sphere (an oblate spheroid) squashed onto a flat map, the **projection / coordinate reference system (CRS)** matters: Mercator distorts sizes (compare Africa across projections), and projected units are **feet or meters** — essential for **distance** calculations, since a degree of lat/lon isn't a constant distance (bigger at the equator, smaller at the poles). A shapefile is basically a **CSV with a `geometry` column** — its **attribute table** has one row per map feature (CT has 169 towns but ~800 polygons because of coastal **islands**). Spatial operations include **clip** (constrain to a study area), **buffer** (a zone around a feature), and the **spatial join** (drop a point on a polygon and inherit its attributes).

## 4.2 Making maps with polygon shapefiles

Mind **spatial resolution** — high (neighborhood) vs. low (state); they can feel backwards. Get free CT/US data from CT DEEP's GIS site (states, zip codes, town boundaries). A **shapefile** is many files together (`.shp`, `.dbf`, `.prj`, `.shx`, …) — all needed. Read it with **GeoPandas**: `import geopandas as gpd; gdf = gpd.read_file("states.shp")` — like pandas but **`read_file`** instead of `read_csv`, and a **`geometry`** column tells the computer how to draw each feature. The US states file has **56** rows (50 + DC + territories) × 15 columns. Then **`.plot()`** draws the map (axes are lon/lat), and **`column="land_area"`** or **`column="region", legend=True`** colors it by an attribute (customize with `cmap`, `legend_kwds`). A `for` loop over polygon centroids can label each state — a real map that would take hours in raw Matplotlib.

## 4.3 Point data, lat/lon & geocoding

Not all data is polygons — sometimes it's **points**. Take CT **house-sales** data (145k rows: address, town, assessed value, sale amount, sales ratio, property type). With only an address you could parse out a town/zip for a join — but a **spatial join** is better: **no formal primary key needed, because the lat/lon *is* the key**. First **geocode**: turn an address string into a lat/lon point. Build a full address by pasting `address + " " + town + " CT"`, then loop each row through a geocoder (**GeoPy**, or an alternative) to get coordinates. Geocoding is slow, so it's pre-canned here — read the resulting `lat_long_real_estate` file and you have coordinates for every house (so you can map them).

## 4.4 Mapping the house sales

Even before making it a geo-object, **`folium`** renders an interactive HTML map inline (centered on CT's centroid). The geocoder isn't perfect (a few CT sales land in Canada — you could add logic: if the address says CT but the point is outside CT, drop it). Convert the DataFrame to a **GeoDataFrame** with a CRS — **EPSG 4326** (WGS84, the GPS standard) for point/lat-lon data. For an intro class, the key is just getting the **longitude/latitude columns named and ordered correctly**. Then read the **zip-code shapefile** (ZCTA, the whole US), and subset **Connecticut** by keeping ZCTA IDs that **start with "06"** (`.str.startswith("06")`) → 283 zip polygons (higher resolution than 169 towns).

## 4.5 The spatial join

The payoff. Your house points are CRS **4326** (WGS84); the CT zip polygons are **4269** (NAD83) — two different projections. **You cannot do a spatial operation across different CRSs**, so convert one: `df = df.to_crs(4269)`. Then **`gpd.sjoin(points, polygons, how="inner", predicate="intersects")`** (note: `op` is deprecated → `predicate`) drops each point onto the polygon it falls in and **attaches all the polygon's attributes** — instantly enriching every house with its zip code, land/water area, etc. Overlay for a great map: plot the polygons first, capture the `ax`, then plot the points on top of that same axis, and color the points by sale amount (use `log` to tame outliers) or property type (cast the column to **string** so the legend renders as categories).

## 4.6 Joining CSVs to shapefiles (CT maps)

Shapefiles carry limited attributes, so — just like enriching data in Chapter 3 — **join a CSV to a shapefile** on a primary key, then map it. Read the **CT towns** shapefile (1,780 polygons; axes in the **hundreds of thousands** because it's a **projected** CRS in **feet**, not lat/lon). Subset CT land (`state == "CT"` and not water). Then `pd.merge` a "children by family type" CSV onto the towns on the town-name key, and **convert the result back to a GeoDataFrame** via its `geometry` column. Subset to one year/family-type → 783 rows (169 towns + ~500 coastal islands; Branford alone has 154 tiny islands). Map a column (e.g. married couples with children under 18) — optionally normalized by population. The pattern is a **nested join**: `pd.merge` (left/right keys) → `gpd.GeoDataFrame(..., geometry=...)`. Keep track of left/right keys and it's not scary.

## 4.7 Enriching house sales with census attributes

The scale play: if you can attach a **zip code** to any point data, you unlock **hundreds of census variables** per zip. Grab the "census summaries by zip code" files (economics ~225, housing ~255, social ~252, rural/urban ~6 columns; ~33,000 zips = all US zip codes). Fix the classic bug: zip codes stored as **integers lose their leading zero**, so **`df["zip"].astype(str).str.zfill(5)`** pads them back. Join the four datasets on the zip key (a `functools.reduce`/lambda one-liner) → a **735-column master table** (225+255+252+6 − 3 duplicate keys). Join *that* to the geocoded house points on the zip → house prices now carry **768 columns** of census context (employment, marriage, kids, school quality) — "solid gold" for explaining *why* a house sold for its price. Finish with **AutoEDA** (`sweetviz`) to auto-profile everything against sale amount.

## 4.8 Resampling zip codes to town resolution

Finally, go the other way — **resample** high-resolution data to a coarser unit. Load the zip-code and town shapefiles (again, `to_crs` so both share one CRS — non-negotiable). Join an economics CSV to the zips (`.str.zfill(5)`), map poverty rate nationwide (pockets in Appalachia, the South), then CT (Hartford, Bridgeport, Waterbury, eastern CT). To push zip data down to **towns**: **`gpd.sjoin`** towns↔zips on `intersects` (→ 1,921 town-zip fragments, since boundaries split), compute an **area-weighted** value (`land_area × poverty_rate`), **aggregate by town** (a weighted average), join back, and map the **weighted poverty rate per town**. That's the essence of resampling between spatial scales — area-weighted, a few lines of code.

**Crash course complete:** vector data (points/lines/polygons), reading shapefiles and making colored maps, geocoding addresses to points, spatial joins to enrich points from polygons, joining CSVs and census data, and resampling across resolutions. You can now make your own maps and say something interesting about spatial patterns in Connecticut.

:::{admonition} 🔗 Notebooks for this chapter
:class: seealso dropdown

- **Intro to geospatial data types** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/2_SamplingDistributions_CI/2_Intro%20to%20Geospatial%20Data%20Types.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/2_SamplingDistributions_CI/2_Intro%20to%20Geospatial%20Data%20Types.ipynb)
- **Making maps with polygons** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/2_SamplingDistributions_CI/3_Intro%20to%20Making%20Maps_Polygons.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/2_SamplingDistributions_CI/3_Intro%20to%20Making%20Maps_Polygons.ipynb)
- **Geocoding & spatial joins** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/1_Geocoding%20and%20Spatial%20Joins.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/1_Geocoding%20and%20Spatial%20Joins.ipynb)
- **CT data joining & maps** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/2_SamplingDistributions_CI/4_CT_Data_Joining_Maps.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/2_SamplingDistributions_CI/4_CT_Data_Joining_Maps.ipynb)
- **Census data enrichment** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/2_Data%20Enrichment.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/2_Data%20Enrichment.ipynb)
- **Resampling zip codes to towns** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/3_ResamplingZipcodes_to_Towns.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/3_ResamplingZipcodes_to_Towns.ipynb)

*Exercises, assignments & extras:*

- **Centroids** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/Centroids.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/Centroids.ipynb)
- **Advanced — spatial analytics & GeoPandas** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/obs/4_advanced_Spatial%20Analytics%20and%20Geopandas.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/3_Geospatial/obs/4_advanced_Spatial%20Analytics%20and%20Geopandas.ipynb)
- **Project 3 — instructions** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/4_Putting_It_Together/Project3_Instructions.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/4_Putting_It_Together/Project3_Instructions.ipynb)
:::

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Intro to Geospatial Data Types
:class: note dropdown
- **Vector** = points/lines/polygons; **raster** = gridded imagery (elevation, crops); we use vector.
- Locate with **latitude/longitude** (positive N, negative W of England); Earth is a sphere → **projection/CRS** matters.
- **Projected units** are feet/meters (needed for distance); a degree of lat/lon isn't a constant distance.
- A shapefile = a table with a **`geometry`** column; ops include **clip**, **buffer**, and the **spatial join**.
:::

:::{admonition} Intro to Making Maps with Polygon Shapefiles
:class: note dropdown
- Read shapefiles with **GeoPandas** (`gpd.read_file`) — like pandas but with a **`geometry`** column.
- A shapefile is many files (`.shp`, `.dbf`, `.prj`, `.shx`…) — all required.
- **`.plot(column=..., legend=True, cmap=...)`** colors the map by an attribute.
- Mind **spatial resolution**; CT has 169 towns but ~800 polygons (coastal islands).
:::

:::{admonition} Introduction to point data, lat/lon and address geocoding
:class: note dropdown
- **Point data** (house sales) vs. polygons; the **lat/lon is the key** — no formal primary key needed for a spatial join.
- **Geocode**: turn an address string (`address + town + "CT"`) into lat/lon with **GeoPy** (or an alternative).
- Geocoding is slow — loop each row; here it's pre-canned into a lat/long file.
- With coordinates, you can map every point.
:::

:::{admonition} Making a map of house sales (downloading zip shapefile)
:class: note dropdown
- **`folium`** renders an interactive HTML map inline (watch for geocoder errors landing outside CT).
- Convert to a **GeoDataFrame** with CRS **EPSG 4326** (WGS84, GPS); get the **lon/lat columns named/ordered** right.
- Read the US **ZCTA** zip shapefile; subset CT with IDs that **start with "06"** → 283 zips.
- Zip codes are higher resolution than the 169 towns.
:::

:::{admonition} Performing the spatial join, making maps
:class: note dropdown
- **CRSs must match** — house points are 4326 (WGS84), CT zips are 4269 (NAD83); `to_crs(4269)` first.
- **`gpd.sjoin(points, polygons, how="inner", predicate="intersects")`** attaches polygon attributes to each point.
- Overlay: plot polygons, capture `ax`, plot points on top; color by sale amount (`log` for outliers).
- Cast categorical columns to **string** so the legend renders as categories.
:::

:::{admonition} CT Data Joining and Maps
:class: note dropdown
- Shapefiles have limited attributes — **`pd.merge` a CSV** onto them by primary key, then remap.
- Convert the merged result back to a **GeoDataFrame** via its `geometry` column (a nested join).
- CT towns are in a **projected** CRS (units in **feet**); 783 polygons = 169 towns + coastal islands.
- Map any joined column (e.g. families with children), optionally normalized by population.
:::

:::{admonition} Enriching the house sales data with census attributes
:class: note dropdown
- Attach a **zip code** to point data to unlock **hundreds of census variables** (~33,000 US zips).
- Fix the leading-zero bug: **`.astype(str).str.zfill(5)`** (zips stored as ints lose the leading 0).
- Join four census files → a **735-column** master table; join to house points → 768 columns of context.
- Profile it fast with **AutoEDA** (`sweetviz`) against sale amount.
:::

:::{admonition} Advanced: resampling zipcode data to town data
:class: note dropdown
- Resample high→low resolution; **both layers must share a CRS** (`to_crs`).
- **`gpd.sjoin`** towns↔zips on `intersects` → fragments where boundaries split.
- Compute an **area-weighted** value (`land_area × rate`) and **aggregate by town** (weighted average).
- Join back and map the weighted rate per town — resampling in a few lines.
:::
