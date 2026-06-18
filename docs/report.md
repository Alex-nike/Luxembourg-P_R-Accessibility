
# Luxembourg P+R accessibility project

## Introduction

---
Over the past decade, Luxembourg City has become a major European hub for finance, technology, and professional services. As the city grows, so too do its mobility demands. However, Luxembourg City is geographically small, making it particularly vulnerable to traffic congestion, especially during peak commuting periods. In March 2020, the government introduced free public transport nationwide to reduce the aforementioned congestion [1]. Despite this, a significant proportion of workers continue to commute by car, particularly cross-border commuters who live outside the city. According to the National Mobility Plan 2035 (NPM 2035), the majority of trips longer than 15 km are still undertaken by private vehicles [2]. As population and employment continue to grow, so too will congestion.

One strategy to mitigate this congestion is the use of Park-and-Ride (P+R) facilities. These facilities allow commuters to park their vehicles in the suburban ring and finish their commute using public transport. The effectiveness of a P+R facility depends on its capacity but more importantly on its convenience. If these facilities are not well connected to important districts, commuters may choose to drive directly to their destination instead. Consequently, understanding the accessibility provided by existing P+R facilities is key to handling congestion.

This project evaluates the accessibility of six major P+R facilities within Luxembourg City. The primary research question is: **How long does it take to travel from a P+R facility to a business district using public transport?** A secondary objective is to determine how the accessibility of different P+R facilities can be quantitatively and qualitatively assessed.

To answer these questions, a multimodal public transport network was constructed using GTFS public transport data and OpenStreetMap walking network data. Major business districts were identified using land-use information, while P+R facilities were derived from OpenStreetMap data tags. Frequency-weighted shortest-path analysis was then used to estimate travel times between P+R facilities and business districts. The results are presented through travel-time matrices, accessibility rankings, and isochrone maps that highlight the spatial distribution of accessibility across Luxembourg City. In addition, the public transport network is validated in comparison to Google Maps and Mobiliteit [3],[4].

## Methodology

---

### Data preparation

There are three main data sources used in forming the multimodal network. These are the GTFS data, the P+R stop data and the commercial district data. All mapping data is saved in EPSG:4326/WGS 84 while analysis data is saved in EPSG:2169/LUREF.

The GTFS data was harvested from the Lux Open Data Portal [5]. The data was first cleaned and validated. Next, it was filtered. Only AVL buses and the LuxTram line were considered and the time frame was restricted to 7-9 AM on weekdays. We assume that the travel time is symmetrical with evening peak hours of 5 to 7 PM. Afterward, bus line level headways were considered. This means that each stop and bus line has its own headway time. This was calculated using the following formula: $$avg\_headway = time\_window/n\_departures$$

where time_window is the total amount of time the departures are filtered by. In this case 2 hours. n_departures are the total number of stop events within that period. This is the number of all departure times within the time window for a route. The waiting time is hence avg_headway/2 for each route, assuming the commuter must wait on average half the headway time.

Next, the P+R stop data was prepared. First, the AVL stops were loaded in, taken from the prepared GTFS data. Then, they were used to create a bounding box. This bounding box was used to find all OSM elements within the AVL coverage area that had the park and ride tag and were polygons — parking lots. Next, a geodataframe was created using a 250m centroid buffer from each P+R facility and spatially joined to the AVL stops. This means that only the PT stops within 250 meters of the P+R lots were used for this analysis.

Last, the commercial stops were prepared. Before using python, Overpass was used to find all commercial or retail land within the municipality of Luxembourg. This land was smoothed out and turned into a multipolygon using QGIS. This is the commercial land geopackage in the data folder. Additionally, quarter polygon data can be found for the municipality of Luxembourg in the open data portal [6].

To find the business districts, we calculate the coverage ratio for each quarter. The coverage ratio is expressed in this equation: $$cov\_ratio = comm\_area/quarter\_area$$

