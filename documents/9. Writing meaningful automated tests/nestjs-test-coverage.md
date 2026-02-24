# Reflection: Focus Bear Coverage Bar and Meaningful Tests

## Summary of tasks

The work was done in the repository [test-repo-for-understanding-the-focus-bear-coverage-bar-and-writing-meaningful-tests](https://github.com/yjfvictor/test-repo-for-understanding-the-focus-bear-coverage-bar-and-writing-meaningful-tests).

Tasks completed:

1. **Sample NestJS project**
   A NestJS application with CRUD for a Person resource was created. Each person has a name, date of birth, sex, id number, and readonly `createdAt` and `updatedAt` timestamps. The database is PostgreSQL running in Docker (`postgres:alpine3.22`). The app runs with `npm run start`.

2. **Jest coverage in NestJS**
   Jest was configured in `jest.config.ts` with `collectCoverage: false` so that coverage is collected only when running `npm run test:cov` (not on every `npm run test`). The config includes `coverageDirectory`, `collectCoverageFrom` (all source files under `src/`, excluding only `*.module.ts`, `*.dto.ts`, and `*.spec.ts`), `coverageReporters` (text, text-summary, html, lcov), and `coverageThreshold` (e.g. 80% for statements, lines, and functions, and a lower threshold for branches). The coverage report was viewed in the terminal and in `coverage/lcov-report/index.html`.

3. **Untested areas and extra tests**
   Gaps were identified from the coverage report. Additional tests were added for creating persons with `Sex.MALE`, `Sex.FEMALE`, and `Sex.OTHER`, and for updating with multiple fields. DTO-only files were excluded from coverage so that the reported metrics focus on executable code. Coverage thresholds were set so that statements, lines, and functions meet at least 80%, with a lower branch threshold because of decorator-generated branches. Unit tests cover all source files: bootstrap (`main.spec.ts`, which mocks `NestFactory` and asserts `bodyParser: false` and middleware order), Content-Type middleware (`content-type.middleware.spec.ts`), Person controller and service (`person.controller.spec.ts`, `person.service.spec.ts`), Person entity and Sex enum (`person.entity.spec.ts`), and the date validator (`valid-date.validator.spec.ts`). A dedicated **valid-date.validator.spec.ts** was added to test the date validator: non-string inputs (number, null, object, array), invalid format, invalid month or day bounds, invalid day in month (e.g. 30 February), and valid boundary dates (e.g. leap year 2024-02-29, last day of year). This gives full coverage of the validator used by the DTOs.

4. **Content-Type validation**
   The server validates the `Content-Type` header for requests with a body (POST, PATCH): it must be `application/json`, and if a charset is present it must be UTF-8 only. To enforce this before body parsing, the default body parser is disabled in `main.ts` and `ContentTypeMiddleware` is registered with `app.use(contentTypeMiddleware.use)` before `express.json()`, so invalid or unsupported charsets (e.g. UTF-7) are rejected with 415 before the body is parsed. Unit tests cover all middleware branches (valid/invalid type, charset present/absent, non-UTF-8 charset).

5. **Meaningful assertions**
   The idea of meaningful test assertions was researched: coverage measures which code was executed, not whether behaviour was actually verified. A weak test that only asserted `expect(result).toBeDefined()` was refactored so that it checks the full response shape (presence and types of `id`, `name`, `dateOfBirth`, `sex`, `idNumber`, `createdAt`, `updatedAt`) and that the service was called with the correct DTO.

6. **Refactor of a weak test**
   The weak test was replaced by a test that asserts on the returned person’s fields and types and on the interaction with the service, so that the test validates real behaviour instead of only that something was returned.

7. **End-to-end tests**
   E2E tests were added in `test/app.e2e-spec.ts` using Jest with `test/jest-e2e.json`. The test app is built with the same middleware as production (body parser disabled, ContentTypeMiddleware, then `express.json()`). The suite uses Supertest to exercise the full person API and many edge cases: GET /person (200, array), POST /person (415 without Content-Type with assertion on 415 response body, 201 with full or minimal valid body, 201 with sex other, 400 for empty body, empty name, missing required fields, invalid sex, invalid date, non-whitelisted property, malformed JSON), GET /person list content (created persons included, at least two after creating two), GET /person/:id (200, 404, 400 for invalid UUID), PATCH /person/:id (415 with body assertion, 200 with empty body leaving person unchanged, 200 with partial updates, 400 for invalid dateOfBirth or sex or non-whitelisted property, 200 when updating only dateOfBirth or only sex), DELETE /person/:id (200, 404, 400, and 404 when fetching the deleted person). E2E tests require PostgreSQL to be running (e.g. via Docker).

---

## What the coverage bar tracks and why it matters

The coverage bar tracks how much of the codebase is executed when the test suite runs. It usually reports statement, line, branch, and function coverage. It matters because it gives a quick, numeric view of which parts of the code are at least run by tests. Low coverage highlights areas that are never executed and may be untested or dead code. The bar is a simple way to see whether new or existing code is included in the test run and to encourage teams to keep coverage above a minimum level.

---

## Why Focus Bear enforces a minimum test coverage threshold

Focus Bear enforces a minimum test coverage threshold (for example 80%) to keep the product stable and to reduce the chance of bugs in production. When every change is required to keep coverage above that level, developers are encouraged to add or update tests for new or modified code. The threshold acts as a shared standard so that no large part of the codebase is left without any test execution. It supports confidence when refactoring and helps prevent regressions.

---

## How high test coverage can still leave functionality untested

High test coverage can still leave real functionality untested because coverage only shows that code was executed, not that its behaviour was checked. For example, a test might call a function and get a result without asserting on the value, so the code runs (and counts as covered) but wrong behaviour would not be caught. Another case is when only the happy path is tested and error handling or edge cases are never run. Coverage can also be raised by tests that always pass and only exist to hit lines, or by testing implementation details instead of observable behaviour. So high coverage numbers can be misleading if the tests do not contain strong, behaviour-focused assertions.

---

## Examples of weak vs strong test assertions

### Weak assertions

- Only checking that a result is defined: `expect(result).toBeDefined()`.
- Not checking that the service or repository was called with the right arguments.
- Asserting that an array has a length without checking the contents.
- Catching an error type but not the message or context.

### Strong assertions

- Checking the exact shape of the result: required fields present, correct types (e.g. `id` is a string, `createdAt` is a Date), and correct values (e.g. `name` matches the input DTO).
- Verifying that dependencies were called the right number of times and with the expected arguments.
- For errors: asserting both the exception type and the message (e.g. `NotFoundException` with "Person with id \"...\" not found").
- Testing both success and failure paths (e.g. person found vs not found) with specific expectations.

---

## Balancing higher coverage with effective tests

A good balance is to treat coverage as one signal among several, and to prioritise tests that verify behaviour. First, write tests that assert on outcomes and contracts (return values, errors, and interactions with dependencies) rather than aiming only to execute more lines. Then run the coverage report and use it to find branches or files that are still uncovered, and add tests that both cover those paths and add meaningful assertions. Prefer testing public behaviour and realistic scenarios over reaching 100% coverage with shallow or redundant tests. Keep the minimum threshold (e.g. 80%) so that coverage does not drop, but do not sacrifice assertion quality for the sake of the number. Review tests in code review for assertion strength as well as coverage impact.
