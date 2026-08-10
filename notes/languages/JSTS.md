- “===” is a strict equality operator. It works like - “==” in python and java and does not attempt to coerce types, immediately returning False if two values are different types 

- Ex: “[] === 0” evaluates to False

- “==” is a loose equality operator. It works like - “===” but instead of returning False for different types it breaks the types down until a common type is found and then evaluates

- Ex: “[] == 0” evaluates to True. [] is coerced to 0 (number) and then compared to 0 (number).

# JavaScript Basic Syntax

Variable declarations: 

- 'var =': For function-scoped variables

- 'let =': block-scoped, mutable

- 'const =': block scoped, immutable

- Data Types: string, number, boolean, nullValue, undefinedValue, symbol

Function declarations:

- ```js
    // Normal function
    function traditionalFunction(x, y) {
        return x + y;
    }
    ```

- ```js
    // Modern anonymous functions (like lambda functions from python)
    const arrowFunction = (x, y) => x + y;
    ```

- ```js
    // Callback functions
    const number = [1, 2, 3];
    const doubled = numbers.map(num => num * 2);
    ```

- Optional parameters can be implemented by setting the parameter to a value in the function declaration (Ex. 'function(x, y = null)')

Objects:

- ```js
    const person = {
        name: "Alice",
        age: 30,
        greet() {
            return `Hello I'm ${this.name}`;
        }
    };
    ```

Arrays: 

- ```js
    const fruits = ["apple", "banana", "orange"];
    fruits.push("grape");
    ```

# TypeScript Basic Syntax

- TypeScript is built on top of JavaScript, meaning all Javascript is valid Typescript

- TypeScript introduces new features to prevent bugs and promote code maintenance

These features include:

- Static Typing: Types can be assigned types in code, meaning you can catch type errors at compile-time instead of runtime

- Better Tools: Autocompletion, refactoring, and code navigation

- Scalability: Large codebases are more maintainable

Basic Type Assignment:

- ```ts
    // Variables
    let username: string = "Alice";
    let age: number = 30;
    let isActive: boolean = True;
    let nothing: null = null;
    let notDefined: undefined = undefined;

    // Arrays
    let numbers: number[] = [1, 2, 3];
    let strings: Array<string> = ["a", "b", "c"];

    // Tuples (fixed length arrays)
    let person: [string, number] = ["Alice", 30];
    ```

- TypeScript also introduces type inference, meaning if a value is not assigned to a variable, TS compiler assigns the type of the first value passed to the variable as the variable's type

- ```ts
    let inferredString; // Type is 'any' until value given
    
    inferredString = "Hello"
    //TypeScript assigns 'string' type to this variable, meaning it cannot be change types later on
    ```

- 'any' is a type which allows any value type to be assigned to a variable

- ```ts
    let flexVar: any;

    flexVar = "hello"; // Valid

    flexVar = 3; // Also valid
    ```

- Assigning type 'unknown' to a variable forces you to assign a type to the variable before attempting to use it, using '|' (Union types) allows multiple selected types to be used

- ```ts
    let spookyVar: unknown;

    const sum = 1 + spookyVar; // Error

    // --

    let twoType: number | string;

    twoType = 3; // Valid

    twoType = "hello" // Valid

    twoType = False // Error
    ```
- Unions can also be used to allow a variable to be only selected values instead of a type

- ```ts
    let status: "active" | "inactive" | "pending";

    status  = "active" // Valid

    status = "inactive" // Valid

    status = "deleted" // Error
    ```
- Typescript also introduces optional parameters without setting the value in the function declaration. Instead, the parameter will have a '?' (Ex. 'function(x, y?)') . The function logic will treat the parameter as undefined if it is not passed by a call

# Interfaces and Custom Types

- An interface is a 'contract' that objects using it must follow

- Let's say you have a User interface defined as follows:
```ts
    interface User {
        id: number;
        name: string;
        email: string;
        isActive?: boolean; // Optional
    }
```

- To create a 'User' object, you must follow this contract. Creating a 'User' object would look like this:
```ts
    const alice: User = {
        id: 1,
        name: "Alice",
        email: "alice@example.com"
        // isActive omitted
    };
