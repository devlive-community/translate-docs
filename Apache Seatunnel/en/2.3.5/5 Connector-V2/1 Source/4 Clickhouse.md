[TOC]

> Clickhouse source connector

## Support Those Engines

> Spark<br/>
> Flink<br/>
> SeaTunnel Zeta<br/>

## Key Features

- [x] [batch]($Intro-To-Connector-V2-Features)
- [ ] [stream]($Intro-To-Connector-V2-Features)
- [ ] [exactly-once]($Intro-To-Connector-V2-Features)
- [x] [column projection]($Intro-To-Connector-V2-Features)
- [ ] [parallelism]($Intro-To-Connector-V2-Features)
- [ ] [support user-defined split]($Intro-To-Connector-V2-Features)

> supports query SQL and can achieve projection effect.

## Description

Used to read data from Clickhouse.

## Supported DataSource Info

In order to use the Clickhouse connector, the following dependencies are required.
They can be downloaded via install-plugin.sh or from the Maven central repository.

| Datasource | Supported Versions |                                                    Dependency                                                    |
|------------|--------------------|------------------------------------------------------------------------------------------------------------------|
| Clickhouse | universal          | [Download](https://mvnrepository.com/artifact/org.apache.seatunnel/seatunnel-connectors-v2/connector-clickhouse) |

## Data Type Mapping

|                                                             Clickhouse Data Type                                                              | SeaTunnel Data Type |
|-----------------------------------------------------------------------------------------------------------------------------------------------|---------------------|
| String / Int128 / UInt128 / Int256 / UInt256 / Point / Ring / Polygon MultiPolygon                                                            | STRING              |
| Int8 / UInt8 / Int16 / UInt16 / Int32                                                                                                         | INT                 |
| UInt64 / Int64 / IntervalYear / IntervalQuarter / IntervalMonth / IntervalWeek / IntervalDay / IntervalHour / IntervalMinute / IntervalSecond | BIGINT              |
| Float64                                                                                                                                       | DOUBLE              |
| Decimal                                                                                                                                       | DECIMAL             |
| Float32                                                                                                                                       | FLOAT               |
| Date                                                                                                                                          | DATE                |
| DateTime                                                                                                                                      | TIME                |
| Array                                                                                                                                         | ARRAY               |
| Map                                                                                                                                           | MAP                 |

## Source Options

|       Name       |  Type  | Required |        Default         |                                                               Description                                                                |
|------------------|--------|----------|------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| host             | String | Yes      | -                      | `ClickHouse` cluster address, the format is `host:port` , allowing multiple `hosts` to be specified. Such as `"host1:8123,host2:8123"` . |
| database         | String | Yes      | -                      | The `ClickHouse` database.                                                                                                               |
| sql              | String | Yes      | -                      | The query sql used to search data though Clickhouse server.                                                                              |
| username         | String | Yes      | -                      | `ClickHouse` user username.                                                                                                              |
| password         | String | Yes      | -                      | `ClickHouse` user password.                                                                                                              |
| server_time_zone | String | No       | ZoneId.systemDefault() | The session time zone in database server. If not set, then ZoneId.systemDefault() is used to determine the server time zone.             |
| common-options   |        | No       | -                      | Source plugin common parameters, please refer to [Source Common Options]($Source-Common-Options) for details.                                 |

## How to Create a Clickhouse Data Synchronization Jobs

The following example demonstrates how to create a data synchronization job that reads data from Clickhouse and prints it on the local client:

```bash
# Set the basic configuration of the task to be performed
env {
  parallelism = 10
  job.mode = "BATCH"
}

# Create a source to connect to Clickhouse
source {
  Clickhouse {
    host = "localhost:8123"
    database = "default"
    sql = "select * from test where age = 20 limit 100"
    username = "xxxxx"
    password = "xxxxx"
    server_time_zone = "UTC"
    result_table_name = "test"
  }
}

# Console printing of the read Clickhouse data
sink {
  Console {
    parallelism = 1
  }
}
```

### Tips

> 1.[SeaTunnel Deployment Document]($Deployment).

