# NestJS Setup and Project Structure

## Goal

Set up a NestJS development environment, understand its project structure, and run a basic application. This reflection summarises the setup steps, default files, and how the framework bootstraps and scales.

---

## 1. Steps to Set Up a New NestJS Project

I followed the steps below and used the test repository [my-test-nestjs-project](https://github.com/yjfvictor/my-test-nestjs-project).

### Prerequisites

- **Node.js** (version 20 or higher). I had this installed on my system.
- A package manager such as **npm** or **yarn** (I used **npm**).

### Installation and initialisation

1. **Install the Nest CLI**

   I ran:

   ```bash
   npm install -g @nestjs/cli
   ```

   Alternatively, I could have used `npx @nestjs/cli@latest new my-test-nestjs-project` without a global install.

2. **Clone my GitHub repo**

   Before creating the Nest project, I ran:

   ```bash
   git clone git@github.com:yjfvictor/my-test-nestjs-project.git
   ```

   so that I had the remote repository ready for the Nest app.

3. **Create the Nest project**

   I ran:

   ```bash
   nest new my-test-nestjs-project
   ```

   The CLI can prompt for a package manager (npm or yarn), or I could pass `--package-manager npm` or `--package-manager yarn` to skip the prompt. Other options include `--strict` for stricter TypeScript, `--skip-git` to skip Git initialisation, and `--skip-install` to skip installing dependencies. I did not use these flags.

4. **Enter the project and run the dev server**

   I ran:

   ```bash
   cd my-test-nestjs-project
   npm run start:dev
   ```

   The app runs at `http://localhost:3000` and recompiles when source files change. I could have used `npm run start` for a one-off run without watch mode, but I did not.

5. **Test a simple endpoint**

   I opened `http://localhost:3000` in a browser. Alternatively, I could also use `curl http://localhost:3000` instead. The default root route returns "Hello World!".

---

## 2. What Files Are Included in a Default NestJS Project?

A project created with `nest new` contains the following.

### Source code (`src/`)

- **`main.ts`**: Entry point. Contains the async `bootstrap()` function that creates the app and starts the HTTP server.
- **`app.module.ts`**: Root module. Declares the app's top-level controllers, services, and imports.
- **`app.controller.ts`**: Default controller with a single GET route (e.g. root returning "Hello World!").
- **`app.service.ts`**: Default service with a simple method used by the controller.
- **`app.controller.spec.ts`**: Unit tests for the default controller.

### Configuration and tooling

- **`nest-cli.json`**: Nest CLI configuration (e.g. source and build paths, compiler options).
- **`tsconfig.json`**: TypeScript compiler options for the project.
- **`tsconfig.build.json`**: TypeScript options used when building for production.
- **`package.json`**: Dependencies and scripts (`start`, `start:dev`, `build`, `lint`, `format`, etc.).
- **`.eslintrc.js`** (or similar): ESLint rules.
- **`.prettierrc`** (or similar): Prettier formatting rules.

### Tests and other

- **`test/`**: E2E test setup (e.g. `jest-e2e.json`, `app.e2e-spec.ts`).
- **`node_modules/`**: Installed dependencies (after `npm install`).

The CLI encourages keeping each feature in its own directory under `src/`, with modules, controllers, and services grouped by domain.

---

## 3. How Does `main.ts` Bootstrap a NestJS Application?

`main.ts` is the application entry point. It does two things: create the Nest application instance and start the HTTP server.

1. **Create the app:** The code calls `NestFactory.create(AppModule)`. `NestFactory` is a core class that builds the Nest app. It receives the **root module** (`AppModule`), which describes the entire application (controllers, services, and imported modules). `create()` is async and returns an object that implements `INestApplication`.

2. **Start listening:** The code then calls `await app.listen(process.env.PORT ?? 3000)`. This starts the underlying HTTP platform (Express by default) and makes the app listen for incoming requests on the given port.

So bootstrap is: load the root module, build the dependency-injection graph and route map from it, then start the HTTP listener. Any global configuration (e.g. validation pipes, CORS, or middleware) can be applied to the `app` object between `create()` and `listen()`.

---

## 4. What Is the Role of `AppModule` in the Project?

`AppModule` is the **root module** of the application. In NestJS, every app has at least one module, and the root is the one passed to `NestFactory.create()`.

Its role is to:

- **Define the application boundary:** It declares which controllers and providers (e.g. services) belong to the app and which other modules it imports. Nothing is part of the running app unless it is reachable from the root module (directly or via imported modules).
- **Wire the default pieces:** In a fresh project, it typically registers `AppController` and `AppService` in its `controllers` and `providers` arrays so the default "Hello World!" route and logic are available.
- **Compose the app:** As the app grows, `AppModule` imports feature modules (e.g. `UsersModule`, `HabitsModule`) instead of listing every controller and service. The root module stays small and acts as a composition root.

So `AppModule` is the top-level descriptor Nest uses to build the dependency-injection container and the routing tree. It does not hold business logic. It declares what the application is made of.

---

## 5. How Does NestJS Structure Help with Scalability?

- **Modular boundaries:** Features are grouped into modules. Each module owns its controllers, services, and exports. New features become new modules (or new parts of existing modules) without scattering code or rewriting the whole app. The root module composes these modules, which keeps the app navigable as it grows.

- **Clear separation of concerns:** Controllers handle HTTP, services hold business logic, and modules define visibility and dependencies. This makes it easier to locate code, change one layer without touching others, and onboard new developers who can work feature-by-feature.

- **Dependency injection and testability:** The built-in IoC container and constructor injection make it easy to replace real services with mocks in tests and to add new providers without manual wiring. Tests can load a single module with mocked imports, which keeps tests fast and focused.

- **Reuse and team scaling:** Modules can export selected providers and import other modules. Shared or global modules (e.g. auth, database, logging) can be reused across the app or in other NestJS projects. Clear public APIs (exports) and dependencies (imports) reduce coupling and support multiple developers or teams working on different modules.

- **Convention and tooling:** The Nest CLI generates modules, controllers, and services in a consistent structure. A predictable layout and CLI support (e.g. `nest generate module users`) keep the codebase consistent as it scales.

Together, these aspects make NestJS well suited to a long-lived, team-maintained backend such as Focus Bear’s, where scalability, maintainability, and clear structure matter more than a minimal, ad-hoc setup.

---

## Summary

- **Setup:** Install Node.js 20+, optionally the Nest CLI, then run `nest new project-name`, `cd` into it, and use `npm run start:dev` to run and test the default endpoint.
- **Default files:** `src/main.ts`, `app.module.ts`, `app.controller.ts`, `app.service.ts`, `app.controller.spec.ts`, plus config files (`nest-cli.json`, `tsconfig.json`, `package.json`, ESLint, Prettier) and a `test/` folder for E2E tests.
- **`main.ts`:** Bootstraps the app by calling `NestFactory.create(AppModule)` to build the app from the root module, then `app.listen(port)` to start the HTTP server.
- **`AppModule`:** Root module that defines the application boundary, registers (or imports) controllers and providers, and composes feature modules.
- **Scalability:** Modular structure, separation of concerns, DI, reusable modules, and CLI conventions support growth, testing, and team collaboration.
