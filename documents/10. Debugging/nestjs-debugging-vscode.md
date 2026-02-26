# Reflection: Debugging NestJS with VS Code and Breakpoints

## Summary of the Tasks

The tasks were completed in the repository [test-repo-for-debugging-with-vs-code-and-breakpoints](https://github.com/yjfvictor/test-repo-for-debugging-with-vs-code-and-breakpoints). The work included researching how to configure VS Code debugging for a NestJS project, adding a `launch.json` with two configurations (Launch NestJS and Attach to NestJS), identifying where to place breakpoints in the sample Person controller and service to inspect request handling, and documenting how to step through execution (Step Over, Step Into, Step Out, Continue) and observe variable values in the Variables panel, Watch panel, and Debug Console. The sample code is a NestJS Person CRUD API with TypeORM and PostgreSQL. The repository README describes the tasks in detail and also explains how to run the application (prerequisites, installing dependencies, environment variables, starting PostgreSQL with docker-compose, and starting the API) and how to debug (launch versus attach, and what to do whilst paused at a breakpoint).

---

## How Breakpoints Help in Debugging Compared to Console Logs

Breakpoints allow you to pause execution at a chosen line and inspect the full application state at that moment. You can view local variables, parameters, the call stack, and nested object properties without adding or removing any code. With console logs, you must decide in advance what to print and where, and you often have to re-run the program after changing log statements. Breakpoints avoid that edit-run cycle and allow you to explore arbitrary expressions in the Debug Console whilst paused. You can also step through code line by line (step over, step into, step out) to see exactly how control and data flow from one function to another. Console logs only show the values you explicitly log and do not show the call stack or allow stepping. For complex logic, breakpoints therefore give deeper insight with less code change and fewer runs.

---

## Purpose of `launch.json` and How It Configures Debugging

The file `launch.json` lives in the `.vscode` folder and tells VS Code how to start or attach to a process for debugging. Each entry in the `configurations` array is one way to run the debugger. For NestJS (a Node.js application), the type is `"node"`. Two common patterns are:

- **Launch:** `"request": "launch"` means VS Code starts the process itself. You can set `program` to the application entry (e.g. `src/main.ts`), and `runtimeArgs` to Node flags such as `--inspect-brk`, `-r ts-node/register`, and `-r tsconfig-paths/register` so that TypeScript runs and the debugger can map execution back to `.ts` files. When you press F5, the app starts under the debugger and stops at the first line (or at your breakpoints) so you can step and inspect.

- **Attach:** `"request": "attach"` means the application is already running (e.g. started in a terminal with `npm run start:debug`, which runs Node with `--inspect` on port 9229). The configuration specifies `"port": 9229` so VS Code connects to that process. After attaching, breakpoints set in your project files are active and will be hit when the code runs.

Other options in `launch.json` can set the working directory (`cwd`), the console (e.g. `integratedTerminal`), `skipFiles` so the debugger does not step into Node internals or `node_modules`, and `restart: true` for attach so the debugger can reconnect after the process restarts. Together, these settings define how the debugger starts or attaches and how it behaves during the session.

---

## How to Inspect Request Parameters and Responses While Debugging

When execution is paused at a breakpoint in a controller or service, the request parameters and the data that will form the response are available as local variables. In the **Variables** panel you can expand the parameter objects (e.g. `createPersonDto`, `updatePersonDto`, `id`) to see their properties. For a controller method, the NestJS request object is not always in scope as a named parameter, but the body and path parameters are passed as method arguments, so you inspect those directly. In the service, you see the same DTOs and IDs passed from the controller, plus any entities or intermediate values (e.g. `person`, `updateData`). To watch specific values as you step, add expressions (e.g. `createPersonDto.name`, `person.id`) in the **Watch** panel. To try out expressions or run small checks, use the **Debug Console** in the current scope. The **Call Stack** shows the chain from the HTTP layer into the controller and then the service, so you can select a frame and see that frame’s variables. Response data is simply the return value of the controller or service method. Set a breakpoint on the `return` line or step until you reach it, then inspect the variable being returned (e.g. the saved `person` or the result of `findOne`) to see exactly what will be sent back to the client.

---

## How to Debug Background Jobs That Do Not Run in a Typical Request-Response Cycle

Background jobs (scheduled tasks, queue workers, cron jobs, or other work triggered outside an HTTP request) run in the same Node process as the rest of the application when using the NestJS CLI. To debug them:

1. **Launch or attach as usual.** Start the app with the debugger (Launch NestJS or Attach to NestJS after `npm run start:debug`). Breakpoints in your job handlers (e.g. in a `@Cron()` method or a queue processor) are then active.

2. **Trigger the job.** Depending on the job type, trigger it by waiting for a schedule, sending a message to a queue, or calling an internal API or script that enqueues work. When the job runs, execution will pause at any breakpoint you set in that code path.

3. **Use the same stepping and inspection tools.** Step through the job logic with Step Over, Step Into, and Step Out, and use the Variables panel, Watch panel, and Debug Console to inspect inputs and state. The call stack will show how the job was invoked (e.g. from a scheduler or queue consumer).

If the job runs in a **separate process** (e.g. a dedicated worker), you must start that process with the Node inspect flag (e.g. `node --inspect-brk dist/worker.js`) and add an attach configuration in `launch.json` that uses the port that process listens on. Then attach to that process and set breakpoints in the worker code. For jobs that run in the same process as the HTTP server, no extra configuration is needed beyond the existing NestJS launch or attach setup.