where the comm_area is the total commercial land area in a quarter and the quarter_area is the total area of the quarter. Next, we sort the result dataframe by the top five coverage ratios and select them. This was done since the top three are in the city center itself. To account for the two emerging districts, the top five were chosen. In our analysis, these correspond to the BD (Gare, Grund and Ville Haute) but also the two newer commercial centers of Kirchberg and Cloche D'or in Gasperich. To find the stops, we filter the comemrical land to the top five quarters. Similar to the P+R stop method, we take a 100m buffer of this land then spatially join it to the stops. Note, that this includes stops within commercial land.

### Network formation

Once all the data is prepared, the network can be formed. It has three layers: the PT layer connecting stops and bus routes; the walking layer that enables on foot connections; and the connector layer connecting these two layers.

The walking network is harvested from OSM data. The span was found similar to that of the P+R where a bounding box was created 300 meters around the AVL stops and the walking network was taken as the longest connected network in that box. The weight is given by length over the walking speed, given in minutes. The walking speed is 1.4 m/s or about 5 km/h.

Next, the PT network is formed. This has two node layers and three edge layers. The two node layers are the stop layer and the stop-route layer. The stop layer connects to the walking network and acts as an interface, allowing the commuter to access many bus lines from one node. The stop-route layer allows commuters on the stop node to choose the quickest bus line. To go from a stop node to a stop-route node, there is an edge with weight equal to that of the waiting time – avg_headway/2. The wait time is estimated to be five minutes if there is no headway found in the data. The edge that goes the opposite direction has no weight since alight time is assumed negligible – the passenger gets off almost instantly. Finally, we have the stop-route to stop-route edges which are the main edges. They derive their weight from the stop_times table in the following expression: 
$$weight = arrival$\_$time$\_$next$\_$stop  - departure\_time$$.

Frequency weighting was used to accurately model departure time intervals. Depending on how often a stop is serviced by a bus, a commuter is more likely to use the stop. As a consequence, if a P+R stop has low frequency it will generally be less connected, even if travel time between the stop and the BD stop is low. The waiting time added accounts for this uncertainty.

Finally, the connector edges connect these two networks together. Their weight is 0 since the walking time between nodes is already calculated from the walking network. The distance between walking nodes and pt_nodes is assumed negligible, in terms of minutes. The multimodal network is the composition of these three graphs. 

### Network analysis

After loading and validating the data, A P+R x BD minimum travel time matrix is created. This is a matrix where the rows are the P+R facilities and the columns are the Business districts. In each record, we have a list of statistics: [min, mean, max]. These are the min, mean, and max of the respective shortest path between all P+R nodes and all BD nodes. For example, the list for Gare x Gare is the minimum time to get to a Gare BD stop from a Gare P+R stop; likewise, the mean is the mean over all of these node-node pairs and the max is the maximum time. This gives us an underestimate, an average and an overestimate of the travel time from the P+R facility to the BD. It should be noted that in the shortest travel time is taken assuming that the commuters start at the P+R stop and end at the BD stop. This model does not include walking to and from work or the facilities. This information can be found using the isochrone maps generated later.

To consolidate these results, a heatmap was created for all mean values in this matrix. This allows us to see which P+R stops have low and high travel times but also lets us see which BDs are great or poor accessibility. Outliers can also be found.

To get a more holistic metric, means are taken over all BDs. These statistics are represented in a barchart and are sorted by mean BD time. The mean BD time is the mean of each statistic over all BDs. For example, the mean mean Gare time is the mean of all the mean times of each BD. This gives us a ballpark value on how connected a P+R stop is to a BD. Shorter times mean that the P+R facility is better connected leading to greater accessibility and vice versa.

Similarly, we can take these means and rank them from slowest to fastest. Then, we can take the mean of each rank to get an overall mean ranking for these facilities. Finally, we can compare the number of mean travel times under an accessibility threshold. In this case it's 25 minutes. To understand the 25 minute threshold, we must understand the size of the city. Though Luxembourg is not circular, it is roughly about 10 km in diameter when approximating. Making sure to take into account the fact that roads wind, the actual distance traveled will be somewhere around 10-15 km. Given traffic and stop lights, the average speed should be a bit faster than a bike, around 30 km/h. Therefore, an average travel time lies around 20 to 30 minutes. To take this all into account, a good approximation for the mean time is 25 minutes. Therefore, accessible P+R facilities are those that can get to BDs under this mean time. We have three metrics to measure the accessibility of a P+R facility, two qualitative and one quantitative.

