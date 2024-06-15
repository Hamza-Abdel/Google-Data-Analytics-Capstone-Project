### Tools used: 
* Excel
* DaxStudio
* MySQL

## First:
### I used Excel where instead of opening each individual file to clean. I used power query and did the following :
1) Downloaded Daxstudio.
2) Put all csv files into one folder.
3) opened up excel > Data tab > Get Data > From File > Choose file containing all the csv files > power query opens automatically > combine > filter rows with missing values > Save & Load to... > check "only create connection" and "Add to data model" > Add-ins tab > DaxStudio > check Power Query > OK > Advanced > Export Data > As CSV

## Second:
### instead of using Import wizard in MySQL which would have taken quite a long time, I did the following:
1) Create empty table with the same columns as in the CSV file.
2) Move the CSV file into the Uploads folder ( which you will usually find through this path 'C:\ProgramData\MySQL\MySQL Server 8.0\Uploads').
3) Used the following propmpt:

```SQL

LOAD DATA INFILE 'file-path' INTO TABLE 2023_trip_data
FIELDS TERMINATED BY ','
IGNORE 1 LINES;

```

4) I created the following (which aren't documented in the code) extra columns:
  * start_time
  * end_time
  * trip_duration_min
  * start_date
  * end_date
  * weekday
  * season
  * month

## Now the data is ready to be further Processed using SQL:

``` SQL

-- Calculate Trip Frequency for casual vs members

SELECT member_casual, COUNT(*) AS trip_count
FROM 2023_trip_data
GROUP BY member_casual;


-- Avg trip duration for members vs casuals

SELECT member_casual, AVG(trip_duration_minutes) AS avg_trip_duration
FROM 2023_trip_data
GROUP BY member_casual;



-- Average activity for hours of the day


SELECT
	member_casual,
    EXTRACT(HOUR FROM start_time) AS hour_of_day,
    COUNT(*) AS ride_count
FROM
	2023_trip_data
GROUP BY
	member_casual, hour_of_day
ORDER BY
	member_casual, hour_of_day;
    

-- Analyze usage by time of day

SELECT
	member_casual,
    CASE
		WHEN EXTRACT(HOUR FROM start_time) BETWEEN 0 AND 11 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM start_time) BETWEEN 12 AND 17 THEN 'Afternoon'
		WHEN EXTRACT(HOUR FROM start_time) BETWEEN 18 AND 23 THEN 'Evening'
	END AS time_of_day,
    COUNT(*) AS ride_count
FROM
	2023_trip_data
GROUP BY
	member_casual, time_of_day
ORDER BY
	member_casual, time_of_day;

-- Ride Type

SELECT
	member_casual,
    rideable_type,
    COUNT(*) AS ride_count
FROM
	2023_trip_data
GROUP BY
	member_casual, rideable_type
ORDER BY
	member_casual, rideable_type;

    
-- day of the week analysis

SELECT
	member_casual,
    weekday,
    COUNT(*) AS ride_count
FROM
	2023_trip_data
GROUP BY
	member_casual, weekday
order by
	member_casual, weekday;
    
-- seasonality analysis

SELECT
	member_casual,
    season,
    COUNT(*) AS ride_count
FROM
	2023_trip_data
GROUP BY
	member_casual, season
ORDER BY
	member_casual, season;

-- by months

ALTER TABLE 2023_trip_data
ADD month text(45);

UPDATE 2023_trip_data
SET month = MONTHNAME(start_date);

SELECT
	month,
    member_casual,
    COUNT(*) AS ride_count
FROM
	2023_trip_data
GROUP BY
	month,
    member_casual;

```
