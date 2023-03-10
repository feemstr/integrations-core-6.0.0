# PostgreSQL Integration
{{< img src="integrations/postgresql/pggraph.png" alt="PostgreSQL Graph" responsive="true" popup="true">}}

## Overview

Get metrics from PostgreSQL service in real time to:

* Visualize and monitor PostgreSQL states
* Be notified about PostgreSQL failovers and events.

## Setup

### Installation

The PostgreSQL check is packaged with the Agent. To start gathering your PostgreSQL metrics and logs, [install the Agent](https://app.datadoghq.com/account/settings#agent).
If you need the newest version of the check, install the `dd-check-postgres` package manually or with your favorite configuration manager.

### Configuration

Create a `postgres.yaml` file in the Agent's `conf.d` directory.

#### Prepare Postgres

To get started with the PostgreSQL integration, create at least a read-only Datadog user with proper access to your PostgreSQL server. Start psql on your PostgreSQL database and run:

```
create user datadog with password '<PASSWORD>';
grant SELECT ON pg_stat_database to datadog;
```

**Note**: When generating custom metrics that require querying additional tables, you may need to grant the `CONNECT` permission on those tables to the datadog user.

To verify the correct permissions run the following command:

```
psql -h localhost -U datadog postgres -c \
"select * from pg_stat_database LIMIT(1);"
&& echo -e "\e[0;32mPostgres connection - OK\e[0m" || \
|| echo -e "\e[0;31mCannot connect to Postgres\e[0m"
```

When it prompts for a password, enter the one used in the first command.

#### Metric Collection

* Edit the `postgres.yaml` file to point to your server and port, set the masters to monitor. See the [sample postgres.yaml](https://github.com/DataDog/integrations-core/blob/master/postgres/conf.yaml.example) for all available configuration options. Configuration Options:

  * **`username`** (Optional) - The user account used to collect metrics, set in the [Installation section above](#installation)
  * **`password`** (Optional) - The password for the user account.
  * **`dbname`** (Optional) - The name of the database you want to monitor.
  * **`ssl`** (Optional) - Defaults to False. Indicates whether to use an SSL connection.
  * **`use_psycopg2`** (Optional) - Defaults to False. Setting this option to `True` will force the Datadog Agent to collect PostgreSQL metrics using psycopg2 instead of pg8000. Note that pyscopg2 does not support SSL connections.
  * **`tags`** (Optional) - A list of tags applied to all metrics collected. Tags may be simple strings or key-value pairs.
  * **`relations`** (Optional) - By default, all schemas are included. Add specific schemas here to collect metrics for schema relations. Each relation will generate 10 metrics and an additional 10 metrics per index. Use the following structure to declare relations:

    ```
    relations:
      - relation_name: my_relation
        schemas:
          - my_schema_1
          - my_schema_2
    ```

  * **`collect_function_metrics`** (Optional) - Collect metrics regarding PL/pgSQL functions from pg_stat_user_functions
  * **`collect_count_metrics`** (Optional) - Collect count metrics. The default value is `True` for backward compatibility, but this might be slow. The recommended value is `False`.

* [Restart the Agent](https://docs.datadoghq.com/agent/faq/start-stop-restart-the-datadog-agent) to start sending PostgreSQL metrics to Datadog.

#### Log Collection

PostgreSQL default logging is to stderr and logs do not include detailed information. This is why we suggest to log into a file with additional details specified in the log line prefix.

* Edit your PostgreSQL configuration file `/etc/postgresql/<version>/main/postgresql.conf` and uncomment the following parameter in the log section:

  ```
  logging_collector = on
  log_directory = 'pg_log'  # directory where log files are written,
                            # can be absolute or relative to PGDATA
  log_filename = 'pg.log'   #log file name, can include pattern
  log_statement = 'all'     #log all queries
  log_line_prefix= ???%m [%p] %d %a %u %h %c ???
  log_file_mode = 0644
  ## For Windows
  #log_destination = ???eventlog???
  ```

* Collecting logs is disabled by default in the Datadog Agent, you need to enable it in datadog.yaml:

  ```
  logs_enabled: true
  ```

*  Add this configuration setup to your `postgres.yaml` file to start collecting your PostgreSQL logs:

  ```
  logs:
      - type: file
        path: /var/log/pg_log/pg.log
        source: postgresql
        sourcecategory: database
        service: myapp
        #To handle multi line that starts with yyyy-mm-dd use the following pattern
        #log_processing_rules:
        #  - type: multi_line
        #    pattern: \d{4}\-(0?[1-9]|1[012])\-(0?[1-9]|[12][0-9]|3[01])
        #    name: new_log_start_with_date
  ```
  Change the `service` and `path` parameter values and configure them for your environment.
  See the [sample postgres.yaml](https://github.com/DataDog/integrations-core/blob/master/postgres/conf.yaml.example) for all available configuration options.

* [Restart the Agent](https://docs.datadoghq.com/agent/faq/start-stop-restart-the-datadog-agent).

**Learn more about log collection [on the log documentation](https://docs.datadoghq.com/logs)**
### Validation

[Run the Agent's `status` subcommand](https://docs.datadoghq.com/agent/faq/agent-status-and-information/) and look for `postgres` under the Checks section:

```
Checks
======

    postgres
    -----------
      - instance #0 [OK]
      - Collected 39 metrics, 0 events & 7 service checks
```

## Compatibility

The PostgreSQL check is compatible with all major platforms

## Data Collected
### Metrics
See [metadata.csv](https://github.com/DataDog/integrations-core/blob/master/postgres/metadata.csv) for a list of metrics provided by this integration.

### Events
The PostgreSQL check does not include any event at this time.

### Service Checks
The PostgreSQL check does not include any service check at this time.

## Troubleshooting

* [PostgreSQL custom metric collection explained](https://docs.datadoghq.com/integrations/faq/postgres-custom-metric-collection-explained)

## Further Reading
### Blog Post
To get a better idea of how (or why) to have 100x faster PostgreSQL performance by changing 1 line with Datadog, check out our [series of blog posts](https://www.datadoghq.com/blog/100x-faster-postgres-performance-by-changing-1-line/) about it.

### Knowledge Base
#### Custom metrics

The Agent generates PostgreSQL metrics from custom query results. For each custom query, four components are required: `descriptors`, `metrics`, `query`, and `relation`.

* **`query`** is where you'll construct a base SELECT statement to generate your custom metrics. Each column name in your SELECT query should have a corresponding item in the `descriptors` section. Each item in `metrics` will be substituted for the first `%s` in the query.
* **`metrics`** are key-value pairs where the key is the query column name or column function and the value is a tuple containing the custom metric name and metric type (`RATE`, `GAUGE`, or `MONOTONIC`). In the example below, the results of the sum of the `idx_scan` column will appear in Datadog  with the metric name `postgresql.idx_scan_count_by_table`.
* **`descriptors`** is used to add tags to your custom metrics. It's a list of lists each containing 2 strings. The first string is for documentation purposes and should be used to make clear what you are getting from the query. The second string will be the tag name. For multiple tags, include additional columns in your `query` string and a corresponding item in the `descriptors`. The order of items in `descriptors` must match the columns in `query`.
* **`relation`** indicates whether to include schema relations specified in the [`relations` configuration option](#configuration-options). If set to `true`, the second `%s` in `query` will be set to the list of schema names specified in the `relations` configuration option.

##### Example 1

```
custom_metrics:
  # All index scans & reads
  - descriptors:
      - [relname, table]
      - [schemaname, schema]
    metrics:
        SUM(idx_scan) as idx_scan_count: [postgresql.idx_scan_count_by_table, RATE]
        SUM(idx_tup_read) as idx_read_count: [postgresql.idx_read_count_by_table, RATE]
    query: SELECT relname, schemaname, %s FROM pg_stat_all_indexes GROUP BY relname;
    relation: false
```

The example above will run two queries in PostgreSQL:

* `SELECT relname, SUM(idx_scan) as idx_scan_count FROM pg_stat_all_indexes GROUP BY relname;` will generate a rate metric `postgresql.idx_scan_count_by_table`.
* `SELECT relname, SUM(idx_tup_read) as idx_read_count FROM pg_stat_all_indexes GROUP BY relname;` will generate a rate metric `postgresql.idx_read_count_by_table`.

Both metrics will use the tags `table` and `schema` with values from the results in the `relname` and `schemaname` columns respectively. e.g. `table: <relname>`

##### Example 2

The `postgres.yaml.example` file includes an example for the SkyTools 3 Londoniste replication tool:

```
custom_metrics:
  # Londiste 3 replication lag
  - descriptors:
      - [consumer_name, consumer_name]
    metrics:
        GREATEST(0, EXTRACT(EPOCH FROM lag)) as lag: [postgresql.londiste_lag, GAUGE]
        GREATEST(0, EXTRACT(EPOCH FROM lag)) as last_seen: [postgresql.londiste_last_seen, GAUGE]
        pending_events: [postgresql.londiste_pending_events, GAUGE]
    query:
        SELECT consumer_name, %s from pgq.get_consumer_info() where consumer_name !~ 'watermark$';
    relation: false
```

##### Debugging

[Run the Agent's `status` subcommand](https://docs.datadoghq.com/agent/faq/agent-status-and-information/) and look for `postgres` under the Checks section:

```
postgres
--------
  - instance #0 [ERROR]: 'Missing relation parameter in custom metric'
  - Collected 0 metrics, 0 events & 0 service checks
```

You should also check the `/var/log/datadog/collector.log` file for more information.
