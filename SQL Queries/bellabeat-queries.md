# Bellabeat Queries
### PostgreSQL in DBeaver

##### List all columns from dataset; Check to see which column names are shared across tables
```
SELECT 
	c."column_name",
 	COUNT(table_name) 
FROM 
	information_schema."columns" c 
WHERE
	table_schema = 'public' --change
GROUP BY 
	1;
```

##### Check if column is in every table
```
 SELECT
 	"table_name",
 	SUM(CASE
     		WHEN column_name = 'Id' THEN 1 --change
   			ELSE
   			0
 		END
   	) AS has_id_column
FROM
 	information_schema."columns" c 
WHERE 
 	table_schema = 'public' --change
GROUP BY
 1
ORDER BY
 1 ASC;
``` 
 
##### Table, column, data type info
```
SELECT
	table_name,
    column_name,
    data_type
FROM
    information_schema.columns
WHERE
    table_schema = 'public';
```    
    
##### This query checks to make sure that each table has a column of a date or time related type
##### If your column types were detected properly prior to upload this table should be empty if HAVING = 0
```
SELECT
	table_name,
 	SUM(CASE
     	WHEN data_type IN ('date','timestamp without time zone','TIMESTAMP', 'DATETIME', 'TIME', 'DATE') THEN 1
	   ELSE
	   0
 	END
   	) AS "has_time_info"
FROM
	information_schema."columns" c 
WHERE
	table_schema = 'public' 
	AND
	data_type IN ('date','timestamp without time zone','TIMESTAMP',
	   'DATETIME',
	   'DATE')
GROUP BY
 	1
HAVING
 	SUM(CASE
     	WHEN data_type IN ('date','timestamp without time zone','TIMESTAMP', 'DATETIME', 'TIME', 'DATE') THEN 1
	   ELSE
	   0
 	END
   	) = 1;
```   
   
##### If we found that we have columns of the type DATETIME, TIMESTAMP, or DATE we can use this query to check for their names
```
SELECT
	CONCAT(table_catalog,'.',table_schema,'.',table_name) AS table_path,
	table_name,
	column_name,
	data_type 
FROM
 	information_schema."columns" c 
WHERE
	data_type IN ('date','timestamp without time zone','TIMESTAMP',
	   'DATETIME',
	   'DATE');
```

##### We now know that every table has an "Id" column but we don't know how to join the dates
##### If we find that not every table has a DATETIME, TIMESTAMP, or DATE column we use their names to check for what might be date-related
##### Here we check to see if the column name has any of the keywords below:
##### date, minute, daily, hourly, day, seconds
```
SELECT
	table_name,
	column_name,
	data_type
FROM
 	information_schema."columns" c 
WHERE
	LOWER(column_name) ~ 'date|minute|daily|hourly|day|seconds'
	AND table_schema = 'public';	  
```
	
##### ADVANCED
##### In the dailyActivity_merged table we saw that there is a column called ActivityDate, let's check to see what it looks like
##### One way to check if something follows a particular pattern is to use a regular expression.
##### In this case we use the regular expression for a timestamp format to check if the column follows that pattern.
##### The is_timestamp column demonstrates that this column is a valid timestamp column
```
SELECT
 "ActivityDate",
 CASE WHEN "ActivityDate"::VARCHAR(50) ~ '^\d{4}-\d{1,2}-\d{1,2}[T ]\d{1,2}:\d{1,2}:\d{1,2}(\.\d{1,6})? *(([+-]\d{1,2}(:\d{1,2})?)|Z|UTC)?$' THEN 'TRUE' ELSE 'FALSE' END AS is_timestamp
FROM
	dailyactivity_merged dm  
LIMIT
 10;
```
 
##### To quickly check if all columns follow the timestamp pattern we can take the minimum value of the boolean expression across the entire table
```
SELECT
 CASE
   WHEN MIN(CASE WHEN "ActivityHour"::VARCHAR(50) ~ '^\d{4}-\d{1,2}-\d{1,2}[T ]\d{1,2}:\d{1,2}:\d{1,2}(\.\d{1,6})? *(([+-]\d{1,2}(:\d{1,2})?)|Z|UTC)?$' THEN 1 ELSE 0 END)>0 THEN 'Valid'
 ELSE
 'Not Valid'
END
 AS valid_test
FROM
 hourlyintensities_merged hm ;
```

