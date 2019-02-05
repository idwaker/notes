# Postgresql

Installation on ubuntu

```shell
$ sudo apt-get install postgresql postgresql-contrib
```

Login as postgres user

```shell
$ su -u postgres
$ sudo su postgres
```

Create new role

```shell
$ createuser --interactive
```

Create new database

```shell
$ createdb mydatabase
```

Login to psql shell

```shell
$ psql -d mydatabase
postgres=# \q
```


# PSQL Commands

Connect to database

```shell
# \c mydatabase
```

list all databases

```shell
# \list
```


list all tables in database

```shell
# \dt
```

dump data to a csv file

```shell
# \o /var/tmp/output.csv
```

exit psql shell

```shell
# \q
```
# SQL

Set postgres password

```sql
ALTER USER postgres PASSWORD 'passpass';
```

Check if pgcrypto extension is available

```sql
SELECT * FROM pg_available_extensions WHERE name='pgcrypto';
```

Create extension for currenct database

```sql
create extension pgcrypto;
```

*__Note__: extension will be enabled to current database. Also requires superuser or database owner role depending on extension.*

Check if pgcrypto is enabled

```sql
SELECT gen_random_uuid();
```
## Selects

```sql
select * from “user” limit 10;
select author, beep::json->>'beep' from beep;
select u.id, u.email, d.beeperhat, d.profile->>'fullname' from "user" as u left join user_detail as d on u.id=d.user_id limit 10;
```
### Group by

```sql
select u.email, count(b.id) from booking as b join "user" as u on u.id = b.user_id group by u.email;
```
## Update

```sql
update user_connection set connected=array_append(connected, 'f4155b03-0d4d-4a8c-a6ed-42262c12b677') where user_id='24773a10-a20a-4984-a298-3efa12fa0e8b';
```