Finally, to get a feel for each facility, Isochrone maps were generated. Each isochrone has 4 bands: 0-15, 15-20, 20-30 and 30-45 minutes. The maps themselves contain the P+R stop centroids, the top five business districts and the AVL lines themselves which were acquired from Geoportail [7].

To validate the model, five test journeys were performed to check whether the time taken was consistent enough with open access models. The journeys were selected such that they covered a robust set of circumstances such as: from city limit to city limit journeys; medium distance journeys; and tram heavy journeys. The two open access models used were Google Maps and Mobiliteit, the government transport agency app. These were chosen since they were the two most commonly used apps when dealing with public transport journey planning in Luxembourg. Times were extracted at 8:00 AM on a weekday for the journey planner. The shortest travel time was used. The absolute value between my model and the open access model was computed for each journey. Furthermore, the mean absolute error (MAE) was found for each platform. Our model is valid if the MAE < 5 minutes, the default waiting time.

## Results

---

The following section collects the results of this analysis. The first figure is a heatmap which shows the average time it takes to get to a BD from each P+R facility. The second figure is a graph denoting the mean time to any BD based on three statistics: minimum, mean and maximum. Furthermore, rank analysis is provided in three tables. The first table compares the rank of each P+R facility for each metric: min, mean, and max. The second uses these metrics to generate an average rank for each P+R facility. Finally, the last table is a quantitative assessment of general accessibility in the mean metric. The table shows the number of BDs that are less than 25 minutes away for each P+R facility. Next, a single isochrone map is examined. Afterwards, the total set of isochrone maps is examined and holes are identified. All of the maps are provided in the maps folder. The validity of the model is checked thereafter. Finally, a summary table is included, summarizing key results.

### P+R to BD mean travel time heatmap

![Figure 1](Figures/heatmap.png)

*Figure 1. This heatmap shows the mean shortest travel time from each P+R facility to each business district (BD). For example, the top left square tells us that it takes on average 5 minutes to get from the P+R stop to a BD stop in the Gare district.*

There are several interesting things to note from this figure. First, Gare and Hollerich are consistently the closest facilities to business districts. Likewise, we can see that Gasperich is the district most connected to P+R stops. On the other hand, Kockelscheuer and Héienhaff are the facilities least connected to the BDs. Similarly, Kirchberg tends to be far away from a lot of the P+R facilities, except for Héienhaff, which is connected by tram.

---

### Mean BD travel time chart

![Figure 2](Figures/barchart.png)

*Figure 2. This chart shows the average time to a BD for every P+R facility for each of three statistics: min, mean and max*

We can see similar trends from Figure 1, expressed as averages here. We can see that the overall shortest travel time P+R facility is Gare, having the lowest time for each statistic. Crucially, this is not true for Kockelscheuer. Its statistics are longer for mean and max but its min statistic is shorter than Héienhaff's. This means that the shortest time to get to a BD from Kockelscheuer is less than that of Héienhaff. Even still, the range of max times is around 17 minutes, the range of min times is 16 minutes and the range of mean times is 16 minutes. This suggests that there is not much variation between statistics. By percentage, Gare's mean BD time is 14.1 minutes while the longest is 30.1 minutes in Kockelscheuer. The percentage difference is 113% or about 2.1 times longer. The rise from the first to the second fastest mean BD time is 30.4% or 1.3 times greater.

---

### Min, Mean, and Max P+R facility ranking

| Min Rank | Mean Rank | Max Rank |
| -------- | -------- | -------- |
| Gare     | Gare     | Gare     |
| Hollerich  | Hollerich     | Hollerich    |
| Howald   | Howald   | Howald   |
| Stadion   | Stadion    | Stadion   |
| Kockelscheuer   | Héienhaff   | Héienhaff   |
| Héienhaff   |Kockelscheuer   | Kockelscheuer   |