##### Say we want to do an analysis based upon daily data, this could help us to find tables that might be at the day level
```
SELECT
 DISTINCT table_name
FROM
 information_schema."columns" c 
WHERE
 LOWER(table_name) ~ 'day|daily';
```

##### Now that we have a list of tables we should look at the columns that are shared among the tables
```
SELECT
 column_name,
 data_type,
 COUNT(table_name) AS table_count
FROM
 information_schema."columns" c 
WHERE
 LOWER(table_name) ~ 'day|daily'
GROUP BY
 1,
 2;
```

##### Now that we have a list of tables we should look at the columns that are shared among the tables
##### We should also make certain that the data types align between tables
```
SELECT
 column_name,
 table_name,
 data_type
FROM
 information_schema."columns" c 
WHERE
 LOWER(table_name) ~ 'day|daily'
 AND column_name IN (
 SELECT
   column_name
 FROM
   information_schema."columns" c 
 WHERE
   LOWER(table_name) ~ 'day|daily'
 GROUP BY
   1
 HAVING
   COUNT(table_name) >=2)
ORDER BY
 1;
SELECT
 A."Id",
 A."Calories",
 A."ActivityDate",
 A."TotalSteps",
 A."TotalDistance",
 A."TrackerDistance",
 A."LoggedActivitiesDistance",
 I."SedentaryMinutes",
 I."LightlyActiveMinutes",
 I."FairlyActiveMinutes",
 I."VeryActiveMinutes",
 I."SedentaryActiveDistance",
 I."LightActiveDistance",
 I."ModeratelyActiveDistance",
 I."VeryActiveDistance",
 S."StepTotal",
 Sl."SleepDay",
 Sl."TotalSleepRecords",
 Sl."TotalMinutesAsleep",
 Sl."TotalTimeInBed" 
FROM
 dailyactivity_merged A
LEFT JOIN
 dailycalories_merged C
ON
 A."Id" = C."Id"
 AND A."ActivityDate"=C."ActivityDay"
 AND A."Calories" = C."Calories"
LEFT JOIN
 dailyintensities_merged I
ON
 A."Id" = I."Id"
 AND A."ActivityDate"=I."ActivityDay"
 AND A."FairlyActiveMinutes" = I."FairlyActiveMinutes"
 AND A."LightActiveDistance" = I."LightActiveDistance"
 AND A."LightlyActiveMinutes" = I."LightlyActiveMinutes"
 AND A."ModeratelyActiveDistance" = I."ModeratelyActiveDistance"
 AND A."SedentaryActiveDistance" = I."SedentaryActiveDistance"
 AND A."SedentaryMinutes" = I."SedentaryMinutes"
 AND A."VeryActiveDistance" = I."VeryActiveDistance"
 AND A."VeryActiveMinutes" = I."VeryActiveMinutes"
LEFT JOIN
 dailysteps_merged S
ON
 A."Id" = S."Id"
 AND A."ActivityDate"=S."ActivityDay"
LEFT JOIN
 sleepday_merged Sl
ON
 A."Id" = Sl."Id"
 AND A."ActivityDate"=Sl."SleepDay";
```

##### Say we are considering sleep related products as a possibility, let's take a moment to see if/ how people nap during the day
##### To do this we are assuming that a nap is any time someone sleeps but goes to sleep and wakes up on the same day
```
SELECT
 "Id",
 "sleep_start" AS sleep_date,
 COUNT("logId") AS number_naps,
 SUM(EXTRACT(HOUR
   FROM
     "time_sleeping")) AS total_time_sleeping
FROM 
(
 SELECT
   "Id",
   "logId",
   MIN("date"::TIMESTAMP WITHOUT TIME ZONE) AS sleep_start,
   MAX("date"::TIMESTAMP WITHOUT TIME ZONE) AS sleep_end,
   (EXTRACT(HOUR FROM (MAX("date") - MIN("date")))::VARCHAR(2)||':'||
     (MOD(EXTRACT(MINUTE FROM (MAX("date") - MIN("date"))),60))::VARCHAR(2)||':'||
     ((MOD(MOD(EXTRACT(SECOND FROM (MAX("date") - MIN("date"))),3600),60))::INT2)::VARCHAR(2) )::TIME AS time_sleeping
 FROM
   minutesleep_merged mm
 WHERE
   value=1
 GROUP BY
   1,
   2
) AS mmm
WHERE
 ("sleep_start"::DATE)=("sleep_end"::DATE)
GROUP BY
 1,
 2
ORDER BY
 3 DESC;
```
 
