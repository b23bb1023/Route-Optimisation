# Nearest Hospital Route Optimisation

A dual-implementation route optimisation system that locates the nearest hospital to a user's real-time GPS position and computes the shortest driving path using graph-based algorithms — implemented both as a Python web application and as a from-scratch C++ engine with hand-built data structures.

---

## Problem Statement

In emergency medical situations, identifying the closest hospital and the fastest route is critical. Generic navigation tools optimise for general travel — this project specifically targets the hospital-finding problem by:

1. Querying OpenStreetMap for hospitals and clinics within a dynamic search radius
2. Building a weighted road-network graph from real geographic data
3. Computing shortest paths using travel-time as the edge weight
4. Rendering the result on an interactive map with distance and ETA

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     IMPLEMENTATION A                         │
│                   Python Web Application                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Browser (index.html)                                       │
│     │  Geolocation API → user lat/lon                        │
│     │  POST /get_route                                       │
│     ▼                                                        │
│   Flask Server (app.py)                                      │
│     ├─ OSMnx → fetch road graph + hospital POIs              │
│     ├─ Geopy → geodesic distance ranking                     │
│     ├─ NetworkX → shortest_path(weight='travel_time')        │
│     └─ Folium → interactive map with route overlay           │
│                                                              │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                     IMPLEMENTATION B                         │
│               C++ Engine (from scratch)                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   c_file_dsa.cpp (650 lines)                                 │
│     ├─ Haversine formula → spherical distance computation    │
│     ├─ QuickSort → rank hospitals by proximity               │
│     ├─ Adjacency List graph → road network representation    │
│     ├─ Min-Heap priority queue → O(log n) extract-min        │
│     ├─ Dijkstra's algorithm → shortest path computation      │
│     └─ Path reconstruction via parent array → route output   │
│                                                              │
│   Data: Binary .DAT files (hospitals, road nodes, links)     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Implementation Details

### Python Web App (`app.py` + `index.html`)

A lightweight Flask application that provides a browser-based interface:

- **Geolocation**: HTML5 Geolocation API captures user coordinates on page load
- **Hospital Search**: OSMnx queries OpenStreetMap for amenities tagged `hospital`, `clinic`, or `doctors` within a 5–10 km radius
- **Graph Construction**: OSMnx downloads the local road network and enriches edges with speed and travel-time attributes
- **Routing**: NetworkX computes the shortest path weighted by `travel_time`
- **Visualisation**: Folium renders an interactive Leaflet map with user marker, hospital marker, and the route polyline with distance and ETA popup

### C++ Engine (`c_file_dsa.cpp`)

A 650-line implementation that solves the same problem using hand-built data structures with no external libraries beyond `<math.h>` and `<stdio.h>`:

| Component | Implementation | Complexity |
|-----------|---------------|------------|
| Distance Calculation | Haversine formula (spherical geometry) | O(1) |
| Hospital Ranking | QuickSort on distance array | O(n log n) |
| Graph Storage | Adjacency list with linked-list nodes | O(V + E) space |
| Priority Queue | Binary min-heap with decrease-key | O(log V) per operation |
| Shortest Path | Dijkstra's algorithm | O((V + E) log V) |
| Path Output | Parent-pointer backtracking | O(V) |

The C++ engine reads road-network data from binary `.DAT` files, constructs a weighted graph, runs Dijkstra from each candidate hospital to the user's nearest graph node, and outputs coordinate sequences for the top 3 shortest routes.

### Google Colab Bridge (`Untitled5.ipynb`)

A notebook that compiles the C++ engine, converts OSMnx graph data into the binary format expected by the C++ code, and orchestrates the two-stage pipeline.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python, Flask |
| Routing (Python) | OSMnx, NetworkX |
| Routing (C++) | Dijkstra's (hand-built min-heap + adjacency list) |
| Distance | Geopy (Python), Haversine formula (C++) |
| Map Rendering | Folium (Leaflet.js wrapper) |
| Frontend | HTML, JavaScript, Geolocation API |
| Data Source | OpenStreetMap (via OSMnx) |
| Notebook | Google Colab |

---

## Quick Start

### Python Web App

```bash
# Install dependencies
pip install flask folium osmnx networkx geopy

# Run the server
python app.py

# Open browser
# Navigate to http://localhost:5000
# Allow location access when prompted
```

The app will detect your location, find the nearest hospital, compute the optimal route, and render it on an interactive map.

### C++ Engine

```bash
# Compile
g++ c_file_dsa.cpp -o route_engine -lm

# Run (requires binary data files: hospitals.DAT, shivajinagarbinary.DAT, Linksbinary.DAT)
./route_engine
```

---

## Project Structure

```
Route-Optimisation/
├── app.py                  # Flask server: OSMnx + NetworkX routing
├── index.html              # Frontend: Geolocation API + fetch
├── c_file_dsa.cpp          # C++ engine: Dijkstra's from scratch (650 lines)
├── Untitled5.ipynb         # Colab notebook: data conversion + pipeline
└── README.md
```

---

## Key Design Decisions

**Why two implementations?**
The Python version prioritises development speed and real-time data access (live OSM queries). The C++ version demonstrates algorithmic depth — every data structure (min-heap, adjacency list, quicksort) is implemented from scratch without library abstractions, optimised for a fixed dataset where sub-millisecond routing matters.

**Why travel-time over distance?**
Shortest distance does not equal fastest route. The Python implementation uses OSMnx edge speed attributes to weight by estimated travel time, producing routes that prefer highways over shorter but slower residential streets.

**Why dynamic search radius?**
The hospital search starts at 5 km and expands to 10 km if no results are found. This prevents empty results in rural areas while keeping urban queries fast.

---

## Limitations & Future Work

- **C++ data files not included**: The binary `.DAT` files for the C++ engine are region-specific (Shivajinagar area) and not committed to the repo. A data generation script would improve reproducibility.
- **Single destination**: Currently routes to the single nearest hospital. A multi-destination comparison view would be more useful.
- **No traffic data**: Travel-time estimates use static speed limits, not real-time traffic.
- **No turn-by-turn directions**: The route is displayed as a polyline but doesn't generate navigation instructions.

---

## License

MIT License