```
- Interfaces induce compile-time type checking for objects

# JavaScript Functions and Scope

- JavaScript uses lexical scope, meaning a function can access variables from where it was defined

- `let` and `const` are block scoped. `var` is function scoped

- A closure is a function retaining access to its outer scope after the outer function returns

    ```js
    function createCounter() {
        let count = 0;

        return function increment() {
            count += 1;
            return count;
        };
    }

    const counter = createCounter();
    counter(); // 1
    counter(); // 2
    ```

- Closures are used for encapsulation, callbacks, factories, memoization, and event handlers

- Function declarations are hoisted with their implementation. Function expressions and arrow functions follow their variable initialization rules

- Arrow functions do not create their own `this`, `arguments`, or constructor behavior

- `this` depends on how a normal function is called, not only where it is written

- Use rest parameters to collect arguments and spread syntax to expand iterables/objects

    ```js
    function sum(...numbers) {
        return numbers.reduce((total, number) => total + number, 0);
    }

    const copy = { ...original, active: true };
    const combined = [...left, ...right];
    ```

# Objects, Prototypes, and Classes

- JavaScript objects store property keys and values

- Property access uses `object.name` or `object[key]`. Bracket syntax accepts computed keys

- Destructuring extracts named or positioned values

    ```js
    const { id, name, role = "user" } = account;
    const [first, second] = values;
    ```

- Optional chaining stops safely on `null` or `undefined`: `user.profile?.avatarUrl`

- Nullish coalescing uses a fallback only for `null` or `undefined`: `value ?? defaultValue`

- JavaScript inheritance is prototype-based. Class syntax is a clearer interface over prototypes

    ```js
    class User {
        #passwordHash;

        constructor(id, email, passwordHash) {
            this.id = id;
            this.email = email;
            this.#passwordHash = passwordHash;
        }

        describe() {
            return `${this.id}: ${this.email}`;
        }
    }
    ```

- Private fields begin with `#` and are enforced by the language

- `Object.freeze` prevents direct changes to one object level but does not deeply freeze nested objects

- Prefer composition and plain data when inheritance does not model a true subtype

# Arrays and Collection Operations

- Common non-mutating operations:

    `map`: Transform every element

    `filter`: Keep matching elements

    `find`: Return the first match

    `some`: Whether any element matches

    `every`: Whether every element matches

    `reduce`: Combine elements into one result

    `slice`: Copy a range

    `toSorted`, `toReversed`, `toSpliced`: Return changed copies

    ```js
    const activeNames = users
        .filter((user) => user.active)
        .map((user) => user.name)
        .toSorted();
    ```