##### Suppose we would like to do an analysis based upon the time of day and day of the week
##### We will do this at a person level such that we smooth over anomalous days for an individual
```
WITH
 user_dow_summary AS (
 SELECT
   "Id",
   EXTRACT(DOW FROM "ActivityHour") AS dow_number,
   TRIM(TO_CHAR(DATE ("ActivityHour"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "ActivityHour") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "ActivityHour") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   (CASE
     WHEN ("ActivityHour"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
     WHEN ("ActivityHour"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
     WHEN ("ActivityHour"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
     WHEN ("ActivityHour"::TIME) >= ('21:00:00'::TIME) OR (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('06:00:00'::TIME) THEN 'Night'
   ELSE
   'ERROR'
 	END)
   AS time_of_day,
   SUM("TotalIntensity") AS total_intensity,
   SUM("AverageIntensity") AS total_average_intensity,
   AVG("AverageIntensity") AS average_intensity,
   MAX("AverageIntensity") AS max_intensity,
   MIN("AverageIntensity") AS min_intensity
 FROM
   "hourlyintensities_merged"
 GROUP BY
   1,
   2,
   3,
   4,
   5),
 intensity_deciles AS (
 SELECT
   DISTINCT "dow_number",
   "part_of_week",
   "day_of_week",
   "time_of_day",
   PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensity_first_decile",
   PERCENTILE_CONT(0.2) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensity_second_decile",
   PERCENTILE_CONT(0.3) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensity_third_decile",
   PERCENTILE_CONT(0.4) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensit_fourth_decile",
   PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensit_fifth_decile",   
   PERCENTILE_CONT(0.6) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensity_sixth_decile",
   PERCENTILE_CONT(0.7) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensity_seventh_decile",
   PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensity_eigth_decile",
   PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensity_ninth_decile",
   PERCENTILE_CONT(1.0) WITHIN GROUP (ORDER BY "total_intensity") AS "total_intensit_tenth_decile"  
 FROM
   "user_dow_summary" 
 GROUP BY
 	"dow_number", "part_of_week", "day_of_week", "time_of_day"
 ),
 basic_summary AS (
 SELECT
   "part_of_week",
   "day_of_week",
   "time_of_day",
   SUM("total_intensity") AS total_total_intensity,
   AVG("total_intensity") AS average_total_intensity,
   SUM("total_average_intensity") AS total_total_average_intensity,
   AVG("total_average_intensity") AS average_total_average_intensity,
   SUM("average_intensity") AS total_average_intensity,
   AVG("average_intensity") AS average_average_intensity,
   AVG("max_intensity") AS average_max_intensity,
   AVG("min_intensity") AS average_min_intensity
 FROM
   "user_dow_summary"
 GROUP BY
   1,
   "dow_number",
   2,
   3)
SELECT
 	"part_of_week",
 	"dow_number",
 	"day_of_week",
 	"time_of_day",
 	ROUND("total_total_intensity", 2) AS total_intensity,
 	ROUND("average_total_intensity", 2) AS avg_total_intensity_per_person,
 	ROUND("average_average_intensity"::NUMERIC, 2) AS average_intensity_per_hour,
 	ROUND("average_max_intensity"::NUMERIC, 2) AS avg_max_intensity_per_hour,
 	ROUND("average_min_intensity"::NUMERIC, 2) AS avg_min_intensity_per_hour,
 	ROUND("total_intensity_first_decile"::NUMERIC, 2) AS total_intensity_10th_percentile,
   	ROUND("total_intensity_second_decile"::NUMERIC, 2) AS total_intensity_20th_percentile,
   	ROUND("total_intensity_third_decile"::NUMERIC, 2) AS total_intensity_30th_percentile,
   	ROUND("total_intensit_fourth_decile"::NUMERIC, 2) AS total_intensity_40th_percentile,
   	ROUND("total_intensit_fifth_decile"::NUMERIC, 2) AS total_intensity_50th_percentile,   
   	ROUND("total_intensity_sixth_decile"::NUMERIC, 2) AS total_intensity_60th_percentile,
   	ROUND("total_intensity_seventh_decile"::NUMERIC, 2) AS total_intensity_70th_percentile,
   	ROUND("total_intensity_eigth_decile"::NUMERIC, 2) AS total_intensity_80th_percentile,
   	ROUND("total_intensity_ninth_decile"::NUMERIC, 2) AS total_intensity_90th_percentile,
   	ROUND("total_intensit_tenth_decile"::NUMERIC, 2)  AS total_intensity_100th_percentile 
FROM
 "basic_summary"
LEFT JOIN
 "intensity_deciles"
USING
 ("part_of_week",
   "day_of_week",
   "time_of_day")
ORDER BY
 "dow_number",
 "day_of_week",
 CASE
   WHEN "time_of_day" = 'Morning' THEN 0
   WHEN "time_of_day" = 'Afternoon' THEN 1
   WHEN "time_of_day" = 'Evening' THEN 2
   WHEN "time_of_day" = 'Night' THEN 3
END
 ;
```
 
