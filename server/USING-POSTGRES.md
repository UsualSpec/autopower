# Postgres database (Server)

Autopower uses a PostgresSQL based database on both the client and server to store measurement and application data. The current schema can be found in `workspace/server/database_schema.sql` or `workspace/client/client_db_schema.sql`. Most analysis should be done on the server since the clients' database is only considered as backup.

## Database schema overview
While the actual database schema is described in the mentioned SQL files and may change, the rough idea should remain stable. We only focus on the server side here.

### Tables (Relations)

measurements: Saves data about measurements. This does **not** contain measurement data but rather metadata about multiple related measurement points. A measurement is started as soon as a client starts measuring - which happens automatically at boot or if a measurement start request is issued from the server.

clients: Saves data about clients. 

`measurement_data`: Saves the actual measurements of each client. `measurement_timestamp` stores the time (including date and time zone) when each measurement sample was measured - this is generated by pinpoint on the client side and the measurement value reported by pinpoint. The `server_measurement_id` links to the measurements table to link to the metadata. This table is the one to analyze.

## Practical connection and some queries

Once connected via SSH to the server, issue `psql -d autopower` on the CLI to connect to the postgres database. You will now be connected to the autopower database.

You can now check which measurements are saved on the server by querying the measurements table with: `SELECT * FROM measurements;`. The `shared_measurement_id` is an identifier generated by the client to uniquely identify a measurement. This is only used for convenience and should be considered as a random string - even if it seems to contain useful data such as the measurement start time. The clients' clock may be out of sync for the first few measurements and thus also for the `shared_measurement_id`: the Pi needs to synchronize its clock right after boot.
Check the `server_measurement_id` and `client_uid` instead! Use those to link to the `measurement_data`.

Measurement data for one measurement can be queried from the `measurement_data` table as follows:

```sql
SELECT measurement_value, measurement_timestamp FROM measurement_data WHERE server_measurement_id = <the_server_measurement_id_you_want_to_query> ORDER BY measurement_timestamp;
```

You can average over the whole result as follows:

```sql
SELECT AVG(measurement_value) AS average_measurement_value FROM measurement_data WHERE server_measurement_id = <server_measurement_id>;
```

To get the last 6 hours worth of data, an SQL query like follows may be of interest:

```sql
SELECT measurement_value, measurement_timestamp FROM measurement_data WHERE server_measurement_id = <server_measurement_id> AND measurement_timestamp > NOW() - INTERVAL '6 hours' ORDER BY measurement_timestamp;
```


To export the result of an SQL query as `.csv` file, the COPY command can be used:

```sql
COPY (<sql_query>) TO 'path/to/output.csv' DELIMITER ',' CSV HEADER;
```

## Further links
* [Postgresql SQL commands documentation](https://www.postgresql.org/docs/current/sql-commands.html)
