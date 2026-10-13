---
name: maintainable-code-craft
description: Write, modify, refactor, and review code for maximum maintainability and quality, using the combined principles of A Philosophy of Software Design (Ousterhout), The Pragmatic Programmer (Hunt and Thomas), and Refactoring (Fowler). Use this whenever writing new code of any size, adding a feature, fixing a bug, refactoring, designing a module, class, function or API, reviewing a pull request, cleaning up legacy code, naming things, writing comments, designing error handling, or writing tests, in any language, even when the user does not mention quality, clean code, or maintainability.
---

# Maintainable Code Craft

This skill distills three books into one working method:

- **A Philosophy of Software Design** (Ousterhout): complexity is the enemy; deep modules, information hiding, define errors out of existence, comments as design.
- **The Pragmatic Programmer** (Hunt and Thomas): Easier To Change (ETC), DRY, orthogonality, decoupling, design by contract, crash early, tracer bullets, testing.
- **Refactoring** (Fowler): small behavior-preserving steps, code smells, the refactoring catalog, self-testing code.

Use it as a lens for every coding decision. The books are opinion backed by experience, not law. Each rule has a "taking it too far" failure mode, so the goal is judgment, not compliance.

---

## 1. The North Star

**Good code is code that is easy to understand and easy to change.** Every principle below is a special case of this.

**Complexity** is anything about the structure of a system that makes it hard to understand and modify. It shows up as three symptoms:

1. **Change amplification**: a simple change requires edits in many places.
2. **Cognitive load**: a developer must hold a lot in their head to make a change safely.
3. **Unknown unknowns** (the worst): it is not obvious what code must change or what you need to know. You only find out when bugs appear.

Complexity has two root causes: **dependencies** (code that cannot be understood or changed in isolation) and **obscurity** (important information that is not obvious). Reduce the number of dependencies, make the remaining ones simple and visible, and make everything important obvious.

Key facts that drive the whole method:

- **Complexity is incremental.** No single shortcut ruins a system; hundreds of small ones do. Adopt zero tolerance for small messes ("no broken windows"). Fix them when found, or at least board them up (a clear TODO with reason, a "not implemented" error) so decay does not spread.
- **Complexity is judged by readers, not writers.** If a reviewer finds code confusing, it is confusing. Do not argue; clarify the code or the comment.
- **Complexity is weighted by how often code is touched.** Messy code nobody ever reads or changes costs little. Hiding complexity where it is rarely seen is almost as good as removing it.
- **Lines of code are not a measure of complexity.** Shorter code can be harder to understand. "Brevity is the soul of wit, but clarity is the soul of evolvable software."
- **Design stamina.** Good internal design lets a team keep moving fast for longer. Poor design is faster for a few weeks, then slower forever. Refactoring and design work exist for economic reasons (they make us faster), not moral ones.

### Strategic, not tactical

Tactical programming aims for "working code as fast as possible" and adds small complexity with every task. Strategic programming aims for **a great design that also works**. Working code is necessary but not sufficient.

- Invest continuously (Ousterhout suggests roughly 10 to 20 percent of effort) in design: trying alternatives, cleaning up, writing good interface docs.
- When you find a design problem, fix it instead of patching around it.
- After any change, the code should look as if it had been designed from the start with that change in mind. If you are not making the design better, you are probably making it worse.
- Under real constraints, ask: "Is this the best design I can achieve given my constraints?" Often a cleaner option costs only slightly more. If a big cleanup truly does not fit, say so explicitly and leave the code no worse.

---

## 2. Workflow: Writing New Code

Follow this sequence for any non-trivial piece of new code. Scale it down for small tasks, but do not skip the thinking.

1. **Understand the real need.** Requirements are learned, not gathered. Probe edge cases the request does not mention (units, empty input, limits, failure modes, who calls this, how often it will change). Separate fixed requirements (semantic invariants, such as "never apply a transaction twice") from changeable policy (such as "only managers may view records"); implement the general mechanism and treat policy as data or configuration ("policy is metadata"). If something important is ambiguous and expensive to guess wrong, ask.

2. **Find the knowledge, not the steps.** Decompose by the pieces of knowledge (design decisions) the system needs, not by the order operations happen in time. Each module should own one or a few design decisions completely (a file format, a protocol, a data structure, an algorithm).

3. **Design it twice.** For each important interface, sketch at least two radically different options (even if you are sure the first is right). Compare: which is simplest for callers, most general, easiest to use for the actual current needs, and efficient? Often the best design combines ideas from several. This takes minutes for a function and an hour or two for a class, and it pays back many times.

4. **Write the interface and its comment before the implementation.** Write the class or module comment, then signatures and interface comments for the main public functions, then the important fields with comments, then the bodies. If the comment is long or hard to write, the abstraction is wrong; redesign now while it is cheap. Look for the design that can be described completely in the fewest words.

5. **Build end to end in thin slices (tracer bullets).** Get a minimal path working through all layers early, especially for the riskiest parts. Tracer code is production quality (real error handling, structure, tests) but incomplete. This differs from a prototype, which is throwaway code built only to learn something; never ship a prototype.