- Mutating methods include `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, and `reverse`

- `Map` allows keys of any value type. `Set` stores unique values

- `WeakMap` and `WeakSet` hold weak object references and are useful for object-associated metadata without preventing garbage collection

- Iterables implement `Symbol.iterator` and work with `for...of`, spread, and destructuring

# Errors

- Throw `Error` objects rather than plain strings

    ```js
    class ValidationError extends Error {
        constructor(message, field) {
            super(message);
            this.name = "ValidationError";
            this.field = field;
        }
    }

    try {
        validate(input);
    } catch (error) {
        if (error instanceof ValidationError) {
            showFieldError(error.field, error.message);
        } else {
            throw error;
        }
    } finally {
        closeResource();
    }
    ```

- Catch errors only where code can recover, translate, add context, or perform boundary logging

- Preserve an original error through `cause` when wrapping it

# Async JavaScript and Event Loop

- A Promise represents a future fulfilled value or rejection

    ```js
    async function loadUser(id, signal) {
        const response = await fetch(`/api/users/${id}`, { signal });

        if (!response.ok) {
            throw new Error(`Request failed: ${response.status}`);
        }

        return response.json();
    }
    ```

- An `async` function always returns a Promise

- `await` pauses only that async function while other event-loop work may continue

- Start independent tasks before awaiting them together

    ```js
    const userPromise = loadUser(id);
    const postsPromise = loadPosts(id);
    const [user, posts] = await Promise.all([userPromise, postsPromise]);
    ```

- Promise combinators:

    `Promise.all`: Fulfill when all fulfill; reject on first rejection

    `Promise.allSettled`: Wait for every outcome

    `Promise.race`: First settled outcome

    `Promise.any`: First fulfilled value

- The event loop processes task queues while the call stack is empty. Promise callbacks use the microtask queue and normally run before the next timer task

- CPU-heavy synchronous JavaScript blocks the event loop. Move large computation to workers/processes or divide it appropriately

- Use `AbortController` for cancellation and always set network/application timeouts where supported

- Avoid unhandled Promise rejections. `try/catch` around `await` or return the Promise to a boundary which handles it

# Modules and Tooling

- ES Modules use `export` and `import`

    ```js
    // math.js
    export function add(a, b) {
        return a + b;
    }

    // app.js
    import { add } from "./math.js";
    ```

- A default export gives one unnamed primary value. Named exports make available members explicit and refactoring-friendly

- Dynamic `import()` loads a module asynchronously and supports code splitting

- Node.js supports ES Modules and CommonJS. Project configuration and file extensions determine module interpretation

- `package.json` declares package metadata, scripts, dependencies, engines, and module behavior

- A lockfile records resolved dependency versions for repeatable installation

- Use `npm ci` in CI when a valid lockfile should be installed exactly

- Common quality tools include TypeScript, ESLint or Biome, Prettier, Vitest/Jest, Testing Library, and Playwright

# TypeScript Narrowing

- TypeScript narrows union types after runtime checks

    ```ts
    function format(value: string | number): string {
        if (typeof value === "number") {
            return value.toFixed(2);
        }

        return value.trim();
    }
    ```

- Narrowing tools include `typeof`, `instanceof`, `in`, equality checks, truthiness, discriminated unions, predicates, and assertion functions

- A discriminated union gives each variant one literal field

    ```ts
    type Result =
        | { kind: "success"; value: string }
        | { kind: "failure"; error: Error };

    function unwrap(result: Result): string {
        switch (result.kind) {
            case "success":
                return result.value;
            case "failure":
                throw result.error;
        }
    }
    ```

- `never` can enforce exhaustive handling when a new union variant is added

- Prefer `unknown` over `any` at untrusted boundaries, then validate/narrow it

- Type assertions (`as Type`) do not validate or convert runtime data

# TypeScript Generics and Utility Types

- Generics preserve relationships between input and output types

    ```ts
    function first<T>(values: readonly T[]): T | undefined {
        return values[0];
    }

    type ApiResponse<T> = {
        data: T;
        requestId: string;
    };
    ```

- Constraints limit generic types: `<T extends { id: string }>`

- `keyof` creates a union of property names. Indexed access obtains a property type

- Mapped types transform every property. Conditional types choose a type from a condition

- Common utility types:

    `Partial<T>`: Every property optional

    `Required<T>`: Every property required

    `Readonly<T>`: Every property read-only

    `Pick<T, K>`: Selected properties

    `Omit<T, K>`: All except selected properties

    `Record<K, V>`: Key/value object type

    `ReturnType<F>`: Function return type

    `Parameters<F>`: Function parameter tuple

- `satisfies` checks that a value matches a type while preserving useful inferred literal details

    ```ts
    const routes = {
        home: "/",
        settings: "/settings",
    } satisfies Record<string, `/${string}`>;
    ```

- `as const` makes literals narrow and properties/tuples read-only in the type system

# TypeScript Classes and Interfaces

- Interfaces can extend other interfaces and merge compatible declarations

- Type aliases can represent primitives, unions, intersections, tuples, mapped types, and conditional types

- Use interfaces for open object contracts and type aliases when union/composition features are needed. Consistency matters more than treating one as always superior

- Access modifiers include `public`, `protected`, and `private`. TypeScript `private` is compile-time visibility; JavaScript `#field` is runtime privacy

- `readonly` prevents assignment through that type but does not guarantee deep runtime immutability

- Abstract classes define shared implementation and required subclass methods

# Runtime Validation and Security

- TypeScript types are removed when JavaScript is emitted

- Validate API bodies, environment variables, storage values, query parameters, and third-party data at runtime

- Schema libraries such as Zod can parse unknown data and infer TypeScript types

    ```ts
    import { z } from "zod";

    const UserSchema = z.object({
        id: z.string().uuid(),
        email: z.string().email(),
    });

    const user = UserSchema.parse(untrustedValue);
    ```

- Avoid `eval`, `new Function`, unsafe HTML injection, prototype pollution paths, and dynamic code from untrusted input

- Keep secrets in server-only code and never depend on frontend checks for authorization

# Testing

- Unit tests verify pure functions and modules

- Component tests verify rendered behavior through user-facing queries

- Integration tests verify cooperating modules and real boundaries

- End-to-End tests verify complete browser workflows

    ```ts
    import { describe, expect, it } from "vitest";

    describe("sum", () => {
        it("adds every number", () => {
            expect(sum(1, 2, 3)).toBe(6);
        });
    });
    ```

- Test behavior and public outcomes instead of implementation details

- Use fake timers, dependency injection, or controlled test servers instead of unreliable real time/network behavior in unit tests