**Table 1.** Table representation of bar chart. Rank 1 is shortest and rank 6 is slowest.

---

### Composite Mean rank

| P+R facility | Composite Mean Rank | 
| -------- | -------- | 
|Gare    | 1    | 
| Hollerich   | 2   | 
| Howald   | 3   | 
| Stadion   | 4   | 
| Héienhaff    | 5.3    | 
| Kockelscheuer   | 5.7   | 

**Table 2.** Mean rank for each P+R facility, based on Table 2 rank positions.

---

### Number of districts reachable under 25 minutes

| P+R facility |   $T_{mean}$ < 25 min       |
| -------- | -------- | 
|Gare    | 4/5   | 
| Hollerich   | 4/5  | 
| Howald   | 3/5   | 
| Stadion   | 2/5  | 
| Héienhaff    | 1/5    | 
| Kockelscheuer   | 1/5   | 

**Table 3.** Number of districts that take less than 25 minutes to reach on average.

---

As expected, the Gare and Hollerich are first and the Kockelscheuer and Héienhaff are last. Interestingly, from Table 2, Kockelscheuer is on average the worst option. We see similar findings in Table 3. The deviation between the top two and the third and fourth is also more noticeable. Stadion still tends to be far away from most BDs while Howald, only around one kilometer away, is close to the majority. It should be noted that Kockelscheuer and Héienhaff are so similar, based on means in Figure 2, that their rank is effectively equivalent, especially in comparison to Gare and Hollerich.

---

### Map 1: Howald Isochrone map

![Map 1](Isochrone%20maps/Howald.png)

*Map 1. For simplicity, the Howald isochrone map is taken as an example map since it sits in the middle of the pack. All areas farther than 45 minutes are unshaded.*

Based on the Howald map, areas within a 1km radius are around 0-15 minutes away but, there are some notable exceptions. The Grund area is yellow for the most part and that can be corroborated by the average value of 26.9 minutes from Figure 1. Likewise, almost all of Kirchberg is within the red band. This goes for all the extremities of the city. Still, using the Howald P+R facility, a commuter can get to all important locations within 45 minutes.

Taking a look at all the isochrone graphs, there is a general trend in BD accessibility. Firstly, the most serviced locations are the central districts. All AVL buses and the tram pass through either the Gare in the Gare quarter or Hamillius in the Ville Haute quarter. As a consequence, these two districts are at most 30 minutes away. The Grund is in the middle area. Since there is a height differential, the travel time is generally longer. Finally, Kirchberg and Gasperich tend to be the hardest to reach on average. Despite that, they have their own dedicated P+R facilities — Gare and Héienhaff for Kirchberg and Stadion and Howald for Gasperich.

Likewise, there are noticeable holes in the coverage. There is a district under construction to the north of Kirchberg. At the moment, only Héienhaff can serve it within 20-30 minutes. All other facilities are more than 30 minutes away, even Gare. The Strassen area, to the west of the city center, is also underserved. This area is only in the yellow band from the Hollerich P+R facility, otherwise it is 30 minutes or higher. Conversely, Bonnevoie, a quarter near Howald, is under construction but it is linked very well with the P+R facilities in Howald, Stadion and Gare.

### Model Validation

| Origin | Destination | Model (min) | Google Maps (min) | Mobiliteit (min) |
|----------|----------|----------:|----------:|----------:|
| Senningerberg, Héienhaff P+R | Gasperich, Stade de Luxembourg | 44.9 | 43 | 39 |
| Luxembourg, Wallis | Kirchberg, Réimerwee | 29.1 | 26 | 25 |
| Hollerich, P+R Bouillon | Gasperich, Louis de Froment | 13.2 | 10 | 12 |
| Kockelscheuer, Patinoire | Centre, Stäreplaz / Étoile (Tram) | 33.0 | 30 | 29 |
| Howald, P&R Howald | Belair, Rheinsheim | 29.4 | 27 | 29 |

**Table 4.** Min travel time from model, Google Maps, and Mobiliteit for various trips.

