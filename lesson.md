# Lesson: Web Service Development and API Design

## Lesson Overview
This lesson builds upon the foundational REST API concepts from the previous lesson and teaches students how to implement complete CRUD (Create, Read, Update, Delete) operations for managing resources. Students will learn the differences between controller annotations, handle HTTP request/response properly using ResponseEntity, implement custom exception handling, and use Lombok to reduce boilerplate code. By working through a practical Customer Resource Management (CRM) example, students will gain hands-on experience building production-ready REST APIs that follow industry best practices for status codes, error handling, and code organization.

---

## Lesson Objectives
By the end of this lesson, students will be able to:
- Differentiate between @Component, @Controller, and @RestController annotations
- Implement complete CRUD operations (Create, Read, Update, Delete) for REST resources
- Use ResponseEntity to return appropriate HTTP status codes (200, 201, 204, 404)
- Create and handle custom exceptions for better error management
- Apply Lombok annotations to reduce boilerplate code in POJOs

---

## Part 1: `@Component`, `@Controller` and `@RestController`

When we annotate with `@Component` or `@Controller`, we are telling Spring Boot to create a bean for us. Recall that a bean is an object that is managed by Spring Boot.

The `@Controller` is actually just a specialized alias of `@Component`. It is used to indicate that a particular class serves the role of a controller.

When we use `@Controller`, our handler methods are expected to return a view. This is useful when we are building a web application that returns HTML pages directly.

To see how this works, let's add a `HomeController.java`.
```java
@Controller
public class HomeController {

  @GetMapping("/home")
  public String home() {
    return "home";
  }
}
```

This will return the `home.html` page in the `resources/templates` folder.

To return a view, we need to add the `spring-boot-starter-thymeleaf` dependency in `pom.xml`.
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Once this is done, a view resolver is automatically configured for us. This means that we can return a view name in our handler method, and Spring Boot will automatically look for the view in the `resources/templates` folder.

You can try to create a `home.html` file in the `resources/templates` folder and load the page at `http://localhost:8080/home`.

If you want to know more about Thymeleaf, you can read the [documentation](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html).

But since we are building a REST API, we will not be returning views. Instead, we will be returning data. The `@ResponseBody` annotation tells a controller that the object returned is automatically serialized into JSON and passed back into the HttpResponse object.
```java
@Controller
@ResponseBody
public class HomeController {

  @GetMapping("/home")
  public String home() {
    return "home";
  }
}
```

The `@RestController` annotation is a convenience annotation that combines `@Controller` and `@ResponseBody`, so we could do this instead:
```java
@RestController // @Controller + @ResponseBody
public class HomeController
```

---

## Part 2: Building Our `simple-crm`

If you have not done the last activity from the previous lesson, you can start creating a new Spring Boot project now.

### `Customer` POJO

Create our `Customer` POJO.
```java
public class Customer {
  private String id;
  private String firstName;
  private String lastName;
  private String email;
  private String contactNo;
  private String jobTitle;
  private int yearOfBirth;

  // Generate getters and setters
}
```

### Storing `Customer` objects

We will use an `ArrayList` to store our `Customer` objects in `CustomerController.java`.
```java
@RestController
public class CustomerController {

  private ArrayList<Customer> customers = new ArrayList<>();

}
```

We will use this as a datastore for now in order to create, read, update, and delete data (CRUD).

### Create

To let our user create a customer by calling an API, we need a `POST` endpoint.
```java
@PostMapping("/customers")
public Customer createCustomer(Customer customer) {
    customers.add(customer);
    return customer;
}
```

Send a `POST` request to `http://localhost:8080/customers` with the following payload:
```json
{
  "id": "123",
  "firstName": "Bruce",
  "lastName": "Banner",
  "email": "bruce@avengers.com",
  "contactNo": "12345678",
  "jobTitle": "Scientist",
  "yearOfBirth": "1975"
}
```

Send the request and check the response. Is it what you expected?

When Postman sends us data, it sends it as a `JSON`. But in our handler method, we are expecting a `Customer` object. Our application does not know how to convert the `JSON` into a `Customer` object. We need to tell our application how to do this.

