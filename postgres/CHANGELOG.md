# CHANGELOG - postgres

2.0.0 / 2018-02-13
==================
### Changes

* [DOC] Adding configuration for log collection in `conf.yaml`
* [DEPRECATING] Starting with agent6 the postgres check no longer tag server wide metrics with instance tags. See [#1073][]

1.2.1 / 2018-02-13
==================

### Changes

* [BUGFIX] Adding instance tags to service check See [#1042][]


1.2.0 / 2017-11-21
==================

### Changes

* [IMPROVEMENT] Adding an option to include the default 'postgres' database when gathering stats [#740][]
* [BUGFIX] Allows `schema` as tag for custom metrics when no schema relations have been defined See[#776][]

1.1.0 / 2017-08-28
==================

### Changes

* [IMPROVEMENT] Deprecating "postgres.replication_delay_bytes" in favor of "postgresql.replication_delay_bytes". See[#639][] and [#699][], thanks to [@Erouan50][]
* [MINOR] Allow specifying postgres port as string. See [#607][], thanks [@infothrill][]

1.0.3 / 2017-07-18
==================

### Changes

* [FEATURE] Collect pg_stat_archiver stats in PG>=9.4.

1.0.2 / 2017-06-05
==================

### Changes

* [IMPROVEMENT] Provide a meaningful error when custom metrics are misconfigured. See [#446][]

1.0.1 / 2017-03-22
==================

### Changes

* [DEPENDENCY] bump psycopg2 to 2.7.1. See [#295][].

1.0.0 / 2017-03-22
==================

### Changes

* [FEATURE] adds postgres integration.

<!--- The following link definition list is generated by PimpMyChangelog --->
[#295]: https://github.com/DataDog/integrations-core/issues/295
[#446]: https://github.com/DataDog/integrations-core/issues/446
[#607]: https://github.com/DataDog/integrations-core/issues/607
[#639]: https://github.com/DataDog/integrations-core/issues/639
[#689]: https://github.com/DataDog/integrations-core/issues/689
[#699]: https://github.com/DataDog/integrations-core/issues/699
[#740]: https://github.com/DataDog/integrations-core/issues/740
[#776]: https://github.com/DataDog/integrations-core/issues/776
[@Erouan50]: https://github.com/Erouan50
[@infothrill]: https://github.com/infothrill