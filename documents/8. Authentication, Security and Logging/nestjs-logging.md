# Reflection: Logging and Error Handling in NestJS

## Summary of the project

The project [test-repo-for-logging-and-error-handling-in-NestJS](https://github.com/yjfvictor/test-repo-for-logging-and-error-handling-in-NestJS) is a NestJS application that demonstrates structured logging and global exception handling. The repository was cloned and extended with the following: integration of **nestjs-pino** for Pino-based structured logging (with environment-based log level and optional pino-pretty in development), a **custom global exception filter** (`AllExceptionsFilter`) that catches every unhandled exception and returns a consistent JSON error response (statusCode, timestamp, path, message, optional errorCode), and **unit tests** for the application controller, service, root module, logger configuration, and the exception filter. The application exposes a root route and two intentional error routes (`/error` and `/http-error`) to illustrate filter behaviour.

---

## What are the benefits of using nestjs-pino for logging?

nestjs-pino brings Pino into the NestJS ecosystem so that application and request logs are structured (typically JSON), fast, and easy to integrate with log aggregators. Benefits include: **structured JSON output** that can be parsed and queried by tools (e.g. ELK, Datadog), **high performance** with minimal overhead compared to plain console logging, **automatic request/response logging** via pino-http so that each HTTP request and its response (and timing) are logged without extra code in every controller, **request context** so that logs emitted during a request can be correlated (e.g. by request id), and **drop-in compatibility** with the standard NestJS Logger API so existing code using `Logger` continues to work while gaining Pino’s backend. Using pino-pretty in development keeps logs human-readable while production can emit raw JSON for pipelines.

---

## How does global exception handling improve API consistency?

A global exception filter runs for every unhandled exception in the application. By centralising the logic that builds the error response, every failure (whether from a controller, service, or guard) is turned into the same JSON shape: statusCode, timestamp, path, message, and optionally errorCode. Clients can rely on a single response format regardless of where the error originated, which simplifies error handling on the front end and in API contracts. It also allows a single place to add security-sensitive behaviour (e.g. hiding stack traces or internal details in production) and to log all errors for monitoring and debugging.

---

## What is the difference between a logging interceptor and an exception filter?

A **logging interceptor** runs in the normal request/response pipeline. It typically wraps the call to the route handler and logs before and after the handler runs (e.g. method, URL, duration, status). It does not handle exceptions: if the handler throws, the exception propagates to the exception layer. An **exception filter** runs only when an exception has been thrown. It receives the exception and the request context and is responsible for sending the HTTP error response (and optionally logging the error). So interceptors are for observability of successful and failed requests along the pipeline, while exception filters are for turning exceptions into a consistent error response and optionally logging them. In this project, request/response logging is provided by pino-http (via nestjs-pino), and the exception filter is used to format and send error responses and to log caught exceptions.

---

## How can logs be structured to provide useful debugging information?

Useful debugging logs are **structured** (e.g. JSON) so that fields can be filtered and aggregated. Important fields include: **timestamp** (ISO 8601) for ordering and correlation, **level** (e.g. debug, info, warn, error) for filtering by severity, **context** or **logger name** to identify the source (e.g. controller or service), **message** for a short human-readable description, and **request context** (e.g. request id, method, path, and optionally user id or correlation id) so that all logs for a single request can be grouped. For errors, include **stack traces** and **error type** where safe (e.g. in non-production or in a dedicated error log stream). Avoid logging sensitive data (passwords, tokens, full PII) or high-volume payloads that would obscure important events. In this project, nestjs-pino and pino-http provide these structures by default (level, time, context, req, res, responseTime), and the custom exception filter adds error-level logs with the exception message and stack when an exception is caught.