##### custom
##### sum of METs grouped by day, time of day
``` WITH x AS (
SELECT
   "Id",
   TO_CHAR("ActivityMinute", 'YYYY-MM-dd') AS date,
   EXTRACT(DOW FROM "ActivityMinute") AS dow_number,
   TRIM(TO_CHAR(DATE ("ActivityMinute"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "ActivityMinute") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "ActivityMinute") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   (CASE
     WHEN ("ActivityMinute"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
     WHEN ("ActivityMinute"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
     WHEN ("ActivityMinute"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
     WHEN ("ActivityMinute"::TIME) >= ('21:00:00'::TIME) OR (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('06:00:00'::TIME) THEN 'Night'
   ELSE
   'ERROR'
 	END)
   AS time_of_day,
   "METs"
FROM
	minutemetsnarrow_merged mm 
)
SELECT 
	"dow_number",
	"day_of_week",
	"part_of_week",
	"time_of_day",
	ROUND(AVG(NULLIF("METs", 0)),2) AS avg_METs_per_min,
	SUM("METs") AS total_METs
FROM 
	x
GROUP BY
	1,
	2,
	3,
	4
ORDER BY 
	"dow_number",
	CASE WHEN "time_of_day" = 'Morning' THEN 0
   		WHEN "time_of_day" = 'Afternoon' THEN 1
   		WHEN "time_of_day" = 'Evening' THEN 2
   		WHEN "time_of_day" = 'Night' THEN 3
	END;
```
	
##### custom
##### average heartrate group by day, time of day
```
WITH x AS (
SELECT
   "Id",
   "Value",
   TO_CHAR("Time", 'YYYY-MM-dd') AS date,
   EXTRACT(DOW FROM "Time") AS dow_number,
   TRIM(TO_CHAR(DATE ("Time"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "Time") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "Time") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   (CASE
     WHEN ("Time"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "Time")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
     WHEN ("Time"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "Time")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
     WHEN ("Time"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "Time")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
     WHEN ("Time"::TIME) >= ('21:00:00'::TIME) OR (DATE_TRUNC('Minute', "Time")::TIME) < ('06:00:00'::TIME) THEN 'Night'
   ELSE
   'ERROR'
 	END)
   AS time_of_day,
   "Value" AS avg_heartrate
FROM
	heartrate_seconds_merged hsm 
)
SELECT
	"dow_number",
	"day_of_week",
	"part_of_week",
	"time_of_day",
	ROUND(AVG("Value"),2) AS avg_heartrate_per_min
FROM 
	x
GROUP BY
	"dow_number",
	"day_of_week",
	"part_of_week",
	"time_of_day"
ORDER BY 
	"dow_number",
	CASE WHEN "time_of_day" = 'Morning' THEN 0
   		WHEN "time_of_day" = 'Afternoon' THEN 1
   		WHEN "time_of_day" = 'Evening' THEN 2
   		WHEN "time_of_day" = 'Night' THEN 3
	END;
```
	
