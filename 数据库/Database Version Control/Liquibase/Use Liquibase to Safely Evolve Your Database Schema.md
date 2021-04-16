# Use Liquibase to Safely Evolve Your Database Schema

> 转载：[Use Liquibase to Safely Evolve Your Database Schema](https://www.baeldung.com/liquibase-refactor-schema-of-java-app)

## **1. Overview**

In this quick tutorial, we'll make use of **[Liquibase](http://www.liquibase.org/) to evolve the database schema** of a Java web application.

We're going to focus on a general Java app first, and we're also going to take a focused look at some interesting options available for Spring and Hibernate.

Very briefly, the core of using Liquibase is **the `changeLog` file** – an XML file that keeps track of all changes that need to run to update the DB.

Let's start with the Maven dependency we need to add into our *pom.xml*:

```xml
<dependency>
    <groupId>org.liquibase</groupId>
     <artifactId>liquibase-core</artifactId>
      <version>3.4.1</version>
</dependency>
```

We can also check if there's a newer version of liquibase-core [here](https://mvnrepository.com/artifact/org.liquibase/liquibase-core).

## **2. The Database Change Log**

Now, let's take a look at a simple *changeLog* file – this one only adds a column “*address*” to the table “*users*“:

```java
<databaseChangeLog 
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog" 
  xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog-ext
   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd 
   http://www.liquibase.org/xml/ns/dbchangelog 
   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">
    
    <changeSet author="John" id="someUniqueId">
        <addColumn tableName="users">
            <column name="address" type="varchar(255)" />
        </addColumn>
    </changeSet>
    
</databaseChangeLog>
```

Note how the change set is identified by an *id* and an *author* – to make sure it can be uniquely identified and only applied once.

Let's not see how to wire this into our application and make sure that it runs when the application starts up.

## **3. Run Liquibase With a Spring Bean**

Our first option to run the changes on application startup is via a Spring bean. There are of course many other ways, but if we're dealing with a Spring application – this is a good, simple way to go:

```java
@Bean
public SpringLiquibase liquibase() {
    SpringLiquibase liquibase = new SpringLiquibase();
    liquibase.setChangeLog("classpath:liquibase-changeLog.xml");
    liquibase.setDataSource(dataSource());
    return liquibase;
}
```

Note how we're pointing it to a valid *changeLog* file that needs to exist on the classpath.

## **4. Use Liquibase With Spring Boot**

If we're using [Spring Boot](http://liquibase.change-log%3Dclasspath:liquibase-changeLog.xml/), there is no need to define a *bean* for Liquibase, but we still need to make sure we add the `liquibase-core` dependency.

Then, all we need is to put our change log in “**db/changelog/db.changelog-master.yaml**” and Liquibase migrations will run automatically on startup.

We can change the default changelog file using the “**liquibase.change-log**” property – for example:

```bash
liquibase.change-log=classpath:liquibase-changeLog.xml
```

## 5. Disable Liquibase in Spring Boot

Sometimes, we may need to disable Liquibase migration's execution on startup.

**The simplest option we have is to use a `spring.liquibase.enabled` property**. This way, all the remaining Liquibase configuration stays untouched.

Here's the example for Spring Boot 2:

```java
spring.liquibase.enabled=false
```

For Spring Boot 1.x, we need to use a *liquibase.enabled* property:

```java
liquibase.enabled=false
```

## **6. Generate the `changeLog` With a Maven Plugin**

Instead of writing the *changeLog* file manually, we can use the Liquibase Maven plugin to generate one and save ourselves a lot of work.

### **6.1. Plugin Configuration**

Here are the changes to our *pom.xml*:

```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-maven-plugin</artifactId>
    <version>3.4.1</version>
</dependency> 
...
<plugins>
    <plugin>
        <groupId>org.liquibase</groupId>
        <artifactId>liquibase-maven-plugin</artifactId>
        <version>3.4.1</version>
        <configuration>                  
            <propertyFile>src/main/resources/liquibase.properties</propertyFile>
        </configuration>                
    </plugin> 
</plugins>
```

### **6.2. Generate a `ChangeLog` From an Existing Database**

We can use the plugin to generate a changelog from an existing database:

```bash
mvn liquibase:generateChangeLog
```

Here are the *liquibase* properties:

```bash
url=jdbc:mysql://localhost:3306/oauth_reddit
username=tutorialuser
password=tutorialmy5ql
driver=com.mysql.jdbc.Driver
outputChangeLogFile=src/main/resources/liquibase-outputChangeLog.xml
```

The end result is a *changeLog* file that we can use either to create an initial DB schema or to populate data. Here's how that would look like for our example app:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<databaseChangeLog ...>
    
    <changeSet author="John (generated)" id="1439225004329-1">
        <createTable tableName="APP_USER">
            <column autoIncrement="true" name="id" type="BIGINT">
                <constraints primaryKey="true"/>
            </column>
            <column name="accessToken" type="VARCHAR(255)"/>
            <column name="needCaptcha" type="BIT(1)">
                <constraints nullable="false"/>
            </column>
            <column name="password" type="VARCHAR(255)"/>
            <column name="refreshToken" type="VARCHAR(255)"/>
            <column name="tokenExpiration" type="datetime"/>
            <column name="username" type="VARCHAR(255)">
                <constraints nullable="false"/>
            </column>
            <column name="preference_id" type="BIGINT"/>
            <column name="address" type="VARCHAR(255)"/>
        </createTable>
    </changeSet>
    ...
</databaseChangeLog>
```

### **6.3. Generate a `ChangeLog` From Diff Between Two Databases**

We can use the plugin to generate a *changeLog* file from the differences between two existing databases (for example development and production):

```bash
mvn liquibase:diff
```

Here are the properties:

```bash
changeLogFile=src/main/resources/liquibase-changeLog.xml
url=jdbc:mysql://localhost:3306/oauth_reddit
username=tutorialuser
password=tutorialmy5ql
driver=com.mysql.jdbc.Driver
referenceUrl=jdbc:h2:mem:oauth_reddit
diffChangeLogFile=src/main/resources/liquibase-diff-changeLog.xml
referenceDriver=org.h2.Driver
referenceUsername=sa
referencePassword=
```

And here's a snippet of the generated *changeLog*:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<databaseChangeLog ...>
    <changeSet author="John" id="1439227853089-1">
        <dropColumn columnName="address" tableName="APP_USER"/>
    </changeSet>
</databaseChangeLog>
```

This is a super powerful way to evolve our DB by – for example – allowing Hibernate to auto-generate a new schema for development, and then using that as a reference point against the old schema.

## **7. Use the Liquibase Hibernate Plugin**

If the application uses Hibernate – we're going to take a look at a very useful way of generating the *changeLog*, which is [the *liquibase-hibernate* plugin](https://github.com/liquibase/liquibase-hibernate/wiki).

### **7.1. Plugin Configuration**

First, let's get the new plugin configured and using the right dependencies:

```xml
<plugins>
    <plugin>
        <groupId>org.liquibase</groupId>
        <artifactId>liquibase-maven-plugin</artifactId>
        <version>3.4.1</version>
        <configuration>                  
            <propertyFile>src/main/resources/liquibase.properties</propertyFile>
        </configuration> 
        <dependencies>
            <dependency>
                <groupId>org.liquibase.ext</groupId>
                <artifactId>liquibase-hibernate4</artifactId>
                <version>3.5</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>4.1.7.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-jpa</artifactId>
                <version>1.7.3.RELEASE</version>
            </dependency>
        </dependencies>               
    </plugin> 
</plugins>
```

### **7.2. Generate a `changeLog` from Diffs Between a Database and Persistence Entities**

Now, for the fun part. We can use this plugin to generate a *changeLog* file from the differences between an existing database (for example production) and our new persistence entities.

So – to make things simple – once an entity is modified, we can simply generate the changes against the old DB schema, getting **a clean, powerful way to evolve our schema in production**.

Here are the liquibase properties:

```bash
changeLogFile=classpath:liquibase-changeLog.xml
url=jdbc:mysql://localhost:3306/oauth_reddit
username=tutorialuser
password=tutorialmy5ql
driver=com.mysql.jdbc.Driver
referenceUrl=hibernate:spring:org.baeldung.persistence.model
  ?dialect=org.hibernate.dialect.MySQLDialect
diffChangeLogFile=src/main/resources/liquibase-diff-changeLog.xml
```

Note that the *referenceUrl* is using package scan, so the *dialect* parameter is required.

## 8. Generate the changeLog in IntelliJ IDEA Using the JPA Buddy Plugin

If we're using a non-Hibernate ORM (e.g. EclipseLink or OpenJPA) or we don't want to add extra dependencies like the *liquibase-hibernate* plugin, we can use [JPA Buddy](https://www.baeldung.com/jpa-buddy-post). **This IntelliJ IDEA plugin integrates useful features of Liquibase into the IDE.**

To generate a differential *changeLog,* simply install the plugin, then call the action from the JPA Structure panel. Select what source you want to compare (database, JPA entities, or Liquibase snapshot) with what target (database or Liquibase snapshot).

**JPA Buddy will generate the \*changeLog\* as shown in the animation below:**

![](https://www.baeldung.com/wp-content/uploads/2021/04/jpabuddy_intellij.gif)

Another advantage of JPA Buddy over the *liquibase-hibernate* plugin is the ability to override default mappings between Java and database types. Also, it works correctly with Hibernate custom types and JPA converters.

## **9. Conclusion**

In this article, we illustrated several ways to use Liquibase and get to a safe and mature way of **evolving and refactoring the DB schema of a Java app**.

The implementation of all these examples and code snippets is available [over on GitHub](https://github.com/Baeldung/reddit-app).

