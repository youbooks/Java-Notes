# Database Migrations with Flyway

> 转载：[Database Migrations with Flyway](https://www.baeldung.com/database-migrations-with-flyway)

## **1. Introduction**

This article describes key concepts of [**Flyway**](https://flywaydb.org/) and how we can use this framework to continuously remodel our application's database schema reliably and easily. At the end, we'll present an example of managing an in-memory H2 database using a Maven Flyway plugin.

Flyway updates a database from one version to a next using migrations. We can write migrations either in SQL with database-specific syntax or in Java for advanced database transformations.

Migrations can either be versioned or `repeatable`. The former has a unique version and is applied exactly once. The latter does not have a version. Instead, they are (re-)applied every time their checksum changes.

Within a single migration run, repeatable migrations are always applied last, after pending versioned migrations have been executed. Repeatable migrations are applied in order of their description. For a single migration, all statements are run within a single database transaction.

In this article, we mainly focus on `how we may use the Maven plugin to perform database migrations`.

## **2. Flyway Maven Plugin**

To install a Flyway Maven plugin, let's add the following plugin definition to our *pom.xml:*

```xml
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>4.0.3</version> 
</plugin>
```

We can check the latest version of the plugin available at [Maven Central](https://mvnrepository.com/artifact/org.flywaydb/flyway-maven-plugin).

This Maven plugin may be configured in four different ways. Please refer to the [documentation](https://flywaydb.org/documentation/usage/maven/migrate) to get a list of all configurable properties.

### **2.1. Plugin Configuration**

We may configure the plugin directly via the *<configuration>* tag in the plugin definition of our *pom.xml:*

```xml
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>4.0.3</version>
    <configuration>
        <user>databaseUser</user>
        <password>databasePassword</password>
        <schemas>
            <schema>schemaName</schema>
        </schemas>
        ...
    </configuration>
</plugin>
```

### **2.2. Maven Properties**

We may also configure the plugin by specifying configurable properties as Maven *properties* in our pom:

```xml
<project>
    ...
    <properties>
        <flyway.user>databaseUser</flyway.user>
        <flyway.password>databasePassword</flyway.password>
        <flyway.schemas>schemaName</flyway.schemas>
        ...
    </properties>
    ...
</project>
```

### **2.3. External Configuration File**

We may also provide plugin configuration in a separate *.properties* file:

```plaintext
flyway.user=databaseUser
flyway.password=databasePassword
flyway.schemas=schemaName
...
```

The default configuration file name is *flyway.properties* and it should reside in the same directory as the *pom.xml* file. Encoding is specified by *flyway.encoding* (Default is *UTF-8*).

If you are using any other name (e.g *customConfig.properties*) as the configuration file, then it should be specified explicitly when invoking the Maven command:

```plaintext
$ mvn -Dflyway.configFile=customConfig.properties
```

### **2.4. System Properties**

Finally, all configuration properties may also be specified as system properties when invoking Maven on the command line:

```bash
$ mvn -Dflyway.user=databaseUser -Dflyway.password=databasePassword 
  -Dflyway.schemas=schemaName
```

Following is an order of precedence when a configuration is specified in more than one way:

1. System properties
2. External configuration file
3. Maven properties
4. Plugin configuration

## **3. Example Migration**

In this section, we walk through the required steps to migrate a database schema to an in-memory H2 database using the Maven plugin. We use an external file to configure Flyway.

### **3.1. Update POM**

First, let's add H2 as a dependency:

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
```

We can, again, check the latest version of the driver available on [Maven Central](https://search.maven.org/classic/#search|ga|1|g%3A"com.h2database" AND a%3A"h2"). We'd also add the Flyway plugin as explained earlier.

### **3.2. Configure Flyway Using External File**

Next, we create *myFlywayConfig.properties* in *$PROJECT_ROOT* with the following content:

```xml
flyway.user=databaseUser
flyway.password=databasePassword
flyway.schemas=app-db
flyway.url=jdbc:h2:mem:DATABASE
flyway.locations=filesystem:db/migration
```

The above configuration specifies that our migration scripts are located in the *db/migration* directory. It connects to an in-memory H2 instance using *databaseUser* and *databasePassword*.

The application database schema is *app-db*.

Of course, we replace *flyway.user, flyway.password,* and *flyway.url* with our own database username, database password, and database URL appropriately.

### **3.3. Define First Migration**

Flyway adheres to the following naming convention for migration scripts:

*<Prefix><Version>__<Description>.sql*

Where:

- *<Prefix>* – Default prefix is *V*, which may be configured in the above configuration file using the *flyway.sqlMigrationPrefix* property.
- *<Version>* – Migration version number. Major and minor versions may be separated by an *underscore*. The migration version should always start with 1.
- *<Description>* – Textual description of the migration. The description needs to be separated from the version numbers with a double underscore.

Example: *V1_1_0__my_first_migration.sql*

So, let's create a directory *db/migration* in *$PROJECT_ROOT* with a migration script named `V1_0__create_employee_schema.sql` containing SQL instructions to create the employee table:

```sql
CREATE TABLE IF NOT EXISTS `employee` (

    `id` int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `name` varchar(20),
    `email` varchar(50),
    `date_of_birth` timestamp

)ENGINE=InnoDB DEFAULT CHARSET=UTF8;
```

### **3.4. Execute Migrations**

Next, we invoke the following Maven command from *$PROJECT_ROOT* to execute database migrations:

```bash
$ mvn clean flyway:migrate -Dflyway.configFile=myFlywayConfig.properties
```

This should result in our first successful migration.

The database schema should now be depicted as follows:

```plaintext
employee:
+----+------+-------+---------------+
| id | name | email | date_of_birth |
+----+------+-------+---------------+
```

We can repeat definition and execution steps to do more migrations.

### **3.5. Define and Execute Second Migration**

Let's see what a second migration looks like by creating a second migration file with name `V2_0_create_department_schema.sql` containing the following two queries:

```sql
CREATE TABLE IF NOT EXISTS `department` (

`id` int NOT NULL AUTO_INCREMENT PRIMARY KEY,
`name` varchar(20)

)ENGINE=InnoDB DEFAULT CHARSET=UTF8; 

ALTER TABLE `employee` ADD `dept_id` int AFTER `email`;
```

We'll execute a similar migration like we did the first time.

And now, our database schema has changed to add a new column to *employee* and a new table:

```plaintext
employee:
+----+------+-------+---------+---------------+
| id | name | email | dept_id | date_of_birth |
+----+------+-------+---------+---------------+
department:
+----+------+
| id | name |
+----+------+
```

We may now verify that both migrations were indeed successful by invoking the following Maven command:

```plaintext
$ mvn flyway:info -Dflyway.configFile=myFlywayConfig.properties
```

## 4. Disabling Flyway in Spring Boot

Sometimes we may need to **disable Flyway migrations under certain circumstances**.

For example, it's a common practice to generate database schema based on the entities during tests. In such a situation, we can disable Flyway under the *test* profile.

Let's see **how easy it is in Spring Boot**.

### 4.1. Spring Boot 1.x

All we need to do is to **set the \*flyway.enabled\* property in our \*application-test.properties\* file**:

```plaintext
flyway.enabled=false
```

### 4.2. Spring Boot 2.x

In the more recent versions of Spring Boot, **this property has been changed to \*spring.flyway.enabled\*:**

```plaintext
spring.flyway.enabled=false
```

### 4.3 Empty *FlywayMigrationStrategy*

If we only want to **disable automatic Flyway migration on startup, but still be able to trigger the migration manually**, then using the properties described above isn't a good choice.

That's because in such a situation Spring Boot will not auto-configure the *Flyway* bean anymore. Consequently, we'd have to provide it on our own which isn't very convenient.

So if this is our use case, we can leave Flyway enabled and implement an empty [*FlywayMigrationStrategy*](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/flyway/FlywayMigrationStrategy.html):

```java
@Configuration
public class EmptyMigrationStrategyConfig {

    @Bean
    public FlywayMigrationStrategy flywayMigrationStrategy() {
        return flyway -> {
            // do nothing  
        };
    }
}
```

This will effectively **disable Flyway migration on application startup**.

But we'll still be able to trigger the migration manually:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ManualFlywayMigrationIntegrationTest {

    @Autowired
    private Flyway flyway;

    @Test
    public void skipAutomaticAndTriggerManualFlywayMigration() {
        flyway.migrate();
    }
}
```

## **5. How Flyway Works**

To keep track of which migrations have already been applied, when and by whom, it adds a special bookkeeping table to your schema. This metadata table also tracks migration checksums and whether or not the migrations were successful.

The framework performs the following steps to accommodate evolving database schemas:

1. It checks a database schema to locate its metadata table (*SCHEMA_VERSION* by default). If the metadata table does not exist, it will create one
2. It scans an application classpath for available migrations
3. It compares migrations against the metadata table. If a version number is lower or equal to a version marked as current, it is ignored
4. It marks any remaining migrations as pending migrations. These are sorted based on version number and are executed in order
5. As each migration is applied, the metadata table is updated accordingly

## **6. Commands**

Flyway supports the following basic commands to manage database migrations.

- **Info:** Prints current status/version of a database schema. It prints which migrations are pending, which migrations have been applied, what is the status of applied migrations and when they were applied.
- **Migrate:** Migrates a database schema to the current version. It scans the classpath for available migrations and applies pending migrations.
- **Baseline:** Baselines an existing database, excluding all migrations, including *baselineVersion*. Baseline helps to start with Flyway in an existing database. Newer migrations can then be applied normally.
- **Validate:** Validates current database schema against available migrations.
- **Repair:** Repairs metadata table.
- **Clean:** Drops all objects in a configured schema. All database objects are dropped. Of course, you should never use clean on any production database.

## **7. Conclusion**

In this article, we've shown how Flyway works and how we can use this framework to remodel our application database reliably.

The code accompanying this article is available on [GitHub](https://github.com/eugenp/tutorials/tree/master/persistence-modules/flyway).

