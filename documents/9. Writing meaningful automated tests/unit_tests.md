# Reflection: Mocking API Calls in Jest

## Summary of the project

The project [test-repo-for-mocking-API-calls-in-Jest](https://github.com/yjfvictor/test-repo-for-mocking-API-calls-in-Jest) is a NestJS application that demonstrates how to mock API calls in Jest when testing asynchronous code. It includes:

- A NestJS backend with an ApiDataService that fetches post data from the JSONPlaceholder API using axios
- A PostDisplay React component that fetches and displays a single post using the native fetch API
- Unit tests for both the service and the React component that use `jest.fn()` to create mock HTTP clients and mock the global fetch
- Tests verify success paths, error handling, and component behaviour without making real network requests

---

## Why is it important to mock API calls in tests?

Mocking API calls in tests is important for several reasons.

First, tests should be fast and reliable. Real network requests introduce latency and can fail due to connectivity issues, rate limits, or changes in the external service. Mocked calls run instantly and return predictable data, so tests complete quickly and pass or fail based on application logic, not on external factors.

Second, tests should be isolated. When testing a component or service that fetches data, we want to verify that it correctly handles the response and renders or processes it. By mocking the API, we control exactly what data is returned and can assert that our code behaves correctly for different scenarios, including error cases, without depending on a live backend.

Third, mocking avoids side effects. Real API calls can create logs, analytics events, or database changes. In unit tests, we typically want to focus on the behaviour of the code under test and avoid such side effects. Mocking keeps the test scope narrow and the environment clean.

---

## What are some common pitfalls when testing asynchronous code?

Several pitfalls often cause flaky or misleading tests when working with asynchronous code.

**Not waiting for async operations to complete.** If an assertion runs before a Promise resolves or a component finishes updating, the test may pass or fail incorrectly. Use `async`/`await`, `waitFor`, or Jest's `resolves`/`rejects` matchers so that the test waits for the async work to finish before asserting.

**Forgetting to clear or reset mocks between tests.** Mocks like `jest.fn()` retain their call history and return values across tests. If one test configures a mock and another assumes a clean state, the second test can be affected by the first. Call `jest.clearAllMocks()` or `mockFn.mockClear()` in `beforeEach` to reset state.

**Not handling Promise rejections.** When mocking a failing API call, ensure the mock returns a rejected Promise or throws. If the rejection is not properly propagated, the test might not catch error-handling bugs, or the test runner may report an unhandled rejection.

**Testing implementation details instead of behaviour.** It can be tempting to assert that a specific function (e.g. fetch or axios.get) was called with exact arguments. While such checks are sometimes useful, over-reliance on them ties tests to implementation. Prefer asserting on the visible outcome, such as rendered content or returned data, so that refactoring internals does not break tests unnecessarily.
