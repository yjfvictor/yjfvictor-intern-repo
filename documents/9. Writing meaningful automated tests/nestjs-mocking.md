# Reflection: Mocking Dependencies and Database Interactions in NestJS

## Summary of the project

The project [test-repo-for-mocking-dependencies-and-database-interactions-in-nestjs](https://github.com/yjfvictor/test-repo-for-mocking-dependencies-and-database-interactions-in-nestjs) is a sample NestJS application that shows how to mock dependencies and database access in unit tests.

It provides a small REST API for an **Item** resource (create, read all, read one, update, delete). The app uses TypeORM with PostgreSQL. The codebase includes controller tests that mock the items service and service tests that mock the TypeORM repository so that no real database or external service is used when running tests. All code is in TypeScript with explicit types. The project can be run with `npm run start` and tested with `npm run test`.

## Why is mocking important in unit tests?

Mocking keeps unit tests fast, stable, and focused. If tests called real services, repositories, or APIs, they would depend on the database being up, network being available, and data being in a known state. That would make tests slower and flaky. By replacing those dependencies with mocks (fake implementations that return controlled values), tests only check the unit under test (for example, that the controller calls the service with the right arguments and returns its result). They do not test the service or the database. That way, unit tests run quickly, do not need infrastructure, and fail only when the unit’s own behaviour changes.

## How do you mock a NestJS provider (e.g. a service in a controller test)?

You use the NestJS testing utilities to build a minimal module where the real provider is replaced by a mock. Call `Test.createTestingModule()` and pass the controller and a custom provider for the service. The provider should use `useValue` (or `useFactory`) and supply an object whose methods are Jest mock functions (e.g. `jest.fn()`). Then compile the module and get the controller with `module.get(ItemsController)`. In each test, configure the mock’s return values with `mockResolvedValue` or `mockRejectedValue` and assert that the controller’s methods call the mock with the expected arguments and return the values you set. The controller never talks to the real service, so the test only verifies controller behaviour.

## What are the benefits of mocking the database instead of using a real one?

Mocking the database (for example, the TypeORM repository) in service tests has several benefits. Tests do not need a running database or migrations, so they run in any environment (CI, local, offline). They are fast because there are no real queries or connections. They are deterministic: you control exactly what the repository “returns,” so you can test error paths (e.g. “not found”) and edge cases without setting up real data. There is no risk of tests affecting shared data or leaving the database in a bad state. The trade-off is that you are not testing the real SQL or repository implementation, so integration or e2e tests are still needed for that.

## How do you decide what to mock vs. what to test directly?

Mock dependencies that are outside the unit you are testing. In a controller test, the unit is the controller, so mock the service (and any other injected providers). In a service test, the unit is the service, so mock the repository (and any other external dependencies). Do not mock the unit under test. Prefer mocking at boundaries (services, repositories, HTTP clients) so that each test file focuses on one layer. Use real instances only for the code that belongs to that layer. For behaviour that spans multiple layers or touches the real database, use integration or e2e tests instead of unit tests with mocks.