6. **Think about tests while designing.** A test is the first user of your code. If it is hard to test, it is too coupled (for example, it reaches for a global database instead of receiving one). Write tests alongside the code (test first or test during; "test later" means test never).

7. **Take small steps with fast feedback.** The rate of feedback is your speed limit. Compile, run tests, or try it in a REPL after each small step. Avoid "fortune telling" (designing for imagined futures); instead make code easy to replace.

8. **Refactor right after it works.** Once tests are green, tidy what you just wrote: names, extraction, duplication, structure. Clean code is easier to refactor later when the tradeoffs change.

9. **Review against the checklist in Section 14** before declaring it done. "Done" means all tests pass and the design is clean, not merely that it runs.

---

## 3. Workflow: Modifying Existing Code

1. **Read before you change.** If you must think hard to understand a piece of code, do comprehension refactoring: rename unclear things, extract a confusing block into a well-named function, so the understanding moves from your head into the code.

2. **Secure a safety net.** Refactoring requires self-checking tests. If coverage is missing around the area, add characterization tests first (write the test with a placeholder, replace it with the actual output, then temporarily break the code to confirm the test fails). In legacy code without tests, find seams to get it under test; prefer IDE-automated refactorings, which are safe without tests.

3. **Make the change easy, then make the easy change** (Kent Beck). If the feature does not fit the current structure, first refactor (preparatory refactoring), then add the feature. This applies to bug fixes too: unifying duplicated code often makes the fix obvious and keeps the bug from coming back.

4. **Wear one hat at a time.** When adding functionality, do not restructure; when refactoring, do not change behavior. Switch hats often, but always know which one you are wearing. Refactoring preserves observable behavior, including existing bugs (fix them separately and deliberately).

5. **Take tiny steps and keep the code working.** Each refactoring step should leave code that compiles and passes tests; commit after each green step. If a test fails and the cause is not obvious, revert to the last green state and redo the change in smaller steps. "If someone says their code was broken for days while refactoring, they were not refactoring."

6. **Leave it better than you found it** (camping rule). Small improvements on every visit add up. For larger messes, improve gradually whenever you pass through, rather than scheduling a big rewrite; use branch by abstraction to replace a library or subsystem incrementally.

7. **Keep integration frequent.** Long-lived branches make refactoring painful (semantic merge conflicts). Integrate at least daily when possible, using feature toggles for unfinished work.

8. **For published interfaces**, keep the old function as a deprecated forwarder to the new one; migrate callers; remove the old one only when safe. For database changes use parallel change (expand, migrate readers and writers, then contract) across releases.

**When not to refactor:** code you do not need to understand or modify (treat it as a black box API), or when a rewrite is clearly cheaper. Weigh a large refactoring against a small feature honestly; refactor more where you work most often.

---

## 4. Designing Modules, Classes, and Interfaces

A module is anything with an interface and an implementation: a function, class, package, service.

### Make modules deep
The best modules provide powerful functionality behind a simple interface. The interface is the cost (what users must learn); the functionality is the benefit. Unix file I/O (five calls hiding a huge implementation) and garbage collectors (no interface at all) are ideal deep modules.

- Beware **shallow modules**: interfaces nearly as complex as their implementation, such as a method that only wraps one line, or a class that only forwards calls. They add an interface to learn without hiding anything.
- Beware **classitis**: splitting into many tiny classes and methods because "small is good". Each split adds interfaces. A larger class that fully owns a capability is often simpler overall.
- **It is more important for a module to have a simple interface than a simple implementation.** Most modules have more users than developers, so take on complexity inside the module to spare callers ("pull complexity downwards"). Only do this when the complexity is related to the module's job, it simplifies callers, and it simplifies the interface.

### Hide information
Each module should encapsulate design decisions (data structures, formats, algorithms, protocols, assumptions) so they do not appear in its interface.

- `private` is not information hiding if getters and setters expose the same details. Avoid getter/setter pairs that just mirror fields.
- **Information leakage** (the same knowledge in two places, such as a reader and a writer that both know a file format) is one of the most important red flags. Fix by merging the code that shares the knowledge, or extracting a module that owns it behind a simple interface.
- **Temporal decomposition** (one class to read, another to parse, another to write, because that is the order things happen) usually causes leakage. Structure around knowledge, not time order.
- Do not return internal data structures (such as a raw map of parameters). Provide operations like `getParameter(name)` and `getIntParameter(name)` that hide representation and do the conversion work.
- Hide only what callers do not need. If a detail genuinely matters to callers (such as when data is flushed to disk), expose it; omitting important details creates a **false abstraction**.

### Make the common case simple
- Provide sensible **defaults**. Things nearly every caller wants (like buffering on file I/O) should happen automatically; rare options should be available without burdening common use. **Overexposure** (forcing callers to learn rarely used features to do common things) is a red flag.
- Avoid **configuration parameters** that push decisions onto users. Ask: "Will callers be able to choose a better value than we can compute here?" If not, compute it (for example, derive a retry timeout from measured latency). When a parameter is necessary, give it a good default. Do not push genuine design debates into configuration to avoid deciding.
- Do externalize values that legitimately vary by environment or deployment (credentials, hosts, ports, log levels, tax rates, feature flags), wrapped behind a thin config API rather than exposed as global data. Never commit secrets.

