# Validating Requests with Pipes in NestJS

## Summary of Tasks

In the GitHub repo [Test-repo-for-validating-requests-with-pipes-in-NestJS](https://github.com/yjfvictor/Test-repo-for-validating-requests-with-pipes-in-NestJS), a full NestJS application was added that demonstrates pipes and request validation.

Research was done on what pipes are in NestJS and how they work. Pipes are classes that implement `PipeTransform`: they run just before a route handler is invoked and can transform input (e.g. string to number) or validate it (pass through or throw). Built-in pipes such as `ValidationPipe` and `ParseIntPipe` were explored. A custom DTO (`CreateItemDto`) was created with `class-validator` decorators (`@IsString()`, `@IsNumber()`, `@IsNotEmpty()`, `@MinLength()`, `@MaxLength()`, `@Min()`, `@Max()`). A second DTO (`FindOneParamsDto`) was added for path parameters using `@IsNumberString()`. A global validation pipe was configured in `main.ts` with `app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }))` so that DTO validation applies across the app. The built-in `ParseIntPipe` was used on a route parameter and a custom pipe (`ParsePositiveIntPipe`) was implemented to show the difference between built-in and custom pipes.

## What is the purpose of pipes in NestJS?

Pipes have two main purposes. First, they can **transform** input data into the form the route handler expects (for example converting a string from the URL into a number). Second, they can **validate** input: if the data is valid they pass it through unchanged, and if not they throw an exception so the route handler is never called. Pipes run at the system boundary, so they are the right place to enforce that only correct, safe data reaches the rest of the application.

## How does ValidationPipe improve API security and data integrity?

`ValidationPipe` improves security and data integrity in several ways. It uses the DTO class and its `class-validator` decorators to check that request bodies (and when used with params or query, those too) match the expected shape and rules. Invalid requests receive a 400 response and the handler is not executed, so bad data never reaches business logic. With `whitelist: true`, properties that are not defined on the DTO are stripped from the payload, which prevents clients from injecting extra fields (e.g. to override server-side values). With `forbidNonWhitelisted: true` you can instead reject requests that contain unknown properties. With `transform: true`, payloads are converted to instances of the DTO class and primitives can be coerced to the right types, so the handler receives consistent, typed data. Together, this ensures that only allowed, well-formed data is processed and reduces risk of injection or inconsistent state.

## What is the difference between built-in and custom pipes?

Built-in pipes are provided by NestJS (`@nestjs/common`) and cover common cases: `ValidationPipe` for DTO-based validation, `ParseIntPipe`, `ParseFloatPipe`, `ParseBoolPipe`, `ParseUUIDPipe`, and others for parsing and basic validation of parameters. They are ready to use and configurable via options. Custom pipes are classes you write that implement `PipeTransform`. You use them when you need behaviour that the built-in pipes do not offer, for example parsing a string to a positive integer only (as in the custom `ParsePositiveIntPipe`), or applying domain-specific validation. Both built-in and custom pipes are bound in the same way (parameter, method, controller, or globally) and integrate with Nest’s exception handling.

## How do decorators like @IsString() and @IsNumber() work with DTOs?

Decorators such as `@IsString()` and `@IsNumber()` come from the `class-validator` package. They are attached to properties of a DTO class and declare validation rules. When `ValidationPipe` runs, it uses the `metatype` of the route parameter (the DTO class) to know which class to validate against. The pipe uses `class-transformer` to turn the plain request payload into an instance of that class, then runs `class-validator`’s `validate()` on that instance. Each decorator checks the property: `@IsString()` ensures the value is a string, `@IsNumber()` that it is a number, and so on. If any check fails, `validate()` returns a list of errors and the pipe throws an exception (e.g. `BadRequestException`), so the controller method is not called. Thus the DTO class is the single place where the allowed shape and rules of the data are defined, and the decorators are the mechanism that enforces them at runtime.
