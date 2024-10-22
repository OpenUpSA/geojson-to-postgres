## Import And Use GeoJSON in PostgreSQL

Example of importing a GeoJSON file into PostgreSQL and then querying it.

The example GeoJSON contains the [Official Planning Suburbs of the City of Cape Town](https://odp-cctegis.opendata.arcgis.com/search?q=%22Official%20Planning%20Suburbs%22).

# Requirements
- `ogr2ogr` from [GDAL](https://gdal.org/en/latest/index.html)
- [Docker](https://www.docker.com/)
- [psql](https://www.postgresql.org/docs/current/app-psql.html)

# Run

```bash
docker compose up
```

Note that `ogr2ogr` is set to `append` data. You can change it to `overwrite` to update the table:
```bash
ogr2ogr -f "PostgreSQL" PG:"host=localhost port=5345 dbname=postgres user=postgres" "suburbs.geojson" -nln suburbs -append
```

Connect to the database:
```bash
psql -h localhost -p 5345 -U postgres
```

Then in the `psql` shell:
```sql
SELECT
    ofc_sbrb_name
FROM
    suburbs
WHERE
    ST_Contains(
        wkb_geometry,
        ST_SetSRID(
            ST_MakePoint(18.50812144380240, -33.99695073329119),
            4326
        )
    );

=> ofc_sbrb_name
---------------
 LANSDOWNE
(1 row)

CREATE TABLE reports (
    name Varchar,
    latitude Decimal(9, 6),
    longitude Decimal(9, 6),
    suburb Varchar
);
=> CREATE TABLE

INSERT INTO
    reports(name, latitude, longitude, suburb)
VALUES
(
        'Test Report 1',
        18.50812144380240,
        -33.99695073329119,
        (
            SELECT
                ofc_sbrb_name
            FROM
                suburbs
            WHERE
                ST_Contains(
                    wkb_geometry,
                    ST_SetSRID(
                        ST_MakePoint(18.50812144380240, -33.99695073329119),
                        4326
                    )
                )
        )
    );

SELECT * FROM reports;
=>     name      | latitude  | longitude  |  suburb
---------------+-----------+------------+-----------
 Test Report 1 | 18.508121 | -33.996951 | LANSDOWNE

```
