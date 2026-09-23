# **F# Coding Guidelines**

How to make profitable F# programming extremely simple, easy, joyful and without any entry in your debugging history. 

These coding guidelines apply for typical F# code created by CS-illiterate dumbs like me, and do not apply to experienced and smart developers and do not apply for very special cases such as highly-performant code or heavy data processing, game development, or graphic. 

Company: Miroslav Husťák (sole owner)

*A note for humans: the entries reflect my experience with coding and the problems I ran into (and, to a lesser extent, issues I noticed in code written by others such as introducing nulls or `.Value` on `Option` types). No idea for an entry was proposed by LLM-based copilots; they only helped with wording, structure and some explanatory remarks.*

## 1. Philosophy

Following a pure functional programming approach, avoiding object-oriented features and mutability unless absolutely necessary for interoperability with .NET libraries or specific frameworks.

## 2. General Principles

**Functional-Only Style**

No use of OOP features such as classes, inheritance, or interfaces unless required for interoperability with .NET or external frameworks (such as Fabulous, TorchSharp, or Elmish.WPF). 

**No Mixing of Paradigms**

No blending of OOP and functional paradigms. Functional purity is preferred. Blending violates the KISS principle.

## 3. Error Handling and Reflection

**Defensive Coding**

All exceptions and nulls originating from .NET libraries are immediately transformed into `Result` or `Option` types at the boundary of the functional code.

**No Exception-Based Flow**

Errors are propagated using types (predominantly the `Result` type), not exceptions. Raising exceptions is avoided unless there is a compelling reason that I have not experienced yet.

**Handling of Nulls Creeping from .NET Libraries**

`Option.ofNull'`, a specific adaptation of FsToolkit's `Option.ofNull` and FSharp.Core's `Option.ofObj`, is used consistently for all nullable .NET types — both reference types and `Nullable<T>` value types — in place of `Option.ofObj`, `Option.ofNull`, and `Option.ofNullable`. This ensures a single uniform null-guarding call and avoids creating variant code. When a `Nullable<T>` value type is passed, the function compiles without error but preserves the `Nullable<T>` wrapper inside `Some`, causing a type mismatch at the downstream consumption site. This is intentional — it is the compiler's signal that `Option.ofNullable` should be used.

**Introducing Nulls into F# Code**

Introducing `nulls` into F# code is strictly prohibited. If you think this rule is too strict, look at your C# debugging history. `Null` is a .NET concept, and F# types do not admit `null` as a value, so the compiler rejects it. `Nulls` can therefore only creep in via .NET types: string, arrays, .NET collections and classes, BCL and third-party return values, and F# classes marked `[<AllowNullLiteral>]`. Assigning `null` to any of these (e.g. `KeywordResults = null` where `KeywordResults` is a string, array, or .NET collection) is prohibited. Represent absence with `option` (or an empty collection or string if appropriate) instead, and convert `nulls` at the boundary where .NET values enter F# code. Do not use `[<AllowNullLiteral>]` on your own types. 

**No `.Value` on `Option`**

Accessing `.Value` on an `Option` (`newValueOpt.Value`) is unacceptable in normal code. It is unsafe when the  `Option` is `None` and turns an explicit absence into a runtime exception - the very thing `Option` exists to prevent. F# is flooded with features for dealing with `Option` types, such as pattern matching or `Option.map/bind/iter, Option.defaultValue, Option.orElseWith` or the `option {} CE`.

It may be tolerated for really quick throwaway testing, when your strict functional boss is not looking and you can't be bothered to type out a proper match. It must never appear in committed code.

The same applies to `Option.get`.

**No `Nullable<T>` in F# Code**

`Nullable<T>` is a .NET interop representation of an optional value and must not appear in F# code. Convert it to `Option<T>` at .NET interop boundaries (see the relevant rule above).

Hand-rolled `match ... Some v -> Nullable(v) | None -> Nullable()` conversions are strictly prohibited.

**Reflection-Free Code**

Reflection moves type checking from compile time to runtime. Renamed members, changed signatures and missing cases become runtime exceptions, string-based member access is invisible to refactoring tools, and reflection can bypass encapsulation and F# invariants (private constructors, immutability, non-null guarantees). It also conflicts with trimming and Native AOT. Therefore reflection is prohibited in application logic.

Reflection means the `System.Reflection` APIs (`GetMethod`, `GetProperty`, `Invoke`, `Activator.CreateInstance`, and the like), `FSharp.Reflection`, and reflection-driven features such as `%A`. Type tests (`:?`), `typeof<T>` not passed to reflection APIs, `nameof`, and attributes consumed by the compiler are not reflection. Prefer `nameof` and source generators over their reflective equivalents.