### Somewhat general-purpose
Design the **interface** general enough to serve multiple uses, while the **functionality** covers only today's needs. A text class with `insert(position, text)` and `delete(start, end)` beats `backspace(cursor)`, `deleteKey(cursor)`, `deleteSelection(sel)`. Ask:
- What is the simplest interface that covers all my current needs?
- In how many situations will this method be used? (Exactly one is a warning sign.)
- Is it still easy to use for my current needs? (If callers need lots of glue code, it is too general or the wrong shape.)

Avoid speculative generality: no hooks, parameters, abstract classes, or plugin points without a current use. Add flexibility only if adding it later via refactoring would be substantially harder (YAGNI, made safe by refactoring). Make code easy to replace instead of trying to predict the future.

### Separate general-purpose from special-purpose code
Push specialization **up** (into the application or UI layer) or **down** (into drivers or plugins behind a general interface). A general mechanism should not contain code for one particular use (**special-general mixture**). Example: a generic `History` with an `Action` interface for undo, with specific actions defined by the modules that know them and grouping policy defined by the UI.

### Different layer, different abstraction
Adjacent layers with the same abstraction add cost without benefit.
- **Pass-through methods** (only forwarding arguments to a method with a similar signature) signal confused responsibility. Fix by exposing the lower layer, redistributing responsibilities, or merging classes. Dispatchers and multiple implementations of one interface are legitimate.
- **Decorators/wrappers** tend to be shallow. Before adding one, consider putting the feature in the base class, in the use case, in an existing decorator, or in a standalone class.
- **Pass-through variables** threaded through many layers: prefer a shared object or a context object referenced by major objects (ideally immutable), over globals.
- The interface should differ from the representation (store lines internally, expose character ranges).

### Together or apart?
Splitting adds interfaces, management code, separation, and sometimes duplication. **Bring code together** when it shares information, when combining simplifies the interface, when one piece is hard to understand without the other, or to eliminate duplication. **Keep it apart** when the pieces are independent, or to separate general from special-purpose code. Choose the structure with the best information hiding, fewest dependencies, and deepest interfaces.

### Decoupling and orthogonality
- **Orthogonality:** unrelated things should not affect each other. Test: if a requirement behind one feature changes dramatically, ideally only one module changes.
- **DRY:** every piece of knowledge has one authoritative representation. DRY is about knowledge and intent, not identical-looking text: two validations with the same code but different meanings are not duplication. Duplication also hides in docs, data (store derived values only as a localized cache), and across teams (make reuse easy).
- **Tell, don't ask:** do not pull state out of an object, decide, and push it back. Ask the object to do the work (`order.applyDiscount(d)`, not `order.getTotals().grandTotal -= d`).
- **Avoid long method chains** (`customer.orders.last().totals().amount`); each dot couples you to another structure. Stable standard-library pipelines are fine. But do not overcorrect into Middle Man classes full of forwarding methods.
- **Avoid global data**, including singletons and mutable external resources used directly. If something must be global, wrap it in an API.
- **Do not depend on things you do not control** (do not use a phone number or email as a primary key). Hide third-party APIs behind your own abstraction so vendors and frameworks are replaceable ("there are no final decisions").
- **Inheritance is coupling.** Children and their users depend on every ancestor. Prefer interfaces for polymorphism, delegation ("has-a") for reuse, and mixins/traits for sharing behavior. Use implementation inheritance only when every superclass method makes sense on the subclass and every subclass instance is a valid superclass instance; keep parent state managed by the parent. Inheritance can only be used for one axis of variation; switch to delegation when it starts to rub.
- **Avoid temporal coupling**: do not require callers to invoke methods in a particular order unless unavoidable, and document it when it is.

---

## 5. Functions and Control Flow

### Length versus depth (reconciling the books)
Fowler favors many tiny, well-named functions; Ousterhout warns that splitting creates shallow, **conjoined** functions you must read together. Both agree on the real criterion: **the semantic distance between what code does and how it does it.**

- **Extract** a block when you had to work to understand what it does and you can name it by intent (`isEligibleForDiscount()`), when it is a cleanly separable subtask (the child can be understood without the parent and vice versa, often general-purpose), when it removes duplication, or when it splits a function doing two separate things into two simpler interfaces.
- **Do not extract** when you cannot find a name better than the code itself, when the pieces would only make sense read together, or when callers would have to call several new functions in sequence passing state between them.
- **Inline** functions whose body is as clear as the name, and needless layers of delegation. Inlining a badly factored group and re-extracting is a valid move.
- A long function with a simple signature, independent sequential blocks, and good block comments can be fine. Depth beats length.
- **Each function should do one thing and do it completely**, with an interface simpler than its implementation.

