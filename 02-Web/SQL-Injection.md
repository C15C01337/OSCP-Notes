# SQL and NoSQL injection

[Home](../README.md) · [Web enumeration](Enumeration.md) · [Direct database access](../01-Enumeration/Databases.md)

## Establish the query context

SQL injection occurs when untrusted input changes a database query's structure. Save a baseline request and vary one input. Identify string/numeric context, quoting, parentheses, comments, likely database and whether results are reflected.

Compare a true and false condition. A single error or slow response is insufficient proof.

```sql
' AND 1=1-- -
' AND 1=2-- -
```

These are string-context examples; they require a compatible surrounding query. In a numeric context the leading quote may be wrong. URL-encode payloads as required by the transport. A `#` typed directly into a browser URL starts a fragment rather than being sent as part of the query; encode it when needed.

## Authentication bypass

Representative patterns from the sources:

```sql
admin'-- -
' OR '1'='1'-- -
') OR ('1'='1')-- -
```

The closing syntax and comment must fit the actual query and database. MySQL supports `#` comments and requires whitespace after `--`; other engines differ. `LIMIT 1` can restrict multiple matching rows in engines that support it, but login bypass also depends on application checks. Large lists of near-identical quote/comment variants have been reduced to these underlying patterns.

## UNION extraction

**Prerequisite:** the vulnerable query returns a compatible result set to the response.

1. Find the number of columns using increasing `ORDER BY` indices or `UNION SELECT NULL` lists.
2. Find which columns accept text and which appear in the response.
3. Confirm database identity, then enumerate relevant schema and data.

```sql
' ORDER BY 1-- -
' ORDER BY 2-- -
' UNION SELECT NULL,NULL,NULL-- -
' UNION SELECT 'marker',NULL,NULL-- -
```

Increase the column count until it matches. An error can also come from incompatible types or syntax. Prefer the smallest reliable result over dumping every table.

### MySQL / MariaDB

The three-column layout below is illustrative; adapt it to the confirmed query:

```sql
' UNION SELECT NULL,version(),database()-- -
' UNION SELECT NULL,table_schema,table_name FROM information_schema.tables-- -
' UNION SELECT NULL,column_name,data_type FROM information_schema.columns WHERE table_schema='app' AND table_name='users'-- -
' UNION SELECT NULL,username,password FROM app.users-- -
```

`GROUP_CONCAT` can combine rows when only one reflected row is useful; delimit fields so the result remains readable. `LOAD_FILE` and `INTO OUTFILE` require separate file privileges and OS access; see [database prerequisites](../01-Enumeration/Databases.md#mysql-and-mariadb--tcp-3306).

### Microsoft SQL Server

Use `@@version`, `DB_NAME()`, `SYSTEM_USER`, `sys.databases` and `INFORMATION_SCHEMA.TABLES/COLUMNS`. A conversion error may reveal a string when the application reflects database error details. Source patterns such as `1=1 IN (SELECT @@version)` rely on implicit type conversion and are not portable syntax for every engine.

Stacked statements and `xp_cmdshell` require support from the driver/query context and sufficient SQL permissions. Enabling a feature is a configuration change; use the [MSSQL procedure](../01-Enumeration/Databases.md#os-execution-with-xp_cmdshell) only after verifying those conditions.

### Oracle

Use `all_tables` and `all_tab_columns` for accessible schema metadata. Simple Oracle `SELECT` expressions may require `FROM dual` depending on version/context. Column count and types still need to match the original query.

### SQLite

Schema metadata is exposed through `sqlite_master`/`sqlite_schema`, for example:

```sql
SELECT name,sql FROM sqlite_master WHERE type='table';
```

SQLite does not have MySQL's `information_schema` or MSSQL's `xp_cmdshell`.

## Boolean and time-based inference

Boolean inference compares a condition's true/false response, including body text, status or length. Time-based inference compares a conditional delay with a control request over multiple trials.

**MySQL example:**

```sql
' AND IF(1=1,SLEEP(3),0)-- -
' AND IF(1=2,SLEEP(3),0)-- -
```

**MSSQL stacked-statement example:**

```sql
'; IF (1=1) WAITFOR DELAY '00:00:03'-- -
```

Timing depends on the query actually evaluating the expression. Network jitter, caching and concurrency can imitate a delay. Compare controls, then change the condition to a specific database question.

## SQL truncation

A registration/update flow may compare an input differently from how it stores it. An overlong username that is accepted, normalized or silently truncated can collide with an existing identity. The source `admin@<DOMAIN> + padding` example is a hypothesis about field length and comparison behavior, not a universal payload. Verify stored length, uniqueness checks, whitespace handling and resulting login identity.

## NoSQL injection

**Prerequisite:** user-controlled input reaches a query object/operator or an evaluated expression without suitable type validation.

```json
{"username":{"$ne":null},"password":{"$ne":null}}
```

An application expecting strings may reject this outright. Compare a controlled operator object with normal valid/invalid string values. The source JavaScript-like `admin'||''==='` pattern applies only to a compatible expression context. MongoDB shell commands and direct record updates belong to [direct database access](../01-Enumeration/Databases.md#mongodb--tcp-27017).

## Tool boundary

The sources include SQLMap request replay (`sqlmap -r request.txt -p item`) alongside conflicting exam advice. Keep it as a lab automation reference, and use the manual workflow here for understanding and reproducibility. The local collection does not establish current exam permission.
