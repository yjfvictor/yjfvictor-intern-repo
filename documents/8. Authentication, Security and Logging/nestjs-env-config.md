# Reflection: Environment Variables and Configuration in NestJS

## Summary of the project

The project [test-repo-for-handling-environment-variables-and-configuration-in-NestJS](https://github.com/yjfvictor/test-repo-for-handling-environment-variables-and-configuration-in-NestJS) is a NestJS application that demonstrates how to manage environment variables and application configuration in a secure and maintainable way. The repository was cloned and extended with the following: integration of `@nestjs/config`, a `.env` file workflow with an `.env.example` template, validation of environment variables using both a custom `validate` function (with class-validator and class-transformer) and a Joi schema, a custom configuration factory that exposes nested config (e.g. `app.port`, `database.host`), and unit tests for the validation logic and for services that use `ConfigService`.

---

## How does `@nestjs/config` help manage environment variables?

`@nestjs/config` provides a `ConfigModule` that loads key-value pairs from a `.env` file (and from the process environment), merges them, and exposes them through a `ConfigService`. The module can be registered with `ConfigModule.forRoot()` so that it loads a `.env` file from the project root (or a custom path), optionally ignores the file in production, and can be made global so that `ConfigService` is available in any module without re-importing. The package uses `dotenv` under the hood, so behaviour matches standard `.env` parsing. In addition, `ConfigModule` supports a `load` array of custom configuration factories that return nested objects, so you can structure configuration (e.g. by feature) and access it via `ConfigService.get('database.port')`. This centralises configuration in one place and keeps environment-specific values out of source code.

---

## Why should secrets (e.g. API keys, database passwords) never be stored in source code?

Secrets in source code are visible to anyone with access to the repository (including history), are duplicated in every clone and fork, and are hard to rotate without code changes. If the repository is public or leaked, those secrets are exposed. In contrast, keeping secrets in environment variables (or a secure vault) allows different values per environment (local, staging, production), avoids committing them when `.env` is gitignored, and supports rotation by changing the environment rather than the code. For these reasons, API keys, database passwords, and similar sensitive values should only live in the environment (or a secret manager), not in source code.

---

## How can you validate environment variables before the app starts?

Two mechanisms are used in this project. First, a custom **validate** function can be passed to `ConfigModule.forRoot({ validate })`. This function receives the merged configuration object (from `.env` and `process.env`), and can transform and validate it (e.g. using class-validator and class-transformer). If it throws an error, NestJS does not finish bootstrapping, so the application never starts with invalid configuration. Second, a **validationSchema** (e.g. a Joi schema) can be passed to `forRoot()`. The `@nestjs/config` package runs this schema against the environment variables and, if validation fails, throws before the app starts. In both cases, validation runs when the configuration is loaded, so missing or invalid required variables (or values that fail constraints) cause an immediate, clear failure rather than runtime errors later.

---

## How can you separate configuration for different environments (e.g. local vs. production)?

Several approaches are used together. First, use a **different `.env` file per environment** (e.g. `.env.development`, `.env.production`). The `envFilePath` option in `ConfigModule.forRoot()` can point to a single file or an array of paths; the order of the array determines precedence. Second, **set `NODE_ENV`** (or a similar variable) so the application knows which environment it is in; the custom configuration factory or validation can then branch or apply different defaults. Third, in **production**, you can set `ignoreEnvFile: true` and supply all variables via the process environment (e.g. from a container or platform), so no `.env` file is read and secrets are provided by the deployment system. Fourth, the **custom configuration factory** in `load` can read `process.env.NODE_ENV` (or similar) and return different structures or defaults per environment. Combining these allows a single codebase to behave correctly in local, staging, and production without storing production secrets in the repo.