##### custom
##### total number/time of naps group by day, time of day
WITH x AS
```
(
 SELECT
   "Id",
   "logId",
   "date",
   "value",
   EXTRACT(DOW FROM "date") AS dow_number,
   TRIM(TO_CHAR(DATE ("date"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "date") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "date") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   (CASE
     WHEN ("date"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "date")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
     WHEN ("date"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "date")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
     WHEN ("date"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "date")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
     WHEN ("date"::TIME) >= ('21:00:00'::TIME) OR (DATE_TRUNC('Minute', "date")::TIME) < ('06:00:00'::TIME) THEN 'Night'
   ELSE
   'ERROR'
 	END)
   AS time_of_day
 FROM
   minutesleep_merged mm
)
SELECT
 dow_number,
 day_of_week,
 part_of_week,
 time_of_day,
 (EXTRACT(HOUR FROM (MAX("date") - MIN("date")))::VARCHAR(2)||':'||
     (MOD(EXTRACT(MINUTE FROM (MAX("date") - MIN("date"))),60))::VARCHAR(2)||':'||
     ((MOD(MOD(EXTRACT(SECOND FROM (MAX("date") - MIN("date"))),3600),60))::INT2)::VARCHAR(2) )::TIME AS time_sleeping,
 COUNT(DISTINCT "logId") AS total_number_naps
FROM x
GROUP BY
 1,
 2,
 3,
 4
ORDER BY 
	"dow_number",
	CASE WHEN "time_of_day" = 'Morning' THEN 0
   		WHEN "time_of_day" = 'Afternoon' THEN 1
   		WHEN "time_of_day" = 'Evening' THEN 2
   		WHEN "time_of_day" = 'Night' THEN 3
	END;
```

##### custom - sleep enhanced
```
SELECT
 "Id",
 "sleep_start" AS sleep_date,
 EXTRACT(DOW FROM "sleep_start") AS dow_number,
TRIM(TO_CHAR(DATE ("sleep_start"::DATE), 'Day')) AS day_of_week,
(CASE
   WHEN EXTRACT(DOW FROM "sleep_start") IN (0, 6) THEN 'Weekend'
   WHEN EXTRACT(DOW FROM "sleep_start") NOT IN (0, 6) THEN 'Weekday'
   ELSE 'ERROR'
 END) AS part_of_week,
(CASE
   WHEN ("sleep_start"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "sleep_start")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
   WHEN ("sleep_start"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "sleep_start")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
   WHEN ("sleep_start"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "sleep_start")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
   WHEN ("sleep_start"::TIME) >= ('21:00:00'::TIME) THEN 'Night'
   ELSE 'Past Midnight'
 END) AS time_of_day,
 COUNT("logId") AS number_naps,
 SUM(EXTRACT(HOUR
   FROM
     "time_sleeping")) AS total_time_sleeping
FROM 
(
 SELECT
   "Id",
   "logId",
   MIN("date"::TIMESTAMP WITHOUT TIME ZONE) AS sleep_start,
   MAX("date"::TIMESTAMP WITHOUT TIME ZONE) AS sleep_end,
   (EXTRACT(HOUR FROM (MAX("date") - MIN("date")))::VARCHAR(2)||':'||
     (MOD(EXTRACT(MINUTE FROM (MAX("date") - MIN("date"))),60))::VARCHAR(2)||':'||
     ((MOD(MOD(EXTRACT(SECOND FROM (MAX("date") - MIN("date"))),3600),60))::INT2)::VARCHAR(2) )::TIME AS time_sleeping
 FROM
   minutesleep_merged mm
 WHERE
   value=1
 GROUP BY
   1,
   2
) AS mmm
WHERE
 ("sleep_start"::DATE)=("sleep_end"::DATE)
GROUP BY
 1,
 2,
 3,
 4,
 5,
 6;
```
 
