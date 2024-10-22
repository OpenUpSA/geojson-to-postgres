## Import And Use GeoJSON in PostgreSQL

suburbs.geojson from https://odp-cctegis.opendata.arcgis.com/datasets/cctegis::official-planning-suburbs/explore

```bash
docker compose up

ogr2ogr -f "PostgreSQL" PG:"host=localhost port=5345 dbname=postgres user=postgres" "suburbs.geojson" -nln suburbs -append

psql -h localhost -p 5345 -U postgres
```

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

select *  from reports;
=>     name      | latitude  | longitude  |  suburb
---------------+-----------+------------+-----------
 Test Report 1 | 18.508121 | -33.996951 | LANSDOWNE

```