### Parameters
- The parameter list should summarize the function's points of variation. Remove parameters the function can derive itself just as easily, unless that adds an unwanted dependency.
- Group data that always travels together into a parameter object (often an immutable value object that will attract behavior, such as `DateRange`).
- Pass the whole object rather than pulling out several fields, unless you want to avoid depending on that object's module.
- **Remove flag arguments** (booleans or enums passed as literals to select behavior); provide explicit functions such as `rushDeliveryDate()` and `regularDeliveryDate()`.
- Replace a hidden dependency on global or mutable state with a parameter to gain referential transparency (pure core, effects at the edges).

### Side effects
- **Command-query separation:** functions that return a value should have no observable side effects. Split functions that both compute and mutate.
- Never assign to input parameters; use a new variable. Give each variable one responsibility (split variables reused for different purposes).
- Prefer immutable data and pure transformations. "Don't hoard state; pass it around." Think of programs as pipelines that transform inputs into outputs; each step takes data and returns new data.

### Conditionals
- **Guard clauses** for unusual cases ("handle it and get out"); if/else when both branches are normal behavior. Single exit is not a goal; clarity is.
- **Decompose conditionals:** extract the condition and each branch into intention-revealing functions.
- **Consolidate** a series of checks with the same result into one named predicate.
- **Replace repeated switches** on the same type code with polymorphism (or a dispatch table). Do not replace every conditional; most should stay simple if/switch.
- **Special case / null object:** if many callers check for the same special value and react the same way, return an object that encapsulates that behavior.
- Eliminate special cases at the design level when possible (an empty selection instead of a "no selection" flag; ranges clamped instead of rejected), so normal-case code handles edge cases naturally.
- Every switch needs a default that flags the "impossible".

### Loops
Prefer collection pipelines (`filter`, `map`, `reduce`) for clarity. One loop should do one thing; split loops that compute unrelated results. Do not prematurely merge loops for performance.

---

## 6. Data and State

- **Data structures are the foundation.** Good structures make behavior code simple; wrong ones force convoluted code everywhere. When a data structure is wrong, fix it promptly (move fields where they belong; a field always passed alongside another record, or updated in several structures, is in the wrong place).
- **Encapsulate mutable data** whose scope is wider than one function behind functions, so updates are visible and controllable. Immutable data needs much less protection ("immutability is a powerful preservative").
- **Collections:** never hand out the internal mutable collection. Provide add/remove operations and return copies or read-only views. Be consistent across the codebase. Watch for in-place operations like sorting.
- **Replace primitives with small types** once they carry behavior or rules: money, units, ranges, phone numbers, IDs, percentages, priorities. A `Percentage` type documents whether 50 means 0.5. Never store phone numbers as numbers. Use integer minor units (cents) or decimal types for money, never floats.
- **Value objects** (immutable, compared by value, with matching hash) for things like money and ranges. **References** (one shared instance from a repository) for entities whose updates must be visible everywhere.
- **Derived values:** compute rather than store and keep in sync, unless the source is immutable or caching is needed and kept private.
- **Split phase:** when code does two different things in sequence (parse then compute; compute then render), separate them with an explicit intermediate data structure.
- Prefer explicit records/classes over generic containers (`Pair`, untyped maps) whose fields have meaningless names. Design for ease of reading, not ease of writing.
- Use a clock wrapper rather than calling the system time directly, so behavior is deterministic and testable. Use UTC internally.

---

## 7. Errors, Contracts, and Resources

### Reduce the number of places errors must be handled
Exceptions add disproportionate complexity: handlers are hard to write, rarely run, and rarely tested. In order of preference:

1. **Define errors out of existence.** Redefine semantics so the "error" is a normal case: `unset` means "ensure the variable does not exist"; `substring` clamps out-of-range indexes; deleting an open file marks it for deletion. Simpler APIs, less caller code, fewer bugs.
2. **Mask exceptions low down** when the low level can fully handle them (retrying lost packets).
3. **Aggregate handling** in one place high up (a single top-level handler per request that turns any request-level error into a response, with the message generated where the error was detected).
4. **Crash** for errors that are not worth handling and indicate bugs or unrecoverable conditions (out of memory, corrupted internal state), with a clear diagnostic.

Do not over-apply: if callers genuinely need to know about a failure (a network module that silently drops errors), expose it. Hide what is unimportant, expose what is important.

Throwing an exception because you do not know what to do just pushes the problem to callers, who usually know even less.

### Design by contract
For each function, know its **preconditions** (what callers must guarantee), **postconditions** (what it promises), and **invariants** (what stays true). Be strict in what you accept and promise as little as you need. Contract violations are bugs, not user errors: validate untrusted external input as normal program logic, and use assertions for internal assumptions. The caller is responsible for meeting preconditions. Minimize preconditions, and document the ones that remain.

### Crash early, assert the impossible
- When something "impossible" happens, stop as close to the cause as possible. A dead program does less damage than a crippled one writing bad data.
- Whenever you think "this can never happen", add an assertion with a descriptive message. Assertions must have no side effects. Keep them on in production unless a specific one is too costly.
- Do not catch, log, and rethrow every exception; it clutters code and couples you to the callee's exception list. Let exceptions propagate to where they can actually be handled.
- Never silently swallow errors. Read the actual error message before acting on it.

