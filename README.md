# cop2806c Module 2 Graded Assignment
 
## Introduction

In this assignment we will use Spring Data JPA to create a simple database 
application which acquires and saves customer data. We will use the H2 
in-memory database with a backing file to store the data. 

The application starts with a Spring Web framework which uses the embedded
Tomcat server, then adds JPA using Hibernate and H2 with Spring Data JPA

Start with a Spring Initializr Project in IntelliJ. 

On the New Project/Spring Initializr Project Settings page, use the following settings:

- name the project group which will be used to create your Java package. 
	- The naming convention for the group name in this course uses a 
	- reversed domain name, which is a standard convention for naming packages:

             edu.fscj.cop2806c
		
- Set the Artifact name. 
	- I used the same name as I did for the project (see below), "firstjpa".
- Set the type to Maven
- Set the Language to Java
- Set the packaging to Jar
- Provide a brief desciption of the project

On the Dependencies page, add the Web/Spring Web dependency to the project.

- Provide a project name of your choice
	- I used "firstjpa", same as the artifact name provided earlier.
	
The template project is now created. Note the Maven dependencies (click on the 
Maven tab).

Use the editor to open the file containing the main method.

## Add a Method to Welcome the User

```

...
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class FirstjpaApplication {
    ...
@GetMapping("/welcome")
public String welcome(
        @RequestParam(value = "myName", defaultValue = "everyone") String name) {
        return String.format("Welcome to COP2806C, %s!", name);
 }

```

The welcome() method takes the name parameter and returns the welcome message combined 
with the parameter value. Everything else is handled by adding Spring annotations:
- The @RestController annotation marks the DemoApplication class as a request handler (a REST controller).
- The @GetMapping("/welcome") annotation maps the sayHello() method to GET requests for /welcome (localhost:8080/welcome)
- The @RequestParam annotation maps the name method parameter to the myName web request parameter. If you don't provide the myName parameter in your web request, it will default to "everyone".

Try providing a value, e.g.
 localhost:8080/welcome?myName=David

# Add a Static Home Page
- Add a static HTML home page with links to the endpoint so we don't have to type it in the URL
- Create an index.html file under /src/main/resources/static/
	- In the Project tool window, right-click the /src/main/resources/static/ directory, 
	  select New | HTML File, specify the name index.html, and press Enter.

# Modify the index.html Template
```

<!DOCTYPE HTML>
<html>
<head>
    <title>First JPA Application</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p><a href="/welcome">Say hello!</a></p>
<form action="/welcome" method="GET" id="nameForm">
    <div>
        <label for="nameField">What should the app call you?</label>
        <input name="myName" id="nameField">
        <button>Say hello!</button>
    </div>
</form>
</body>
</html>

```

# Run the application and connect
- IntelliJ creates a Spring Boot run configuration that you can use to run your
  new Spring application. It should be selected by default, so you can press
  Shift+F10. You can also use the Run icon in the gutter of the Java file
  next to the class declaration or the main() method declaration.
- The Say Hello dialog will display and allow you to enter your name

# Adding JPA/H2
- Now we will dependencies for JPA and H2 that enable the Spring application
 to store and retrieve relational data.
- H2 is an open-source, lightweight Java database that can be run in memory
  and used for development and testing that can be embedded in Java applications
  or run in client-server mode. 

# Add the JPA/H2 Dependencies
- Open the pom.xml file in your project root directory.
- From the main menu, select Code | Generate Alt+Insert and then select Dependency.
- In the Maven Artifact Search dialog, find and add the necessary Maven dependencies:
	- org.springframework.boot:spring-boot-starter-data-jpa
	- com.h2database:h2
	- javax.persistence
- Alternatively, you can add these dependencies to pom.xml manually:
```

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>

```

- Click The Load Maven Changes button in the popup or press Ctrl+Shift+O 
  to load the updated project structure.

# Create an Entity
- Create a JPA Customer entity that will be stored in the database﻿.
- The application will create and store Customer objects that have an 
  ID, a first name, and a last name.

- To create the Customer class:

	- In the Project tool window, select the

		/src/main/java/edu/fscj/cop2806c/firstjpa 

      (substitute your project name for firstjpa if different) directory and
	  then select File | New | Java Class from the main menu

	- Select Class and type the name of the class: Customer

Modify the default template with the following Java code:
```

package edu.fscj.cop2806c.firstjpa;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String firstName;
    private String lastName;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}

```

# Create a Repository Interface
- Spring Data JPA creates an implementation from a Repository interface at runtime.

