# Laboratory 6: Introductory Spatial Database Modeling

## Contents
- [Required Dependencies](#required-dependencies)
- [Verify PostgreSQL Installation](#verify-postgresql-installation)
- [Troubleshooting](#troubleshooting)
- [Persistence Decision Table](#persistence-decision-table)
- [Reflections](#reflections)
- [Author Information](#-author)

## Required Dependencies
Before starting, make sure that you have installed the following dependencies:

- **[PostgreSQL 18](https://www.postgresql.org/download/macosx/)**
- PostGIS
- `psql` (included with PostgreSQL) and [pgAdmin 4](https://www.pgadmin.org/)
- *(Optional)* **[VSCode](https://code.visualstudio.com/)**

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
```bash
SELECT current_database();
```
> This should show the current database name.
```bash
SELECT version();
```
> This should show the current version of PostgreSQL.
```bash
SELECT postgis_full_version();
```
> This should show a the current version of PostGIS and the installed extensions.
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

## Directory Structure
```text
├── diagrams/                       # Design Documentation
│   └── lab6_relational_model.png       # ER diagram for the Lab 6 database model
├── sql/                            # SQL commands
│   ├── 01_schema.sql                   #
│   ├── 02_seed.sql                     #
│   └── 03_queries.sql                  #
├── evidence/                       # 
│   └── query_results.md                # 
├── .gitignore                      # 
└── README.md                       # Project overview, setup, and usage instructions
```

## Persistence Decision Table
| Object-model element       | Persist? | Reason |
|---|---|---|
| `Parcel.id` | Yes | Primary key for the Parcel table. |
| `Parcel.geometry` | Yes | Core spatial attribute; needed for intersection/distance queries (PostGIS geometry column). |
| `Parcel.zone` | Yes | For querying/filtering by zone. Also needed to evaluate AllowedZoneRule against stored data |
| `Parcel.area_sqm` | Yes | For querying/filtering by area. Needed to evaluate MinimumAreaRule without recomputing from geometry every time. |
| `Parcel.intersects(other)` | No | A method is behavior; intersection can be recomputed on demand from stored geometry. |
| `Building.building_id` | Yes | This will be used as the primary key for the Building table. |
| `Building.floors` | Yes | Persistent domain state describing the building. |
| `Building.geometry` | Yes | Core spatial state for the building; needed for spatial queries. |
| `Building → Parcel relationship` | Yes | Defines the relationship between Parcel and Boundary — a building belongs to exactly one parcel, so this relationship will need a key reference (`parcel_id` as a `foreign key` on Building). |
| `Road.road_id` | Yes | Primary key for the Road table. |
| `Road.geometry` | Yes | Core spatial attribute; needed for RoadAccessRule distance calculations. |
| `HazardZone.zone_id` | Yes | Primary key for the HazardZone table. |
| `HazardZone.geometry` | Yes | Core spatial attribute; needed for spatial overlap checks against Parcel geometry. |
| `HazardZone.hazard_type` | Yes | For querying/filtering by hazard type. |
| `HazardZone.severity` | Yes | For querying/filtering by hazard severity. |
| `AssessmentRule.evaluate(parcel)` | No | Executable rule behavior remains application code in this introductory exercise. |

## Reflections

## 👤 Author
**ALLAN FRITZGERALD N. AMISTOSO** <br>
*2014-73618* <br>
MS Geomatics Engineering - Geoinformatics <br>
University of the Philippines Diliman
