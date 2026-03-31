# Self Studies: Web Service Development and API Design

## Overview

In this lesson you will build a fully functional CRUD REST API from scratch using Spring Boot. The self-study materials below will help you arrive with a clear understanding of what CRUD operations are, how HTTP status codes work, and what Lombok does — so you can focus on coding during the lesson rather than catching up on concepts.

**Estimated Prep Time:** 60–80 minutes

---

## Task 1: Spring Boot CRUD REST API

This video walks you through building a complete CRUD API with Spring Boot — POST, GET, PUT, and DELETE endpoints — and covers how to use `ResponseEntity` to return the correct HTTP status codes. This maps directly to Part 2 of the lesson.

**Watch:** Spring Boot CRUD REST API Tutorial
🎬 https://www.youtube.com/watch?v=7nonQ2dYgiE

**Then read:** Lesson 3.13 — Part 1 and Part 2

**Guiding Questions:**
- What is the difference between `@Controller` and `@RestController`?
- What does `@RequestBody` do and why is it needed for POST and PUT requests?
- Why should a POST endpoint return a `201 Created` status code instead of `200 OK`?
- What happens when a client requests a resource that does not exist — which status code should be returned?

---

## Task 2: Project Lombok

Lombok is a small but widely used library in Java projects. This video gives you a quick overview of the most common annotations so that the Lombok section in Part 3 of the lesson feels familiar.

**Watch:** Project Lombok Tutorial for Beginners
🎬 https://www.youtube.com/watch?v=745W-dng3wk&t=73s

**Then read:** Lesson 3.13 — Part 3

**Guiding Questions:**
- What problem does Lombok solve?
- What do `@Getter` and `@Setter` generate for you?
- Can Lombok be used in any Java project, or only Spring Boot?

---

## Active Engagement Strategies

- After watching the first video, try to sketch out the full CRUD flow on paper — what endpoint, HTTP method, and status code maps to each operation (Create, Read, Update, Delete)
- Before the lesson, look at the `Customer` POJO from Lesson 3.11 and count how many lines of boilerplate code (getters, setters, constructors) Lombok would eliminate
- If you are unsure about any HTTP status code, browse the [MDN HTTP Status reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) — knowing the common ones (200, 201, 204, 404) will help you follow the lesson

---

## Additional Reading Material

- [HTTP Status Codes Reference — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [ResponseEntity in Spring — Baeldung](https://www.baeldung.com/spring-response-entity)
- [Introduction to Project Lombok — Baeldung](https://www.baeldung.com/intro-to-project-lombok)
- [Custom Exception Handling in Spring Boot — Baeldung](https://www.baeldung.com/exception-handling-for-rest-with-spring)