# Attraction Catchment & Resource Proximity Analysis — Edmonton
**Author:** Martins
**Date:** May 11, 2026
**City:** Edmonton, Alberta, Canada
**Software:** QGIS (EPSG:3776 — NAD83 / Alberta 10-TM Forest)
**Project File:** `Attraction_Catchment.qgz`

---

## 1. Project Overview

This project analyses the **spatial relationship between Edmonton's 56 major
civic attractions and 502 community arts and recreation resources** by measuring
how many community resources fall within 500 m, 1 km, and 2 km of each
attraction. The goal is to distinguish attractions that function as **isolated
destination anchors** — drawing visitors into areas with little surrounding
community programming — from those **embedded in dense service clusters** that
reinforce neighbourhood-level recreation ecosystems.

This distinction is directly relevant to tourism planning, community investment
decisions, and equitable service distribution across Edmonton's wards and districts.

---

## 2. Input Data

| Layer | Source | Features | Geometry | Original CRS |
|-------|--------|----------|----------|--------------|
| `geo_export` | City of Edmonton Attractions | 56 | Point | EPSG:4326 |
| `211 Arts and Recreation Resources` | 211 Alberta | 502 | Point | EPSG:3857 |
| `City Boundary` | City of Edmonton | 1 | Polygon | EPSG:3776 |

All layers were reprojected to **EPSG:3776 (NAD83 / Alberta 10-TM Forest)**
prior to analysis. This CRS uses metres as its unit, making distance-based
operations (buffers, proximity counts) accurate for the Edmonton area.

---

## 3. Methodology

### 3.1 Reprojection
All three layers were reprojected to EPSG:3776 to ensure consistent metric
units throughout. Buffers in geographic CRS (degrees) would produce incorrect
and inconsistent distances.

### 3.2 Multi-Ring Buffer Generation
Three concentric buffer zones were generated around each of the 56 attraction
points at radii of **500 m**, **1,000 m**, and **2,000 m**. Buffers were not
dissolved, preserving one polygon per attraction per radius for individual
attribution. A 32-segment approximation was used for smooth circles.

### 3.3 Resource Counts per Catchment Zone
The `native:countpointsinpolygon` algorithm was applied independently for each
buffer layer against the 502 community resources layer, producing per-attraction
counts:
- `res_500m` — resources within 500 m walking distance
- `res_1000m` — resources within 1 km (approx. 10–12 min walk)
- `res_2000m` — resources within 2 km (approx. 20–25 min walk or 5 min cycle)

### 3.4 Cluster Tier Classification

Each attraction was classified into one of four tiers based on its resource
counts:

| Tier | Criteria | Count | Meaning |
|------|----------|-------|---------|
| 🔴 Isolated Anchor | `res_500m = 0` AND `res_1000m ≤ 2` | 18 | No surrounding community programming |
| 🟠 Sparse Cluster | `res_500m ≤ 3` AND `res_2000m ≤ 10` | 10 | Minimal nearby services |
| 🔵 Moderate Cluster | `res_500m ≤ 8` (otherwise) | 26 | Embedded in a service neighbourhood |
| 🟦 Dense Cluster | `res_500m > 8` | 2 | Core of a major recreation hub |

---

## 4. Key Findings

### 4.1 Summary Statistics

| Metric | Value |
|--------|-------|
| Total attractions analysed | 56 |
| Avg resources within 500m | 1.3 |
| Avg resources within 1km | 9.8 |
| Avg resources within 2km | 28.2 |
| Max resources within 2km | 118 (Jasper Place Annex) |
| Min resources within 2km | 0 (Telus World of Science) |

### 4.2 Most Resource-Rich Attractions (2 km catchment)

| Attraction | 500m | 1km | 2km | Tier |
|-----------|------|-----|-----|------|
| Jasper Place Annex | 14 | 51 | 118 | Dense Cluster |
| Scona Pool | 0 | 53 | 95 | Moderate Cluster |
| Oliver Outdoor Swimming Pool | 0 | 0 | 92 | Isolated Anchor* |
| Mill Woods Recreation Centre | 0 | 52 | 91 | Moderate Cluster |
| Kenilworth Arena | 0 | 54 | 91 | Moderate Cluster |