When evaluating third-party libraries, distinguish between two cases:
- **Reflection hidden behind a clean API boundary** is acceptable if all of the following hold:
  - It does not leak into your code.
  - It does not degrade performance in critical paths.
  - It is compatible with trimming/AOT, if you use them.
  - No reflection-free alternative of comparable quality exists (prefer source-generated variants).
  - Values produced by reflection-based deserialization are validated at the boundary, because such libraries can violate F# invariants, including nullness.
- **Reflection that surfaces in your code** is prohibited on the same grounds as reflection in application logic. This means your code must not pass `Type` values, `MethodInfo`, or member names as strings to reflect over. Attributes that are a library's documented public contract are not in themselves a violation.

**Explicit Deserialization over Implicit Mapping**

Prefer deserialization libraries that require explicit field declarations, such as `Thoth.Json.Net`, so that structural mismatches between expected and actual data are caught eagerly rather than silently swallowed.

Exception: type providers infer the schema from a sample, and erased JSON providers typically fail lazily, only when your code accesses a field missing from the real data. Use  type providers where you control the shape of the data or a representative sample is guaranteed. For external or untrusted payloads, use explicit decoders.

**`%A` Format Specifier**

`%A` uses reflection at runtime, so it is an exception to the Reflection-Free Code rule, allowed only in test code, only for diagnostics, and sparingly.

- Production: Prohibited. It is slow, allocation-heavy, incompatible with trimming/AOT, and its output is neither stable nor complete (large structures are truncated). Use explicit formatting (`%s`, `%d`, `%O`) or a structured logger.
- Tests: Use with caution, preferably not in loops, generators, or on large collections.

## 4. Code Structure & Patterns

**Separation of Pure and Impure Functions**

When possible, separate pure and impure functions; you will thank yourself later. Avoid hiding impurity in function signatures. Consider using a non-monadic emulation of Haskell’s monads with a custom-made SCDU, such as `type Impure<'a> = Impure of (unit -> 'a)`.

**Functions, not Members**

Keep records, DUs and SCDUs purely as data. Use separate functions to operate on them, rather than defining members — except in custom CEs, where members on SCDUs are unavoidable by design.

**Monadic and ROP-Style Abstractions**

Use the built-in monadic CEs (`result{}`, `option{}`, `asyncResult{}`, `asyncOption{}`) and custom monadic/monad-inspired/ROP-style CEs (with SCDUs for builders) where appropriate. If monadic composition or ROP-style function composition are used - functions from `FSharp.Core`, `FsToolkit`, and `FsToolkit.ErrorHandling`, it is recommended to consider limiting their usage to a limited number of functions, preferably to: `Option.map/bind/iter`, `Result.map/bind`, `Option.defaultValue`, `Option.orElseWith`, `Option.toResult`,`Result.defaultWith`, `Result.defaultValue`, `Result.sequence`, `Result.either`, and `Result.mapError`.

**Avoid Non-Monadic Bind in Computation Expressions**

When defining custom computation expressions, ensure that `Bind` satisfies monad laws (left identity, right identity, and associativity). If `Bind` is designed to break these laws on purpose, use an alias to avoid confusion.

**Asynchronous Code**

Use F#'s `async {}` workflows - C#-style `async/await` is prohibited. `Async.Parallel` is preferred for concurrency. .NET's `Task` or `Array.Parallel` may be used for performance-sensitive, CPU-bound operations.

**Functional Control Flow**

Control flow is managed using pattern matching and active patterns instead of `if...then...else` constructs. Looping is typically implemented using either Haskell-like collection functions such as `map`, `iter`, or `fold`, or through tail-recursive functions or continuation-passing style (CPS) recursive functions (with an accumulator if needed), ensuring tail-call optimization or CPS compliance. Query expressions are not employed.

**String Combination**

Use type-safe `sprintf` exclusively for combining strings unless there is a compelling reason to use specialized .NET methods like `Path.Combine` or `String.concat`.

**Avoiding Imperative Constructs** 

- No LINQ expressions
- No query expressions
- No mutable state
- No `if...then...else` constructs (most of potential ones are actually monads or monad-like structures anyway)
- No `for` and `while` loops (I do mean it. The `for` loop is not as harmless as you might think.)

**Code Organisation**

