# Reflection: Using Jest and Supertest for API Testing in NestJS

## Project Summary

The project [test-repo-for-using-jest-and-supertest-for-api-testing-in-nestjs](https://github.com/yjfvictor/test-repo-for-using-jest-and-supertest-for-api-testing-in-nestjs) is a sample NestJS application that demonstrates how to write both **unit tests** and **integration (e2e) tests** for an API using Jest and Supertest. The repository contains a simple items API with a public GET endpoint, a POST endpoint with request body validation (using class-validator), and a JWT-protected GET endpoint for the authenticated user's items.

**Unit tests** cover all TypeScript source files under `src/` except the bootstrap entry `main.ts`. They include: `AppModule`, `ItemsModule`, `AuthModule` (module compile and provider resolution); `ItemsController` (all endpoints with mocked `ItemsService`); `ItemsService` (CRUD); `CreateItemDtoClass` and `UpdateItemDtoClass` (class-validator validation); JSON error middleware and `JsonErrorExceptionFilter`; content-type/encoding middleware; `JwtStrategy`, `JwtAuthGuard`, and `MockJwtAuthGuard`.

**E2e tests** use Supertest to drive the app over HTTP and a mock JWT guard so that protected routes can be tested without issuing real tokens. The project can be run with `npm run start`, and all tests (unit then e2e) are run with `npm run test`.

## How Supertest Helps Test API Endpoints

Supertest is a library that lets you send HTTP requests to your application and assert on the response without starting a real server on a port. In NestJS, you pass the result of `app.getHttpServer()` to Supertest so that requests are handled by the same Nest pipeline (middleware, guards, pipes, controllers) as in production. You can call `.get()`, `.post()`, `.put()`, and other methods with a path and optional body or headers, then chain `.expect(statusCode)` and `.expect((res) => { ... })` to check status and body. This makes it easy to cover success and error cases (for example 200 vs 400 when validation fails) and to verify that the API contract (status codes and response shape) is correct.

## Difference Between Unit Tests and API Tests

Unit tests focus on a single class or function in isolation. Dependencies are usually mocked or stubbed so that only the unit under test is exercised. In this project, for example, the `ItemsService` unit tests call `findAll()`, `create()`, and `findByUserId()` directly and assert on return values without any HTTP layer; the `ItemsController` tests use a mocked `ItemsService` and assert on controller return values and thrown exceptions. API tests (integration or e2e tests) instead drive the application through HTTP. They use Supertest to send requests to the app and assert on status and response body. That way they test the full stack: routing, validation pipes, guards, controllers, and services together. Unit tests are fast and pinpoint failures to a small piece of code. API tests are slower but catch issues in how components interact and how the API behaves for a client. The sample repo uses both: comprehensive unit tests for every source file and e2e tests for the HTTP API.

## Why Authentication Should Be Mocked in Integration Tests

In integration tests we usually want to verify that a protected endpoint returns the right data when the user is authenticated, without depending on a real auth backend or token issuance. If we used real JWTs, we would need a running auth service, valid credentials, and token handling in the test, which makes tests brittle and environment-dependent. By overriding the JWT guard with a mock guard that always allows the request and sets a fixed test user on the request (for example `req.user = { sub: 'test-user-id' }`), we can test the behaviour of the protected route (for example that GET /items/me returns only that user's items) in a repeatable way. The authentication mechanism is still tested elsewhere (for example in unit tests of the guard or in a dedicated auth e2e suite), but for general API tests, mocking auth keeps tests simple and focused on the endpoint logic.

## Structuring API Tests to Cover Success and Failure Cases

A good approach is to group tests by endpoint or by scenario and then add at least one test for the happy path and one or more for failure paths. For a GET endpoint, that might mean one test that expects 200 and a valid response shape. For a POST endpoint with validation, add a test that sends a valid body and expects 201 and the created resource, then add tests that send invalid data (missing required field, empty string, value that exceeds max length, or wrong type) and expect 400. Use descriptive test names (for example "with valid body returns 201 and created item" and "with empty title returns 400") so that when a test fails it is clear which case broke. Reuse the same app instance in `beforeAll` (and close it in `afterAll`) so that the test suite stays fast while still testing the real request pipeline.
