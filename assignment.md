# Assignment (Optional)

## Brief

Create a Spring Boot project called LibraryManagementAPI and build a complete CRUD API with proper exception handling and Lombok integration.

1. **Complete CRUD API for Book Management**
   - Create a `Book` class with the following attributes:
     - `String id` (final, auto-generated using UUID)
     - `String title`
     - `String author`
     - `String isbn`
     - `int publicationYear`
   - Use Lombok annotations (@Getter, @Setter) to reduce boilerplate code
   - Create a `BookController` with `@RestController` and `@RequestMapping("/books")`
   - Use an `ArrayList<Book>` to store books in memory
   - Implement all CRUD endpoints with proper `ResponseEntity` and HTTP status codes:
     - `POST /books` - Create a new book (return 201 Created)
     - `GET /books` - Get all books (return 200 OK)
     - `GET /books/{id}` - Get a specific book by ID (return 200 OK or 404 Not Found)
     - `PUT /books/{id}` - Update an existing book (return 200 OK or 404 Not Found)
     - `DELETE /books/{id}` - Delete a book (return 204 No Content or 404 Not Found)
   - Pre-populate the ArrayList with at least 3 books in the controller constructor
   - Test all endpoints using Postman or Thunder Client

2. **Custom Exception Handling for Book API**
   - Create a custom exception `BookNotFoundException` that extends `RuntimeException`
   - The exception should accept an `id` parameter and set the message as "Could not find book with id: {id}"
   - Create a helper method `getBookIndex(String id)` that:
     - Returns the index of the book in the ArrayList if found
     - Throws `BookNotFoundException` if the book is not found
   - Update all endpoints that access a specific book to:
     - Use the `getBookIndex()` helper method
     - Catch `BookNotFoundException` and return appropriate status codes (404 Not Found)
   - Create a `BookStatistics` class (as a POJO or Record) with:
     - `int totalBooks`
     - `String oldestBook` (title of the oldest book by publication year)
     - `String newestBook` (title of the newest book by publication year)
   - Use Lombok's @Getter and @Setter on the BookStatistics class
   - Create an endpoint `GET /books/statistics` that:
     - Calculates and returns BookStatistics based on the current books in the ArrayList
     - Returns 200 OK with the statistics
   - Add logging using SLF4J:
     - Log INFO when a book is created, updated, or deleted
     - Log WARN when a BookNotFoundException is caught
   - Test all scenarios including error cases (invalid IDs)

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- PostgreSQL: https://www.postgresql.org/docs/
- OWASP: https://cheatsheetseries.owasp.org/