---
title: "Migrating Databases with Liquibase"
date: 2022-11-26T11:23:52+01:00
author: "Florian M."
tags: ["Data"]
brief: "Integrating automated Database Migrations into Spring using Liquibase"
---

Updating an application sometimes requires changing the underlying database schema.
We will cover how to implement automated database migrations using Liquibase in Spring Boot.

## Liquibase - a short Introduction
Liquibase is used in many different Java applications to perform automated database migrations on application startup.</br>
This is done by integrating so-called Changelog files in the application's executable.</br> 
These Changelogs can contain multiple Changesets which contain the logic for transforming the database schema.

Liquibase transforms these Changesets into SQL queries at runtime, independent of the database type being used.

## Prerequisites
As Liquibase uses the application's database connection a Spring managed Datasource is required to run Changesets.
This is done by [integrating Spring Data]({{< ref "/blog/springboot/spring-data-setup" >}}) into your project.

## Integrating Liquibase
### Adding the Dependency
Add Liquibase as a dependency in your project's `build.gradle` file:
```groovy
dependencies {
    implementation 'org.liquibase:liquibase-core'
}
```
There is no need to specify a version of the dependency as Liquibase is a Spring-managed dependency and is kept
up-to-date automatically when updating the Spring Boot version of your project.

### Configuring the application
Liquibase searches for the file `db/changelog/db.changelog-master.yaml` on the application's classpath for Changelogs by default.</br>
As we are going to use XML as our Changelog format we have to add the following line to the `application.properties` file in our project:
```properties
spring.liquibase.change-log=classpath:/db/changelog/db.changelog-master.xml
```

## Writing your first Changelog
Create a new file called `db.changelog-master.xml` under `src/main/resources/db/changelog/` in your application.
The following Changelog contains one changeset which creates a new Table called `RegisteredCustomers`:

```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
		http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd">

    <changeSet author="DB-Author-1" id="AddRegisteredCustomersTable">
        <createTable tableName="RegisteredCustomers">
            <column name="id" type="BIGINT" autoIncrement="true">
                <constraints primaryKey="true"/>
            </column>
            <column name="firstName" type="VARCHAR(64)" />
            <column name="lastName" type="VARCHAR(64)" />
        </createTable>
    </changeSet>

</databaseChangeLog>
```