### Resources
- **Finish what you start:** the code that acquires a resource releases it, in the same scope. Use language scoping (`with`, `using`, try-with-resources, RAII, `defer`) rather than manual pairing across functions.
- Acquire before the `try`, release in `finally`; if acquisition happens inside the `try`, a failed acquire leads to releasing something never acquired.
- Release in reverse order of acquisition; acquire multiple resources in a consistent order to avoid deadlock.
- Think about resources that accumulate over time (logs, temp files, caches) and how they are cleaned up.

---

## 8. Naming

Names are a form of abstraction and documentation. A poor name can cause bugs that take months to find.

- **Create an image.** Seen in isolation, the name should let a reader guess what the thing is and what it is not. Aim for precision with two to three words.
- **Be precise:** `numActiveIndexlets` not `getCount`; `charIndex, lineIndex` not `x, y`; `cursorVisible` not `blinkStatus`; booleans as predicates (`isReady`, `hasItems`). Avoid `data`, `info`, `manager`, `result` (except for an actual return value), `temp`, `status`.
- **Name by role and intent, not mechanism:** `applyDiscount(Percentage discount)` not `deductPercent(double amount)`; `buyer` not `user` when the domain says buyer.
- **Use domain vocabulary consistently** (keep a glossary). Always use the same name for the same concept, never use it for anything else, and keep the concept narrow enough that all uses behave the same (`fileBlock` and `diskBlock`, not `block` for both; distinct types are even better). Use prefixes to distinguish instances (`srcFileBlock`, `dstFileBlock`).
- **Avoid noise words** (`fileObject`), type encodings (Hungarian notation), and repeating the class name in its fields.
- **Scope rule:** the larger the distance between declaration and use, the more descriptive the name. Short names (`i`, `x` in a one-line lambda) are fine in tiny scopes.
- **Read names in context:** `Fib.of(n)` reads better than `Fib.fib(n)`.
- **Honor the language culture** (camelCase vs snake_case, idioms).
- **If a good name is hard to find, the design is probably unclear** (the variable may represent several things, or the function may do several things). Fix the design.
- **Rename when meaning drifts.** A misleading name (`getData` that writes) is worse than a vague one. If renaming is hard, that is a coupling problem to fix.

---

## 9. Comments and Documentation

Comments capture what was in the designer's mind that the code cannot express. Without them, there is no abstraction: callers would have to read the implementation. But comments must not repeat the code.

### What to document
- **Interface comments** (most important): for every class, describe the abstraction it provides, what an instance represents, and limitations (thread safety, performance). For every non-trivial function: a sentence or two on behavior as seen by callers; each parameter and return value precisely (units, ranges, inclusive or exclusive bounds, meaning of null/empty); side effects; errors raised; preconditions; ordering constraints. A caller should never need to read the body.
- **Field/variable comments:** describe what it represents (nouns, not verbs): units, invariants, ownership (who frees it), what a missing entry means.
- **Implementation comments:** describe what and why, not how. Put a short comment before each major block of a long function and before complex loops ("each iteration extracts one request and appends a response"). Explain non-obvious reasons, workarounds, rejected alternatives, and trade-offs ("we bypass the cache here because...").
- **Cross-module decisions:** put them where developers will find them (for example, a checklist next to an enum listing every place that must change when a value is added), or in a central design notes file referenced by short pointers.
- Explain non-obvious behavior: event handlers (when they are invoked), constructors that start threads, code violating reader expectations.

### How to write them
- **Do not repeat the code** or reuse the words in the name. Test: could someone who has never seen the code write this comment just by reading the line next to it? Then it adds nothing.
- Write at a **different level of detail** than the code: lower-level for precision (exact semantics) or higher-level for intuition (what is this block trying to accomplish).
- **Keep interface and implementation documentation separate.** If an interface comment must describe the implementation, the module is shallow.
- **Write comments first, as a design tool.** A comment that is long or hard to write signals a bad abstraction.
- **Keep comments near the code** they describe (in the implementation file, at the narrowest scope); the farther away, the more abstract it should be. Document each decision once; reference rather than duplicate (including external specs: link the RFC). Do not explain a callee at the call site.
- **Put knowledge in the code, not the commit message**, when future developers need it.
- **Refactor before commenting** when a comment would only be compensating for bad names or structure: extract a well-named function, rename, or add an assertion instead. Comments are not deodorant for bad code.
- Check diffs before committing: are the docs still accurate? Any leftover debug code or TODOs?
- Follow the language's documentation conventions (Javadoc, docstrings, JSDoc, rustdoc, godoc).

---

## 10. Testing

