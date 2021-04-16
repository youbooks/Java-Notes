# Rolling Back Migrations with Flyway

> 转载：[Rolling Back Migrations with Flyway](https://www.baeldung.com/flyway-roll-back)

## 1. Introduction

In this short tutorial, we'll explore a couple of ways to rollback a migration with [Flyway.](https://www.baeldung.com/database-migrations-with-flyway)

## 2. Simulate Rollback with a Migration

In this section, we'll rollback our database using a standard migration file.

In our examples, we'll use the command-line version of Flyway. However, the core principles are equally applicable to the other formats, such as the core API, Maven plugin, etc.

### 2.1. Create Migration

First, let's add a new *book* table to our database. In order to do this, we'll create a migration file called `V1_0__create_book_table.sql`:

```sql
create table book (
  id numeric,
  title varchar(128),
  author varchar(256),
  constraint pk_book primary key (id)
);
```

Secondly, let's apply the migration:

```bash
./flyway migrate
```

### 2.2. Simulate Rollback

Then, at some point, say we need to reverse the last migration.

In order to restore the database to before the *book* table was created, let's create migration called `V2_0__drop_table_book.sql`:

```sql
drop table book;
```

Next, let's apply the migration:

```bash
./flyway migrate
```

Finally, we can check the history of all the migrations using:

```bash
./flyway info
```

which gives us the following output:

```bash
+-----------+---------+-------------------+------+---------------------+---------+
| Category  | Version | Description       | Type | Installed On        | State   |
+-----------+---------+-------------------+------+---------------------+---------+
| Versioned | 1.0     | create book table | SQL  | 2020-08-29 16:07:43 | Success |
| Versioned | 2.0     | drop table book   | SQL  | 2020-08-29 16:08:15 | Success |
+-----------+---------+-------------------+------+---------------------+---------+
```

Notice that our second migration ran successfully.

As far as Flyway is concerned, the second migration file is just another standard migration. The actual restoring of the database to the previous version is done entirely through SQL. For example, in our case, the SQL of dropping the table is the opposite of the first migration, which creates the table.

Using this method, **the audit trail doesn't show us that the second migration is related to the first**, as they have different version numbers. In order to get such an audit trail, we need to use Flyway Undo.

## 3. Using Flyway Undo

Firstly, it's important to note that **Flyway Undo is a commercial feature of Flyway and isn't available in the Community Edition.** Therefore, we'll need either the Pro Edition or Enterprise Edition in order to use this feature.

### 3.1. Create Migration Files

First, let's create a migration file called `V1_0__create_book_table.sql`:

```sql
create table book (
  id numeric,
  title varchar(128),
  author varchar(256),
  constraint pk_book primary key (id)
);
```

Secondly, let's create the corresponding undo migration file `U1_0__create_book_table.sql`:

```sql
drop table book;
```

In our undo migration, notice how the filename-prefix is ‘U' compared with the normal migration prefix of ‘V'. Also, in our undo migration files, **we write the SQL that reverses the changes of the corresponding migration file**. In our case, we're dropping the table that's created by the normal migration.

### 3.2. Apply Migrations

Next, let's check the current state of the migrations:

```bash
./flyway -pro info
```

This gives us the following output:

```bash
+-----------+---------+-------------------+------+--------------+---------+----------+
| Category  | Version | Description       | Type | Installed On | State   | Undoable |
+-----------+---------+-------------------+------+--------------+---------+----------+
| Versioned | 1.0     | create book table | SQL  |              | Pending | Yes      |
+-----------+---------+-------------------+------+--------------+---------+----------+
```

Notice the last column, *Undoable*, which indicates Flyway has detected an undo migration file that accompanies our normal migration file.

Next, let's apply our migrations:

```bash
./flyway migrate
```

When it completes, our migrations are complete, and our schema has a new book table:

```bash
                List of relations
 Schema |         Name          | Type  |  Owner   
--------+-----------------------+-------+----------
 public | book                  | table | baeldung
 public | flyway_schema_history | table | baeldung
(2 rows)
```

### 3.3. Rollback the Last Migration

Finally, let's undo the last migration using the command line:

```bash
./flyway -pro undo
```

After the command has run successfully, we can check the status of the migrations again:

```bash
./flyway -pro info
```

which gives us the following output:

```bash
+-----------+---------+-------------------+----------+---------------------+---------+----------+
| Category  | Version | Description       | Type     | Installed On        | State   | Undoable |
+-----------+---------+-------------------+----------+---------------------+---------+----------+
| Versioned | 1.0     | create book table | SQL      | 2020-08-22 15:48:00 | Undone  |          |
| Undo      | 1.0     | create book table | UNDO_SQL | 2020-08-22 15:49:47 | Success |          |
| Versioned | 1.0     | create book table | SQL      |                     | Pending | Yes      |
+-----------+---------+-------------------+----------+---------------------+---------+----------+
```

Notice how the undo has been successful, and the first migration is back to pending. Also, in contrast to the first method, the **audit trail clearly shows the migrations that were rolled back.**

Although Flyway Undo can be useful, [it assumes that the whole migration has succeeded.](https://flywaydb.org/documentation/command/undo) For example, it may not work as expected if a migration fails partway through.

## 4. Conclusion

In this short tutorial, we looked at restoring our database using a standard migration. We also looked at the official way of rolling back migrations using Flyway Undo. As usual, all of our code that relates to this tutorial can be found [over on GitHub.](https://github.com/eugenp/tutorials/tree/master/persistence-modules/flyway)