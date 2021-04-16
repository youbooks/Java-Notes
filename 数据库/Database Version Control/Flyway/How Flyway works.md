# How Flyway works

> 转载：[How Flyway works](https://flywaydb.org/documentation/getstarted/how)

The easiest scenario is when you point **Flyway** to an **empty database**.

![2021-04-16-yXbUaP](https://image.ldbmcs.com/2021-04-16-yXbUaP.jpg)

It will try to locate its **schema history table**. As the database is empty, Flyway won't find it and will **create** it instead.

You now have a database with a single empty table called `flyway_schema_history` by default:

![2021-04-16-GRpvYo](https://image.ldbmcs.com/2021-04-16-GRpvYo.jpg)

This table will be used to track the state of the database.

Immediately afterwards Flyway will begin **scanning** the filesystem or the classpath of the application for **migrations**. They can be written in either Sql or Java.

The migrations are then **sorted** based on their **version number** and applied in order:

![2021-04-16-k0Nb9t](https://image.ldbmcs.com/2021-04-16-k0Nb9t.jpg)

As each migration gets applied, the schema history table is updated accordingly:

**flyway_schema_history**

| installed_rank | version | description   | type | script                | checksum   | installed_by | installed_on          | execution_time | success |
| :------------- | :------ | :------------ | :--- | :-------------------- | :--------- | :----------- | :-------------------- | :------------- | :------ |
| 1              | 1       | Initial Setup | SQL  | V1__Initial_Setup.sql | 1996767037 | axel         | 2016-02-04 22:23:00.0 | 546            | true    |
| 2              | 2       | First Changes | SQL  | V2__First_Changes.sql | 1279644856 | axel         | 2016-02-06 09:18:00.0 | 127            | true    |

With the metadata and the initial state in place, we can now talk about **migrating to newer versions**.

Flyway will once again scan the filesystem or the classpath of the application for migrations. The migrations are checked against the schema history table. If their version number is lower or equal to the one of the version marked as current, they are ignored.

The remaining migrations are the **pending migrations**: available, but not applied.

![2021-04-16-m6SvTL](https://image.ldbmcs.com/2021-04-16-m6SvTL.jpg)

They are then **sorted by version number** and **executed in order**:

![2021-04-16-WyhNWk](https://image.ldbmcs.com/2021-04-16-WyhNWk.jpg)

The **schema history table** is **updated** accordingly:

**flyway_schema_history**

| installed_rank | version | description   | type | script                | checksum   | installed_by | installed_on          | execution_time | success |
| :------------- | :------ | :------------ | :--- | :-------------------- | :--------- | :----------- | :-------------------- | :------------- | :------ |
| 1              | 1       | Initial Setup | SQL  | V1__Initial_Setup.sql | 1996767037 | axel         | 2016-02-04 22:23:00.0 | 546            | true    |
| 2              | 2       | First Changes | SQL  | V2__First_Changes.sql | 1279644856 | axel         | 2016-02-06 09:18:00.0 | 127            | true    |
| 3              | 2.1     | Refactoring   | JDBC | V2_1__Refactoring     |            | axel         | 2016-02-10 17:45:05.4 | 251            | true    |

**And that's it!** Every time the need to evolve the database arises, whether structure (DDL) or reference data (DML), simply create a new migration with a version number higher than the current one. The next time Flyway starts, it will find it and upgrade the database accordingly.

