##### Practice SQL - Finding relatives

```
CREATE TABLE relatives_table (PERSON_ID varchar, RELATIVE_ID1 varchar, RELATIVE_ID3 varchar);
```

```
INSERT INTO relatives_table 
	VALUES 
		('ATR-1', NULL, NULL),
		('ATR-2', 'ATR-1', NULL),
		('ATR-3', 'ATR-2', NULL),
		('ATR-4', 'ATR-3', NULL),
		('ATR-5', 'ATR-4', NULL),
		('BTR-1', NULL, NULL),
		('BTR-2', NULL, 'BTR-1'),
		('BTR-3', NULL, 'BTR-2'),
		('BTR-4', NULL, 'BTR-3'),
		('BTR-5', NULL, 'BTR-4'),
		('CTR-1', NULL, 'CTR-3'),
		('CTR-2', 'CTR-1', NULL),
		('CTR-3', NULL, NULL),
		('DTR-1', 'DTR-3', 'ETR-2'),
		('DTR-2', NULL, NULL),
		('DTR-3', NULL, NULL),
		('ETR-1', NULL, 'DTR-2'),
		('ETR-2', NULL, NULL),
		('FTR-1', NULL, NULL),
		('FTR-2', NULL, NULL),
		('FTR-3', NULL, NULL),
		('GTR-1', 'GTR-1', NULL),
		('GTR-2', 'GTR-1', NULL),
		('GTR-3', 'GTR-1', NULL),
		('GTR-4', 'GTR-1', NULL),
		('GTR-5', 'GTR-1', NULL),
		('HTR-1', 'GTR-1', NULL), 
		('HTR-2', 'GTR-1', NULL),
		('HTR-3', 'GTR-1', NULL),
		('HTR-4', 'GTR-1', NULL),
		('HTR-5', 'GTR-1', NULL)	
```

```
ALTER TABLE relatives_table 
RENAME COLUMN relative_id3 TO relative_id2
```

```
SELECT *
FROM relatives_table 
```

```
CREATE VIEW base_query AS
	SELECT relative_id1 AS "relative", substring(person_id, 1, 3)  AS "fam_group" 
	FROM relatives_table
	WHERE relative_id1 IS NOT NULL
	UNION
	SELECT relative_id2 AS "relative", substring(person_id, 1, 3) AS "fam_group"
	FROM relatives_table
	WHERE relative_id2 IS NOT NULL ORDER BY 1
```

```
SELECT *
FROM base_query 
```
	
```
WITH x AS (
SELECT person_id, fam_group 
FROM base_query 
JOIN relatives_table 
ON base_query."relative" = relatives_table.relative_id1
OR base_query."relative" = relatives_table.relative_id2
	)
SELECT *
FROM base_query
UNION
SELECT *
FROM x
ORDER BY 1
	
WITH RECURSIVE related_fam_members AS
		(SELECT *
		FROM base_query
		UNION
		SELECT r.person_id AS "relative", rfm.fam_group AS fam_group
		FROM related_fam_members rfm
		JOIN relatives_table r 
		ON rfm."relative" = r.relative_id2
		OR rfm."relative" = r.relative_id1
		),
	base_query AS
		(SELECT relative_id1 AS "relative", substring(person_id, 1, 3)  AS "fam_group" 
		FROM relatives_table
		WHERE relative_id1 IS NOT NULL
		UNION
		SELECT relative_id2 AS "relative", substring(person_id, 1, 3) AS "fam_group"
		FROM relatives_table
		WHERE relative_id2 IS NOT NULL ORDER BY 1
		),
	no_relatives AS 
		(SELECT person_id
		FROM relatives_table
		WHERE relative_id1 IS NULL AND relative_id2 IS NULL
		AND person_id NOT IN (SELECT base_query.relative FROM base_query)
		)
SELECT concat('F_', ROW_NUMBER() OVER (ORDER BY x."relatives")) AS family_id, x."relatives"
FROM
	(SELECT DISTINCT string_agg("relative",', ' ORDER BY "relative") AS relatives
	FROM related_fam_members rfm2
	GROUP BY fam_group
	UNION
	SELECT *
	FROM no_relatives
	ORDER BY 1
	) x
```

```
DROP TABLE relatives_table 
```

```
DROP VIEW base_query, relatives_table_view 
```
