# Syntax

The syntax of workflow definition part is borrow from Golang, since golang provide a great solution for bridging the synchronouce and asynchronous code.
The reason is that there may no need to have a general programing language that cover lots of nucance but for defining and executing workflow, two topics are unavoid:

- is it support control flow?
- can things run in parallel? 

If the async or lock concepts for concurency programing is not introduced, there is no hybrid way to accomplish both.

The syntax should distinguish from the platform language, say in the python SDK, the syntax can python-like but developers should easily distinguish it is programming on the workflow.
Because the workflow will then pass to the runtime that is not python VM and required different error handling. 
The way to debug and to inspect the result can similar to python but not the same.
By delibrately distinguish a bit on the syntax make user aware when the problem happened where they need to go to check and to ask the questions, to python or the workflow engine.
Or the internal engine error should be treat as panic and not exposed to user (when user hit such kind of error, they are supposed to report to me as a bug).
To make the panic nice for user, can consider to use https://github.com/rust-cli/human-panic. 
If the panic happened in the daemon side, a message should send back to the client to tell user where to the log file to get the log and upload (or automatic this procedure if possible, by asking for the permission.)
The syntax and workflow runtime error are exposed to user with extensive help information so they can fix by themself.

- python-like or non-python-like?
- syntax customized for different base language or single syntax?

`{` and `}` pairs are used for separate the scope of variables passing between processes.
The newline is treat as statement terminator, since I didn't expect multilines statement in building a workflow.

#### Coerce to limit the versatile of syntax

I don't want user who write workflow write super complex flows that make the DSL looks like a generic programming language.
Therefore, I'll make some assumptions and simply throw compile time unsupport error when unexpected syntax shows up.
parallel inside while is one of them, and I'll come up with more rules that users may think about but should not support.

Basic language primitives should all support, such as `while`, `for`, `if..else if.. else` and `break/continue`.
The unsupport list are for things like nested para block, or para block inside para while etc.