*Oliver Outdoor Swimming Pool scores highly at 2km but has zero resources
within 1km — indicating a large underserved immediate zone despite regional
density.

### 4.3 Most Isolated Attractions (0 resources within 2 km)

| Attraction | 500m | 1km | 2km | Tier |
|-----------|------|-----|-----|------|
| Telus World of Science Edmonton | 0 | 0 | 0 | Isolated Anchor |
| Kinsmen Twin Arenas | 0 | 0 | 1 | Isolated Anchor |
| Castle Downs Recreation Centre | 0 | 0 | 1 | Isolated Anchor |
| Kinsmen Pitch & Putt | 0 | 0 | 2 | Isolated Anchor |
| Jasper Place Fitness & Leisure | 0 | 0 | 2 | Isolated Anchor |

**Telus World of Science** — Edmonton's flagship science attraction — has
**zero community recreation resources within 2 km**, making it the most
isolated major attraction in the city. This has direct implications for
visitor experience, transit connectivity, and neighbourhood service investment.

---

## 5. Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | String | Attraction name |
| `type` | String | Attraction type (Attraction, Recreation, etc.) |
| `address` | String | Street address |
| `quadrant` | String | City quadrant (NE, NW, SE, SW) |
| `url` | String | Official website |
| `res_500m` | Integer | Community resources within 500 m |
| `res_1000m` | Integer | Community resources within 1 km |
| `res_2000m` | Integer | Community resources within 2 km |
| `cluster_tier` | String | Tier: Isolated Anchor / Sparse / Moderate / Dense |

---

## 6. Output Files

```
Attraction Catchment & Resource Proximity Analysis/
├── Attraction_Catchment.qgz              ← QGIS project (open this)
├── attraction_catchment.gpkg
│   ├── attraction_catchment              ← Scored attractions (primary output)
│   ├── buffers_500m                      ← 500 m catchment zones
│   ├── buffers_1000m                     ← 1 km catchment zones
│   ├── buffers_2000m                     ← 2 km catchment zones
│   ├── resources                         ← Community resources (EPSG:3776)
│   └── city_boundary                     ← Edmonton city boundary
└── README.md
```

---

## 7. Map Symbology

### Attractions (cluster_tier)
| Tier | Colour | Symbol |
|------|--------|--------|
| Isolated Anchor | `#d7191c` Red | Star |
| Sparse Cluster | `#fdae61` Amber | Star |
| Moderate Cluster | `#abd9e9` Light blue | Star |
| Dense Cluster | `#2c7bb6` Dark blue | Star |

### Buffer Zones
| Ring | Colour | Style |
|------|--------|-------|
| 500 m | Light blue | Dashed outline, 60% opacity fill |
| 1 km | Medium blue | Dashed outline, 45% opacity fill |
| 2 km | Dark blue | Dashed outline, 35% opacity fill |

Community resources are rendered as small green circles.

---

## 8. Limitations

- **211 resource completeness** — the 502 resources represent programs
  registered with 211 Alberta and may not capture all informal or privately
  operated community recreation services.
- **Point-in-polygon counts only** — the analysis uses straight-line distance,
  not network (walking/cycling) distance. Actual pedestrian accessibility
  depends on road network, river valley barriers (notably the North
  Saskatchewan River), and transit routes.
- **Buffer overlap not handled** — where attraction buffers overlap, resources
  are counted independently for each attraction. A resource near two
  attractions will be counted twice across both.

---

## 9. Recommended Next Steps

- **Network distance analysis** — replace Euclidean buffers with walking or
  cycling isochrones using the QGIS ORS Tools plugin and Edmonton's road network
  to compute true accessibility catchments.
- **Ward-level equity report** — aggregate cluster tier counts by `civic_ward`
  to identify whether isolated anchors are disproportionately concentrated in
  specific wards, indicating systemic service gaps.
- **Cross with language accessibility** — join the resource language field to
  identify whether isolated attraction catchments also lack multilingual
  services, compounding barriers for newcomer communities.
- **Tourism planning overlay** — overlay the isolated anchor list with hotel
  and accommodation locations to assess whether visitor-serving areas are
  disconnected from community recreation infrastructure.

---

*Generated with QGIS MCP Plugin + Claude (Anthropic) — May 11, 2026*
