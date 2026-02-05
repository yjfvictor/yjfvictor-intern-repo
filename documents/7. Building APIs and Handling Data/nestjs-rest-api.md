# Creating REST APIs with NestJS

## Summary of tasks

The goal was to learn how to build RESTful APIs using NestJS controllers and services. The work was done in the repository [Test-Repo-Creating-REST-APIs-with-NestJS](https://github.com/yjfvictor/Test-Repo-Creating-REST-APIs-with-NestJS).

Tasks completed:

1. **Research**  
   REST APIs in NestJS are structured around controllers (routes), services (business logic), and modules (grouping). HTTP method decorators such as `@Get()`, `@Post()`, `@Put()` and `@Delete()` map to handlers, and request data is read via `@Body()`, `@Param()` and `@Query()`.

2. **Controller with CRUD routes**  
   An `ItemsController` was added with basic CRUD: `GET /items` (list all), `GET /items/:id` (get one), `POST /items` (create), `PUT /items/:id` (update), `DELETE /items/:id` (delete).

3. **Service for business logic**  
   An `ItemsService` was created to hold all item operations (in-memory store for the demo). The controller only forwards HTTP input to the service and returns the service result.

4. **Testing**  
   The endpoints can be tested with a tool such as Postman or cURL. The repo README includes example cURL commands for each operation. After running `npm install` and `npm run start:dev`, the API is available at `http://localhost:3000`.

---

## What is the role of a controller in NestJS?

A controller is responsible for handling **incoming HTTP requests** and sending **responses** back to the client. It does not implement business rules. Its job is to:

- Define which URLs and HTTP methods it handles (routing)
- Extract input from the request (path params, query, body) using decorators such as `@Param()`, `@Query()` and `@Body()`
- Call one or more services to perform the work
- Return the result (or throw exceptions), and let NestJS serialise the response (e.g. to JSON) and set status codes

So the controller is the **HTTP boundary** of the application. It stays thin and delegates all business logic to providers (e.g. services).

---

## How should business logic be separated from the controller?

Business logic should live in **services** (or other injectable providers), not in the controller. The controller should only:

1. Receive and validate (or pass to validation pipes) the request data
2. Call the appropriate service method with that data
3. Return the value returned by the service, or let exceptions bubble up

The service holds the actual logic: creating, reading, updating and deleting data, applying rules, and possibly calling other services or repositories. This separation is achieved by:

- Defining one or more service classes marked with `@Injectable()`
- Registering them in the module’s `providers` array
- Injecting the service into the controller via the constructor so the controller can call it

The controller then contains no business logic, only HTTP handling and delegation.

---

## Why is it important to use services instead of handling logic inside controllers?

Using services has several benefits:

- **Testability**  
  Services can be unit tested without HTTP. Controllers can be tested by mocking the service. Logic in the controller would require more complex request/response mocks.

- **Reusability**  
  The same service can be used by multiple controllers (e.g. REST and GraphQL), by other services, or by background jobs. Logic locked inside a controller is not reusable.

- **Single responsibility**  
  Controllers focus on HTTP. Services focus on domain and application logic. This keeps the codebase easier to understand and change.

- **Maintainability**  
  When business rules change, only the service is updated. Controllers stay stable. This reduces the risk of breaking the API surface when fixing or extending behaviour.

For these reasons, NestJS encourages a thin controller layer and a service layer that holds the real logic.

---

## How does NestJS automatically map request methods (GET, POST, etc.) to handlers?

NestJS uses **decorators** and **metadata** to build a routing map at startup.

1. **Controller path prefix**  
   The `@Controller('items')` decorator on the class sets a path prefix (e.g. `/items`) for all routes in that controller.

2. **Method decorators**  
   On each handler method, a decorator declares the HTTP method and optional path:
   - `@Get()` maps to GET (and no extra path, or a path given as `@Get(':id')`)
   - `@Post()` maps to POST
   - `@Put(':id')` maps to PUT for a path with an `:id` segment
   - `@Delete(':id')` maps to DELETE for that path

3. **Metadata and routing map**  
   NestJS (and the underlying platform) uses the metadata attached by these decorators to build a map from (method, path) to the correct controller instance and method. When a request arrives, the framework matches method and path and invokes the corresponding handler.

4. **Parameter decorators**  
   Decorators such as `@Param('id')`, `@Body()` and `@Query()` tell NestJS how to fill the handler’s arguments from the request. The framework injects the right values before calling the method.

So the mapping is “automatic” in the sense that you do not manually register routes. You declare them with decorators, and NestJS discovers and wires them from the metadata.