The Mean Absolute Error (MAE) for Google maps with respect to the model is 2.72 minutes while that of Mobiliteit is 3.12. Further still, when not taking the absolute, these errors are all positive. The model consistently overestimates the public platforms.

### Results Summary


| Facility | Mean BD Time (min) | Overall Rank | Districts Reachable <25 min |
|-----------|-------------------:|-------------:|----------------------------:|
| Gare | 14.1 | 1 | 4/5 |
| Hollerich | 18.2 | 2 | 4/5 |
| Howald | 23.0 | 3 | 3/5 |
| Stadion | 24.7 | 4 | 2/5 |
| Héienhaff | 29.2 | 5 | 1/5 |
| Kockelscheuer | 30.1 | 6 | 1/5 |

**Table 5.** Summary of all P+R facility mean BD time results.

## Discussion

### Discussion of Results

Regarding the travel time based results, P+R facilities outside the center are specialized. They are made to access only nearby districts, like Héienhaff for Kirchberg. Likewise, central facilities have great accessibility. Central facilities are generally better due to better connectivity like how Gare has about half of all AVL buses running through it – the other half running through Hamillius in Ville Haute. This better connectivity leads to shorter mean times. Likewise, the poor connectivity on the peripherals is due to the lack of connections and hence the increase in waiting time for transfers.

Noting the gaps in coverage, Bonnevoie and Howald have already been addressed. With the tram extension to Stadion going through the two districts as well as Cloche d'Or, the renovated districts were built with accessibility in mind. When looking at the PNM 2035, we see a couple important areas that are expected to be expanded [2]. The tram network is to be expanded with stops in: Hollerich, connecting to the P+R facility there; northern Kirchberg, where there were accessibility holes; and Strassen. This suggests that the Hollerich and Strassen areas are to be developed next. Indeed a Nei-Hollerich district is planned to be built around Hollerich and Belair as well [8].

Looking at the P+R portion of the plan, there are several important things to note. The directive is multimodality so, the intention is that bus corridors and tram stops are connected to the lots. As well, the LuxExpo lot is named as a P+R facility even though it is not labeled as such. A new P+R facility is planned to open, near the CHL in North Belair, called P+R Ouest (West). This should cover the majority of the new developed area, allowing new business districts in the east as the city continues to sprawl.

Another interesting thing to note is the lack of P+R stops in the south-east of the city. The closest facility is the Howald lot. Why doesn't the city build up that area? The main issue is terrain. Since Kirchberg is a plateau, it was easy to develop; however, due to topological considerations, areas outside of the plateaus are considerably harder to develop. Furthermore, that land is owned by farmers so to develop that land requires far too much logistics to be practical. On top of that, ecological concerns and preserving a green belt are also considered essential for the city [9]. Finally, Luxembourg has a policy of redevelopment where existing land is renovated, like that in Cloche D'or. Therefore, adding new stops to renovated districts is more efficient than sprawling outwards [10],[11].


### Limitations

In this section, the limitations of the methodology will be discussed. First and foremost are the scope limiting decisions. These include: omitting border travel time, omitting P+R to P+R travel time, omitting RGTR busses and omitting the Funicular in Kirchberg. All of these decisions were made to reduce the scope and were based on available open data. The question of which P+R facility is the best depends on what border town you come from. Additionally, road and traffic data for the highways is needed for this analysis which is not easily accessible. This is also true for the P+R to P+R travel time. The Hollerich lot may be a better lot if we factor in the travel time it takes just to get to the Gare lot. Likewise, omitting RGTR busses was done to limit the scope of the project to that of the VDL. Despite this, there are many RGTR busses that have routes to and from BDs. When performing the validation, some of the journeys had a faster time when a RGTR bus was used. This is similar to the Funicular which may affect the Grund results. Furthermore, the funicular is not included in the GTFS data itself meaning one would have to encode it themselves. 

For the P+R data, as previously discussed, the parking lot at LuxExpo was labelled a P+R stop in the PNM but not elsewhere. In fact, when only keeping the ways, that lot was dropped. Many parking lots were dropped during this way filtering. It is uncertain the effect that this had on the results however, if LuxExpo would be included, it would just be a better Héienhaff. Since Héienhaff is not technically in the VDL, it stands to reason that it is technically out of scope for this analysis. It is kept since it is a major Quai for bus routes as well as for the tram. However so is LuxExpo. Therefore, the only reason it was kept was due to the P+R label.