- In the Project tool window, select the 

        /src/main/java/edu/fscj/cop2806c/firstjpa 

   directory and then select File | New | Java Class from the main menu 

- Select Interface and type the name of the interface: CustomerRepository.
- Modify the default template with the following Java code:

```

package edu.fscj.cop2806c.firstjpa;

import org.springframework.data.repository.CrudRepository;

public interface CustomerRepository 
              extends CrudRepository<Customer, Integer> {
    Customer findCustomerById(Integer id);
}

```

- This repository works with Customer entities and an Integer ID.
	- It also declares the findCustomerById() method.

- Spring Data JPA will derive a query based on this method's signature, which will select the Customer object for the specified ID.

- You can try adding other methods to see how IntelliJ provides completion suggestions based on available JPA entities (in this case, the Customer class).

# Create a Web Controller﻿
- A controller handles HTTP requests for a Spring application. 
- The application will use the /add endpoint to add Customer objects 
  to the database, the /list endpoint to fetch all Customer objects 
  from the database, and the /find/{id} endpoint to find the customer 
  with the specified ID.
- Create a class named DemoController with content shown here:

```

package edu.fscj.cop2806c.firstjpa;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
public class DemoController {

    @Autowired
    private CustomerRepository customerRepository;

    @PostMapping("/add")
    public String addCustomer(@RequestParam String first, @RequestParam String last) {
        Customer customer = new Customer();
        customer.setFirstName(first);
        customer.setLastName(last);
        customerRepository.save(customer);
        return "Added new customer to repo!";
    }

    @GetMapping("/list")
    public Iterable<Customer> getCustomers() {
        return customerRepository.findAll();
    }

    @GetMapping("/find/{id}")
    public Customer findCustomerById(@PathVariable Integer id) {
        return customerRepository.findCustomerById(id);
    }
}

```

# Spring Annotations for Web Controller
- The @Autowired annotation tells Spring to inject the customerRepository bean, 
  which is implemented from the repository interface. IntelliJ designates it 
  with The autowired dependencies icon in the gutter, which you can click to 
  navigate to the corresponding autowired dependency.
- The @PostMapping("/add") annotation maps the addCustomer() method to POST 
  requests for /add.
- The @GetMapping("/list") annotation maps the getCustomers() method to GET 
  requests for /list.
- The @GetMapping("/find/{id}") annotation maps the findCustomerById() method 
  to GET requests for /find/{id}.
- The @PathVariable annotation maps the value in place of the id variable from 
  the URL to the corresponding method parameter.
- IntelliJ designates the web request methods with a request mapping icon in 
  the gutter, which you can click to execute the corresponding request.
  
# Run the Application
- Press Shift+F10 or use the Run icon in the gutter to run your application.
- By default, IntelliJ shows your running Spring Boot application in the Run tool 
  window.
- The Console tab shows the output of Spring log messages. By default, the 
  built-in Apache Tomcat server is listening on port 8080.
- Open a web browser and go to http://localhost:8080/list. You should see your 
  application respond with an empty list [ ] because you don't have any customers
  in the database.

# Adding Customers and Querying the DB
- To add customers, send a POST request to the /add endpoint with request 
  parameters that specify the first and last name. 
- IntelliJ has an HTTP client built into the editor to compose and execute
  HTTP requests.
	- Open DemoController.java and click The Open in HTTP Request Editor icon
      in the gutter next to the addCustomer() method.
	- In the request file, compose the following POST request:

```
        POST http://localhost:8080/add?first=Homer&last=Simpson
```

- Click The Run HTTP Request icon in the gutter to execute the request.
	- IntelliJ opens the Run tool window and prints both the request and the response:

```
		POST http://localhost:8080/add?first=Homer&last=Simpson
		...
		Saved new customer!
```

- Open the HTTP requests file again and compose a GET request to fetch all customers:

```
		GET http://localhost:8080/list
		...
		[
			{
				"id": 1,
				"firstName": "Homer",
				"lastName": "Simpson"
			}
		]
```

- Try running more POST requests to add other customers. Each one will get a unique ID. 
- Then compose and execute a GET request to return a customer with a specific ID. 
- For example:

```
		GET http://localhost:8080/find/1

```

# TODO
- Other things to try:
	- Implement more classes with relationships, e.g. one-to-one and one-to-many,
	  then add/access via the Web Controller
	- Integrate app with a different database (e.g. mySQL or MSSQL Server)
	- Implement a CommandLineRunner (more convenient execution)




