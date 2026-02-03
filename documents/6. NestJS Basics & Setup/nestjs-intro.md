# NestJS Introduction

## 🎯 Overview

This document reflects on the NestJS framework: how it differs from Express.js, why it uses decorators, how dependency injection works, and what benefits its modular architecture brings to large-scale applications such as Focus Bear’s backend.

---

## 1. Key Differences Between NestJS and Express.js

### Philosophy and structure

- **Express.js** is **unopinionated** and **minimal**. It gives developers full freedom over project structure, routing, and architecture. There is little built-in convention beyond middleware and basic HTTP handling.
- **NestJS** is **opinionated** and **structured**. It provides a fixed architecture (inspired by Angular) with Modules, Controllers, Services, and Providers. It runs on top of Express (or Fastify) but adds a clear, convention-based layer on top.

### Architecture

- **Express.js**: You choose how to organise code. This is flexible for small or quick prototypes but can lead to inconsistent or messy structure as the app grows.
- **NestJS**: Comes with a defined structure (Modules, Controllers, Services). This promotes consistency, reusability, and maintainability, especially in larger teams and codebases.

### Features

| Aspect                | Express.js                          | NestJS                                                                 |
| --------------------- | ----------------------------------- | ---------------------------------------------------------------------- |
| Structure             | Minimal. You define it              | Built-in modular architecture                                          |
| Dependency injection  | Not built-in (manual or third-party) | Built-in IoC container and DI                                         |
| TypeScript            | Optional, manual setup              | First-class, native support                                            |
| Decorators            | Not part of the framework           | Core: `@Controller()`, `@Injectable()`, `@Module()`, route decorators  |
| Testing               | You wire everything                 | Easier unit and integration testing via DI and modules                 |
| Underlying engine     | Standalone                          | Can use Express or Fastify (platform-agnostic)                         |

### When to use which

- **Express.js**: Small services, quick prototypes, or when you want maximum flexibility and minimal abstraction.
- **NestJS**: Larger or long-lived backends (like Focus Bear) where scalability, maintainability, and team consistency matter more than minimal setup.

---

## 2. Why NestJS Uses Decorators Extensively

Decorators are *metadata and behaviour markers* that NestJS uses to understand and wire the application without lots of boilerplate.

### What decorators do in NestJS

1. **Declare intent**  
   A class or method is marked with a single decorator so the framework knows its role:
   - `@Controller('path')` → this class handles HTTP requests for that path.
   - `@Injectable()` → this class is a provider and can be injected.
   - `@Module({ ... })` → this class describes a module and its dependencies.

2. **Attach metadata**  
   TypeScript/JavaScript decorators work with the `reflect-metadata` API. NestJS reads this metadata at runtime to:
   - Build the routing map (which controller and method handle which URL and verb).
   - Know what to inject into each class (constructor parameter types).
   - Know how modules, controllers, and providers relate.

3. **Reduce boilerplate**  
   Instead of manually registering every route and every dependency, you add a decorator and the framework discovers and wires things from the metadata.

### Examples

- **`@Controller('users')`**  
  Tells NestJS: “This class is a controller and groups its routes under `/users`.”  
  Method decorators like `@Get()`, `@Post()`, `@Body()` then define exact routes and parameters.

- **`@Injectable()`**  
  Tells NestJS: “This class can be created and injected by the DI container.”  
  Without it, the framework would not treat the class as a provider, and constructor injection would not work reliably.

NestJS therefore uses decorators extensively to keep the code declarative, to drive routing and dependency injection via metadata, and to avoid repetitive configuration code.

---

## 3. How NestJS Handles Dependency Injection

NestJS has a **built-in inversion of control (IoC) container** that creates and injects dependencies for you.

### Core idea

- You **declare** what a class needs (via its constructor).
- The framework **creates** those dependencies (and their dependencies) and **injects** them.
- You do not use `new` or manual wiring for application services. The container does it.

### Mechanism (simplified)

1. **Metadata**  
   With `@Injectable()` and TypeScript’s `emitDecoratorMetadata`, NestJS knows the **types** of constructor parameters. It uses these types as “tokens” to resolve what to inject.

2. **Registration**  
   Providers are registered in a module’s `providers` array (or via `@Injectable()` and module discovery). The container keeps a map from token → how to create the instance.

3. **Resolution**  
   When a class is instantiated (e.g. a controller or another service), the container:
   - Looks at the constructor parameters.
   - Resolves each dependency (creating it if necessary, and recursively resolving its dependencies).
   - Instantiates the class and passes in those dependencies.

So dependency injection in NestJS is **constructor-based**: you list your dependencies in the constructor, and the container fills them in.

### Scopes

- **DEFAULT (singleton)**: One instance per application, created at startup and reused.
- **REQUEST**: One instance per incoming HTTP request (useful for request-scoped state).
- **TRANSIENT**: One new instance per consumer that asks for it.

### Benefits

- **Testability**: You can replace real services with mocks/stubs by registering different providers in tests.
- **Loose coupling**: Classes depend on abstractions (interfaces or class tokens), not on concrete `new` calls.
- **Single place to configure**: The container is the single place that decides how to create and connect objects.

---

## 4. Benefits of Modular Architecture in a Large-Scale Application

NestJS’s **modular architecture** (grouping the app into **Modules** that contain Controllers, Services, and other providers) gives several benefits as the app grows.

### Clear boundaries and separation of concerns

- Each **module** groups one area of the app (e.g. “users”, “habits”, “payments”).
- Controllers handle HTTP, services hold business logic, and modules define what is visible to the rest of the app.
- This makes it easier to find code and understand what each part is responsible for.

### Maintainability and navigation

- Related code lives together instead of being scattered.
- New developers can work feature-by-feature (one module at a time).
- Changes in one feature are less likely to break unrelated features if modules are well isolated.

### Reusability

- Modules can **export** selected providers and **import** other modules.
- Shared logic (e.g. auth, database, logging) can live in shared or global modules and be reused across the app or in other NestJS projects.

### Easier testing

- You can load and test a single module with mocked imports.
- Integration tests can run with a subset of modules, which keeps tests faster and more focused.

### Team collaboration

- Different people or teams can own different modules.
- Clear public APIs (exports) and dependencies (imports) reduce merge conflicts and accidental coupling.

### Scalability

- New features become new modules (or new parts of existing modules) without rewriting the whole app.
- NestJS also supports monorepos and workspaces, so multiple apps or services can share modules in a structured way.

For a product like Focus Bear, where the backend is expected to grow and be maintained by a team, this modular architecture supports **scalability, modularity, and maintainability** in a way that a minimal, unstructured Express app would not.

---

## ✅ Summary

- **NestJS vs Express**: NestJS adds an opinionated, modular structure and built-in DI on top of Express (or Fastify). Express is minimal and flexible. NestJS suits larger, long-lived backends.
- **Decorators**: They declare roles and attach metadata so NestJS can build routes and the dependency graph with minimal boilerplate.
- **Dependency injection**: Constructor-based. The IoC container uses TypeScript metadata to resolve and inject dependencies, improving testability and decoupling.
- **Modular architecture**: Modules give clear boundaries, easier maintenance, reuse, testing, and team scalability, all of which are important for a backend like Focus Bear’s.
