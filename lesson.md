# Lesson 3.13: Web Service Development and API Design

## Lesson Overview
This lesson builds upon the foundational REST API concepts from the previous lesson and teaches students how to implement complete CRUD (Create, Read, Update, Delete) operations for managing resources. Students will learn the differences between controller annotations, handle HTTP request/response properly using ResponseEntity, implement custom exception handling, and use Lombok to reduce boilerplate code. By working through a practical Customer Resource Management (CRM) example, students will gain hands-on experience building production-ready REST APIs that follow industry best practices for status codes, error handling, and code organization.

---

## Lesson Objectives
By the end of this lesson, students will be able to:

1. **Differentiate** between `@Component`, `@Controller`, and `@RestController` annotations
2. **Implement** complete CRUD operations for REST resources using `ResponseEntity` with appropriate HTTP status codes
3. **Create** and handle custom exceptions for better error management
4. **Apply** Lombok annotations to reduce boilerplate code in POJOs

---

## Part 1: Annotations, Beans, and Controllers

### What is an Annotation?

An annotation is metadata you attach to a class, method, or field using the `@` symbol. The annotation itself contains no logic — it is a signal to the framework. When Spring Boot starts up, it scans your code, reads these annotations, and acts on them automatically.

Think of annotations as labels on a box. The label doesn't do anything by itself — but the person (Spring) reading the label knows exactly what to do with that box.

```java
@RestController          // Label: "this class handles HTTP requests and returns data"
public class CustomerController {

    @GetMapping("/customers")   // Label: "this method handles GET /customers"
    public String getCustomers() {
        return "customers";
    }
}
```

### What is a Bean?

A **bean** is simply an object that is created and managed by Spring Boot. Instead of you writing `new CustomerController()` yourself, Spring Boot creates it for you, manages its lifecycle, and makes it available throughout your application.

When you annotate a class with `@Component` (or any of its specializations like `@Controller`, `@RestController`, `@Service`), you are telling Spring Boot: *"Please create an instance of this class and manage it for me."*

This is the foundation of **Dependency Injection** — Spring manages your objects so you don't have to wire them together manually.

### `@Component`, `@Controller`, and `@RestController`

These three annotations form a hierarchy:

**`@Component`** is the most generic. It simply tells Spring: *"Create a bean for this class."* You use it for utility classes or any Spring-managed object that doesn't fit a more specific role.

```java
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        return email.contains("@");
    }
}
```

**`@Controller`** is a specialization of `@Component`. It signals that this class handles web requests. It inherits everything `@Component` does, but adds the context that this is a web layer class. By default, handler methods in a `@Controller` are expected to return a **view name** (an HTML page).

**`@ResponseBody`** changes that behavior. When added to a method (or the class), it tells Spring: *"Don't look for a view — serialize the return value directly into the HTTP response body as JSON."*

**`@RestController`** is simply a shortcut that combines both:

```java
@RestController
// is exactly the same as:
@Controller
@ResponseBody
```

Since we are building a REST API and always returning JSON — not HTML pages — we will always use `@RestController`.

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

To get a specific customer, we need to know the `id` of the customer. We can get the `id` from the URL using the `@PathVariable` annotation.

Since we are storing the data in an array, we need to find the index of the customer in the array.

Let's create a helper method to do this since we will be using it in multiple places.
```java
private int getCustomerIndex(String id) {
    for (Customer customer : customers) {
        if (customer.getId().equals(id)) {
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

> **What happens when we try to retrieve a customer that does not exist?**
> You will get a `500 Internal Server Error`. This is technically wrong — the server didn't crash, the client sent a bad ID. We will fix this properly in the Custom Exception section below.

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

The `PUT` method is used to replace the current representation of the target resource with the request payload. To keep our implementation simple, we will only update if the record exists.

Note that you can also use the `PATCH` method to apply partial modifications to a resource, rather than replacing it entirely. See [MDN HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) for more.

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

Now, we are currently just returning JSON data. We should also specify the HTTP status code, so that the consumer of our API gets a more meaningful response.

**What is `ResponseEntity`?**

An HTTP response has three parts: a **status code**, **headers**, and a **body**. Up until now, Spring Boot has been handling the status code automatically — always returning `200 OK`. `ResponseEntity` gives you explicit control over all three parts of the response.

```
HTTP/1.1 201 Created          ← status code
Content-Type: application/json ← header
                               ← blank line
{ "id": "abc123", ... }        ← body
```

`ResponseEntity<T>` is a generic wrapper where `T` is the type of your response body.

Currently all endpoints return `200`, but we should use the correct status codes:

- `200` - OK, used when a resource is retrieved
- `201` - Created, used when a new resource is created
- `204` - No Content, used when a resource is deleted
- `404` - Not Found, used when a resource is not found

Reference: [MDN HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

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

### 👨‍💻 Activity **(10 minutes)**

Update the rest of the endpoints to use `ResponseEntity` with the appropriate status codes.

### `@RequestMapping`

We can reduce repetition in the code with `@RequestMapping`. By adding it to the class level, we can specify the base path for all the endpoints in the class.
```java
@RestController
@RequestMapping("/customers")
public class CustomerController {

}
```

The rest of the paths can then be updated to remove the `/customers` prefix — for example `@GetMapping("/customers/{id}")` becomes `@GetMapping("/{id}")`.

### Custom Exception

Currently, when we enter an invalid id, we get a `500 Internal Server Error`. This is because we are trying to get the index of the customer in the `customers` list, but the customer does not exist.

Technically, it is not a server error — it is the client that is sending an invalid request. We should return a `404` status code instead.

To handle this we can create a custom exception.
```java
public class CustomerNotFoundException extends RuntimeException {
  public CustomerNotFoundException(String id) {
    super("Could not find customer with id: " + id);
  }
}
```

Then, in our helper method, we can throw this exception instead of returning `-1`.
```java
private int getCustomerIndex(String id) {
    for (Customer customer : customers) {
        if (customer.getId().equals(id)) {
            return customers.indexOf(customer);
        }
    }

    // Not found
    throw new CustomerNotFoundException(id);
}
```

Since this exception is propagated up the call stack, we need to catch it in our handler methods. Let's update `getCustomer` first.
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

### 👨‍💻 Activity **(20 minutes)**

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

Lombok is a library that helps us reduce boilerplate code. It does this by generating code for us at **compile time** — so the bytecode contains all the getters, setters, and constructors, but your source file stays clean.

### Installation

To install Lombok, add the dependency in `pom.xml`.
```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
```

### Common Lombok Annotations

| Annotation | What it generates |
|---|---|
| `@Getter` | Getter methods for all fields |
| `@Setter` | Setter methods for all fields |
| `@NoArgsConstructor` | A no-argument constructor |
| `@AllArgsConstructor` | A constructor with all fields as parameters |
| `@Data` | `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor` |

In real Spring Boot projects, you will see `@Data` used most commonly on entity and POJO classes.

### Example Usage

Without Lombok, our `Customer` class requires manually written getters, setters, and constructors — dozens of lines of repetitive code. With Lombok:

```java
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
public class Customer {
  private final String id = UUID.randomUUID().toString();
  private String firstName;
  private String lastName;
  private String email;
  private String contactNo;
  private String jobTitle;
  private int yearOfBirth;
}
```

`@Data` generates all getters and setters. `@NoArgsConstructor` generates the default constructor. The `id` field remains `final` and is auto-generated — Lombok will not generate a setter for `final` fields.

For further reading, see the [Lombok documentation](https://projectlombok.org/features/all).

---

END