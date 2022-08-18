##### Cyclistic Queries - BigQuery

```
WITH
  data AS 
  (
SELECT
  ride_id,
  member_casual,
  rideable_type,
  DATE_DIFF(ended_at, started_at, SECOND) AS length_of_trip_sec,
  FORMAT_DATE('%A', started_at) AS start_day_of_week,
  FORMAT_DATE('%T', started_at) AS start_time_of_day,
  started_at,
  ended_at,
  start_station_name,
  start_station_id,
  end_station_name,
  end_station_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng
FROM
  `case-study-projects-355820.CyclisticTripData.202*`
WHERE
  CAST(started_at AS DATE) BETWEEN '2021-06-01' AND '2022-05-31' AND
  started_at < ended_at
  )
SELECT
  ride_id,
  member_casual,
  rideable_type,
  length_of_trip_sec,
  ROUND((length_of_trip_sec/60), 3) AS length_of_trip_min,
  ROUND((length_of_trip_sec/86400), 3) AS length_of_trip_day,
  start_day_of_week,
  start_time_of_day,
  TIME_TRUNC(TIME(started_at), HOUR) AS hour,
  started_at,
  ended_at,
  start_station_name,
  start_station_id,
  end_station_name,
  end_station_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng
FROM
    `data`
ORDER BY
  length_of_trip_sec DESC
```

```
SELECT
  column_name, 
  COUNT(1) AS nulls_count
FROM
  `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05` AS data,
UNNEST(REGEXP_EXTRACT_ALL(TO_JSON_STRING(data), r'"(\w+)":null')) AS column_name
GROUP BY column_name
ORDER BY nulls_count DESC
```

```
SELECT
  member_casual,
  rideable_type,
  COUNT(member_casual) AS count,
  ROUND(SUM(length_of_trip_min),2) AS total_length_of_trip_min,
  ROUND(SUM(length_of_trip_day),2) AS total_length_of_trip_day
FROM
  `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05`
GROUP BY
  member_casual,
  rideable_type
ORDER BY
  total_length_of_trip_min
```

```
SELECT
  rideable_type,
  start_station_name,
  COUNT(*) AS count,
  SUM(COUNT(*)) OVER (PARTITION BY start_station_name) AS total_rides_by_station,
  SUM(COUNT(*)) OVER (PARTITION BY rideable_type) AS total_rides_by_bike_type,
  ROUND(SUM(length_of_trip_min),2) AS total_length_of_trip_min,
  ROUND(SUM(length_of_trip_day),2) AS total_length_of_trip_day
FROM
  `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05`
GROUP BY
  rideable_type,
  start_station_name
ORDER BY
  start_station_name, total_length_of_trip_min
```

```
SELECT
  member_casual,
  rideable_type,
  start_station_name,
  COUNT(*) AS rides,
  SUM(COUNT(1)) OVER (PARTITION BY start_station_name) AS total_rides_at_station,
  SUM(COUNT(1)) OVER (PARTITION BY start_station_name, rideable_type) AS total_rides_at_station_by_bike_type,
  SUM(COUNT(1)) OVER (PARTITION BY member_casual, rideable_type) AS total_rides_by_mem_cas_bike_type,
  ROUND(SUM(length_of_trip_min),2) AS total_length_of_trip_min,
  
  ROUND(SUM(length_of_trip_day),2) AS total_length_of_trip_day
FROM
  `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05`
GROUP BY
  member_casual,
  rideable_type,
  start_station_name
ORDER BY
  start_station_name, rideable_type
```

```
SELECT 
  member_casual,
  ROUND(AVG(length_of_trip_min),2) AS mean_length_of_trip_min,
  ROUND(AVG(length_of_trip_sec),2) AS mean_length_of_trip_sec,
  MAX(length_of_trip_min) AS max_length_of_trip_min,
  MAX(length_of_trip_day) AS max_length_of_trip_day
FROM 
  `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05` 
GROUP BY
  member_casual
```

```
WITH weekend AS
  (
    SELECT
      member_casual,
      rideable_type,
      COUNT(1) AS weekend_count
    FROM
      `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05`
    WHERE
      start_day_of_week = 'Saturday' OR start_day_of_week = 'Sunday'
    GROUP BY
      member_casual,
      rideable_type
  ),
weekday AS
  (
    SELECT  
      member_casual,
      rideable_type,
      COUNT(1) AS weekday_count
    FROM 
      `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05`
    WHERE
      start_day_of_week != 'Saturday' AND start_day_of_week != 'Sunday'
    GROUP BY
      member_casual,
      rideable_type
  )
SELECT  
  weekday.member_casual,
  weekday.rideable_type,
  weekday.weekday_count,
  weekend.weekend_count,
  (weekend.weekend_count + weekday.weekday_count) AS total_count,
  ROUND(weekday.weekday_count/(weekday.weekday_count + weekend.weekend_count), 2) AS weekday_pct,
  ROUND(weekend.weekend_count/(weekday.weekday_count + weekend.weekend_count), 2) AS weekend_pct
FROM 
  weekday
JOIN weekend
  ON weekday.member_casual = weekend.member_casual AND weekday.rideable_type = weekend.rideable_type
```

```
SELECT
  member_casual,
  rideable_type,
  SUM(COUNT(1)) OVER (PARTITION BY member_casual) AS total_rides_cas_mem,
  SUM(COUNT(1)) OVER (PARTITION BY rideable_type) AS total_rides_bike,
  (ROUND(SUM(length_of_trip_min)/60, 2)) AS total_hours,
  5824 AS bike_inv,
  COUNT(DISTINCT start_station_name) AS num_stations,
  1105 AS total_stations
FROM
  `case-study-projects-355820.CyclisticTripData_CaseStudy.1_project_data_2021-06_to_2022-05`
GROUP BY
  member_casual,
  rideable_type
```
