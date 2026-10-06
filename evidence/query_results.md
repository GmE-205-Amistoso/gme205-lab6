# Query Results

### Query 1: First Verification — Persistent State
```bash
SELECT parcel_id, land_use, ST_Area(geom) AS area_sqm
FROM lab6.parcels;
```
**Result:**
```text
 parcel_id |  land_use   | area_sqm 
-----------+-------------+----------
 P-001     | Residential |    10000
(1 row)
```
>***Interpretation:*** <br><br> The seed data from the `02_seed.sql` file has been successufully executed. PostGIS has also been verified to work since the `ST_Area(geometry)` spatial function has returned a valid result.

### Query 2: Second Verification — Object Relationship → Foreign Key
```bash
SELECT b.building_id,
       b.floors,
       p.parcel_id,
       p.land_use
FROM lab6.buildings as b
JOIN lab6.parcels as p
ON p.parcel_id = b.parcel_id;
```
**Result:**
```text
 building_id | floors | parcel_id |  land_use   
-------------+--------+-----------+-------------
 B-001       |      2 | P-001     | Residential
(1 row)
```
>***Interpretation:*** <br><br> The `JOIN` reconstructs the `Building -> Parcel` relationship by using the persistent `parcel_id` key stored in both tables to associate each **building** with a specific **parcel** it belongs to.

### Query 3: Topological check
```bash
SELECT b.building_id,
       p.parcel_id,
       ST_Within(b.geom, p.geom) AS is_inside
FROM lab6.buildings AS b
JOIN lab6.parcels AS p 
ON p.parcel_id = b.parcel_id;
```
**Result:**
```text
 building_id | parcel_id | is_inside 
-------------+-----------+-----------
 B-001       | P-001     | t
(1 row)
```
>***Interpretation:*** <br><br> The spatial relationship of building `B-001` and parcel `P-001` has been correctly identified by the spatial function `ST_Within`. The query first uses the shared `parcel_id` to identify the corresponding parcel, then determines whether the building's geometry is within the geometry of that parcel.

### Query 4: Spatial Relationship
```bash
SELECT p.parcel_id,
       r.road_id,
       ST_DWithin(p.geom, r.geom, 15) AS within_15m
FROM lab6.parcels AS p
CROSS JOIN lab6.roads AS r;
```
**Result:**
```text
 parcel_id | road_id | within_15m 
-----------+---------+------------
 P-001     | R-001   | t
(1 row)
```
>***Interpretation:*** <br><br> The spatial relationship of parcel `P-001` and road `R-001` has been correctly identified by the spatial function `ST_DWithin`. Unlike the previous query, this relationship was determined based solely on the geometries of the two features, without requiring an explicit relational key between the tables.