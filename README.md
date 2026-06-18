# Luxembourg P+R Accessibility Analysis Project


## Overview
This project takes a look at P+R facilities in the VDL (Ville de Luxembourg) and determines their accessibility to business districts (BDs) for commuter travel time analysis. 
To do this, a multimodal network was constructed using the AVL and LuxTram PT modes and the OSM walking map.
Using this network, the shortest time between P+R stops and BD stops was determined using frequency-weighted shortest time analysis.
The results were displyaed using a mean travel time P+R x BD matrix, a mean BD travel time barchart, accessibility rankings, and isochrones for qualitative analysis. Furthermore, a set of isochrone maps were produced for each P+R facility.
The model was validated using Google Maps and Mobiliteit

## Objectives
- Form a multimodal public transport network from strach
- Generate travel time isochrones for each P+R facility
- Compare accessibility metrics between facilities
- Find trends in P+R conenctivity and identify holes in the network.

## Study Area
Luxembourg City, Luxembourg. (VDL)

Analysed P+R facilities:

- Gare
- Hollerich
- Howald
- Stadion
- Héienhaff
- Kockelscheuer

## Repo Structure

```text
project/
├── data/
│ ├── raw/      # Data needed before any notebooks will run
│ ├── processed/ # Data produced from notebooks
├── notebooks/   # All scripts are in the form of code blocks that run and output results and processed data
├── outputs/     
│ ├── maps/      # Isochrone maps
│ ├── figures/   # P+R x BD matrix heatmap and mean BD time barchart
├── docs/        # Problem defintion and report coverting the results in detail
└── README.md
```

## Data Sources

Raw data sources can be found in the raw folder. All data processed by the notebooks can be found in processed, including ischrone layers.

The raw data sources are:
- GTFS data Luxembourg (Spans the whole country)
- Commerical land (Preprocesed, needed for BD classification)
- VDL quarters polygons (used in BD classification)
- AVl and Luxtram lines (used in mapping)
- OSM walking network (extracted using osmnx)

## Requirements

Python 3.11+

## How to use this repo

1. Download all data from the raw folder
2. Run the notebooks in the following order:
   - GTFS prep,
   - P+R stop creation and Business district formation
   - netowrk creation
   - network analysis
  
## Outputs

These notebooks output:
- A usable multimodal network during peak hours. Can be changed by tweaking GTFS prep notebook
- Heatmap and bar cahrt for mean times
- Statistical ranking tables
- Isochrone layers that can be used for mapping

## Key methodology

- GTFS data filtered to AVL and LuxTram stops at 7-9 AM on weekdays. Waiting times calculated using average stop-route headway.
- P+R stops found using osmnx query
- BDs classified using OSM land use data and nearby stops found
- multimodal network formed using PT layer, walking layer and connection layer
- Shortest freqeuncy weighted travel time between P+R stops and BD stops run.
- Isochrones generated
- Means calcualted for min, mean and max statistics for each P+R facility.

## Results 

Full results can be found in docs/report.md.

Summary Table:
| Facility | Mean BD Time (min) | Overall Rank | Districts Reachable <25 min |
|-----------|-------------------:|-------------:|----------------------------:|
| Gare | 14.1 | 1 | 4/5 |
| Hollerich | 18.2 | 2 | 4/5 |
| Howald | 23.0 | 3 | 3/5 |
| Stadion | 24.7 | 4 | 2/5 |
| Héienhaff | 29.2 | 5 | 1/5 |
| Kockelscheuer | 30.1 | 6 | 1/5 |

## Limitations
- Static GTFS timetable analysis; no real-time traffic or delays.
- Car travel to P+R facilities not modelled.
- RGTR services and the Kirchberg funicular were excluded.
- Business districts are approximated using commercial land-use data.
- Results depend on assumptions regarding waiting times and P+R facility selection.

## Author
Luca Bachiri
June 2026
