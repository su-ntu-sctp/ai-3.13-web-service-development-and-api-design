# Lesson 3.13: Web Service Development and API Design

## Lesson Overview
This lesson builds upon the foundational REST API concepts from the previous lesson and teaches students how to implement complete CRUD (Create, Read, Update, Delete) operations for managing resources. Students will learn how Spring uses annotations to create and manage objects, handle HTTP request/response properly using ResponseEntity, implement custom exception handling, and use Lombok to reduce boilerplate code. By working through a practical Customer Resource Management (CRM) example, students will gain hands-on experience building production-ready REST APIs that follow industry best practices for status codes, error handling, and code organization.

---

## Lesson Objectives
By the end of this lesson, students will be able to:

1. **Explain** how `@Component`, `@RestController` and `@ResponseBody` work, and why a data class should never be a Spring bean
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

When you annotate a class with `@Component` (or a specialization such as `@RestController` or `@Service`), you are telling Spring Boot: *"Please create an instance of this class and manage it for me."*

This is the foundation of **Dependency Injection** — Spring manages your objects so you don't have to wire them together manually.

**Important:** beans are **singletons** by default. Spring creates *one* instance and shares it across the whole application, for every request and every thread. Remember this — it determines what should and should not be a bean.

### `@Component`

`@Component` is the most generic bean annotation. It simply tells Spring: *"Create a bean for this class."* You use it for classes that **do work** — validators, helpers, and later on, services and repositories.

```java
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        return email.contains("@");
    }
}
```

A useful rule of thumb, which we will come back to shortly:

> **Inject the things that *do work*. Create the things that *hold data*.**

### `@RestController` and `@ResponseBody`

`@RestController` is what we use on every controller class in this module. It does two things at once:

1. It registers the class as a bean (it is a specialization of `@Component`).
2. It tells Spring that whatever a method returns **is the data** — serialize it straight into the HTTP response body as JSON.

That second behaviour comes from `@ResponseBody`, which is already built into `@RestController`. **You never need to add `@ResponseBody` separately.**

```java
@RestController
public class CustomerController {

    @GetMapping("/customers")
    public String getCustomers() {
        return "customers";   // returned as JSON data
    }
}
```

Under the hood, Spring uses the **Jackson** library to convert your Java object into JSON. We will see the reverse of this shortly with `@RequestBody`, which uses the same machinery to convert incoming JSON into a Java object.

> **Sidenote — `@Controller`:** you will see an older annotation called `@Controller` in tutorials and older codebases. That belongs to the traditional style where the server built and returned a complete HTML page. We are building APIs — our frontend is separate and we always return JSON — so we use `@RestController` throughout this module.

### Meta-annotations

`@RestController` is an example of a **meta-annotation**: a single annotation that is itself made up of other annotations. Rather than writing several annotations on every controller, you write one that bundles them.

You have already been using another one without realising it. `@SpringBootApplication` on your main class is a meta-annotation that bundles together the annotations that enable auto-configuration and tell Spring which packages to scan for beans.

This is why component scanning "just works": `@SpringBootApplication` scans its own package and everything below it. If a class sits outside that package tree, Spring will never find it, and it will never become a bean.

---

## Part 2: Postman — Testing Your API

### What is Postman?

A browser can only make `GET` requests easily — you can't send a `POST` with a JSON body just from the address bar. **Postman** is a tool that lets you send any HTTP request (GET, POST, PUT, DELETE) with full control over the URL, headers, and request body. It shows you the response status code and body clearly, making it the standard tool for testing REST APIs during development.

### Installation

