Prototype for Substrate client refactoring.

# Objectives compared to Substrate

This code base proposed an opinionated approaches that differ from Substrate.

## Readability

The number one objective of this code base is to conform to the Substrate/Polkadot specs and not have any bug or security issue.

Beyond that, the most important metric of quality of this source code is how easy it is to understand.

The reference point of this metric is the documentation generated by `cargo doc`. It can more or less be seen as cargo-doc-driven development.

In details:

- Code must be properly documented, and the context for why the code exists should be given.
- Examples must be written as much as possible.
- When possible, use types found in the standard library rather than types defined locally or defined in other libraries. For example, always use `[u8; 32]` rather than `H256`.
- Trait definitions are only ever allowed for implementation details. Custom traits **must not** be exposed in any public API.

## Purity

When applicable, code must not have any side effect and must only ever return an output that directly depends on its inputs.

In particular, it must (when applicable) not perform any operation that directly or indirectly requires help from the operating system, such as getting the current time, accessing files, or sleeping the thread.

In details:

- No global variables (except for niche optimizations).
- No thread-local variables (except for niche optimizations).
- Never sleep the current thread. Everything must be asynchronous.
- No logging (no `log` library).

One must strive to make the code compile for `no_std` contexts if theoretically possible. As such, any code that directly or indirectly requires help from the operating system must be optional and disableable at compile-time.

## Reusability: don't mix concerns

Substrate is based on an architecture where each piece of code plays a specific role in a grander vision. The author of substrate-lite considers that this grander vision is too complicated for this kind of architecture.

In substrate-lite, however, almost all the modules of the source code are provided as tools, as if they were small libraries that are available to be used.

For example, features such as Prometheus metrics or the RPC endpoints **must not** be rooted in the code. It **must** be theoretically easy to remove support for this kind of feature from the source code. Prefer *pulling* information from components from a higher-level rather than passing `Arc` objects around.

Other example: the module that verifies whether a block respects the Babe algorithm must be passed as input the information required for this verification, and doesn't try, for example, to load the information from a database. The Babe verification code should only not be concerned with the concept of a database.

While the guidelines here a blurry, here are a few points:

- Dependency injection is almost always a bad thing.
- Exposing `Arc`s in your public API is almost always a bad thing.

## Assumption that specs will not change

This project is implemented following the current state of the Substrate/Polkadot specifications, and assumes that these specifications will never change.

For example, the code that decodes block headers in written in a way that would be quite annoying (though straight-forward) to modify if the format of a block header ever changes. However, we simply assume that the format of block headers will rarely, if ever, change.

In particular, there is an assumption that the list of consensus algorithms is known in advance and will rarely change. Substrate-lite prefers the explicitness of code specific to every single consensus code, rather than giving the fake impression to the user that they can simply plug their own algorithm and expect it to work.

## Fail fast

Code **must not** panic as a result of unexpected input from the user or from the network.

However, code **must** panic if its internal consistency is compromised. In other words, if the only possible reason for the panic is a bug in the logic of the code.

The author of this crate considers that it is dangerous to try to continue running the program if it is known that it does not run according to expectations.
