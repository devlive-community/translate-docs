[TOC]

> Neo4j source connector

## Description

Read data from Neo4j.

`neo4j-java-driver` version 4.4.9

## Key features

- [x] [batch]($Intro-To-Connector-V2-Features)
- [ ] [stream]($Intro-To-Connector-V2-Features)
- [ ] [exactly-once]($Intro-To-Connector-V2-Features)
- [x] [column projection]($Intro-To-Connector-V2-Features)
- [ ] [parallelism]($Intro-To-Connector-V2-Features)
- [ ] [support user-defined split]($Intro-To-Connector-V2-Features)

## Options

|            name            |  type  | required | default value |
|----------------------------|--------|----------|---------------|
| uri                        | String | Yes      | -             |
| username                   | String | No       | -             |
| password                   | String | No       | -             |
| bearer_token               | String | No       | -             |
| kerberos_ticket            | String | No       | -             |
| database                   | String | Yes      | -             |
| query                      | String | Yes      | -             |
| schema                     | Object | Yes      | -             |
| max_transaction_retry_time | Long   | No       | 30            |
| max_connection_timeout     | Long   | No       | 30            |

### uri [string]

The URI of the Neo4j database. Refer to a case: `neo4j://localhost:7687`

### username [string]

username of the Neo4j

### password [string]

password of the Neo4j. required if `username` is provided

### bearer_token [string]

base64 encoded bearer token of the Neo4j. for Auth.

### kerberos_ticket [string]

base64 encoded kerberos ticket of the Neo4j. for Auth.

### database [string]

database name.

### query [string]

Query statement.

### schema.fields [string]

returned fields of `query`

see [column projection]($Intro-To-Connector-V2-Features)

### max_transaction_retry_time [long]

maximum transaction retry time(seconds). transaction fail if exceeded

### max_connection_timeout [long]

The maximum amount of time to wait for a TCP connection to be established (seconds)

## Example

```
source {
    Neo4j {
        uri = "neo4j://localhost:7687"
        username = "neo4j"
        password = "1234"
        database = "neo4j"
    
        max_transaction_retry_time = 1
        max_connection_timeout = 1
    
        query = "MATCH (a:Person) RETURN a.name, a.age"
    
        schema {
            fields {
                a.age=INT
                a.name=STRING
            }
        }
    }
}
```

## Changelog

### next version

- Add Neo4j Source Connector
