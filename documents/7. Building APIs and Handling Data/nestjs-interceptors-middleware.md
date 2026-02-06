# Using Interceptors and Middleware in NestJS

## Summary of Tasks

In the GitHub repository [test-repo-for-using-interceptors-and-middleware-in-NestJS](https://github.com/yjfvictor/test-repo-for-using-interceptors-and-middleware-in-NestJS), a full NestJS application was created that demonstrates interceptors and middleware.

Research was done on the difference between interceptors and middleware in NestJS. Middleware runs early in the request lifecycle, before guards, pipes, and interceptors. It operates on raw Express request and response objects directly. Interceptors run closer to the route handler, both before and after it is called. They have access to the execution context and can transform both incoming requests and outgoing responses.

Built-in interceptors such as `ClassSerializerInterceptor` were explored. `ClassSerializerInterceptor` uses `class-transformer` and the `@Exclude()` and `@Expose()` decorators to control which properties are included in API responses, helping to protect sensitive data.

A custom `LoggingInterceptor` was implemented that logs the HTTP method, URL, and execution time of each request. It wraps the request-response stream using RxJS `tap()` and logs before the handler runs and after the response is sent.

A `RequestLoggerMiddleware` class was created and applied to specific routes (ItemsController) via `MiddlewareConsumer.configure()`. A functional `globalLogger` middleware was also created and applied globally to all routes via `app.use()` in `main.ts`. This demonstrates both route-specific and global middleware patterns.

The application includes an `ItemEntity` with `@Exclude()` on the `internalSecret` field and `@Expose()` on public fields, used with `ClassSerializerInterceptor` to show serialisation control.

## What is the difference between an interceptor and middleware in NestJS?

**Execution timing.** Middleware runs first in the request lifecycle, before guards, pipes, and interceptors. Interceptors run closer to the route handler, both before and after it is invoked. The typical order is: Middleware, Guards, Pipes, Interceptors (before), Controller, Interceptors (after), Response.

**Access and context.** Middleware operates on raw Express (or Fastify) request and response objects directly, even before NestJS features take effect. Interceptors have access to the `ExecutionContext`, which includes the request, response, and controller arguments. Interceptors can wrap the entire request-response stream using RxJS operators.

**Capabilities.** Middleware can execute code, modify the request and response objects, and end or continue the request-response cycle. It cannot directly transform controller handler responses or access route handler results. Interceptors can transform both incoming requests and outgoing responses, implement caching, handle exceptions, measure execution time, and extend or override function behaviour.

**Registration.** Middleware is registered via `MiddlewareConsumer.configure()` in a module that implements `NestModule`, or globally via `app.use()`. Interceptors are registered with `@UseInterceptors()` on controllers or methods, or globally with `app.useGlobalInterceptors()` or the `APP_INTERCEPTOR` provider.

## When would you use an interceptor instead of middleware?

Use an interceptor when you need to:

- **Transform responses.** Wrap response data in a standard format (for example `{ data: result }`), convert `null` to empty strings, or apply other response transformations. Middleware cannot modify controller output.

- **Measure execution time.** Interceptors run both before and after the handler, so they can record the start time, call `next.handle()`, and calculate elapsed time in the `tap()` or `map()` callback. Middleware only runs before the handler.

- **Log at the controller level.** If you need to log the actual response body or status code returned by the handler, use an interceptor. Middleware runs before the handler and does not see the handler result.

- **Handle exceptions.** Interceptors can use RxJS `catchError()` to transform thrown exceptions, log them, or return a different response. Middleware runs before exceptions are thrown by the handler.

- **Apply serialisation rules.** `ClassSerializerInterceptor` and similar interceptors use DTO or entity metadata to control output. This requires access to the handler result, which middleware does not have.

Use middleware when you need to:

- Log all requests including 404s and unmatched routes
- Parse tokens or headers before guards run
- Apply CORS or other request-level processing
- Manipulate headers or cookies before the request reaches route handlers

## How does LoggerErrorInterceptor help?

There is no built-in `LoggerErrorInterceptor` in NestJS. The concept refers to an interceptor pattern that combines logging with error handling. Such an interceptor helps by:

- **Capturing exceptions.** Using RxJS `catchError()`, it intercepts exceptions thrown from route handlers before they reach exception filters.

- **Logging errors.** It logs the exception (stack trace, message, context) to a logger or logging service. This provides a central place to record errors for debugging and monitoring.

- **Optionally transforming errors.** It can rethrow the original exception, transform it into a different exception (for example standardise error format), or return a fallback response. This keeps error responses consistent and avoids exposing internal details.

- **Adding context.** Because interceptors have access to `ExecutionContext`, they can log the HTTP method, URL, user id, or other context along with the error, making logs more useful.

In practice, you would implement a `LoggingErrorInterceptor` (or similar name) that uses `catchError()` to log exceptions and optionally map them before rethrowing or returning an alternative response.