A single logical unit that provides a complete big-picture overview of the component must be kept in one file and must never be split. Splitting can easily become a maintainability trap - a split logic is often hard to review, test, and evolve.
Code that implements one complete MVU (Model-View-Update) logic per UI component is considered a single logical unit and must not be split under any circumstances as the consequences can be dire (such as unmaintainability or a "big picture" lost). If the file seems to be too big, it may be a sign (and usually is) that a collection of logical units was created (instead of a single logical unit) or that nested, independent MVU components should have been implemented.

**Collections**

Immutable collections (`List`, `Seq`, `Set`, `Map`) shall be employed unless there is a compelling reason to use `Array`. Exercise caution when using lazy-evaluated sequences (`Seq`).

**Single-Direction Dependency**

Avoid the `and` keyword and recursive namespaces to preserve single-direction dependencies. Exception: `and` is allowed between mutually recursive functions that form one inseparable algorithm (a CPS pair, for example).

## 5. Enhancing Type Safety

**Type-Driven Development**

Preference is given to type-driven development using single-case discriminated unions (SCDUs) where reasonable, as the memory overhead is negligible in most use cases. 
Use built-in units of measure for numeric values involving physical quantities or consistent arithmetic (e.g. distances, weights, rates), as they provide compile-time safety with zero runtime overhead.

Where zero runtime overhead and simple interop are required, units-of-measure–based phantom types (via FSharp.UMX) may be used to distinguish otherwise identical primitive types (for example different kinds of IDs or numeric values).

**Using `ignore` with Type Parameters**

Using `ignore` with type parameters (for example `ignore<FileInfo>`) catches partial application errors at compile time, preventing subtle bugs in cases where side effects (like logging or printing) are silently skipped.

## 6. Testing

**Testing Philosophy**

Pure functions are assumed to be correct by design especially when type-driven development is applied. Unit tests are optional for these; instead, integration tests (if at all necessary) and PBT (recommended) are used. For performance, load, stress, and security testing, standard industry practices apply.

## 7. Logging

Use any appropriate logging library. Creating a custom logging system is acceptable.

## 8. LLM-based Copilots 

**Use of LLM-Based Copilots**

Exercise caution when using LLM-based copilots. These tools can greatly enhance productivity for developers with strong programming skills; however, for those with limited experience, reliance on copilots may result in errors, security vulnerabilities, or inefficient solutions.

Vibe coded output makes perfect sense for boilerplate-heavy or repetitive work, but shall never be dropped unchanged — always verify, adapt, and refactor. Treat it like a copy-pasted code from Stack Overflow. Tag significant copilot-assisted sections so that the code reviewers can apply extra scrutiny where it matters.

**Copilot-assisted review preparation**

Copilot-assisted review preparation is acceptable, but exercise caution.

**No Copilots for Final Code Review**

Copilots have an uncanny talent for turning a simple code review into a full-scale disaster. And when the phenomenon called “model collapse” finally catches us, it will be a catastrophe of epic proportions :-). So, unless you have been given an extra week to fix the chaos they will inevitably cause, resist the urge to let them anywhere near your pull requests. 

## 9. Async-by-Default

**Preferring Asynchronous Versions**

If an asynchronous variant of an API exists, it is preferred. Adopting the async model opens up future possibilities — such as cancellation or non-blocking constructs with minimal refactoring. That said, caution should be exercised when using asynchronous variants within parallel loops.

## 10. Data Handling

**No Fully-Fledged or Micro Object-Relational Mappers (ORMs)**

Preferring plain SQL for database interactions, avoiding ORMs (such as Entity Framework Core) and micro-ORMs.

**Vanilla SQL and SQL Type Providers**

Exercise caution when using SQL Type Providers. While they offer excellent compile-time safety, they can significantly increase compilation times or hit schema-mapping limitations when used against large, complex enterprise databases (consider using vanilla SQL instead).  

**Type Providers for Non-Database Scenarios**

Type providers for CSV, XML, and JSON are preferred over equivalent .NET libraries (subject to the conditions in the **Explicit Deserialization over Implicit Mapping** sub-entry) due to better type inference and significantly less boilerplate.

**Separation of Data and Operations on Data**

Keeping data separate from operations on data, in accordance with functional programming principles. This separation reinforces the decision to avoid mixing paradigms.

## 11. Learning Lessons from Others 

Learn lessons from others:

[https://colton.dev/blog/curing-your-ai-10x-engineer-imposter-syndrome/?utm_source=tldrnewsletter](https://colton.dev/blog/curing-your-ai-10x-engineer-imposter-syndrome/?utm_source=tldrnewsletter)

[https://youtu.be/ua6zxKSiQ_g?t=591
](https://youtu.be/ua6zxKSiQ_g?t=591)



