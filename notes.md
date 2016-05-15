### Preprocess data
#### Create tables

In order to get the data in a format that was easy to use in Tableau, the CSV
data was loaded into SQLite. The first thing we need to do is create a new
database...

```
$ sqlite3 bikedata.sqlite

```
and then add a couple tables.
```
create table rawdata(
    tripduration, starttime, stoptime, start_station_id, start_station_name,
    start_station_latitude, start_station_longitude, end_station_id,
    end_station_name, end_station_latitude, end_station_longitude, bikeid,
    usertype, birth_year, gender
);

create table paths(
    path_id, station_id, station_name, avg_duration, num_trips, latitude,
    longitude, trips_filter, avg_duration_filter, day, from_station, to_station
);
```

#### Load data

Before importing the CSV data, we need to remove the header row.

```
$ sed 1d citibike.csv > citibike_v2.csv
```

Import the CSV data into SQLite.

```
sqlite.mode csv
.import citibike_v2.csv rawdata
```

Aggregate the raw data to create paths in Tableau. This first row is for the
start station.

```
insert into paths
select
    start_station_id || "-" || end_station_id, start_station_id,
    start_station_name, 0,0, start_station_latitude, start_station_longitude,
    count(*), avg(tripduration), substr(starttime,1,10), start_station_id,
    end_station_id
from rawdata
group by
    start_station_id, end_station_id, substr(starttime,1,10);
```

We also need an end station row.

```
insert into paths_by_month
select
    start_station_id || "-" || end_station_id, end_station_id, end_station_name,
    avg(tripduration), count(*), end_station_latitude, end_station_longitude,
    count(*), avg(tripduration), substr(starttime,1,7), start_station_id,
    end_station_id
from rawdata
group by
    start_station_id, end_station_id, substr(starttime,1,7);
```

#### Clean data

There are multiple latitudes / longitudes for each station, which is causing
issues with the data quality. Running some queries to troubleshoot.

```
select * from (
select start_station_id, end_station_id, min(start_station_latitude) as min_lat,min(start_station_longitude) as min_long, max(start_station_latitude) as max_lat,max(start_station_longitude) as max_long
from rawdata
group by
    start_station_id, end_station_id, substr(starttime,1,7),
    start_station_name)
where min_lat <> max_lat or min_long <> max_long;

select start_station_id, start_station_latitude, start_station_longitude, count(*)
from rawdata
where start_station_id in ('262','521')
group by
    start_station_id, start_station_latitude, start_station_longitude;
```

Looks like these stations moved by about a block.

    262|40.6917823|-73.9737299|1633
    262|40.69178232|-73.97372989|5083
    521|40.75044999|-73.99481051|80395
    521|40.75096735|-73.99444208|20155

```
update paths
set latitude = '40.69178232', longitude = '-73.97372989'
where station_id = '262';

update paths
set latitude = '40.75044999', longitude = '-73.99481051'
where station_id = '521';
```

####Flows
In order to avoid duplicating the main data source (CitiBike.csv), I need to add a new table.

```
create table flows(station_id, day, trips_to, trips_from);

insert into flows
select ss.station_id, ss.day, ifnull(ss.count,0), ifnull(es.count,0)
from (
  select
    start_station_id as station_id,
    count(start_station_id) as count,
    substr(starttime,1,10) as day
  from rawdata
  group by start_station_id, substr(starttime,1,10) ) ss
left join (
  select
    end_station_id as station_id,
    count(end_station_id) as count,
    substr(stoptime,1,10) as day
  from rawdata
  group by end_station_id, substr(stoptime,1,10) ) es
on ss.station_id = es.station_id and ss.day = es.day
UNION
select es.station_id, es.day, ifnull(ss.count,0), ifnull(es.count,0)
from (
  select
    end_station_id as station_id,
    count(end_station_id) as count,
    substr(stoptime,1,10) as day
  from rawdata
  group by end_station_id, substr(stoptime,1,10) ) es
left join (
  select
    start_station_id as station_id,
    count(start_station_id) as count,
    substr(starttime,1,10) as day
  from rawdata
  group by start_station_id, substr(starttime,1,10) ) ss
on ss.station_id = es.station_id and ss.day = es.day;
```