- **Self-checking, automated, fast tests** are the foundation for refactoring and safe change. Run the relevant tests every few minutes and the full suite at least daily (and in CI on every push).
- **Testing is design feedback.** Hard-to-test code is coupled code; fix the design (inject dependencies, separate logic from I/O and UI).
- **Test against the contract:** valid inputs across the range, boundaries (zero, one, empty, max, negative, blank strings, off-by-one and fencepost cases, overflow, duplicates, wrong types where inputs are untrusted), and rejection of invalid inputs. Test subcomponents' contracts before composites.
- **Risk-driven, not method-driven.** Focus on complex and likely-to-break areas; do not bother testing trivial accessors. Incomplete tests that run beat complete tests never written.
- **Make each test fail at least once** (temporarily inject a fault) to prove it can catch a bug. For bigger confidence, use saboteurs or mutation testing.
- **Every bug gets a test first:** reproduce it with a failing test, then fix. Find bugs once.
- **Property-based tests** for invariants (sorting preserves length and order; stock taken plus remaining equals original); turn any failing generated case into a regular regression test.
- **Isolated fixtures:** build a fresh fixture for each test (setup, exercise, verify); never share mutable fixtures between tests. Prefer one logical assertion per test.
- **Avoid fragile tests:** do not depend on exact timestamps, widget positions, or exact error wording; control time and randomness.
- Treat test code with production-code care. Keep all tests passing all the time; a tolerated failing test is a broken window.
- Code coverage finds untested areas but does not prove quality. Think in **states**, not lines (`a / (a + b)` fails only when `a + b == 0`). The real measure: are you confident a defect would make some test fail?
- TDD (red, green, refactor in minutes) is valuable, but keep the big picture in view: units of development should be **abstractions, not features**, and passing tests are not a substitute for design.

---

## 11. Performance

- Clean, simple code is usually fast enough and is far easier to tune. Simple code often runs faster (fewer special-case checks, fewer layer crossings).
- Know what is fundamentally expensive (network round trips, disk and flash I/O, allocation, cache misses, N+1 queries) and choose naturally efficient designs when they cost no extra complexity (a hash map rather than a list for lookups).
- **Estimate algorithmic order** for loops and recursion (nested loops are O(n*m)); ask how large inputs can grow. Do not hand-roll algorithms the standard library provides better.
- **Measure before optimizing.** Intuition is unreliable, even for experts. Profile, find the hot spot, change it in small steps, re-measure, and back out changes that do not help (unless they also simplify).
- Prefer fundamental fixes (caching, better algorithm, batching). If still too slow, design around the **critical path**: imagine the minimal code for the common case, then find a clean structure close to it, with one check that diverts all special cases off the fast path.
- Refactor first, then tune. Do not let speculative performance worries block clear structure (splitting a loop or replacing a temp with a query rarely matters).

---

## 12. Concurrency and Shared State

- **Shared mutable state is incorrect state.** Check-then-act on shared data is a race.
- Prefer, in order: no sharing (immutable data, message passing, actors, pipelines); resources that are transactional themselves (one method that checks and acts under a lock, with release guaranteed in `finally` or a scoped lock); explicit locking as a last resort.
- Watch hidden shared state: current working directory, globals, singletons, files, databases.
- Random, intermittent failures are often concurrency bugs.
- Avoid unnecessary temporal coupling; identify work that can run concurrently (activity analysis), especially waits on I/O.

---

## 13. Security Basics

- Minimize attack surface: less and simpler code, fewer endpoints, fewer privileged accounts.
- **Never trust external input.** Validate and sanitize at boundaries; use parameterized queries and safe APIs instead of building SQL or shell commands with string concatenation.
- Least privilege, for the shortest time. Secure defaults.
- Do not leak information in errors, logs, or stack traces shown to users; mask sensitive identifiers.
- Encrypt sensitive data; never commit secrets to version control; use environment or secret managers.
- Never invent your own cryptography or password storage; use vetted libraries and established auth providers. Follow modern password guidance (allow long passwords and paste, no arbitrary composition rules).
- Keep dependencies patched. Ask of every change: "Have I protected the user? Would I use this myself?"

---

## 14. Consistency and Professional Habits

- **When in Rome:** before writing in an existing codebase, look at how it does things (naming, structure, error handling, test style) and follow it. Consistency gives readers safe assumptions.
- **Do not change existing conventions** for a merely better idea. Change only with significant new information and a willingness to convert all old uses.
- But do not force genuinely different things into the same pattern; consistency means similar things done similarly and different things done differently.
- Automate enforcement (formatters, linters, pre-commit hooks, CI). Do not rely on manual procedures.
- **Don't program by coincidence.** Know why your code works. Rely only on documented behavior; remove calls you added "just in case"; never fix symptoms with +1/-1 adjustments without a correct model; test your assumptions with assertions.
- **Debugging:** do not panic; read the error message; reproduce with a single command and a failing test; binary-search (stack frames, input data, commits via `git bisect`); assume your code is wrong before the library or OS; prove, do not assume. After fixing: why was it not caught earlier, where else could it occur?
- **Decide what matters.** Separate what is important from what is not; build the structure around the important things (make them prominent in interfaces, names, and central types) and hide or de-emphasize the rest. Two mistakes: treating too many things as important (clutter, shallow APIs) and missing something important (hidden essentials, unknown unknowns).
- **Program close to the problem domain:** use the domain's vocabulary in code, and prefer simple data-driven designs (tables, plain-text formats like JSON or YAML) or plain functions over heavy custom mini-languages unless they clearly save more effort than they cost.
- Use design patterns only when they genuinely fit; challenge every fashionable practice by asking whether it really reduces complexity here.
- Listen to discomfort: if code feels like walking uphill in mud, the design or the problem framing may be wrong. Step back, explain it aloud (rubber duck), or prototype to learn.