##### custom
##### asleep vs not asleep in bed,, group by day
```
WITH x AS
(
 SELECT
   "Id",
   "TotalSleepRecords",
   "TotalMinutesAsleep",
   "TotalTimeInBed",
   ("TotalMinutesAsleep"/"TotalTimeInBed"::FLOAT) AS Pct_Time_Sleeping_In_Bed,
   EXTRACT(DOW FROM "SleepDay") AS dow_number,
   TRIM(TO_CHAR(DATE ("SleepDay"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "SleepDay") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "SleepDay") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week
 FROM
   sleepday_merged sm 
)
SELECT
 dow_number,
 day_of_week,
 part_of_week,
 ROUND(SUM("TotalSleepRecords"),2) AS Total_Sleep_Records,
 ROUND(AVG("TotalSleepRecords"),2) AS Avg_Sleep_Records,
 ROUND(AVG("TotalMinutesAsleep"),2) AS Avg_Minutes_Asleep_In_Bed,
 ROUND(AVG("TotalTimeInBed"),2) AS Avg_Minutes_In_Bed,
 ROUND(AVG("TotalTimeInBed"-"TotalMinutesAsleep"),2) AS Avg_Minutes_Not_Sleeping_In_Bed,
 ROUND((AVG(Pct_Time_Sleeping_In_Bed))::NUMERIC,2) AS Pct_Time_Sleeping_In_Bed
FROM x
GROUP BY
 1,
 2,
 3
ORDER BY 
	"dow_number"
```

##### custom
##### total calories group by day, time of day
```
WITH x AS (
SELECT
   "Id",
   TO_CHAR("ActivityMinute", 'YYYY-MM-dd') AS date,
   EXTRACT(DOW FROM "ActivityMinute") AS dow_number,
   TRIM(TO_CHAR(DATE ("ActivityMinute"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "ActivityMinute") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "ActivityMinute") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   (CASE
     WHEN ("ActivityMinute"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
     WHEN ("ActivityMinute"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
     WHEN ("ActivityMinute"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
     WHEN ("ActivityMinute"::TIME) >= ('21:00:00'::TIME) OR (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('06:00:00'::TIME) THEN 'Night'
   ELSE
   'ERROR'
 	END)
   AS time_of_day,
   "Calories"
FROM
	minutecaloriesnarrow_merged mc
)
SELECT 
	"dow_number",
	"day_of_week",
	"part_of_week",
	"time_of_day",
	AVG("Calories") AS avg_calories_per_min,
	SUM("Calories") AS total_calories
FROM 
	x
GROUP BY
	1,
	2,
	3,
	4
ORDER BY 
	"dow_number",
	CASE WHEN "time_of_day" = 'Morning' THEN 0
   		WHEN "time_of_day" = 'Afternoon' THEN 1
   		WHEN "time_of_day" = 'Evening' THEN 2
   		WHEN "time_of_day" = 'Night' THEN 3
	END;
```
	
##### custom
##### total steps group by day, time of day
```
WITH x AS (
SELECT
   "Id",
   EXTRACT(DOW FROM "ActivityMinute") AS dow_number,
   TRIM(TO_CHAR(DATE ("ActivityMinute"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "ActivityMinute") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "ActivityMinute") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   (CASE
     WHEN ("ActivityMinute"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
     WHEN ("ActivityMinute"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
     WHEN ("ActivityMinute"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
     WHEN ("ActivityMinute"::TIME) >= ('21:00:00'::TIME) OR (DATE_TRUNC('Minute', "ActivityMinute")::TIME) < ('06:00:00'::TIME) THEN 'Night'
   ELSE
   'ERROR'
 	END)
   AS time_of_day,
   "Steps"
FROM
	minutestepsnarrow_merged msn
)
SELECT 
	"dow_number",
	"day_of_week",
	"part_of_week",
	"time_of_day",
	AVG("Steps") AS avg_steps_per_min,
	SUM("Steps") AS total_steps
FROM 
	x
GROUP BY
	1,
	2,
	3,
	4
ORDER BY 
	"dow_number",
	CASE WHEN "time_of_day" = 'Morning' THEN 0
   		WHEN "time_of_day" = 'Afternoon' THEN 1
   		WHEN "time_of_day" = 'Evening' THEN 2
   		WHEN "time_of_day" = 'Night' THEN 3
	END;
```
	
