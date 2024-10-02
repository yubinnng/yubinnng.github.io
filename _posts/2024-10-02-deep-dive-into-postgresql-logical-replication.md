---
title: Deep Dive Into PostgreSQL Logical Replication
date: 2024-10-18 18:27:16 +08:00
categories: [Database]
tags: [PostgreSQL, Database]
---

# Write-Ahead Logging (WAL)

Before diving into PostgreSQL logical replication, we need to know the Write-Ahead Logging (WAL).

WAL’s central concept is that changes to data files (where tables and indexes reside) must be written only after those changes have been logged, that is, after WAL records describing the changes have been flushed to permanent storage. [1]

# Logical decoding

PostgreSQL’s [logical decoding](https://www.postgresql.org/docs/current/static/logicaldecoding-explanation.html) feature was introduced in version 9.4. It is a mechanism that allows the extraction of the changes that were committed to WAL and the processing of these changes in a user-friendly manner with the help of an [output plugin](https://www.postgresql.org/docs/current/static/logicaldecoding-output-plugin.html) [2].

A more official definition can be found in [PostgreSQL: Documentation: Logical Decoding](https://www.postgresql.org/docs/current/logicaldecoding.html).

## Output Plugins

Output plugins enable clients to consume the decoded WAL changes. It transforms the data from the WAL’s internal representation into the format the consumer of a replication slot desires [3].

## pgoutput plugin

As of PostgreSQL 10+, there is a built-in logical decoding output plugin, called **pgoutput**. This means that we can consume that replication stream without the need for additional plug-ins. This is particularly valuable for environments where the installation of plug-ins is not supported or not allowed [4].

## Transaction Consistency

Logical decoding ensures that changes from each transaction are **processed and output as a complete unit**, starting with `BEGIN`, followed by all the changes, and then ending with `COMMIT` (or `ROLLBACK` if it happens). Therefore, only changes in a committed transaction are visible in the logical decoding output. Besides, the changes from different transactions will **not be interleaved** in the logical replication stream [5].

# Logical Replication

Leveraging the logical decoding feature, PostgreSQL supports logical replication, which allows changes from a database to be streamed in real-time to an external system. Logical replication uses a publish and subscribe model with one or more subscribers subscribing to one or more publications on a publisher node [6].

## Publication

A publication defines the publishing rules for table change events. A publication is created using the `[CREATE PUBLICATION](https://www.postgresql.org/docs/current/sql-createpublication.html)` command.

A more official definition can be found in [PostgreSQL: Documentation: Publication](https://www.postgresql.org/docs/current/logical-replication-publication.html).

## Subscription

A _subscription_ is the downstream side of logical replication. The node where a subscription is defined is referred to as the _subscriber_. A subscription defines the connection to another database and set of publications (one or more) to which it wants to subscribe.

## pgJDBC replication API

Other than consuming the changes in another PostgreSQL with subscriptions, this article [PostgreSQL® Extensions to the JDBC API | pgJDBC](https://jdbc.postgresql.org/documentation/server-prepare/#logical-replication) describes how to use _pgJDBC_ replication API to consume the logical changes programmatically.

## Replication Slot

PostgreSQL logical replication uses replication slots to manage the replication position, reserve WAL logs on the server, and define which decoding plugin to use [7].

See [PostgreSQL: Documentation: Streaming Replication Protocol](https://www.postgresql.org/docs/current/protocol-replication.html) to know how to manage replication slots.

# References

1. [PostgreSQL: Documentation: Write-Ahead Logging (WAL)](https://www.postgresql.org/docs/current/wal-intro.html)
2. [Debezium connector for PostgreSQL :: Debezium Documentation](https://debezium.io/documentation/reference/stable/connectors/postgresql.html#postgresql-overview)
3. [PostgreSQL: Documentation: Output Plugins](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html#LOGICALDECODING-EXPLANATION-OUTPUT-PLUGINS)
4. [PostgreSQL 10+ logical decoding support (pgoutput) :: Debezium Documentation](https://debezium.io/documentation/reference/stable/connectors/postgresql.html#postgresql-pgoutput)
5. [PostgreSQL: Documentation: Output Plugin Callbacks](https://www.postgresql.org/docs/current/logicaldecoding-output-plugin.html#LOGICALDECODING-OUTPUT-PLUGIN-CALLBACKS)
6. [PostgreSQL: Documentation: Logical Replication](https://www.postgresql.org/docs/current/logical-replication.html)
7. [PostgreSQL: Documentation: Streaming Replication Protocol](https://www.postgresql.org/docs/current/protocol-replication.html)