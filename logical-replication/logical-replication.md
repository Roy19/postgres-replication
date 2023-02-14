# Start logical replication in Postgres

Create a database and tables inside leader database whose tables you want to replicate in the follower database instance.

In the leader database instance:

```sql
create database leader_database;
\c leader_database -- enter change to the leader_database
create table tbl_1 (id int primary key, name text);
```

In the follower database instance:

```sql 
create database leader_database;
```

Copy the DDL statements from the main database into the follower database using the pg_dump command.

```
$ pg_dump -t tbl -s -d leader_database -U postgres -p 5432
```

Insert some dummy data into the dummy table just created in `tbl` inside the `leader_database`.

```sql
insert into tbl values (generate_series(1, 10), 'name' || generate_series(1, 10));
```

Now create a publication inside the `leader_database`.

```sql
create publication my_publication for all tables; -- can also specify the specific tables we want to publish
```

Create a subscription inside the follower database to receive updates from the publication `my_publication`.

```sql
create subscription my_subscription connection 'host=db_leader user=postgres password=postgres dbname=leader_database' publication my_publication;
```

Check if `INSERT`, `UPDATE`, `DELETE` and `TRUNCATE` are getting reflected back in the follower table.