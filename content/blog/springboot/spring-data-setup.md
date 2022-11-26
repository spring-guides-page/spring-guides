---
title: "Accessing databases using Spring Data JPA"
date: 2022-11-26T17:36:52+01:00
author: "Florian M."
tags: ["Data"]
---
## Introduction
Whenever we need to handle persistent data in our application a Database connection of some kind
is required.</br>
Spring Data helps us to deal with those situations by providing an easy-to-use suite of libraries into its own
Framework's architecture.

We are going to focus on the JPA flavor of Spring Data in this article as it's easiest to understand when coming from
non-Spring based Projects (Using Hibernate directly for example).

You can also add automatic [Database schema migrations using Liquibase]({{< ref "/blog/springboot/spring-liquibase" >}}) after integrating Spring Data into your application.

## Integrating Spring Data JPA
Spring Data JPA can be added as a dependency in your `build.gradle` file. You can add the H2 Database dependency for 
local testing as well:

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    // Optional: A JDBC Driver for the specific Database you are trying to connect to
    // implementation 'com.mysql:mysql-connector-j'
    // implementation 'org.mariadb.jdbc:mariadb-java-client'
    // implementation 'com.microsoft.sqlserver:mssql-jdbc'

    // Optional, for local testing
    implementation 'com.h2database:h2'
}
```

## Connecting to your Database
Spring creates a Datasource object for creating connections to the Database.
The connection parameters can be configured in the `application.properties` file.</br>
We are using a local H2 Database instance in this example:

```properties
spring.datasource.url=jdbc:h2:file:./database
spring.datasource.username=sa
spring.datasource.password=
```

## Using Spring Data JPA
### Creating Entity objects
JPA annotations are used to create a mapping between Java Objects and Database entries.
We are going to create a simple Customer object which matches our Database schema.
Create a new entity class called `Customer` in your project:

```java
@Entity
@Table(name = "RegisteredCustomers")
public class Customer {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column(name = "firstName")
	private String firstName;

	@Column(name = "lastName")
	private String lastName;

	public Customer() {
		// Default constructor for Hibernate
	}

	public Customer(String firstName, String lastName) {
		this.firstName = firstName;
		this.lastName = lastName;
	}
	
	// Getters omitted
}
```

We are using the @Entity annotation on our class to tell Spring (to be exact: Hibernate which comes with Spring Data) that
this class maps to some entry in our database.</br>
Using the @Table annotation we are specifying to which table's entries it should be mapped.

### Setting up the Repository
To interact with the entities and our database we first need to create a Repository interface.
These implement some CRUD operations by default and can be extended at will.</br>
We need to create a `CustomerRepository` interface with a simple custom query for this example:

```java
@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {

    List<Customer> findByLastName(String lastName);

}
```

As you can see we are only extending the default JpaRepository and specifying the two type parameters `<Customer, Long>`:</br>
`Customer` as our entity which is handled by this interface and `Long` as the primary key type being used for this entity.

We also added a single Method called `findByLastName` with a parameter `lastName` to our interface.
The naming of this method is important as Spring is able to derive the `SQL SELECT` Query just from the method name itself -
no implementation or special `@Query` annotation needed.

### Building our Service Layer
Service classes are the only Database-related classes which should be visible to the rest of our business logic.
