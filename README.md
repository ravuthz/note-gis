# PostGIS PSQL


Install posthis package first
```bash

-- On ubuntu server
sudo apt install postgis

-- On mac using homebrew
brew install postgis

```

```sql
-- Check postgis extension is exists
SELECT * FROM pg_available_extensions WHERE name = 'postgis';

-- Install postgis extension
CREATE EXTENSION postgis;

-- Check the postgis version
SELECT PostGIS_Full_Version();

```


## Testing Postgis
```sql

CREATE TABLE IF NOT EXISTS test_postgis (name varchar, geometry geometry);

INSERT INTO test_postgis VALUES
  ('Point', 'POINT(0 0)'),
  ('Linestring', 'LINESTRING(0 0, 1 1, 2 1, 2 2)'),
  ('Polygon', 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
  ('PolygonWithHole', 'POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))'),
  ('Collection', 'GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))');

INSERT INTO test_postgis VALUES
  ('Point', ST_GeomFromText('POINT(0 0)')),
  ('MultiPoint', ST_GeomFromText('MULTIPOINT(0 0, 0 1)')),
  ('Linestring', ST_GeomFromText('LINESTRING(0 0, 1 1, 2 1, 2 2)')),
  ('MultiLinestring', ST_GeomFromText('MULTILINESTRING((0 0, 1 1, 2 1, 2 2),(3 3, 4 4, 5 5))')),
  ('Polygon', ST_GeomFromText('POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))')),
  ('MultiPolygon', ST_GeomFromText('MULTIPOLYGON(((0 0, 1 0, 1 1, 0 1, 0 0)),((2 2, 3 2, 3 3, 2 3, 2 2)))')),
  ('PolygonWithHole', 'POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))'),
  ('Collection', 'GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))');

```

```sql
SELECT name, geometry FROM test_postgis;


+---------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|name           |geometry                                                                                                                                                                                                                                                                                                                                                          |
+---------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|Point          |010100000000000000000000000000000000000000                                                                                                                                                                                                                                                                                                                        |
|Linestring     |01020000000400000000000000000000000000000000000000000000000000F03F000000000000F03F0000000000000040000000000000F03F00000000000000400000000000000040                                                                                                                                                                                                                |
|Polygon        |0103000000010000000500000000000000000000000000000000000000000000000000F03F0000000000000000000000000000F03F000000000000F03F0000000000000000000000000000F03F00000000000000000000000000000000                                                                                                                                                                        |
|PolygonWithHole|01030000000200000005000000000000000000000000000000000000000000000000002440000000000000000000000000000024400000000000002440000000000000000000000000000024400000000000000000000000000000000005000000000000000000F03F000000000000F03F000000000000F03F0000000000000040000000000000004000000000000000400000000000000040000000000000F03F000000000000F03F000000000000F03F|
|Collection     |0107000000020000000101000000000000000000004000000000000000000103000000010000000500000000000000000000000000000000000000000000000000F03F0000000000000000000000000000F03F000000000000F03F0000000000000000000000000000F03F00000000000000000000000000000000                                                                                                            |
+---------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

```sql
SELECT name, ST_SRID(geometry), ST_AsText(geometry) FROM test_postgis;

+---------------+-------+-------------------------------------------------------------+
|name           |st_srid|st_astext                                                    |
+---------------+-------+-------------------------------------------------------------+
|Point          |0      |POINT(0 0)                                                   |
|Linestring     |0      |LINESTRING(0 0,1 1,2 1,2 2)                                  |
|Polygon        |0      |POLYGON((0 0,1 0,1 1,0 1,0 0))                               |
|PolygonWithHole|0      |POLYGON((0 0,10 0,10 10,0 10,0 0),(1 1,1 2,2 2,2 1,1 1))     |
|Collection     |0      |GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0,1 0,1 1,0 1,0 0)))|
+---------------+-------+-------------------------------------------------------------+


```

## Convert between 4326 and 32648 using ST_Transform

```sql

## Add samples data with SRID
INSERT INTO test_postgis VALUES
  ('TestMultiPolygon1', 'MULTIPOLYGON(((104.8809019096807 11.386177840390399,104.88092335561748 11.38621570868837,104.88098473537255 11.386174738161007,104.88097638802223 11.386144997385077,104.8809019096807 11.386177840390399)))'),
  ('TestPoint1', 'POINT(104.88531045545238 11.384859335257575)'),
  ('TestMultiPolygon2', 'MULTIPOLYGON(((487006.4879027305 1258682.5112933894,487008.8293594435 1258686.6975650848,487015.5239948292 1258682.164561993,487014.6119552875 1258678.8763925983,487006.4879027305 1258682.5112933894)))'),
  ('TestPoint2', 'POINT(487487.3999284887 1258536.5258183747)');

UPDATE test_postgis SET geometry = ST_SetSRID(geometry, 4326) WHERE name IN ('TestMultiPolygon1', 'TestPoint1');
UPDATE test_postgis SET geometry = ST_SetSRID(geometry, 32648) WHERE name IN ('TestMultiPolygon2', 'TestPoint2');


```

```sql
SELECT name, ST_SRID(geometry), ST_AsText(geometry) FROM test_postgis;