##### custom
##### intensities grouped by id, day of week, time of day
```
WITH x AS
(
SELECT
   "Id",
   EXTRACT(DOW FROM "ActivityHour") AS dow_number,
   TRIM(TO_CHAR(DATE ("ActivityHour"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "ActivityHour") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "ActivityHour") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   (CASE
     WHEN ("ActivityHour"::TIME) >= ('06:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('12:00:00'::TIME) THEN 'Morning'
     WHEN ("ActivityHour"::TIME) >= ('12:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('18:00:00'::TIME) THEN 'Afternoon'
     WHEN ("ActivityHour"::TIME) >= ('18:00:00'::TIME) AND (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('21:00:00'::TIME) THEN 'Evening'
     WHEN ("ActivityHour"::TIME) >= ('21:00:00'::TIME) OR (DATE_TRUNC('Minute', "ActivityHour")::TIME) < ('06:00:00'::TIME) THEN 'Night'
   ELSE
   'ERROR'
 	END)
   AS time_of_day,
   SUM("TotalIntensity") AS total_intensity,
   SUM("AverageIntensity") AS total_average_intensity,
   AVG("AverageIntensity") AS average_intensity,
   MAX("AverageIntensity") AS max_intensity,
   MIN("AverageIntensity") AS min_intensity
 FROM
   "hourlyintensities_merged"
 GROUP BY
   1,
   2,
   3,
   4,
   5
)
SELECT 
	"Id",
	"dow_number",
	"day_of_week",
	"part_of_week",
	"time_of_day",
	"total_intensity",
	ROUND(total_average_intensity::NUMERIC, 2) AS total_average_intensity,
	ROUND(average_intensity::NUMERIC, 2) AS average_intensity,
	ROUND(max_intensity::NUMERIC, 2) AS max_intensity,
	ROUND(min_intensity::NUMERIC, 2) AS min_intensity
FROM x
ORDER BY 
	"Id",
	"dow_number",
	CASE
   WHEN "time_of_day" = 'Morning' THEN 0
   WHEN "time_of_day" = 'Afternoon' THEN 1
   WHEN "time_of_day" = 'Evening' THEN 2
   WHEN "time_of_day" = 'Night' THEN 3
END;
```

##### custom
##### distance grouped by day of week
```
WITH x AS (
SELECT
   "Id",
   EXTRACT(DOW FROM "ActivityDay") AS dow_number,
   TRIM(TO_CHAR(DATE ("ActivityDay"::DATE), 'Day')) AS day_of_week,
   (CASE
     WHEN EXTRACT(DOW FROM "ActivityDay") IN (0, 6) THEN 'Weekend'
     WHEN EXTRACT(DOW FROM "ActivityDay") NOT IN (0, 6) THEN 'Weekday'
   ELSE
   'ERROR'
   END)
   AS part_of_week,
   "SedentaryMinutes",
   "LightlyActiveMinutes",
   "FairlyActiveMinutes",
   "VeryActiveMinutes",
   "SedentaryActiveDistance",
   "LightActiveDistance",
   "ModeratelyActiveDistance",
   "VeryActiveDistance"
FROM
	dailyintensities_merged di
)
SELECT 
	"dow_number",
	"day_of_week",
	"part_of_week",
	ROUND(AVG("SedentaryMinutes"),2) AS "Avg_Sedentary_Min",
	SUM("SedentaryMinutes") AS "Total_Sedentary_Min",
	ROUND(AVG("LightlyActiveMinutes"),2) AS "Avg_Lightly_Active_Min",
	SUM("LightlyActiveMinutes") AS "Total_Lightly_Active_Min",
	ROUND(AVG("FairlyActiveMinutes"),2) AS "Avg_Fairly_Active_Min",
	SUM("FairlyActiveMinutes") AS "Total_Fairly_Active_Min",
	ROUND(AVG("VeryActiveMinutes"),2) AS "Avg_Very_Active_Min",
	SUM("VeryActiveMinutes") AS "Total_Very_Active_Min",
	ROUND(AVG("LightActiveDistance")::NUMERIC,2) AS "Avg_Light_Active_Distance",
	ROUND(SUM("LightActiveDistance")::NUMERIC,2) AS "Total_Light_Active_Distance",
	ROUND(AVG("ModeratelyActiveDistance")::NUMERIC,2) AS "Avg_Moderately_Active_Distance",
	ROUND(SUM("ModeratelyActiveDistance")::NUMERIC,2) AS "Total_Moderately_Active_Distance",	
	ROUND(AVG("VeryActiveDistance")::NUMERIC,2) AS "Avg_Very_Active_Distance",
	ROUND(SUM("VeryActiveDistance")::NUMERIC,2) AS "Total_Very_Active_Distance"
FROM 
	x
GROUP BY
	1,
	2,
	3
ORDER BY 
	"dow_number";
```
