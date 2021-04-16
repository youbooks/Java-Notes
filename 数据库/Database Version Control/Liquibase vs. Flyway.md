# Liquibase vs. Flyway

> 转载：[Liquibase vs. Flyway](https://www.liquibase.org/liquibase-vs-flyway)

*Understand the differences and similarities of these two database migration tools so you can decide which one will work best for your use case, team structure, and workflow.*

Anyone who has ever developed software will tell you, you shouldn’t develop application code without version control. The same is true for database code. With the rise of Agile and DevOps methodologies that are needed to accomplish continuous integration and deployment, it’s more important than ever to [apply CI/CD to databases](https://www.liquibase.com/ci-cd).

There are two leading open source tools for database version control: Liquibase and Flyway. Both are popular options for versioning and organizing database changes, deploying changes when they need to be deployed, and tracking what’s been deployed.

## Here’s what Flyway and Liquibase have in common:

- Use a migrations-based approach to database change
- At update time, both tools check if a change has already been deployed
- Run from command line or Maven
- Offer support and enhanced features as a paid add-on to their open source offering
- Use SQL-based migrations (both can use plain old SQL)
- Repeatable migrations (both can perform rerunnable vs. non-rerunnable changes)

## Differences between Liquibase and Flyway

While both tools are based on [Martin Fowler’s Evolutionary Database](https://martinfowler.com/articles/evodb.html), there are some differences. Here’s where [Liquibase](https://www.liquibase.org/) and [Flyway](https://flywaydb.org/) differ.

|                           FlywayDB                           |                          Liquibase                           |                                                              |
| :----------------------------------------------------------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|                 Diff (Compare two databases)                 |                              –                               | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
|                    Generates SQL for you                     |                              –                               | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
|                           Rollback                           | ![money icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/money-icon-30x30.png) | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
|           Targeted rollback for any single change            |                              –                               | ![money icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/money-icon-30x30.png) |
|            Targeted rollback for a set of changes            |                              –                               | ![money icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/money-icon-30x30.png) |
|                       Defining changes                       |                             SQL                              | SQL, XML, JSON, YAML                                         |
|                    Java-based migrations                     | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) | –                                                            |
|                    Repeatable migrations                     | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
|                           Dry runs                           | ![money icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/money-icon-30x30.png) | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
|                        Preconditions                         |                              –                               | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
|                    Selective deployments                     |                          Many Files                          | One File                                                     |
|                 Manage the order of changes                  |                             Hard                             | Easy                                                         |
|              Ability to work with stored logic               |                              –                               | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
|         Flexibility in how change files are managed          |                              –                               | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |
| [Monitoring & reporting dashboard to view and organize changes](https://hub.liquibase.com/) |                              –                               | ![check mark icon](https://www.liquibase.org/wp-content/uploads/sites/6/2020/04/check-icon-30x30.png) |

### The way changes are defined

Flyway uses SQL exclusively. (This means you need to manually create and specify the SQL you want). Some people love that. Some people find it restricting.

[Liquibase can use plain old SQL](https://www.liquibase.org/plain-sql), but it also allows you to specify the change you want in several different abstract, database-agnostic formats including XML, YAML, and JSON. Liquibase makes it easy to define database changes in a format that’s familiar and comfortable to each user and then automatically generates database-specific SQL for you.

### Script and file management

When it comes to scaling for multiple developers and the ability to branch/merge, Liquibase and Flyway handle this differently.

#### With Flyway, the filename is king.

This can have limitations. Let’s take a closer look.

Flyway is built around a concept of a linear database versioning system which starts at version 1. After a change is added, the version is incremented to 2, then 3, etc. Users often end up doing gymnastics with filenames to manage execution order. **For example, changes 1, 2, 3, 4, 5 are all deployed. Now imagine you want to add a change between 2 and 3,so you add a “2.5”. What happens to environments where 1-5 were already deployed?** Version control systems, like Git, can’t handle this for you. 

Another limitation comes to light when you’re working on a team with other developers and there’s a filename conflict or a change ordering issue. All of these scenarios — adding a new change, filename conflicts, and reordering changes — need to be dealt with manually when using Flyway.

#### With Liquibase, changes are managed with one ledger, referred to as the changelog.

Liquibase uses a unique identification scheme: **an ID and author**, along with the name and path of the file all in a [changelog](https://docs.liquibase.com/concepts/databasechangelog-table.html). This makes it easy to manage the order in which database changes are made. This way of managing the changes makes developer conflicts and collisions less likely since there are multiple factors that drive the uniqueness of a given change. The only possible problems you can encounter are with XML merge conflicts, which can typically be resolved with version control systems like Git. 

With Liquibase, it’s also easy to reorder changes in the changelog when you need to roll out changes to lower environments.

### Rework and selective deployments

Let’s face it, rework happens. It happens to all of us, every day. Nobody gets everything right the first time.

Flyway can be pretty unforgiving when it comes to reworking database changes and performing selective deployments.

#### **Reworking changes**

With Flyway, users have two options for reworking changes: roll everything back or purchase their Pro or Enterprise offerings for undo functionality. With the open source version of Flyway, there’s no real way to automate or clean up an environment where a bad change has been deployed.

The free version of Liquibase allows you to undo changes you have made to your database, either automatically or via custom rollback SQL with the [rollback command.](https://docs.liquibase.com/workflows/liquibase-community/using-rollback.html) Liquibase Pro also adds [Targeted Rollbacks](https://www.liquibase.com/blog/targeted-rollback), which allow users to rollback a specific change or set of specific changes without rolling everything else back.

#### **Selective deployments**

If your use case requires you to selectively deploy changes, it’s harder to do that with Flyway than it is with Liquibase. Let’s say you want to only deploy to a test environment and you’re managing scripts that populate certain configuration or test data to that environment. With Flyway, you would have to set up a different configuration (properties file) for each of the affected environments in order to only apply certain changes to those specific databases. Since Liquibase uses one ledger (changelog), it’s more straightforward to [add labels and contexts](https://www.liquibase.org/blog/contexts-vs-labels) to ensure that this is set up in one place.

#### **Liquibase preconditions**

Liquibase has a powerful feature called [*preconditions*](https://docs.liquibase.com/concepts/preconditions.html). Preconditions allow users to apply changes based on certain conditions such as to check whether or not a table exists or for another example to check whether a column contains data before dropping it. This allows you to control whether or not a change should be applied or not based on the state of the database. A changeset will only execute if the precondition passes. If the precondition fails, you can tell Liquibase what you’d like to happen next, such as halt, mark, ran, etc. Flyway does not do this.

### Stored logic

Liquibase Pro enables users to snapshot and work with Stored Logic. Snapshots allow you to get a static view of your database at a particular point in time and is useful for reporting and safeguarding your data by comparing databases (performing diffs) to find differences.

Liquibase Pro users have Stored Logic *changeSets* in changelogs generated through the [diffChangeLog](https://docs.liquibase.com/commands/community/diff.html) and [generateChangeLog](https://docs.liquibase.com/commands/community/generatechangelog.html) command.

Since Flyway only uses SQL, the expectation is for users to generate the SQL themselves using a database’s native tool to export the SQL. Or alternatively, the developer would just write the stored procedure as a SQL file for Flyway to deploy as needed. SQL files that contain stored logic would have to be named differently than other files (for example, use an “R_” prefix rather than a “V1.2_” prefix.

Flyway also doesn’t have the ability to perform a diff to compare two databases, like Liquibase — Flyway expects that you will also use some external tool to do this. This also means Flyway users can’t do the equivalent of diffChangeLog.

### Monitoring & reporting dashboard

Liquibase offers a free SaaS dashboard called [Liquibase Hub](https://hub.liquibase.com/) that allows teams to easily view, organize, and report on their database change process. Flyway doesn’t offer any dashboard.

## Wrapping it up

Both Liquibase and Flyway are tools that help with managing, tracking, and deploying database schema changes. They are both migration-based tools that are looking to fill a need that many teams have for treating database code like app code with source control and automation.

Flyway can solve your Day One problems: managing, tracking, and deploying database schema changes. Flyway’s file structure and raw SQL focus work fine at the beginning. It’s easy and quick. However, everyone eventually hits the rollback/rework/merge/conditional logic use cases.

**Liquibase has the power and flexibility to handle your Day One problems AND your Day 50 problems more gracefully from the get-go.** Sure, there are workarounds for Flyway to handle some of these more complex needs, but they’re not pretty. Liquibase handles these very typical database schema problems in stride.

There are always tradeoffs to make when choosing between tools. If you are looking for a tool that checks the following boxes, [Liquibase](https://www.liquibase.org/) may be right for you:

- Define database changes in a format that’s familiar and comfortable to each user and then automatically generates database-specific SQL
- Support updating and managing the same schema across multiple database vendors using the same changelog file
- Easily add new changes and reorder them without running into filename conflicts
- Rollback changes without having to purchase a paid offering
- Snapshot and work with Stored Logic (including reverse engineering)
- Selectively deploy changes as needed to different environments
- View real-time database change information on a free dashboard and share it with your team

[Give Liquibase a try](https://www.liquibase.org/download) to take a closer look and check out all the great features we covered in this comparison.

Looking for support and enterprise features? Liquibase offers a range of solutions for every team and use case. [View Liquibase products and pricing](https://www.liquibase.com/products).