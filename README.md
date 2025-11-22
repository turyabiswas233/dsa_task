
# dsa_task — Route planner for Dhaka (C++)

Overview
- This project implements a simple route planner for Dhaka using CSV route/road datasets. It builds a graph from road and route files (metro/bus), runs Dijkstra to compute a shortest path between two geographic coordinates, prints a step-by-step itinerary with simple cost estimates, and produces a route.kml file suitable for visualizing the path in Google Earth / Google My Maps.

Key features
- CSV parsers for:
  - Road network lines (Roadmap-Dhaka.csv)
  - Route lines (metro/bus) with station names (Routemap-*.csv)
- Graph construction with unique nodes for each (lat,lon)
- Shortest-path computation with Dijkstra's algorithm (by distance)
- Per-edge type output (metro, bus, road, walk) and a simple cost model (cost = distance * 2)
- KML output (route.kml) for visualization (includes altitude where available)
- Command-line usage; reads four dataset files (1 road dataset + 3 route datasets)

What the program expects (deduced from the code)
- Program file: final 1507.cpp (main implementation)
- Data files in the repo (examples present):
  - Roadmap-Dhaka.csv — road lines. Format (inferred):
    - First token per line: a dataType label
    - Then a sequence of latitude,longitude pairs
    - At the end of the line there may be altitude and distance fields (parsed if present)
    - Each adjacent coordinate pair creates a directed road edge with the parsed distance and altitude
  - Routemap-*.csv (e.g., Routemap-DhakaMetroRail.csv, Routemap-BikolpoBus.csv, Routemap-UttaraBus.csv) — route lines:
    - First token: transport type (e.g., "metro" or "bus")
    - Then a sequence of lat,lon pairs describing the route geometry
    - Last two fields are start station name and end station name (for metro routes)
    - Each route line is turned into an edge from first coordinate to last coordinate with a distance equal to the summed Haversine distance along the route

How it works (summary)
- Parsing:
  - readDhakaStreet reads road CSV lines, creates nodes from lat/lon pairs, and creates edges between consecutive node pairs.
  - readDhakaRoute reads route CSV lines (metro/bus), collects coordinates, sums segment distances, and creates a single edge from the route start to end with station names for metros.
- Graph:
  - Nodes: unique (latitude, longitude) pairs mapped to integer indices.
  - Edges: struct holding source, destination, distance (km), altitude, type (road/bus/metro), and optionally station names.
  - Adjacency list: map<int, vector<int>> from node index to outgoing node indices (directed edges).
- Pathfinding:
  - Dijkstra runs on the constructed directed graph using edge.distance as cost.
  - After finding the path, the program reconstructs and prints a human-friendly itinerary and computes a simplistic fare: cost = distance * 2 (local currency symbol used in print).
- Output:
  - Console: node/edge counts, itinerary, per-segment cost, total distance.
  - KML: route.kml containing the line string of the returned path; altitude is included when present on edges.

Build & run
- Compile (example):
  - g++ -std=c++17 -O2 "final 1507.cpp" -o route_planner
  - If you get math-related link errors, add -lm.
- Run (program requires 4 dataset arguments):
  - ./route_planner Roadmap-Dhaka.csv Routemap-DhakaMetroRail.csv Routemap-BikolpoBus.csv Routemap-UttaraBus.csv
- After starting, the program prompts:
  - Enter the Source Latitude, Longitude, Destination Latitude, Longitude
  - Important: enter latitude then longitude for source and destination (e.g., sourceLat sourceLon destLat destLon).
- Example coordinates (from comments in code — verify correct ordering when entering):
  - Example A (close-by points):
    - Input: 23.741813 90.439973 23.742218 90.439341
  - Example B (larger example):
    - Input: 23.813352 90.344319 23.810823 90.342514
- Program prints:
  - Number of nodes, number of edges
  - Shortest path step-by-step with per-segment cost and type (metro/bus/road/walk)
  - Total distance in km
  - Generates route.kml in the working directory if a path exists

Sample console output (approximate)
- Number of nodes: 1234
- Number of edges: 567
- Enter the Source Latitude, Longitude, Destination Latitude, Longitude
- 23.741813 90.439973 23.742218 90.439341
- Source: (23.741813, 90.439973)
- Destination: (23.742218, 90.439341)
- Shortest Path from (...) to (...):
  - Cost: ৳1.20: Ride Bus from (23.741813, 90.439973) to (23.742000, 90.439600)
  - Cost: ৳1.50: Ride Metro from [Station A] (23.742000, 90.439600) to [Station B] (23.742218, 90.439341)
- Total Distance: 0.85 km
- Generating KML file...
- KML file generated successfully.