Download from [https://www.postman.com/downloads](https://www.postman.com/downloads). Install and create a free account, or skip sign-in and use it directly.

### Key Areas of the UI

- **Method selector** — dropdown on the left (GET, POST, PUT, DELETE)
- **URL bar** — where you enter your endpoint URL
- **Body tab** — where you attach a JSON payload for POST/PUT requests
- **Response panel** — bottom half; shows status code, response time, and response body

### Making a GET Request

1. Select `GET` from the method dropdown
2. Enter the URL: `http://localhost:8080/customers`
3. Click **Send**
4. Check the response panel — you should see your JSON data and a `200 OK` status

### Making a POST Request

1. Select `POST` from the method dropdown
2. Enter the URL: `http://localhost:8080/customers`
3. Click the **Body** tab → select **raw** → select **JSON** from the dropdown
4. Paste your JSON payload:
```json
{
  "firstName": "Bruce",
  "lastName": "Banner",
  "email": "bruce@avengers.com",
  "contactNo": "12345678",
  "jobTitle": "Scientist",
  "yearOfBirth": 1975
}
```
5. Click **Send**
6. Check the response — you should see the created customer with a generated `id` and a `201 Created` status

### PUT and DELETE

Follow the same pattern as POST for `PUT` — select `PUT`, add the `id` to the URL (`/customers/{id}`), and include the updated JSON body.

For `DELETE` — select `DELETE`, add the `id` to the URL, no body needed.

> **Note — why JSON looks messy in a browser but neat in Postman.** The server sends exactly the same response to both. Postman reads the `Content-Type: application/json` header and pretty-prints it for you; a browser just dumps the raw text on one line. If you want readable JSON in the browser, either install a JSON formatter extension, or open DevTools → Network → click the request → Preview.

---

## Part 3: Building Our `simple-crm`

We continue with the **`simple-crm`** project you created at the end of the previous lesson. Do not create a new project — this is the project we build on for the rest of the module.

> **Package/folder structure — standing rule for `simple-crm`:** every class goes in a folder matching its layer. Create these folders inside your base package (`sg.edu.ntu.simple_crm`) and place each class accordingly:
> - Controller classes → `controller` folder
> - Entity/POJO classes (e.g. `Customer`) → `model` folder
> - Custom exception classes (e.g. `CustomerNotFoundException`) → `exceptions` folder
> - (Later lessons) Service interfaces + implementations → `service` folder; repository classes/interfaces → `repository` folder

> **Moving existing classes into folders.** Dragging a file into a new folder in VS Code moves the file but does **not** update the `package` line at the top, which is why you get a red error afterwards. Either use **right-click on the class name in the editor → Refactor → Move**, which updates the package and all references for you, or drag the file and then fix the `package` line by hand.
>
> If the application then fails to start with a `ConflictingBeanDefinitionException`, it means an old copy of the class is still in the original location. Delete it and run `mvn clean` before restarting.

### Cleanup: remove the `@Component` / `@Autowired` from `Customer`

In the previous lesson, you annotated `Customer` with `@Component` and injected it into `CustomerController` with `@Autowired`. That was done purely to demonstrate how the two annotations work together. **We now need to undo it**, before we build our CRUD endpoints.

Make these three changes:

1. Remove `@Component` from the `Customer` class.
2. Remove the `@Autowired private Customer customer;` field from `CustomerController`.
3. Remove the old `/customer` endpoint that returned a single preset customer.

**Why?** Because beans are singletons. Annotating `Customer` with `@Component` means Spring creates exactly **one** `Customer` object and shares that same instance across the entire application. Every request would be reading and writing the same object — one user's data would overwrite another's.

But a CRM needs *many* customers, each with its own id and its own values. `Customer` is **data**, not a service. Data objects are created with `new`, or built by Jackson from incoming JSON, or (later in this module) loaded by JPA from a database row. They are never created by the Spring container.

This is the rule from Part 1 in action:

> **Inject the things that *do work*. Create the things that *hold data*.**

Dependency injection is genuinely useful, and we return to it properly in the next lesson when we build a **service** and a **repository**. Those are working classes — they have behaviour, they hold no per-request data, and there is real value in one shared instance. That is where `@Autowired` belongs.

### `Customer` POJO

Create our `Customer` POJO. Place this class in the **`model`** folder (e.g. `sg.edu.ntu.simple_crm.model.Customer`).
```java
import com.fasterxml.jackson.annotation.JsonPropertyOrder;

@JsonPropertyOrder({ "id", "firstName", "lastName", "email", "contactNo", "jobTitle", "yearOfBirth" })
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

> **Why `@JsonPropertyOrder`?** Without it, the fields appear in the JSON in an unpredictable order — `id` might show up in the middle rather than first. Jackson builds the JSON from the getters it discovers by reflection, and the JVM gives no guarantee about the order those come back in. `@JsonPropertyOrder` simply tells Jackson the order to write them in. This is cosmetic only: JSON is an unordered set of key-value pairs, and every client reads fields by name, not position. Nothing breaks without it — it just makes the response easier to read.

> **Why is `id` a `String` and not a number?** Because we are about to generate it with `UUID.randomUUID()`. A UUID is a 128-bit value written as hex with dashes (`a1b2c3d4-e5f6-...`) — it will not fit in a `long`, so `String` is the natural type. We use a UUID because our "database" is currently just an `ArrayList` in memory: there is no auto-increment column to assign ids for us, so each object generates its own. When we move to JPA later in the module, you will see the database-assigned `Long` id approach instead.

### Storing `Customer` objects

We will use an `ArrayList` to store our `Customer` objects in `CustomerController.java`. Place this class in the **`controller`** folder (e.g. `sg.edu.ntu.simple_crm.controller.CustomerController`).
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

> **What actually happens right now, without any conversion instruction?** The request does not fail or error out. Spring still creates a `Customer` object using the no-arg constructor and adds it to the `customers` list — but since Spring has no way to populate that object's fields from a JSON request body, every field comes back `null` (or `0` for `yearOfBirth`). So an object *is* added to the list, but the actual values you sent (`"Bruce"`, `"Banner"`, etc.) are not captured anywhere — they're lost. Check your `ArrayList` and you'll find an extra entry with blank fields, not the customer you sent.

This is done by adding the `@RequestBody` annotation to our handler method.
```java
@PostMapping("/customers")
public Customer createCustomer(@RequestBody Customer customer) {
    customers.add(customer);
    return customer;
}
```

The `@RequestBody` annotation tells our application to convert the JSON into a `Customer` object. Spring Boot is now able to de-serialize the JSON into a `Customer` object, which is why we are able to add it to our `customers` list.

Notice that `@RequestBody` and `@ResponseBody` are two directions of the same mechanism: Jackson converting JSON into a Java object on the way in, and a Java object into JSON on the way out.

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

Note that we create these with `new` — exactly as described earlier. `Customer` holds data, so we create it ourselves rather than asking Spring for it.

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

Let's create a helper method to do this since we will be using it in multiple places. Note that it is `private` — it is an internal detail of the controller, not part of our API.
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

Update the rest of your endpoints to use `ResponseEntity` with the appropriate status code.

> **Watch out:** each endpoint has a *success* status. It is very easy to copy a `ResponseEntity` line from one method to another and leave the wrong status behind — for example returning `NOT_FOUND` from a delete that actually succeeded. Check each one individually.

### `@RequestMapping`

We can reduce repetition in the code with `@RequestMapping`. By adding it to the class level, we can specify the base path for all the endpoints in the class.
```java
@RestController
@RequestMapping("/customers")
public class CustomerController {

}
```

The rest of the paths can then be updated to remove the `/customers` prefix — for example `@GetMapping("/customers/{id}")` becomes `@GetMapping("/{id}")`.

Keep the leading `/` on every method-level path so they all look consistent.

### Custom Exception

Currently, when we enter an invalid id, we get a `500 Internal Server Error`. This is because we are trying to get the index of the customer in the `customers` list, but the customer does not exist.

Technically, it is not a server error — it is the client that is sending an invalid request. We should return a `404` status code instead.

To handle this we can create a custom exception. Place this class in the **`exceptions`** folder (e.g. `sg.edu.ntu.simple_crm.exceptions.CustomerNotFoundException`).
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

---

### 👨‍💻 Activity **(25 minutes)**

Both tasks are done in your existing `simple-crm` project. Do not create any new classes.

#### Task 1 — Finish the exception handling

We handled `CustomerNotFoundException` in `getCustomer` together. Now do the same for the two endpoints that were left:

- `updateCustomer` — wrap the lookup in a `try`/`catch`, return `200 OK` with the updated customer on success, `404 Not Found` if the id does not exist.
- `deleteCustomer` — same pattern. Return `404 Not Found` if the id does not exist.

For the success case of `deleteCustomer`, try both of these and compare them in Postman:

- `200 OK` with the deleted customer in the body
- `204 No Content` with no body at all — `return new ResponseEntity<>(HttpStatus.NO_CONTENT);`

Which one you choose is a genuine API design decision. `200` is useful if the client wants confirmation of exactly what was removed; `204` is cleaner when the client only needs to know it worked.

Test every endpoint with both a valid and an invalid id before moving on.

#### Task 2 — Refactor `Customer` with Lombok

Read the Lombok section below first, then apply it to your `Customer` class:

1. Add the Lombok dependency to `pom.xml`.
2. Delete all hand-written getters and setters.
3. Add `@Data` and `@NoArgsConstructor`.
4. Move the UUID generation inline onto the field, and delete the no-arg constructor that used to set it.
5. Keep the two-argument `Customer(String firstName, String lastName)` constructor.

Then re-run the application and test **all five endpoints** again. Nothing should behave differently — the whole point of Lombok is that it generates exactly what you deleted. If something breaks, the most likely cause is a getter that is no longer being generated the way you expected.

---

## Part 4: Intro to Lombok

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

### What to Remove and What to Keep

When applying Lombok to an existing class, not everything gets deleted. Here is the rule:

**Remove:**
- All manually written getters — `@Data` generates them
- All manually written setters — `@Data` generates them
- The no-arg default constructor — replace with `@NoArgsConstructor`
- The no-arg constructor that sets `this.id = UUID.randomUUID().toString()` — no longer needed once `id` is initialized inline (see below)

**Keep:**
- Any constructor that contains **custom logic** — Lombok cannot generate these. Our `Customer(String firstName, String lastName)` constructor stays because it is used for preloading data. It is not just assigning fields mechanically.

### Inline UUID Initialization

Previously our no-arg constructor was responsible for generating the UUID:
```java
public Customer() {
    this.id = UUID.randomUUID().toString();
}
```

With Lombok, we remove this constructor. To ensure every instance still gets a UUID regardless of which constructor is called, move the initialization inline:
```java
private final String id = UUID.randomUUID().toString();
```

Now the UUID is generated at the field level — both constructors (and any future ones) automatically get a unique `id` without you having to set it manually.

### Final `Customer` Class with Lombok

```java
package sg.edu.ntu.simple_crm.model;

import com.fasterxml.jackson.annotation.JsonPropertyOrder;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.util.UUID;

@Data
@NoArgsConstructor
@JsonPropertyOrder({ "id", "firstName", "lastName", "email", "contactNo", "jobTitle", "yearOfBirth" })
public class Customer {
  private final String id = UUID.randomUUID().toString();
  private String firstName;
  private String lastName;
  private String email;
  private String contactNo;
  private String jobTitle;
  private int yearOfBirth;

  // Keep this constructor — it has custom logic used for preloading data
  public Customer(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}
```

`@Data` generates all getters and setters. `@NoArgsConstructor` generates the default no-arg constructor. Lombok will not generate a setter for `final` fields, so `id` remains immutable.

Notice there is no `@Component` here. `Customer` holds data — it is never a Spring bean.

For further reading, see the [Lombok documentation](https://projectlombok.org/features/all).

---

END