---

## 15. Red Flags and Code Smells: Diagnosis to Cure

When you notice one of these, stop and look for a better design. The cures name refactorings from Section 16.

| Red flag / smell | What it means | Typical cures |
|---|---|---|
| Shallow module / lazy element | Interface about as complex as implementation | Merge or inline; give the module more responsibility; Inline Function or Class; Collapse Hierarchy |
| Information leakage | Same design decision in several modules | Merge modules; extract one module owning the knowledge |
| Temporal decomposition | Structure follows execution order | Restructure around knowledge |
| Overexposure | Common use requires knowing rare features | Defaults; separate advanced API |
| Pass-through method / Middle Man | Method or class mostly forwards | Remove Middle Man; redistribute responsibilities; merge |
| Pass-through variable | Parameter threaded through layers that ignore it | Shared or context object |
| Repetition / duplicated code | Same logic in several places | Extract Function; Slide Statements; Pull Up Method; Parameterize Function |
| Special-general mixture | General mechanism contains use-specific code | Push specialization up or down |
| Conjoined methods | Cannot understand one without the other | Rejoin or re-split along cleaner lines |
| Long function | Semantic gap between what and how | Extract Function (named by intent); Decompose Conditional; Split Loop; Replace Temp with Query; Replace Function with Command |
| Long parameter list | Too much context passed | Introduce Parameter Object; Preserve Whole Object; Replace Parameter with Query; Remove Flag Argument; Combine Functions into Class |
| Global data | Mutable from anywhere | Encapsulate Variable; narrow scope; inject dependencies |
| Mutable data | Updates cause distant breakage | Encapsulate; Split Variable; Separate Query from Modifier; Remove Setting Method; Replace Derived Variable with Query; Change Reference to Value |
| Divergent change | One module changed for many different reasons | Split Phase; Extract Class; Move Function |
| Shotgun surgery | One change touches many modules | Move Function or Field into one place; Combine Functions into Class or Transform; inline then re-extract |
| Feature envy | Function uses another module's data more than its own | Move Function to the data |
| Data clumps | Same fields travel together | Extract Class; Introduce Parameter Object |
| Primitive obsession | Strings and numbers for domain concepts | Replace Primitive with Object; value objects |
| Repeated switches | Same type switch in several places | Replace Conditional with Polymorphism |
| Speculative generality | Hooks and parameters with no current use | Inline; Collapse Hierarchy; remove unused parameters; Remove Dead Code |
| Temporary field | Field only valid sometimes | Extract Class; Introduce Special Case |
| Message chains | `a.b().c().d()` | Hide Delegate; move the using code down the chain |
| Insider trading | Modules exchange too much private knowledge | Move Function or Field; Hide Delegate; replace inheritance with delegation |
| Large class | Too many fields or responsibilities | Extract Class (look for shared prefixes or client subsets); Extract Superclass |
| Data class | Fields with no behavior, manipulated by others | Move behavior in; Encapsulate Record (fine for immutable result records) |
| Refused bequest | Subclass rejects parent's interface | Replace Subclass or Superclass with Delegate |
| Comment repeats code | Comment adds nothing | Delete it, or write what is not obvious |
| Comment as deodorant | Comment compensates for unclear code | Extract and name; rename; Introduce Assertion |
| Implementation docs in interface | Users told about internals | Move details inside; deepen the abstraction |
| Vague name / hard to pick name | Concept is unclear | Precise name; rethink the design |
| Hard to describe | Interface comment must be long | Simplify the abstraction |
| Nonobvious code | First guess about behavior is wrong | Better names, structure, comments; follow conventions |
| Flag argument | Boolean literal selects behavior | Explicit functions |
| Dead or commented-out code | Unused code still costs reading time | Delete it; version control remembers |
| Catch-log-rethrow everywhere | Noise and coupling | Let exceptions propagate to a real handler |

---

## 16. Refactoring Quick Reference

Core mechanics for all refactorings: ensure tests exist; make one small behavior-preserving change; compile and test; commit; repeat. "The trickier the situation, the smaller the steps." Prefer IDE-automated refactorings when available.

**Basic**
- **Extract Function:** name by intent; move variables only used inside; pass read-only locals as parameters; return the one modified variable (if many are modified, first Split Variable or Replace Temp with Query). Then look for duplicates to replace. Inverse: **Inline Function** (not for polymorphic methods).
- **Extract Variable / Inline Variable:** name a sub-expression (no side effects; make it immutable); promote to a function if meaningful beyond the function.
- **Change Function Declaration** (rename, add or remove parameter): simple path (change declaration and all callers) or migration path (extract new function with a temporary name, move callers one at a time, inline old, rename). Do renames and parameter changes as separate steps.
- **Encapsulate Variable** before moving or renaming widely used data. **Rename Variable/Field.**
- **Introduce Parameter Object; Combine Functions into Class** (shared data, mutable source); **Combine Functions into Transform** (derive enriched read-only data); **Split Phase.**

