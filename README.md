# **F# Coding Guidelines**

Company: Miroslav Husťák (sole owner)

## 1. Philosophy

Following a pure functional programming approach, avoiding object-oriented features unless absolutely necessary for interoperability with .NET libraries or specific frameworks.

## 2. General Principles

**Functional-Only Style**

No use of classes, inheritance, or interfaces unless required for interoperability with .NET or external frameworks (such as Fabulous or Elmish.WPF).

**No Mixing of Paradigms**

No blending of OOP and functional paradigms. Functional purity is preferred.

## 3. Error Handling and Reflection

**Defensive Coding**

All exceptions and nulls originating from .NET libraries are immediately transformed into `Result` or `Option` types at the boundary of the functional code.

**No Exception-Based Flow**

Errors are propagated using types (predominantly the `Result` type), not exceptions. Raising exceptions is avoided unless there is a compelling reason.

**Handling of Nulls Creeping From .NET Libraries**

`Option.ofNull` rather than `Option.ofObj` or `Option.ofNullable` is used consistently for .NET types — both reference and value types (incl. strings originating from a .NET library) — for simplicity and uniformity. Strings may be handled with the help of `string`, `Option.ofNullEmpty` or `Option.NullEmptySpace`.  

**Reflection-Free Code**

Reflection is not used in application logic. Libraries that depend on reflection (such as some serialisers) are avoided unless no viable alternative exists.

**Preferred (De)Serialiser:**

Favouring non-reflection-based libraries such as `Thoth.Json.Net` for (de)serialisation.

## 4. Code Structure & Patterns

**Monadic Abstractions**

Monadic custom computation expressions (with single-case discriminated unions used for creating builders) are employed where appropriate. If monadic function composition is used (functions from `FSharp.Core`, `FsToolkit`, and `FsToolkit.ErrorHandling`), it is recommended to limit its usage to the following functions: `Option.map/bind`, `Result.map/bind`, `Option.defaultValue`, `Result.defaultValue/defaultWith`, `Result.sequence`, and `Result.mapError`.

**Asynchronous Code**

Using F#'s `async { ... }` workflows; avoiding C#-style `async/await`. `Async.Parallel` is preferred for concurrency. `Task` may be used for performance-sensitive, CPU-bound operations.

**Functional Control Flow**

Using pattern matching and active patterns instead of `if...then...else`. Looping is achieved through tail-recursive functions or Haskell-like collection functions such as `map`, `iter`, or `fold`. No query expressions.

**Avoiding Imperative Constructs** 

- No LINQ expressions
- No mutable state
- No `for` and `while` loops
- `Array` to be immediately transformed into immutable collections at the boundary of the functional code unless retained for performance-sensitive use cases.

**MVU Organisation**

MVU (Model-View-Update) logic is kept in a single module per UI component. Splitting into multiple files/modules is avoided for the sake of readability.

**Collections**

Immutable collections (`List`, `Seq`, `Set`, `Map`) shall be employed unless there is a compelling reason to use `Array`. Exercise caution when using lazy-evaluated sequences (`Seq`).

**Single-Direction Dependency**

Avoiding the use of the `and` keyword and recursive namespaces to preserve the single-direction dependency concept.

## 5. Type-Driven Design and Testing

**Type-Driven Design**

Preference is given to type-driven development (using single-case discriminated unions) when reasonable and when memory overhead is acceptable.

**Testing Philosophy**

Pure functions are assumed to be correct by design especially when type-driven development is applied. Unit tests are optional for these; instead, integration tests and property-based testing are used.

## 6. Async-by-Default

**Preferring Asynchronous Versions**

If an asynchronous variant of an API exists, preference is given to it. Even if `Async.RunSynchronously` is initially used, the async model opens up future possibilities (such as cancellation or non-blocking constructs) with minimal refactoring.

## 7. Data Handling

**No ORMs and Micro ORMs (Object-Relational Mappers)**

Preferring plain SQL for database interactions, avoiding ORMs and micro-ORMs.

**SQL Over SQL Type Providers**

SQL is written explicitly rather than through SQL type providers as it is unsure how SQL type providers handle large databases.

**Type Providers for Non-Database Scenarios**

Type providers for CSV, Excel and JSON are preferred over equivalent .NET libraries due to better type inference and significantly less boilerplate.

**Separation of Data and Operations on Data**

Keeping data separate from operations on data, in accordance with functional programming principles. This separation reinforces the decision to avoid mixing paradigms.

