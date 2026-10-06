# Laboratory 6: Introductory Spatial Database Modeling

## Contents
- [Required Dependencies](#required-dependencies)
- [Verify PostgreSQL Installation](#verify-postgresql-installation)
- [Troubleshooting](#troubleshooting)
- [Persistence Decision Table](#persistence-decision-table)
- [The Entity-Relationship Diagram](#the-entity-relationship-diagram)
- [Running the SQL Files](#running-the-sql-files)
- [Reflections](#reflections)
- [Author Information](#-author)

## Required Dependencies
Before starting, make sure that you have installed the following dependencies:

- **[PostgreSQL 18](https://www.postgresql.org/download/macosx/)**
- PostGIS extension
- `psql` (included with PostgreSQL) and [pgAdmin 4](https://www.pgadmin.org/)
- *(Optional)* **[VSCode](https://code.visualstudio.com/)**

## Directory Structure
```text
├── diagrams/                       # Design Documentation
│   └── lab6_relational_model.png       # ER diagram for the Lab 6 database model
├── evidence/                       # Contains evidences of SQL commands
│   └── query_results.md                # Documentation of query results
├── sql/                            # SQL commands
│   ├── 01_schema.sql                   # Database schema definition
│   ├── 02_seed.sql                     # Seed data
│   └── 03_queries.sql                  # Sample queries
├── .gitignore                      # Ignored folders/files
└── README.md                       # Project overview, setup, and usage instructions
```

## Verify PostgreSQL Installation

Check PostgreSQL version:

```bash
psql --version
```
You should see something like:

> psql (PostgreSQL) 18.6 (Homebrew)

List databases:

```bash
psql -l
```

At minimum you should see `postgres`, `template0`, and `template1`:

``` text
                         List of databases
   Name    |     Owner     | Encoding | ...
-----------+---------------+----------+----
 postgres  | youruser      | UTF8     | ...
 template0 | youruser      | UTF8     | ...
 template1 | youruser      | UTF8     | ...
```
Use the default database `postgres`:
```bash
psql -d postgres
```
Inside `psql`:

Check the name of the current database used.
```bash
SELECT current_database();
```

Check PostgreSQL version:
```bash
SELECT version();
```

Install the PostGIS extension to the current database.
```bash
CREATE EXTENSION IF NOT EXISTS postgis;
```

Show full PostGIS version, build configuration, and underlying library information. 
```bash
SELECT postgis_full_version();
```

To exit `psql`:
```bash
\q
```

## Troubleshooting
| Problem | Likely cause and fix |
|---|---|
| `'psql' is not recognized as an internal or external command, operable program or batch file.` | By default, the PostgreSQL installer does not automatically add the command-line tools to your system's `PATH` environment variable. Add the PostgreSQL `bin` directory to your system Environment Variables. |
| `role "postgres" does not exist` | Homebrew creates a role named after your macOS user. Use that username in `psql` and pgAdmin. |
| PostGIS is not enabled. | Run the command `CREATE EXTENSION IF NOT EXISTS postgis;` |
| `extension "..." is not available` | The package isn't installed for your running PostgreSQL version. Check `pg_config --version` against `SHOW server_version;`. |
| `database "..." does not exist` | The database might not exist. You may run the `psql -l` command to check for existing databases. |

***

## Persistence Decision Table
> *Note*: The table has been restructured into **one row per attribute** format so that it is easier to map against the ERD.

| Object-model element | Persist? | Reason |
|---|---|---|
| `Parcel.parcel_id` | Yes | Identity must survive — used as the primary key for the Parcel table. |
| `Parcel.land_use` | Yes | Mapped attribute state must survive after the program stops. Can also be considered later on as a separate entity (e.g. `LandUse`) and will be referenced by the `Parcel` table through foreign key.|
| `Parcel.geom` | Yes | Mapped spatial state must survive after the program stops. |
| `Parcel.area()` | No | A behavior since area can be recalculated from the stored geometry whenever it's needed. |
| `Parcel.intersects(other)` | No | This method is a behavior and can be computed from the stored geometries of the objects evaluated. |
| `Building.building_id` | Yes | Identity must survive — used as the primary key for the Building table. |
| `Building.floors` | Yes | Persistent domain attribute describing the building. |
| `Building.geom` | Yes | Persistent spatial state for the building. |
| `Building → Parcel relationship` | Yes | This relationship must be retained across restarts, so it requires a stored key reference (`parcel_id` as a foreign key on `Building`). |
| `Road.road_id` | Yes | Identity must survive — used as the primary key for the Road table. |
| `Road.road_class` | Yes | Persistent attribute describing the road's classification. Can also be considered later on as a separate entity (e.g. `RoadClass`) and will be referenced by the `Road` table through foreign key.|
| `Road.geom` | Yes | Persistent spatial state for the road. |
| `SpatialRule.evaluate(parcel)` | No | Rule-checking logic stays as application code; it is behavior, not data to store. |
| `RuleResult.rulename, passed, message` | Yes | Unlike a rule's logic, the outcome of evaluating a rule against a parcel is a fact worth keeping — it can't be regenerated later without re-running the exact same evaluation against the parcel's state at that point in time, so it must be stored if any history of past assessments is required. |

## The Entity-Relationship Diagram
<img src="diagrams/lab6_relational_model.png" alt="lab5_uml" width="800">

## Running the SQL files

### Method 1: Using the Terminal
Run the SQL files from the root directory using the `psql` command-line utility with the `-f` flag.
``` bash
psql -U <your_username> -d <your_database> -f sql/<sql_file>.sql
```
>***Sample usage:*** psql -U juandlc -d postgres -f sql/01_schema.sql

### Method 2: Using the pgAdmin
1. Open the `query tool` of your specific database.
2. Click the `Open File` icon (folder symbol) in the toolbar.
3. Navigate to and select the SQL file to load it into the editor.
4. Click the Execute/Refresh button (or press F5) to run the script.

>***Important Note:*** If running the SQL files for the **first time**, make sure to run the `01_schema.sql` first then the `02_seed.sql` next. Running `03_queries.sql` before the other two files will produce an error as the schema does not exist and/or the tables have not been populated yet. <br><br>If both `01_schema.sql` and `02_seed.sql` have already been executed, no need to run the them again befure the succeeding runs of `03_queries.sql`.

## Reflections

### *1. Name one attribute from your object model that became a database column. Why must it persist?*
One attribute from my object model that became a database column is the `parcel_id`. This attribute must persist because it serves as the `primary key` (the persistent identity) of the records in the `parcels` table. This attribute is also being referenced by other table such as the `buildings` table, ensuring that the `Building → Parcel` relationship would survive even after the program terminates.

### *2. Name one method or behavior that did not become a column. Why not?*
One method/behavior that did not become a column was the `area()` method. Its result can be calculated on demand from the stored geometry (e.g. using `ST_Area`), so there's no need to persist it directly. That being said, a related design consideration is whether to store a persistent pre-computed value like the `area_sqm` as its own colum. This would allow for quick checks, filtering, or sorting by area without having the additional overhead of computing this every single time, at the cost of needing to keep that stored value in sync if the `geometry` ever changes.

### *3. How did the Building → Parcel object relationship become a relational relationship?*
The `Building → Parcel` object relationship became a relational relationship by having the Building table reference the Parcel table through the use of a `foreign key (parcel_id)`. Through this, a `one-to-many` cardinality is reflected: each `Building` belongs to **exactly one** `Parcel`, while a `Parcel` may have **zero or many** `Buildings`. A `Building` **cannot exist** without a valid `Parcel` reference (enforced by the `NOT NULL` foreign key constraint), but a `Parcel` **can exist independently** with no Buildings at all.

### *4. Why can Road proximity be discovered spatially instead of storing parcel_id in roads?*
The `road proximity` can be discovered **spatially** rather than stored as a **foreign key** because a `road` being near a `parcel` doesn't mean that the `road` belongs to or is owned by that `parcel`. Unlike the `Building → Parcel` relationship, there is no structural ownership to record. Instead, `PostGIS` can calculate this relationship on demand using the `ST_DWithin(geom1, geom2, distance)` spatial method which checks whether the two geometries (geom1 and geom2) are within a given distance (dist) with each other. Storing `parcel_id` in `roads` would misrepresent a geometric/spatial relationship as a **fixed relationship**.

### *5. What is one advantage of explicitly storing geometry type and SRID in the schema?*
One advantage of explicitly storing the `geometry type` (e.g. `Polygon`, `LineString`) is that the database enforces that **only geometries of the correct shape** can be inserted into the table. This would prevent, for example, a line accidentally being stored in a column meant for parcel boundaries. Storing the `SRID` (e.g. 32651) similarly ensures every geometry in that column uses a **known, consistent coordinate reference system,** so spatial calculations like `ST_Area` or `ST_DWithin` produce meaningful, comparable results instead of silently mixing incompatible projections.

## 👤 Author
**ALLAN FRITZGERALD N. AMISTOSO** <br>
*2014-73618* <br>
MS Geomatics Engineering - Geoinformatics <br>
University of the Philippines Diliman
