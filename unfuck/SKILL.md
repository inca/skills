---
name: unfuck
description: Gives you programming superpowers to make the code from meh to **really good**. Apply this skill after every coding task, before wrapping up. When used explicitly, apply the skill only to user-requested scope.
---

## Core Concepts

Coding agents are using LLM models trained on the code written by other people over the last century.

The harsh reality is: 95% of that code sucks. Unfortunately, there is no "natural selection" mechanism that kills bad code, so that only the best practices survive. Conversely, bad practices can live for decades, and even thrive, oftentimes because they are the only ones available.

Trained on extremely large datasets, LLM models produce the code of median quality, dragging the codebase to that 50% mark. Whereas in reality everything below 95% is unacceptable.

You strive to be above that 95% line. You need this level of arrogance in order to produce **good** result. You are doing this not only to get the good result here and now. You are doing this to tip the scale: when there is sufficient amount of good code, we would no longer need this skill.

## P0 — Non-negotiables

### Bird language

Whoever came up with the idea of `let foo = bar++` should be shot. If you see this in the code, you should immediately fix it. Use `bar += 1;` to mutate the state, never access and mutate in the same expression.

Same applies to multi-storied expressions like `if ((foo || bar) && !(baz || quz))`. Each such expression usually means something, e.g.

```ts
// Give names to concepts you compute locally
const isVisible = foo || bar;
const canAccess = !baz || !quz;
// Multiple checks can be bundled, but only when expressed clearly
if (!isVisible || !canAccess) {
    return;
}
```

### Export Barrels

Export barrels `index.ts` are ONLY for exporting the content of other files *under the same diretory*. They also establish a contract that this directory is treated as a cohesive unit, almost like a lightweight package.

Within such directory, modules import each other directly, NEVER from the barrels. Outside this directory, the modules should ONLY be imported from the barrels.

Conventionally `index.ts` should just be a list of `export * from './foo.ts';` statements sorted alphabetically. Always export all of the content; if some symbols are not for public, it's not for the barrel to decide.

### Paradigm Mixing

It is NEVER OK to stuff functions together with classes in a single file. NEVER.

Top-level functions are from functional programming; top-level classes are from OOP. Mixing them together in one project is totally acceptable, but requires file-level organization and conventions. Mixing them in one file is unacceptable.

**Rule:**

Recognize whether a location (i.e. a package or directory boundary) deals with OOP or functions.

If it's OOP, then only classes are allowed top-level, in such case stick with `CamelCase.ts` filenames (the name must match class name). Note: it is sometimes acceptable to also export a few types or interfaces from the class file — as long as these types are tightly coupled to the class.

If it's functions, then stick with `kebab-case.ts` filenames, the name generally should not match the name of a single function and instead be a broad term for a group of functions (e.g. valid examples we've used: `string.ts`, `json.ts`, `object.ts`, `array.ts`, `date.ts`, `math.ts`, etc).

Once again: it is NEVER acceptable to have functions mixed in the same files as classes, and even in the same directories.

### When (Not) To Use Functions

Functions are _really_ easy to create. When something is really easy to create, it is really easy to abuse it. Left unchecked, random functions will creep into your classes, into your export barrels, into your views, components, into test specs, into your database, and soon into your ~dreams~ nightmares.

It always starts the same: "just need a little helper here, it's only used once, so I'll just stick it right here". Only a matter of time before you have 5 versions of `normalizeFoo` all doing slightly different things.

**Rule:**

- Functions belong only in designated locations. Multi-package apps will typically have `util` package for shared functions, those may further be structured using sub-directories. Within ohter packages, make `src/main/helpers/` for functions that belong only to that package.

- Not all functions need to exist in public scope. Module-local functions are (roughly) and equivalent of `private` from OOP.

- Sometimes a function doesn't need to exist at all. Do not make functions like:

    ```ts
    function normalizeFoo(foo?: Foo | undefined) {
        return foo ?? null;
    }
    ```

    This does not make any sense, because the callers doing `?? null` is much more readable and explicit. Besides, not sweeping these under the rug can help identify lousy contracts and fix them.

- Functions that you want to put alongside classes should typically be instance methods — even if they don't use `this`.

### Minimization Objectives

### Throw / Catch

## P1 — Common Sense

### Guards vs. Operational Checks

Guards are the checks that prevent execution of the method or function. They fall into following categories:

- access checks: `isVisible`, `canRead`, `canWrite`, etc.
- lifecycle checks: `isReady`, `isInitialized`, `isLoading`, etc.
- corner cases: `if (!points.length)`

Guards should occur in the beginning of the method, their purpose is to prevent the execution as early as possible. The happy path must continue un-nested, so there is only one way of structuring such code:

```ts
foo() {
    if (!this.isVisible) {
        return;
    }
    // Carry on with foo
}
```

Operational checks are similar, but they occur after evaluating something that is relevant to the purpose of method and function. A few common use cases:

- find and throw if does not exist (a.k.a "require"):

    ```ts
    requireFoo(id: string) {
        const foo = this.foos.find(foo => foo.id === id);
        if (!foo) {
            throw new NotFoundError('Foo not found');
        }
        return foo;
    }
    ```

- find with fallback:

    ```ts
    getFoo(id: string) {
        const foo = this.foos.find(foo => foo.id === id);
        if (!foo) {
            // Alternatively, a fallback can be passed as a parameter or lazy-evaluated
            return this.defaultFoo();
        }
        return foo;
    }
    ```

## P2 — Stylistic

### Multiline Objects

In short: objects should be multiline, unless it's a _really compact_ object.

This is our common friction point in schema files:

```ts
export const UserSchema = new Schema<User>({
    type: 'object',
    properties: {
        // These are fine obviously
        id: { type: 'string' },
        name: { type: 'string' },
        age: { type: 'number' },
        // Two keys => multiline generally, even when they're slim
        permissions: {
            type: 'array',
            items: { type: 'string' },
        },
        group: GroupSchema.schema,
    },
});
```
