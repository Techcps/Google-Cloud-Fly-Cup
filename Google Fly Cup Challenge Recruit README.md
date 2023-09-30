
# Google Fly Cup Challenge: Recruit

* In the GCP Console open the Cloud Shell and enter the following commands:

```
for file in `gsutil ls gs://spls/gsp394/tables/*.csv`; do TABLE_NAME=`echo $file | cut -d '/' -f6 | cut -d '.' -f1`; bq load --autodetect --source_format=CSV --replace=true drl.$TABLE_NAME $file; done
```

#Task 1:

* Replace City
```
SELECT name FROM `drl.events` WHERE city = 'ENTER_YOUR_CITY_NAME'
```

#Task 2:

```
SELECT `drl.pilots`.name, `drl.event_pilots`.id FROM `drl.event_pilots` LEFT JOIN `drl.pilots` ON `drl.pilots`.id = `drl.event_pilots`.pilot_id
```

#Task 3:

* Replace Event Name
```
SELECT `drl.pilots`.name, `drl.events`.name AS event_name FROM `drl.event_pilots` LEFT OUTER JOIN `drl.pilots` ON `drl.pilots`.id = `drl.event_pilots`.pilot_id LEFT OUTER JOIN `drl.events` ON `drl.events`.id = `drl.event_pilots`.event_id WHERE `drl.events`.name = 'ENTER_YOUR_EVENT_NAME'
```

#Task 4:

```
WITH cte AS (SELECT `drl.round_standings`.minimum_time FROM `drl.round_standings` WHERE `rank` = 1)

SELECT time

(timestamp_seconds

(CAST

  (AVG

    (UNIX_SECONDS

      (PARSE_TIMESTAMP('%H:%M.%S', minimum_time))

    )

AS INT64)

)

)

AS avg FROM cte
```

#Task 5:

```
CREATE TABLE drl.time_trial_cleaned AS (

SELECT

`drl.time_trial_group_pilot_times`.id AS time_trial_group_pilot_times_id,

`drl.time_trial_group_pilots`.id AS time_trial_group_pilot_id,

`drl.time_trial_groups`.id AS time_trial_group_id,

round_id,

CASE

WHEN `drl.time_trial_group_pilot_times`.time_adjusted IS NOT null then `drl.time_trial_group_pilot_times`.time_adjusted

WHEN `drl.time_trial_groups`.racestack_scoring = 0 then `drl.time_trial_group_pilot_times`.time
ELSE`drl.time_trial_group_pilot_times`.racestack_time

END

AS time

FROM `drl.time_trial_group_pilot_times` LEFT OUTER JOIN `drl.time_trial_group_pilots` ON `drl.time_trial_group_pilot_times`.time_trial_group_pilot_id = `drl.time_trial_group_pilots`.id LEFT OUTER JOIN `drl.time_trial_groups` ON `drl.time_trial_group_pilots`.time_trial_group_id = `drl.time_trial_groups`.id

)
```

#Task 6:

* Replace Event Name

```
WITH cte AS

(SELECT

`drl.rounds`.event_id,

`drl.rounds`.name,

`drl.events`.name AS event_name,

time

FROM `drl.time_trial_cleaned`

LEFT OUTER JOIN `drl.rounds` ON `drl.time_trial_cleaned`.round_id = `drl.rounds`.id

LEFT OUTER JOIN `drl.events` ON `drl.events`.id = `drl.rounds`.event_id)
SELECT MIN(time) as fastest_time FROM cte WHERE event_name = 'ENTER_YOUR_EVENT_NAME' AND name = 'Time Trials'
```
#Task 7:

* Replace Name

```
SELECT

`drl.pilots`.name AS pilot_name,

`drl.heat_standings`.heat_id AS heat_id,

`drl.heat_standings`.minimum_time,

`drl.heat_standings`.points

FROM `drl.heat_standings`

LEFT JOIN `drl.event_pilots` ON `drl.event_pilots`.id = event_pilot_id

LEFT JOIN `drl.pilots` ON `drl.pilots`.id = `drl.event_pilots`.pilot_id

WHERE

name = 'ENTER_NAME'

AND

minimum_time != 'NURK'

AND

minimum_time != ''
```

#Task 8:

* Replace Name

```
WITH cte AS

(SELECT `drl.pilots`.name, `drl.heat_standings`.heat_id, `drl.heat_standings`.points, `drl.heat_standings`.minimum_time

FROM `drl.heat_standings`

LEFT JOIN `drl.event_pilots` ON `drl.event_pilots`.id = event_pilot_id

LEFT JOIN `drl.pilots` ON `drl.pilots`.id = `drl.event_pilots`.pilot_id
WHERE name = 'ENTER_NAME' AND minimum_time != 'DNF' AND minimum_time != '')

SELECT

name AS pilot_name,

heat_id

minimum_time,

points,

time

(timestamp_seconds

  (CAST

    (AVG

      (UNIX_SECONDS

        (PARSE_TIMESTAMP('%H:%M.%S', minimum_time))

      )

    OVER (ORDER BY heat_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

  AS INT64)

  )

)

AS running_avg

FROM cte
```

#Task 9:

* Replace Name

```
WITH cte AS

(SELECT

`drl.pilots`.name,

`drl.heat_standings`.heat_id,

`drl.heat_standings`.points,

`drl.heat_standings`.minimum_time

FROM `drl.heat_standings`

LEFT JOIN `drl.event_pilots` ON `drl.event_pilots`.id = event_pilot_id

LEFT JOIN `drl.pilots` ON `drl.pilots`.id = `drl.event_pilots`.pilot_id
WHERE name = 'ENTER_NAME' AND minimum_time != 'DNF' AND minimum_time != ''),

cte2 AS

(SELECT

name AS pilot_name,

heat_id,

minimum_time,

points,

time

(timestamp_seconds

 (CAST

   (AVG

     (UNIX_SECONDS

       (PARSE_TIMESTAMP('%H:%M.%S', minimum_time))

     )

   OVER (ORDER BY heat_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

 AS INT64)

 )

)

AS running_avg FROM cte)

SELECT *,

TIME_DIFF(PARSE_TIME('%H:%M.%S', minimum_time), running_avg, SECOND) as time_diff_from_avg FROM cte2
```


# Congratulations, you're all done with the lab 😄
# If you consider that the video helped you to complete your lab, so please do like and subscribe
# Thanks for watching :)