**Encapsulation:** Encapsulate Record, Encapsulate Collection, Replace Primitive with Object, Replace Temp with Query (only for values computed once and not snapshots), Extract Class, Inline Class, Hide Delegate, Remove Middle Man, Substitute Algorithm (decompose first, compare old and new outputs in tests).

**Moving features:** Move Function (copy, adapt, delegate from old, test, inline old), Move Field, Move Statements into Function or to Callers, Replace Inline Code with Function Call (unless the similarity is coincidental), Slide Statements (cannot slide over code that modifies data both refer to), Split Loop, Replace Loop with Pipeline, Remove Dead Code.

**Organizing data:** Split Variable, Rename Field, Replace Derived Variable with Query (verify with an assertion first), Change Reference to Value, Change Value to Reference.

**Conditionals:** Decompose Conditional, Consolidate Conditional Expression, Replace Nested Conditional with Guard Clauses, Replace Conditional with Polymorphism, Introduce Special Case, Introduce Assertion.

**APIs:** Separate Query from Modifier, Parameterize Function, Remove Flag Argument, Preserve Whole Object, Replace Parameter with Query, Replace Query with Parameter, Remove Setting Method, Replace Constructor with Factory Function, Replace Function with Command (only when you need undo, staged parameters, or heavy decomposition; plain functions 95 percent of the time), Replace Command with Function.

**Inheritance:** Pull Up Method, Field, or Constructor Body; Push Down Method or Field; Replace Type Code with Subclasses (when several functions switch on the type or some fields only apply to some types); Remove Subclass; Extract Superclass; Collapse Hierarchy; Replace Subclass with Delegate; Replace Superclass with Delegate.

Useful techniques that recur across mechanics:
- To change many call sites safely, first extract the repeated code (for example, the special-value check) into one function, then change call sites one at a time.
- To reorganize two tangled things, inline them together, then extract along better boundaries.
- Give temporary functions easily searchable names, then rename at the end.
- Add a temporary assertion to verify a hypothesis (such as "this stored value always equals this calculation") before relying on it.
- Briefly break new code (or a new subclass) to confirm tests actually exercise it.

---

## 17. Final Review Checklist

Before presenting or committing code, check:

**Design**
- [ ] Does each module hide meaningful complexity behind a simple interface? Any shallow modules or pass-throughs?
- [ ] Is each design decision (format, protocol, rule) owned by exactly one place? Any information leakage or duplicated knowledge?
- [ ] Is the common case simple, with sensible defaults? Any configuration that could be computed?
- [ ] Are general mechanisms free of special-case code? Is anything speculative?
- [ ] Do adjacent layers provide different abstractions?
- [ ] Will likely changes stay local (ETC)? Are third-party dependencies behind my own interfaces?
- [ ] Is mutable state minimized, encapsulated, and never shared unsafely? Are there globals?
- [ ] Is inheritance used only where substitutability truly holds?

**Code**
- [ ] Can a reader's first guess about every function and variable be right (obvious code)?
- [ ] Are names precise, consistent, and in domain vocabulary?
- [ ] Do functions do one thing completely, with clean interfaces and no flag arguments? Queries free of side effects?
- [ ] Are conditionals clear (guard clauses, named predicates, no repeated type switches)?
- [ ] Are errors defined away, masked, or aggregated where possible, and crashed on when impossible states occur? No swallowed exceptions?
- [ ] Are resources released in the scope that acquired them?
- [ ] Are external inputs validated, and secrets kept out of code?
- [ ] Does it follow the existing codebase conventions?

**Documentation and tests**
- [ ] Do classes and non-trivial functions have interface comments that make reading the body unnecessary? Do comments explain what is not obvious (why, units, bounds, invariants) without repeating code?
- [ ] Are tests self-checking, focused on risky behavior and boundaries, and proven able to fail? Does each fixed bug have a regression test?
- [ ] Do all tests pass? Is there leftover debug code, dead code, or an unresolved TODO?

When reporting work to the user, briefly mention notable design decisions and any trade-offs or known limitations, so they can judge the result.

---

## 18. Balancing the Principles

The authors disagree in places, and each idea fails when taken too far. Use these resolutions:

- **Small functions vs deep functions:** extract for clean abstraction and clear naming, not to hit a line count. Avoid shallow, conjoined fragments.
- **Comments:** first try to make the code itself clear (names, extraction, assertions); then document the interface and anything still not obvious. Never repeat the code.
- **General vs specific:** general interface, current functionality; no speculative hooks.
- **Hiding vs exposing:** hide what callers do not need; expose what matters to them.
- **Tell, don't ask vs middle men:** encapsulate navigation, but do not wrap every delegate method.
- **Inheritance vs composition:** a judicious mixture; switch to delegation when inheritance starts to hurt.
- **Polymorphism vs conditionals:** polymorphism for repeated type-based switching; plain conditionals otherwise.
- **Consistency vs fit:** follow conventions, but do not force dissimilar things into the same mold.
- **Quality vs delivery:** make quality a requirement; build good enough software, and know when to stop polishing. Never trade away correctness, security, or a clean structure for a trivial time saving.

The ultimate test for every decision: **does this make the code easier to understand and easier to change, for the next person who reads it?**