+-----------------+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|name             |st_srid|st_astext                                                                                                                                                                                                   |
+-----------------+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|Point            |0      |POINT(0 0)                                                                                                                                                                                                  |
|Linestring       |0      |LINESTRING(0 0,1 1,2 1,2 2)                                                                                                                                                                                 |
|Polygon          |0      |POLYGON((0 0,1 0,1 1,0 1,0 0))                                                                                                                                                                              |
|PolygonWithHole  |0      |POLYGON((0 0,10 0,10 10,0 10,0 0),(1 1,1 2,2 2,2 1,1 1))                                                                                                                                                    |
|Collection       |0      |GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0,1 0,1 1,0 1,0 0)))                                                                                                                                               |
|TestMultiPolygon1|4326   |MULTIPOLYGON(((104.8809019096807 11.386177840390399,104.88092335561748 11.38621570868837,104.88098473537255 11.386174738161007,104.88097638802223 11.386144997385077,104.8809019096807 11.386177840390399)))|
|TestPoint1       |4326   |POINT(104.88531045545238 11.384859335257575)                                                                                                                                                                |
|TestMultiPolygon2|32648  |MULTIPOLYGON(((487006.4879027305 1258682.5112933894,487008.8293594435 1258686.6975650848,487015.5239948292 1258682.164561993,487014.6119552875 1258678.8763925983,487006.4879027305 1258682.5112933894)))   |
|TestPoint2       |32648  |POINT(487487.3999284887 1258536.5258183747)                                                                                                                                                                 |
+-----------------+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

```sql
# From 4326 to 32648 using ST_Transform
SELECT ST_SRID(geometry), ST_SRID(ST_Transform(geometry, 32648)), ST_AsText(ST_Transform(geometry, 32648))
FROM test_postgis WHERE ST_SRID(geometry) = 4326;
+-------+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|st_srid|st_srid|st_astext                                                                                                                                                                                                |
+-------+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|4326   |32648  |MULTIPOLYGON(((487006.4879027305 1258682.5112933894,487008.8293594435 1258686.6975650848,487015.5239948292 1258682.164561993,487014.6119552875 1258678.8763925983,487006.4879027305 1258682.5112933894)))|
|4326   |32648  |POINT(487487.3999284887 1258536.5258183747)                                                                                                                                                              |
+-------+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


# From 32648 to 4326 using ST_Transform
SELECT ST_SRID(geometry), ST_SRID(ST_Transform(geometry, 4326)), ST_AsText(ST_Transform(geometry, 4326))
FROM test_postgis WHERE ST_SRID(geometry) = 32648;
+-------+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|st_srid|st_srid|st_astext                                                                                                                                                                                                  |
+-------+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|32648  |4326   |MULTIPOLYGON(((104.8809019096807 11.386177840390397,104.88092335561748 11.38621570868837,104.88098473537255 11.386174738161007,104.88097638802223 11.38614499738508,104.8809019096807 11.386177840390397)))|
|32648  |4326   |POINT(104.88531045545238 11.384859335257575)                                                                                                                                                               |
+-------+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

## Find Point in Polygons - ST_DWithin, ST_Within, ST_Contains and ST_Intersects

```sql
SELECT
    ST_AsText(point) AS "PointText",
    ST_AsText(geometry) AS "GeometryText",

    ST_SRID(point) AS "PointSRID",
    ST_SRID(geometry) AS "GeometrySRID",

    ST_DWithin(geometry::geography,point, 100),
    ST_DWithin(point, geometry::geography,100),

    ST_Intersects(geometry, point),
    ST_Intersects(point, geometry),

    ST_Within(geometry,point),
    ST_Within(point, geometry),

    ST_Contains(geometry, point),
    ST_Contains(point, geometry)
FROM (
    SELECT
    (SELECT geometry FROM test_postgis WHERE name = 'TestMultiPolygon1') AS "geometry",
    (SELECT geometry FROM test_postgis WHERE name = 'TestPoint1') AS "point"
) AS location;
```




# ArcGIS PSQL

```sql

CREATE TABLE test_arcgis (name varchar, shape sde.st_geometry);

INSERT INTO test_arcgis VALUES (
 'Point',
 sde.st_point ('point (10.01 50.76)', 4326)
);

SELECT name, ST_AsText(shape) FROM test_arcgis;

```

## Casting

```sql

-- Convert to SRID = 32648
SELECT ST_SetSRID(geometry, 32648);

-- Convert to 3D or HasZ = true
SELECT ST_Force3D(geometry);

-- Print Text for hummand readable
SELECT ST_AsText(geometry);

```

### LatLng (4326) to UTM (32648)

```sql
SELECT
        ST_AsText(shape) shape_text,
        ST_AsText(geometry) geometry_text,
        ST_X(ST_Centroid(shape)) AS utm_easting_x,
        ST_Y(ST_Centroid(shape)) AS utm_northing_y,
        ST_SRID(shape) shape_srid,
        ST_SRID(geometry) geometry_srid,
        FLOOR((ST_X(ST_Centroid(shape)) + 180) / 6) + 1 AS utm_zone
FROM (
    SELECT
        ST_GeogFromText('POINT(104.8717520846559 11.629526891411311)') AS geometry,
        ST_Transform(ST_GeogFromText('POINT(104.8717520846559 11.629526891411311)')::geometry, 32648) as shape
) latlng;

+-------------------------------------------+-------------------------------------------+-----------------+------------------+----------+-------------+--------+
|shape_text                                 |geometry_text                              |utm_easting_x    |utm_northing_y    |shape_srid|geometry_srid|utm_zone|
+-------------------------------------------+-------------------------------------------+-----------------+------------------+----------+-------------+--------+
|POINT(486020.2633959938 1285591.1348916255)|POINT(104.8717520846559 11.629526891411311)|486020.2633959938|1285591.1348916255|32648     |4326         |81034   |
+-------------------------------------------+-------------------------------------------+-----------------+------------------+----------+-------------+--------+

```

#### Reference Docs

https://postgis.net/documentation/getting_started/

https://postgis.net/documentation/getting_started/install_macos/

https://desktop.arcgis.com/en/arcmap/latest/manage-data/using-sql-with-gdbs/st-astext.htm


