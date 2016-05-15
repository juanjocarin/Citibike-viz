# W209 Final Project
##### *By Juanjo Carin, Kevin Allen, Matt Hayes, and Rosalind Lee*
### Visualizing Citi Bike Data
Check out the [web site on Github](http://kevinallen.github.io/citibike-viz/).

The data that was used for this project can be found on the [Citi Bike NYC web site](https://www.citibikenyc.com/system-data). The source data consists of a single file per month, all with the same schema.

 - Trip Duration (seconds)
 - Start Time and Date
 - Stop Time and Date
 - Start Station Name
 - End Station Name
 - Station ID
 - Station Lat/Long
 - Bike ID
 - User Type (Customer = 24-hour pass or 7-day pass user; Subscriber = Annual Member)
 - Gender (Zero=unknown; 1=male; 2=female)
 - Year of Birth

Here is an example row:
```
520,2014-06-01 0:00:02,2014-06-01 0:08:42,358,Christopher St & Greenwich St,40.73291553,-74.00711384,426,West St & Chambers St,40.71754834,-74.01322069,18840,Subscriber,1979,1
```
Because of the popularity of the bike share program, and the limited resources and performance of Tableau Public, it was necessary to focus on only a single year. This was still 8,018,751 rows!

### Cleaning the Data
Tableau is great for visualizing data, just make sure the data is structured properly first. Many hours were spent in vain attempting to manipulate the source data in Tableau.

SQLite made quick work of shaping and aggregating the data in a way amenable to Tableau. For example, in order to create a path between two points on a map in Tableau, it is necessary to have two rows for every row our the Citi Bike source data. After putting the raw data in a table in SQLite, the following query was used to get the data in the proper format for plotting paths.

```SQL
INSERT INTO paths
SELECT
    start_station_id || "-" || end_station_id, start_station_id,
    start_station_name, 0,0, start_station_latitude, start_station_longitude,
    COUNT(*), AVG(tripduration), SUBSTR(starttime,1,10), start_station_id,
    end_station_id
FROM rawdata
GROUP BY
    start_station_id, end_station_id, substr(starttime,1,10);

INSERT INTO paths
SELECT
    start_station_id || "-" || end_station_id, end_station_id, end_station_name,
    AVG(tripduration), COUNT(*), end_station_latitude, end_station_longitude,
    COUNT(*), AVG(tripduration), SUBSTR(starttime,1,10), start_station_id,
    end_station_id
FROM rawdata
GROUP BY
    start_station_id, end_station_id, substr(starttime,1,10);
```
More information on the cleaning process can be found in the [notes file](https://github.com/kevinallen/citibike-viz/blob/gh-pages/notes.md).

### Usability Testing
Usability testing was conducted in order to find confusing and intuitive elements of the visualizations. The usability testing was conducted as a one-on-one interview session with the users. The users were asked to answer questions, and their audio responses and interaction with the visualization were recorded.

#####Interviewee #1
 - [Popular Citi Bike Trips](https://drive.google.com/open?id=0BzZOSSpYDxvYd0dYZ19vb01QdUE)
 - [Popular Citi Bike Origins and Destinations](https://drive.google.com/open?id=0BzZOSSpYDxvYNmxnNFIxOVdsUmM)
 - [Bike Rental Intervals](https://drive.google.com/open?id=0BzZOSSpYDxvYVGFocjRtNC1maWM)
 - [Distribution of Traffic](https://drive.google.com/open?id=0BzZOSSpYDxvYcDNJMlgySzhnS2M)
 - [Nearby Stations](https://drive.google.com/open?id=0BzZOSSpYDxvYTGh1bFpaeThwWWM)
 - [Maintenance](https://drive.google.com/open?id=0BzZOSSpYDxvYTkV3SXVudU1IU28)

#####Interviewee #2
 - [Popular Citi Bike Trips](https://drive.google.com/open?id=0BzZOSSpYDxvYaUJ1Smw5a2kwM28)
 - [Popular Citi Bike Origins and Destinations](https://drive.google.com/open?id=0BzZOSSpYDxvYemlialFHc0dGSjg)
 - [Bike Rental Intervals](https://drive.google.com/open?id=0BzZOSSpYDxvYUkJmeFV6c1R3YW8)
 - [Distribution of Traffic](https://drive.google.com/open?id=0BzZOSSpYDxvYemRSVEJ2MTN1T1k)
 - [Nearby Stations](https://drive.google.com/open?id=0BzZOSSpYDxvYTXMtaHBjM2NxSDQ)
 - [Maintenance](https://drive.google.com/open?id=0BzZOSSpYDxvYNzVaT1JEanVtV3c)

#####Interviewee #3
 - [Bike Rental Intervals](https://drive.google.com/open?id=0B3cqLO1xs1GIMWt5WlVaQUJ1MWs)
 - [Distribution of Traffic](https://drive.google.com/open?id=0B3cqLO1xs1GIUlA4anFMeXhENlU)
 - [Nearby Stations](https://drive.google.com/open?id=0B3cqLO1xs1GIN0FuRVl6cTJvYTQ)
# Citibike-viz