When looking at the validation, we see that the model consistently overestimates the public transit models. Interestingly, the custom designed model, Mobilitet, has the largest error. This could be due to the average headway calculations. When performing the analysis, the model starts at the bus stop and always waits half that time. In comparison, the journey planner starts at the bus stop as soon as bus arrives. The only wait time is then for transfers. If I wanted this model to be more accurate, I would remove the starting waiting time. However, this fails to consider delays, traffic and other mishaps that occur during peak hours. It very well could be that the overestimate accounts for this. Furthermore, this model does not presuppose the human planning done by the commute before they get to the stop. The commuter knows the departure times so will get there on time. Overall, this suggests that these results are an overestimate but cautious at that. Further still, since car traffic was not included, delays could also come in that form. Despite this, since the MAE is less than 5 minutes for both models, the effects are not crucial enough for our model to be invalid. It's important to note that traffic tends to be accounted for using GTFS data but since this model is not dynamic, there is some uncertainty as a consequence.

Finally, regarding business districts. Ideally, one would use STATEC job density data per quarter however, that is not openly accessible. There is data similar to that in the PNM so the data does exist, just not in the hands of the public. The commercial land method was used to compromise. Likewise, as discussed, Strassen is being redeveloped as well as a new district called Nei-Hollerich. The problem is that Strassen is not a quarter of the VDL. Therefore, getting comemrical land data for it is not possible. The quarters of Luxembourg City and municipalities of Luxembourg are the lowest subdivisions available. Since the AVL often stretches past the limits of the VDL municipality, there may be business districts that were missed.

### Future research

In lieu of the limitations and discussions, the next step for this project would be to account for traffic. A simple extension would add a traffic layer to the network. Then, times would be compared between taking a car to each P+R facility versus simply starting at a P+R facility. However, this needs a known starting point thus using the border towns as an example, extending the model to a full border town model would be more accurate. Likewise, a comparison between this car + P+R + PT model vs a Train + PT model or bus + PT model for each town would allow us to see which method is the best overall for border town commuters. Another improvement would be the first stop waiting time removal such that at the first stop, the commuter should not wait, assuming they get to the stop on time for the departure.

## Conclusion

In this project, the question of "How long does it take to travel from a P+R facility to a business district using public transport" was posed. To answer this question, a multimodal public transit network was created and frequency-weighted shortest path analysis was used on it, using the P+R stops as origins and BD stops as destinations. This analysis showed that the Gare and Hollerich P+R stops were the fastest with mean times of 14 and 18.2 minutes to a BD while Héienhaff and Kockelscheuer were the slowest with mean times of 29.1 and 30 minutes. This shows that more central P+R facilities are more accessible to business districts while those on the periphery are less. One major flaw in this project is lacking a car travel time factor. In a future project, a car layer with traffic should be added to see whether driving to Gare or driving to Hollerich is a better option.

## Citations

[1] <https://transports.public.lu/en/plus/faq/gratuite-transports-publics.html>

[2] <https://gouvernement.lu/en/dossiers/2022/pnm2035.html>

[3] <https://maps.app.goo.gl/9B3GYSQ1vSn3spZcA>

[4] <https://www.mobiliteit.lu/en/>

[5] <https://data.public.lu/fr/datasets/horaires-et-arrets-des-transport-publics-gtfs/>

[6] <https://data.public.lu/en/datasets/carte-topographique-quartiers-de-la-ville-de-luxembourg/>

[7] <https://data.public.lu/en/datasets/transport-en-commun/>

[8] <https://www.neihollerich.lu/fr/>

[9] <https://sustainlux.lu/en/initiative/the_green_belt>

[10] <https://www.vdl.lu/en/topic/urban-development-and-housing>

[11]<https://www.vdl.lu/en/city/projects-and-commitments/urban-development/concept-and-objectives>