This is done by adding the `@RequestBody` annotation to our handler method.
```java
@PostMapping("/customers")
public Customer createCustomer(@RequestBody Customer customer) {
    customers.add(customer);
    return customer;
}
```

The `@RequestBody` annotation tells our application to convert the JSON into a `Customer` object. Spring Boot is now able to de-serialize the JSON into a `Customer` object, which is why we are able to add it to our `customers` list.

#### `uuid`

Currently, we are manually setting the `id` of our `Customer` object. We can use the `UUID` class to generate a unique id for us whenever a new `Customer` object is created.
```java
import java.util.UUID;

public Customer() {
  this.id = UUID.randomUUID().toString();
}
```

Let's also make the `id` field `final` so that it cannot be changed once it is set. The corresponding setter method can be removed.
```java
private final String id;
```

Now try to create a new `Customer` object using Postman. What is the `id` of the new `Customer` object?

### Read

For read, we will usually create 2 endpoints. One to get all the objects, and another to get a specific object.

#### Get all customers
```java
@GetMapping("/customers")
public ArrayList<Customer> getAllCustomers() {
    return customers;
}
```

Let's preload some data into our `customers` list by adding them to the constructor. We can just add the first names and last names.
```java
public CustomerController() {
    customers.add(new Customer("Bruce", "Banner"));
    customers.add(new Customer("Peter", "Parker"));
    customers.add(new Customer("Stephen", "Strange"));
    customers.add(new Customer("Steve", "Rogers"));
}
```

This will mean we need a constructor in our `Customer` class that takes in the first name and last name.
```java
public Customer(String firstName, String lastName) {
    this.id = UUID.randomUUID().toString();
    this.firstName = firstName;
    this.lastName = lastName;
}
```

Now try to get all the customers using Postman.

#### Get a specific customer

To get a specific customer, we need to know the `id` of the customer. We can get the `id` from the URL. using the `@PathVariable` annotation.

Since we are storing the data in an array, we need to find the index of the customer in the array.

Let's create a helper method to do this since we will be using it in multiple places.
```java
private int getCustomerIndex(String id) {
    for( Customer customer: customers) {
        if(customer.getId().equals(id)) {
            return customers.indexOf(customer);
        }
    }

    // Not found
    return -1;
}
```

Now we can create our `getCustomer` method.
```java
@GetMapping("/customers/{id}")
public Customer getCustomer(@PathVariable String id) {
    int index = getCustomerIndex(id);
    return customers.get(index);
}
```

Try retrieving a customer using Postman.

What happens when we try to retrieve a customer that does not exist?

### Update

To update a customer, similarly we need to get the `id` of the customer using the `@PathVariable` annotation.

We can use the previous helper method to get the index of the customer in the `customers` list.
```java
@PutMapping("/customers/{id}")
public Customer updateCustomer(@PathVariable String id, @RequestBody Customer customer) {
    int index = getCustomerIndex(id);
    customers.set(index, customer);
    return customer;
}
```

Based on IETF's [HTTP specification](https://tools.ietf.org/html/rfc7231#section-4.3.4), the `PUT` method is used to replace the current representation of the target resource with the request payload.

This means that if the `id` does not exist, we should create a new `Customer` object with the `id` and the request payload. Note that this is not always the case, and it depends on the API design.
```java
@PutMapping("/customers/{id}")
public Customer updateCustomer(@PathVariable String id, @RequestBody Customer customer) {
    int index = getCustomerIndex(id);

    if( index == -1) {
        customers.add(customer);
        return customer;
    }

    customers.set(index, customer);
    return customer;
}
```

To keep our implementation simple, we will use the former method i.e., to only update if the record if it exists.

Note that you can also use the Patch method to update a resource. The `PATCH` method is used to apply partial modifications to a resource.

### Delete

To delete a customer, again, we need to use `@PathVariable` to get the `id` of the customer.

