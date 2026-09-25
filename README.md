# Datomic

Database training, a project for 42Nice. Everything is done in a vm, this document are my notes, and reminder on SQL syntax.

## SQL

Query ordering
```SQL
SELECT ...
FROM ...
JOIN ... ON ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY
```

Selecting stuff

```SQL
SELECT column_name, other_column FROM table_name WHERE <condition>;
-- a condition can look like : 
-- column_name = "foobar"
-- column_name is NULL
-- and remember sql statements always end in ;
```

Ordering stuff
```SQL
SELECT colum_name, other_colum FROM table_name ORDER BY some_column;
-- there is other syntax for different ordering
```

Limmiting selections
```SQL
SELECT colum_name, other_colum FROM table_name ORDER BY some_column LIMIT 2;
-- ORDER BY come_column DESC
-- ASC is the deafult
```

Grouping stuff
```SQL
SELECT d.name, ROUND(AVG(c.mission_hours), 2) FROM crew as c
JOIN deck as d ON c.deck_id = d.id
GROUP BY d.id;
-- when grouping you need some sort of coalessing function in the select
-- bug with the thing, you need to round when they ask for average
```

Sub quiries
```SQL
SELECT c.callsign, c.uptime_pct FROM crew as c
JOIN 
(
	SELECT AVG(uptime_pct) as aup , module_id FROM crew WHERE module_id IS NOT null GROUP BY module_id
) as avgs 
ON c.module_id = avgs.module_id
WHERE avgs.aup < c.uptime_pct
;
```

Having
```SQL
SELECT s.name, COUNT(m.mission_id)
FROM crew c
JOIN squad s ON s.id = c.squad_id
JOIN mission_log m ON c.id = m.crew_id
GROUP BY s.name
HAVING COUNT(DISTINCT c.id) >= 6
ORDER BY COUNT
```

CTE (common table expression)
```SQL
WITH cte AS (
 SELECT c.id, SUM(m.hours)
 FROM crew c 
 LEFT JOIN mission_log m
 ON c.id = m.crew_id
 GROUP BY c.id
)

SELECT * FROM cte

;
```

## NOTES on some bugs found

WTF does this mean!
```
For each deck, compute the average of cumulative mission hours. Return the deck name and the value.
```

The sql query entry does not support comments, it should I think:

`03 Group by in psql`
```SQL
SELECT * FROM crew as c JOIN deck as d ON c.deck_id = d.id;
-- SELECT d.name, AVG(c.mission_hours) FROM crew as c JOIN deck as d ON c.deck_id = d.id GROUP BY d.id;
```
return
```
Query error: can't exectue empty query
```
```
SELECT c.callsign, c.uptime_pct FROM crew AS c WHERE .uptime_pct > (
SELECT AVG(c.uptime_pct) FROM crew AS a
JOIN module AS m 

);
```

WHY is this wrong!!

> For each squad, report the number of logged missions, keeping only squads with at least 6 distinct crew appearing in the mission logs.
```SQL
SELECT c.squad_id, COUNT(DISTINCT ml.id) AS mission_count 
FROM mission_log AS ml 
JOIN crew AS c ON ml.crew_id = c.id
WHERE c.squad_id IS NOT null
GROUP BY c.squad_id
HAVING COUNT(DISTINCT ml.crew_id) >= 6
;
```
