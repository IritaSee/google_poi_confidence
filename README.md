# Google POI Confidence
Reflect how similar POI data from OpenStreetMap and Google Maps Scrapping
(for Kuwala.io Junior Data Engineer role)
## Task:
```
At Kuwala, we have two pipelines to get POI information. The first is the OpenStreetMap
(OSM) POI pipeline which parses relevant tags and geo-boundaries from OSM objects. The
second pipeline is a scraper for Google Maps. The scraper takes a search string and returns
the data for the first corresponding result. Sometimes the result is an exact match and
sometimes it is not. Since it is not feasible to check each result manually we need to provide
a confidence score that indicates how well the Google POI returned from the scraper
actually matches the OSM POI the query has been generated on.

The search strings that are sent to the Google scraper are based on the name of the OSM
POI (if available) and its address tags.

The task is to calculate the confidence score between 0 and 1 indicating the quality
of the result returned from the Google scraper. Add this score as a new column to the
google_osm_poi_matching table and save it as a CSV.
```



## Steps: 
1. Read all csv (from google maps, OSM, and the matching table)
#### (In Preparations Tab)
2. From the matching table, create a table from OSM data consist of locations that we want to match up
3. Just like process 2, but for google maps data
#### Coordinate Matching
4. Create two lists `latdelta` and `longdelta` to save the difference of latitude and longitude from both data source
5. Using Euler distance, calculate how far the difference was from both of the data sources, and put it in `distancedelta`
#### Name Matching
##### in `Built in Sequencer` tab
6. Using Built-in Sequence Matcher, compare each location's `name` from both data sources, then add them all in `score_sequencer`
##### in `Using external Library` tab
7. To enhance the result, we will use [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance), so let's install `thefuzz` package!
8. From now on, all string matching will use Levenshtein Distance algorithm
9. Make `fuzz_name` list to save each ratio (1 means exacly same, 0 means totally different)
10. Make `is_with_subs` to remember "is there any exact match as a substring in names from both side?"
11. Iterate for every name in OSM and Gmaps data, make it lowercase, and compare each
12. divide the result by 100 so it's between 0 to 1
#### Category Matching
13. See all categories from OSM and Gmaps data,  
14. Make `fuzz_cat` list to save each ratio (1 means exacly same, 0 means totally different)
15. Iterate for every name in OSM and Gmaps data, make it lowercase, trim one character from each side(removes the {})
16. From step 15, any NaN (empty category) will represented as character 'a', make an if to replace every 'a' with '---' so it's not mistaken as a legit category
17. Take the token ratio (difference score), create a simple if to add 10 points to the ratio if there is any match as a substring from both side
18. Divide the result by 100 so it's between 0 to 1
#### Address Matching
19. We know that both have different format, but for the sake of time, we are going to just simply join all address related from OSM, and compare it with google's, so now, let's make a list `addr_type` to list all of address types, 
20. Loop as many as the matching rows, then create another for loop to each address types, if it's not empty (NaN), write them to `temp`
21. Remove the two last characters, since it's an automated comma and space
22. Still in the main loop of step 20, we add the `temp` string to `joined_addr` list, so we can compare each of this with address from google maps data
23. Make `fuzz_addr` list to save each ratio (1 means exacly same, 0 means totally different)
24. Make `addr_with_subs` to remember "is there any exact match as a substring in address from both side?"
25. Iterate for every name in OSM and Gmaps data, make it lowercase, and compare each
26. divide the result by 100 so it's between 0 to 1
#### Scoring
27. Add results from previous steps to `matching` table, as:
```
matching['latlong_score'] represents distancedelta
matching['name_score'] represents fuzz_name
matching['name_has_substring'] represents is_with_subs
matching['categories_score'] represents fuzz_cat
matching['address_score'] represents fuzz_addr
and
matching['addr_has_substring'] represents addr_with_subs
```
28. Now we can see everything in one workspace, 
29. Create `final` list to save all the final confidence score
30. For `distancedelta` we need to map them to 0 to 1, and reverse them (so the most similar coordinate will represented as a value near one, and vice versa)
```
distance delta mapping algorithm:
- get the highest value in distancedelta
- get the lowest value in distancedelta
- mapped distancedelta is the old distancedelta substracted with lowest value, 
  divided by maximum value 
```
31. Final score can be calculated as:
```
((1-delta_mapped(distancedelta))*latlong_weight) 
+ (fuzz_name)*name_weight 
+ (is_with_subs*is_with_subs_weight) 
+ (fuzz_cat*category_weight) 
+ (fuzz_addr*address_weight) 
+ (addr_substring_weight*addr_with_subs)

```
where all weights are naively choosen as:
```
latlong_weight=0.4
name_weight=0.3
is_with_subs_weight=0.1
category_weight=0.1
address_weight=0.05
addr_substring_weight=0.05
```
32. Save each final score to `final` list
33. Create `confidence` column and add `final` as its value
34. Save `matching` to CSV

## Future Works:
This requires more research, but since i am running out of time, here is what I got:
1. we need look further into the weighting system, A.K.A. How each scores represent the true final score, for now, I will just naively pick some score
2. Address is the trickiest one, requires a lot of chopping, joining, etc
3. I could not access this useful [research](https://www.researchgate.net/publication/2834776_A_comparison_of_string_DISTANCE_metrics_for_name-matching_tasks
) yet, that could help the string matching,


Thank you! ^^ All comments and advices are appreciated!