We can use the previous helper method to get the index of the customer in the `customers` list.
```java
@DeleteMapping("/customers/{id}")
public Customer deleteCustomer(@PathVariable String id) {
    int index = getCustomerIndex(id);
    return customers.remove(index);
}
```

### `ResponseEntity`

Now, we are currently just returning JSON data. We should also specify the status code, so that the consumer of our API gets a more meaningful response.

`ResponseEntity` is a generic type that allows us to specify the response body and the status code.

Currently it is always returning a `200` status code, which generally means that the request was successful. But there are other status codes that we should use to indicate the status of the request e.g.

- `200` - OK, used when a resource is retrieved
- `201` - Created, used when a new resource is created
- `204` - No Content, used when a resource is deleted
- `404` - Not Found, used when a resource is not found

https://developer.mozilla.org/en-US/docs/Web/HTTP/Status

We can use the `HttpStatus` enum to specify the status code.
```java
@PostMapping("/customers")
public ResponseEntity<Customer> createCustomer(@RequestBody Customer customer) {
    customers.add(customer);
    return new ResponseEntity<>(customer, HttpStatus.CREATED);

    // Alternate syntax
    // return ResponseEntity.status(HttpStatus.CREATED).body(customer);
}
```

### 👨‍💻 Activity

Update the rest of the endpoints to use `ResponseEntity` with the appropriate status codes.

### `@RequestMapping`

We can reduce repetition in the code with `@RequestMapping`. By adding it to the class level, we can specify the base path for all the endpoints in the class.
```java
@RestController
@RequestMapping("/customers")
public class CustomerController {

}
```

The rest of the paths can then be updated to remove the `/customers` part.

### Custom Exception

Currently, when we enter an invalid id, we get an Internal Server Error. This is because we are trying to get the index of the customer in the `customers` list, but the customer does not exist.

Technically, it is not really a server error because it is the client that is sending an invalid request. We should return a `404` status code instead.

To handle this we can create a custom exception.
```java
public class CustomerNotFoundException extends RuntimeException {
  CustomerNotFoundException(String id) {
    super("Could not find customer with id: " + id);
  }
}
```

Then, in our helper method, we can throw this exception if the customer is not found.
```java
private int getCustomerIndex(String id) {
    for( Customer customer: customers) {
        if(customer.getId().equals(id)) {
            return customers.indexOf(customer);
        }
    }

    // Not found
    throw new CustomerNotFoundException(id);
}
```

Since this exception is propagated up the call stack, we need to catch it in our handler method.

Let's catch this exception in the methods that call this helper method so that we can return the appropriate status code when a customer `id` is not found.
```java
@GetMapping("/{id}")
public ResponseEntity<Customer> getCustomer(@PathVariable String id) {
  try {
    int index = getCustomerIndex(id);
    return new ResponseEntity<>(customers.get(index), HttpStatus.OK);
  } catch (CustomerNotFoundException e) {
    return new ResponseEntity<>(HttpStatus.NOT_FOUND);
  }
}
```

Proceed to update the rest of the endpoints to handle the `CustomerNotFoundException`.

### 👨‍💻 Activity

Practice creating CRUD endpoints with another resource called `Product` by yourself.

The `Product` class should have the following fields:

- id
- name
- description
- price

Create the following endpoints:

- `GET /products` - Get all products
- `GET /products/{id}` - Get a specific product
- `POST /products` - Create a new product
- `PUT /products/{id}` - Update a product
- `DELETE /products/{id}` - Delete a product

---

## Part 3: Intro to Lombok

Lombok is a library that helps us to reduce boilerplate code. It does this by generating the code for us during compile time.

### Installation

To install Lombok, we need to add the Lombok dependency in VSCode.
```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
```

### Example Usage

For now, we will just use the `@Getter` and `@Setter` annotations. These annotations will generate the getters and setters for us. This keeps our code clean and concise.
```java
@Getter
@Setter
public class Customer {
  private String id;
  private String firstName;
  private String lastName;
  private String email;
  private String contactNo;
  private String jobTitle;
  private int yearOfBirth;
}
```

For further reading, you can read the [documentation](https://projectlombok.org/features/all).

---

END