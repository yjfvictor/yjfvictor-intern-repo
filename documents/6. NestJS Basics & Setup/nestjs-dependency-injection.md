# Dependency Injection in NestJS

## Goal

Understand how **dependency injection (DI)** works in NestJS and why it is fundamental to the framework. Focus Bear’s backend uses DI to keep the codebase **modular, testable, and maintainable**.

---

## 1. How does dependency injection improve maintainability?

**Dependency injection** means that a class receives its dependencies from the outside (typically via the constructor) instead of creating them itself. NestJS provides an **Inversion of Control (IoC) container** that performs this injection.

- **Loose coupling**: Classes depend on abstractions or class tokens, not on concrete instantiation. A controller or service does not need to know how to construct its dependencies. The container does. This reduces coupling and makes the design more flexible. Changing an implementation (for example, swapping a real email service for a mock) is done in one place without scattering `new` calls across the codebase.

- **Single place for wiring**: The module’s `providers` (and the container) define who gets what. All dependency wiring is centralised. When a service implementation changes, only the module or provider registration needs to be updated, not every class that uses it.

- **Clear dependencies**: Constructor parameters explicitly list what a class needs. This makes the dependency graph visible and easier to reason about. New developers can see at a glance what each component relies on.

- **Easier refactoring**: Because dependencies are injected rather than constructed inline, moving or replacing a service does not require hunting down every `new Service()` call. The container continues to resolve dependencies as long as they are registered correctly.

For a codebase like Focus Bear’s backend, this improves maintainability by keeping the structure predictable, dependencies explicit, and changes localised.

---

## 2. What is the purpose of the `@Injectable()` decorator?

The `@Injectable()` decorator marks a class as a **provider** that the NestJS IoC container can create and inject.

- **Registration**: It tells the framework that this class is eligible for dependency injection. Without it, Nest would not treat the class as a provider, and constructor injection would not work reliably.

- **Metadata**: Together with TypeScript’s `emitDecoratorMetadata`, the decorator allows Nest to read the **types** of constructor parameters at runtime. The framework uses these types as tokens to look up the correct provider in the container and inject it.

- **Scope configuration**: The decorator can accept an options object, for example `@Injectable({ scope: Scope.REQUEST })`, to set the provider’s lifecycle scope (singleton, request-scoped, or transient).

In short, `@Injectable()` declares that a class is managed by the DI container and enables automatic resolution of its dependencies via constructor injection.

---

## 3. What are the different types of provider scopes, and when would you use each?

NestJS supports three provider scopes that control how instances are created and shared.

| Scope | Behaviour | When to use it |
| ----------- | ---------- | -------------- |
| **DEFAULT** | A single instance is shared across the entire application. Created once at startup and reused for all consumers. | Default and recommended for most services. Use for stateless logic, database access, or any provider that does not need request-specific state. |
| **REQUEST** | A new instance is created for each incoming HTTP request and disposed after the request completes. | Use when you need per-request state, such as request-specific caching, request ID tracking, or multi-tenancy where the tenant is derived from the request. Be aware that any controller or provider that depends on a request-scoped provider becomes request-scoped too. |
| **TRANSIENT** | Each consumer that injects the provider receives a new, dedicated instance. Instances are never shared between consumers. | Use when each consumer must have its own copy of the provider (for example, to avoid shared mutable state between different services that use the same helper). |

- **DEFAULT (singleton)** is the default and is suitable for most application services. It is efficient and safe in Node.js because the runtime is single-threaded.

- **REQUEST** is useful when behaviour or data must be isolated per HTTP request. Overusing it can affect performance because a new instance (and its dependency tree) is created for every request.

- **TRANSIENT** is useful when you need isolation per consumer rather than per request. Each service that injects a transient provider gets its own instance.

---

## 4. How does NestJS automatically resolve dependencies?

NestJS uses an **IoC container** that resolves dependencies automatically through the following process.

- **Metadata and decorators**: The `@Injectable()` decorator marks classes as providers. With TypeScript’s `emitDecoratorMetadata` enabled, Nest can read the **types** of constructor parameters at runtime. These types act as **tokens** that the container uses to look up the right provider.

- **Registration**: When the application boots, Nest builds a **dependency graph** from the root module. Every provider listed in a module’s `providers` array is registered in the container and associated with its token (usually the class itself). The container knows how to create each provider and what it depends on.

- **Resolution**: When a class (for example, a controller or another service) is instantiated, the container inspects its constructor parameters. For each parameter, it looks up the corresponding token in the registry, creates the dependency if it does not yet exist (or reuses it according to scope), and recursively resolves that dependency’s own dependencies. Finally, it calls the constructor with the resolved instances.

- **Scopes**: The container respects each provider’s scope. For DEFAULT scope it reuses the same instance. For REQUEST scope it creates a new instance per request. For TRANSIENT scope it creates a new instance per consumer. This happens automatically during resolution.

The developer only has to declare dependencies in the constructor (for example, `constructor(private readonly userService: UserService)`) and register the provider in a module. NestJS handles the rest: it discovers what is needed, creates or retrieves the right instances, and injects them. There is no need to manually call `new` or pass dependencies from the outside.

---

## Summary

- **Maintainability**: Dependency injection improves maintainability by centralising wiring, reducing coupling, making dependencies explicit, and simplifying refactoring.
- **`@Injectable()`**: It marks a class as a provider, enables constructor-based injection, and supports scope configuration.
- **Scopes**: DEFAULT (singleton) for most services, REQUEST for per-request state, TRANSIENT for per-consumer isolation.
- **Automatic resolution**: Nest uses decorator metadata and a dependency graph to resolve constructor parameters by type and inject the correct instances according to each provider’s scope.

Understanding these concepts is essential for working effectively with NestJS and with Focus Bear’s backend.
