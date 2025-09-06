# Designs

## Real-world goals

- Quantum ESPRESSO simulation but easier.
- SSSP verifications but minimal man labor.
- Bioinformation using container technologies.
- Machine learning using hyperqueue.

### Milestones

- [ ] run cp2k simulations on local laptop with native container runtime integrated (bare machine launch).
- [ ] run cp2k simulations on HPC through SSH. (let QE go where ever it is.)
- [ ] run python based machine learning on hybrid local + GPU cluster.
- [ ] run bio-information typical RNA data processing pipeline on cloud.
- [ ] support WDL by parsing spec to my ast.
- [ ] run some example through hyperequeue.

## Roadmap

In the prototype phase, nice to show powerfulness of new self-baked syntax and runtime with following features.

- [x] process as the inner task dirver constructed as a generic state-machine.
- [x] trivial arithmetic task that wrap as the process for consistency.
- [x] shell task that can pause/resume/kill asynchronously. 
- [x] base syntax tree to evaluate arithmetic expressions.
- [x] base syntax tree representation to evaluate shell commands.
- [x] pipeline shell tasks through syntax tree.
- [x] tracing log, separate print statement and log to tracing.
- [x] Builtin binary expressions.
- [x] std lexing and parsing to my syntax tree.
- [x] control flow: if..else, while loop 
- [x] array-like type for holding multiple return and used as iter in for loop (python-like syntax).
- [x] para block using shell example.
- [x] customized provenace to file.
- [x] pipeline shell syntax sugar.
- [x] miette to pop nice syntax error.
- [x] design doc for the syntax specifications, through a mdbook (https://github.com/oxiida/book).
- [x] FFI through pyo3 to embed oxiida-lang into python script.
- [x] (*) workflow reuse 
- [x] (*) static and strong type system.
- [ ] (*) customized type
- [ ] (*) module import. 
- [ ] (*) stdlib with dsl functionalities.
- [ ] (*) Interfaces for wfdev for tasks and workflow build. 
- [x] Clear scope and variable management (plan to use var stack)
- [x] Performance, workaround GIL with multiprocessing.
- [x] versatile runtime that listen to the tasks and launch them.
- [x] sqlite as default persistence
- [x] Default config folder using `.config/oxiida/` and support profile switch by switch config. Should also manage proper persistence target of persistence between run and submit to daemon.
- [ ] query db and provide a lame version of graph print.
- [ ] (*) the ffi call from python should able to return value in python, the workflow can return result as a expression.
- [x] (*) traverse ast and spot parsing time error as much as possible.
- [ ] (*) statement should return value of last expression, para block should return an array with lexical order.
- [ ] ~~`para while`~~ (error-prone, thus disallowed) and `para for` syntax. (check https://docs.julialang.org/en/v1/manual/multi-threading/#The-@threads-Macro)
- [ ] (*) snapshot of a syntax tree and serialize details to restore. (this requires to change from recursive interpretor to flat loop interpretor, a big refactoring)
- [ ] graceful cancellation with [termination signals](https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html), use SIGTERM
- [ ] type checking for the ffi function declaration.
- [ ] separate the bin and lib parts into independent crates.
- [x] pre-runtime type check for array and variable assignment.
- [ ] tracing for crutial flow nodes.
- [ ] (*) Support container tasks
- [ ] Support ssh tasks
- [ ] Support container tasks over ssh wire.
- [ ] chores: TODO/XXX
- [ ] make every example test one specific type of feature.

Small tasks collect around the prototype tasks above.

- [x] distinguish integer and float.
- [ ] bool shall be explicitly casting with stdlib.
- [ ] remove dict and replace it with customized composit type.
- [ ] reinvstigate wether I need incline variable for attr access.
- [ ] Type as a `TypExpression` not to mix with `Expression` in tree walking.
- [ ] `for` loop julia behavior
- [ ] type error can collect and show in one time
- [x] `workflow` has same behavior as `function` and fix function examples.

After prototype, incrementally add more nice to have proper language things:

- [ ] reintrospect the design, especially where the reactor pattern is used. Draw figures for clear demostration.
- [ ] error handling of task, how to represent in syntax, rust way with `match` keyword?
- [ ] ergonomic: py daemon attach env mutate info to the reply message to client.
- [ ] when using =_= syntax, provide way to control upbound number for the available processes.
- [ ] FFI call from julia/lua
- [ ] traverse the ast and generate the control flow graph. 
- [ ] `break` and `continue` keywords.
- [ ] let's build a REPL, oxiida its a interpret language.
- [ ] ?? `local` keyword for variable shadowing.
- [ ] performance: expressions in array can be evaluated concurrently.
- [ ] parallel the expression evaluation and hold the future to join when use.
- [ ] separate CLI and library part and make library part dedicate crate.
- [ ] separate pyo3 part as python SDK.
- [ ] release fixed version of specification with the basic language primitives.
- [ ] clean the dependencies that can be implemented by my own. list are
    - `corfig` and `directories-rs` crates for profile/config setting.
    - `rmp-serde` for codec to msgpack.
- [ ] For python binding, clean the dependencies that can be implemented by my own. list are
    - `serde-pythobject` for convert pyAny back and forth.
    - `serde_json`, because should not bind to json as serializable format.
- [ ] Considered native win support. Now I use `nix` which is a issue for the moment for win support, but should consider to support it as a DSL.

Now I can public the repo and release 0.1.0 to crates.io

After 0.1.0
- [ ] anoymous function and workflow.
- [ ] parser for WDL
- [ ] parser for nextflow
- [ ] parser for CWL

## Misc

