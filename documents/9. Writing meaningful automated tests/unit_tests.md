# Reflection: Unit Testing

This document contains reflections on two projects: mocking API calls in Jest, and introduction to unit testing with Jest.

---

## Reflection: Mocking API Calls in Jest

### Summary of the Mocking API Calls project

The project [test-repo-for-mocking-API-calls-in-Jest](https://github.com/yjfvictor/test-repo-for-mocking-API-calls-in-Jest) is a NestJS application that demonstrates how to mock API calls in Jest when testing asynchronous code. It includes:

- A NestJS backend with an ApiDataService that fetches post data from the JSONPlaceholder API using axios
- A PostDisplay React component that fetches and displays a single post using the native fetch API
- Unit tests for both the service and the React component that use `jest.fn()` to create mock HTTP clients and mock the global fetch
- Tests verify success paths, error handling, and component behaviour without making real network requests

### Why is it important to mock API calls in tests?

Mocking API calls in tests is important for several reasons.

First, tests should be fast and reliable. Real network requests introduce latency and can fail due to connectivity issues, rate limits, or changes in the external service. Mocked calls run instantly and return predictable data, so tests complete quickly and pass or fail based on application logic, not on external factors.

Second, tests should be isolated. When testing a component or service that fetches data, we want to verify that it correctly handles the response and renders or processes it. By mocking the API, we control exactly what data is returned and can assert that our code behaves correctly for different scenarios, including error cases, without depending on a live backend.

Third, mocking avoids side effects. Real API calls can create logs, analytics events, or database changes. In unit tests, we typically want to focus on the behaviour of the code under test and avoid such side effects. Mocking keeps the test scope narrow and the environment clean.

### What are some common pitfalls when testing asynchronous code?

Several pitfalls often cause flaky or misleading tests when working with asynchronous code.

**Not waiting for async operations to complete.** If an assertion runs before a Promise resolves or a component finishes updating, the test may pass or fail incorrectly. Use `async`/`await`, `waitFor`, or Jest's `resolves`/`rejects` matchers so that the test waits for the async work to finish before asserting.

**Forgetting to clear or reset mocks between tests.** Mocks like `jest.fn()` retain their call history and return values across tests. If one test configures a mock and another assumes a clean state, the second test can be affected by the first. Call `jest.clearAllMocks()` or `mockFn.mockClear()` in `beforeEach` to reset state.

**Not handling Promise rejections.** When mocking a failing API call, ensure the mock returns a rejected Promise or throws. If the rejection is not properly propagated, the test might not catch error-handling bugs, or the test runner may report an unhandled rejection.

**Testing implementation details instead of behaviour.** It can be tempting to assert that a specific function (e.g. fetch or axios.get) was called with exact arguments. While such checks are sometimes useful, over-reliance on them ties tests to implementation. Prefer asserting on the visible outcome, such as rendered content or returned data, so that refactoring internals does not break tests unnecessarily.

---

## Reflection: Introduction to Unit Testing with Jest

### Summary of the Introduction to Unit Testing project

The project [test-repo-for-introduction-to-unit-testing-with-jest](https://github.com/yjfvictor/test-repo-for-introduction-to-unit-testing-with-jest) is a NestJS application set up to introduce unit testing with Jest. The repository includes:

- A minimal NestJS backend with a root module, controller, and service that expose `GET /` and `GET /health`
- A utility module with a single pure function that adds two numbers (`add(a, b)`)
- Jest configured for TypeScript via ts-jest, with tests in the `test/` directory
- Unit tests for the add function covering positive integers, zero, negative numbers, and floating-point values
- All source and test code written in TypeScript with explicit types and structured documentation

### Why is automated testing important in software development?

Automated testing is important for several reasons.

First, it helps catch regressions. When code is changed or new features are added, tests can be run repeatedly to ensure that existing behaviour still holds. Without automated tests, manual checks are slow and easy to skip or forget.

Second, it supports refactoring. Developers can change implementation details with more confidence if a test suite passes before and after the change. Tests act as a safety net and encourage cleaner, more maintainable code.

Third, it documents behaviour. Well-named tests describe what the code is supposed to do. New team members can read tests to understand expected inputs, outputs, and edge cases.

Fourth, it improves reliability in production. Projects that rely on automated testing (such as Focus Bear) can release updates with less risk of breaking existing features, because the test suite is run as part of the development and release process.

### What did you find challenging when writing your first Jest test?

A few things can be challenging when writing a first Jest test.

One is knowing what to test. For a simple function like `add`, the behaviour is straightforward, but deciding how many cases to include (e.g. zero, negatives, floats) and how to name each test so it clearly describes the scenario takes a bit of practice.

Another is test environment and configuration. Ensuring Jest is installed, ts-jest is configured so TypeScript compiles correctly, and that the test runner can find the spec files (e.g. correct `testRegex` and placement of `*.spec.ts` files) can be confusing the first time. Getting the first passing test often involves fixing path or configuration issues.

A third is floating-point assertions. For a function that adds two numbers, testing `0.1 + 0.2` with `expect(result).toBe(0.3)` can fail due to floating-point representation. Using `toBeCloseTo(0.3)` instead of `toBe(0.3)` is the right approach, but it is something you learn when writing or debugging that first test.

Once the first test passes and the setup is in place, adding more tests for the same or other functions becomes much easier.
