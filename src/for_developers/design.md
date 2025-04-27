# Design decisions

(This is just a partial backup for the design decision copied from private oxiida repo)

## What is workflow?

It is unavoid to have a domain specific language to write workflow, since workflows can be regard as programming to control the remote resource.
The anology can be made by mapping the concept of remote HPC as the CPUs and the database as the RAM (optional if the runtime keep on running and do not need to recover tasks from persistent storage, then just use RAM as "RAM").
Therefore, `oxiida` has an async runtime based on the rust tokio and has its own syntax (the standard procedures of a programming language design with lexing/parsing/interpreting) to mark as a small language for workflow construction.

## runtime

Runtime is a huge topic requires a lot mental efforts to make the design correct.
I need to come back many times to re-think how to make it well implemented to get both performance and maintainability.
There are hard concept such as `Pin`/`Unpin` need to go throught and have a well understand on deep native tokio runtime implementation to borrow ideas and to make the pattern consistent.

The inner state machine is called "Process", it hold states and return handler for the runtime controller to communicate from outside of runtime.
The "Task" is a wrapper of the "Process", it contains also the input and output of the runable target. 
Inside the process, it defines generic running logic which in the context of workflow usually require involving the resource management. 
The process running logic is essentially how the input port being used to modified the output port.

Therefore the "Task" interface require `task.new` as the constructer (will builder pattern fit better??) and require `task.output` to access the output later when task completed.
It also require the `task.execute` to define how to use inputs to modify the mutated output port. 

### concurrent processes

The reserved keyword `para` is introduced to decorate the block (and expression??) to run in concurrent manner.
The concurrent under this context means for inside a workfllow, the instructions can not just goes in the lexical order but statements can run "in parallel" or more precisely asynchronous in the tokio runtime.
Because if the workflows or processes are launched separately they are run in runtime asynchronously.
In oxiida, a DSL is introduced there for there are lexical orders for the code (for construct the workflow) are written.
By default, tasks run in sequence mode, only if the task is annotated as parallel, it will spin up to run concurrently.
This is key to avoid suprising execution order of workflow. 
But between workflows, since there are no code written to communicate between workflows to run one after another, it is safe to have them run in any order with their own variable environment.

Under the hood, sequence run and parallel run are distinguished by the join handler of the spawned couroutine. 
For the sequence tasks, the join handlers are immediately await after the spawn of the task.
This ensure the async runtime will not execute other task until the await join handler returned.
While for the parallel tasks, the join handlers are later await when the leave the parallel scope, or even later when the workflow finished.

It is essential that the join handlers are all properly await so that no tasks abort unexpectly when the runtime finished, which may cause remote resource leaking.
The design here is to have a handler type check at the end of runtime, the joiner is an enum type that can be either `JoinHandler` or resolved output (`dyn` needed??).
If it is a join handler which means there are no explicity consume of the output, the runtime will trigger the await and warn the user that "Some output is not used and runtime is await to make sure it is finished without resource leaking".

In principle, since I have the whole DSL parser implemented from scratch, I can design the syntax for inline parallelism.
Semanticlly, it means an expresion can have partial section of an one line execution run in parallel. 
For example, `(6 + 4) * (5 + 4)` can have left-hand side and right-hand side of `*` operator run at same time by violate the left association law of `*` operator. 
At the moment, I didn't see much use case of adding such complicity, so I just limit the parallelism can declared to certain type of statements apply to certain type of task, e.g. `ShellTask`, `SchedulerTask`.
But the parallelism of evaluating the array element expression may not cause any unexpected behaviors since they are by default not rely on the others.
It thus worth to support the parallel run of expressions in the array.

It requires some restrictions for the statements that can be run in parallel block. 

Every statement has their own variable scope, so they are able to run in any order. 
Actually it is natural when implement in rust the borrow checker force me to deal with variable environment nicely by making them independent for each statement.
Because the environment is passed as `&mut var_env` so to make it can be used for another corountine, I choose to clone it and move the ownership to the statement run in a async block.
This makes it again not so sure if the scope is needed or not.

All the statements in the para block will be launched concurrently and no hybrid mode support yet to have single statement that is run in sync.
In principle it is allow to have same variable name for different statements since they have own local scope, but usually it reveal some potential risk of a bug, this is raise as error spotted in the ast resolve phase.
The statement is not allow to redeclare or modified the variable in the parent scope (the shared var_env are read-only), this restriction make it possible avoid using mutex.
At the end of para block all the handlers are wait to join so nothing goes no control.

The statement contains evaluation of expressions, and I do not know in advance whether the expression has side effect or not.
The side effect means the statement does not mutate the resource, a more detail categorise on which expression should be regard having side effect should be well defined, at the moment, the assignment is one example of not having side effect, while the tasks with process run in runtime treat as having side effect.

In terms of para expression, it is not clear whether it require to support inline decorate expression with `para` to make an expression can immediately return the control.
I can see use cases that in the resource wise it is effecient to launch things concurrently and continue the other instructions.
But this means the expression need to return a future type (the `JoinHandler` in my runtime implementation) that need to be resolved to complete manually (or automatically at the end of scope) to avoid resource leak. 
Not very sure how complex the syntax goes to do it right, so leave it as a feature after prototype phase.
When the expression does not has any side effect the ast traverse phase will detect such problem and throw error to warn a potential bug and ask to put the statement in the block to form a valid statement.


## Storage for provenance

One of the key feature to mark a scientific workflow is the ability to record the dynamic running logic. 
The data nodes as the input or output of the tasks will be generated and written into the persistent storage system where the runtime runs.
It can be a file, a database or even the remote object store. 
This requires the input/output data is serializable. 
Powered by the `serde` crate, combined with the type checking of the DSL I introduced in the syntax, it is clear what should be the basic type for the tasks' input/output.
The basic types are serializable values defined in: https://docs.rs/serde_json/latest/serde_json/enum.Value.html

(Now by default everything is recorded, do I need control the storage in a fine grid??) Whether to store input/output along with the process is controlled in the block statement level.
By default, everything is not stored for performance consideration and so that provenance can be an optional feature.
When the block is marked as "proof" by the leading `@` for `@{ stmts }`, it casade into the statements inside.
I can not see use case of unmarking for a particular expression inside because once the provenance needed it requires the provenance to be complete.
For the customized task, it is in the task implementation define which data should be stored.

When an expression marked as stored, the terminator data expression will attached with an unique ID so that when the terminator value output become the input of another expression, the ID indicate that it sterms from the idendical data.
Meanwhile, the data with the ID will be dump to the storage defined.
The storage is controlled by passing through the CLI.
I don't have idea yet what command line API should be for the storage.
For prototyping, I'll implement two types of storage system. 
One is the dummy (named `NullPersistent`) one that nothing will actually goes to persistent storage but just emit the storing operation to the stdout. 
Then it is file in the disk (named `FilePersistence`) as the persistent to the file system. 
Both of them share the same interface showing how a persistent system should operated.

There are three types of data nodes that will stored in the persistent storage system for future provenance.
It consist of terminator value data note, task process node and edge node.
Each of them will have their unique id when stored in the persistence media such as file or database.

Distinguishing between the value array and expression array, where value array contains only terminator values that are serializable and will be the value nodes.
The expression array contains expressions that will be further evaluated towards to the value array.

Those types are mapping to the terminator expression of the syntax tree. 
Then storing the data just become serialize the terminator data syntax tree node and calling the persistent storage system, which can be passed to the runtime for switching where to store the provenance.
Implementing the different storage system are just fitting for the interface with the dependency injection pattern, neat.

Storing of input and output is independent from running process, because process is where inputs are consumed to generate output.
The storing require inputs data that already exist before process is running and require outputs data that only exist after process complete.
Meanwhile, there are expressions that has data connection that requires no process to generate data such as `Expression::Attribute`.
For the assignment case, the output is deduct from the input by `.` operator which trigger a hashmap lookup rather than running a process.

## Uncategorized ideas and notes

### How much APIs exposed by Process?

Implementing a certain type of `Process` is quite sophisticate, it may result in infinite loop or putting functions in wrong mode (async v.s. sync).
Thus, the core part should provide all the implemented types processes to try to cover some generic cases, any new process type will regard as new features.
Provided process types are:
- `SchedulerProcess`: similar to AiiDA `CalcJob`, bundle the remote file operations through transport and perform scheduler (slurm first and see what requirement it has, make it plugable) operations through transport.
- `LocalProcess`: no remote communication required, but use the resources where the service is running. The process spawn using spawn-blocking in other threads for performance, but require tense monitoring to avoid draining the resource and starving the service.
- `KubernetesProcess`: communicate to a k8s cluster and start customized pod on demand, i.e. cloud processes.
- `HyperequeueProcess`: or sub-scheduler using hyperqueue (rust crate, therefore can call through native APIs) as native sub-scheduler once a large abstract resource is available. 

See if the categories above needed, some might be categorize under same abstraction.

### Paused state

The paused state is special.
It can be recovered from the middle means it hold the process infos so it knows how to advance to next state without starting from beginning.
In the runtime, the process state stay there waiting for a resume signal. 
Since it may stay in the runtime for long time, in the paused state it should carry as less resource as possible. 
It requires when transition to paused state, all resouces from previous state should be well closed.
In anycase, it should be gaurenteed by between state transition, no resources should be carried over. 
For example, if the process need SSH communication over a SSH transport through an actor in the transport pool.
In pausing, this resource should be droped and recreated when resume.

The pausing is happened before the `proc.advance()` which is the function call where the really operation happeneds.
Therefore, in resuming the state is go back to which before the pausing.

### Resource actors pool

The workflow consist of process which act as the entity to communicate with resources. 
In order to communicate with remote resource, it require some way to transport the information over the wire, for example SSH protocol is one of the typical transport method.
Frequetly access remote may bring unnecessary overhead on initializing procedures such as handshake or authentication. 
Wrost case, too frequent accessing may be regaurd as attack and banned from the resource provider. 

By design, there will be only one actor represent the communication to one resource pool. 
Which means new communication request from process first goes to the pool to take an ready to use resource and use it.
Through this approach, it avoids the frequently open and drop the resources.
However, the remote resource may have timeout for the open connections and close from server side forcely. 
To overcome the issue, the requests can poll and keep on move to next one in the pool so after sometime the older resources fade away.

The mechanism require more design consideration and it is a key part that influence the reliability of the tool.

### run process in local thread

The Process does not need to be `Send + Sync`, since I made it only run in `local_set` theread. 
The reason is send over can be expensive when process carry a lot of data.
Meanwhile, process is `Send` means the type of its inner data it carries needs to be `Send`, for instance the input and output. 
This make it cumbersome to having explicitly write `Send` trait bound for its data.

### The create state

The created state seems a bit redundant since the first running state can be the entry for launching.
Maybe it is useful when need to write persistent proc info intot disk i.e. DB.

## An interpreter

### Syntax

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
The syntax and workflow runtime error are exposed to user with extensive help information so they can fix by themself.

- python-like or non-python-like?
- syntax customized for different base language or single syntax?

`{` and `}` pairs are used for separate the scope of variables passing between processes.
The newline is treat as statement terminator, since I didn't expect multilines statement in building a workflow.

#### Variable declaring and shadowing

I personally think a proper generic programming language should separate the declaration and assignment a variable instead of mix them like python.
But as a small DSL, it might be a bit too verbose to have explicit declaration.
However, does not separate two statement means when the assignment appears the second time, it either a valid assignment or wrongly override the old value by overlook it is already used.
I can reject any reuse of the variable but it will end up having too many variables just for simple stuff.
To mitigate the problem, I apply the pre-runtime check to throw an error only when the variable is assigned with a different type of value.

For the variable shadowing, if I go with only have a global shared var_env, the shadowing will take the Lua way by using `local` keyword to declaring the variable to the local environment.

- ? how to deal with variable confliction?
- ? when there is also ffi var_env, how to manage it together?
- ? when ox module import introduce to allow workflow reuse, how to solve confliction?

#### Control flow

- ~a while loop evaluates to an array containing each element that the body evaluated to, like CoffeeScript?~ no, this is quite unusual syntax, and while is then a stmt. 

I support the for and while loop. 
The for loop is loop over a lenght fixed array thus it will always end the loop.
The while loop has the risk to use to not exit the loop, this will happend when the incremental expression is missed to run.
So first of all, recommend to use for loop over while loop.
Then for the while loop, it require to add a max iter to avoid the infinite loop, which is an outlandish syntax therefore to introduce it nicely I'll have a resolve time checker point to how to use this syntax when it is not provided in the first place.

There is an ambiguity when combine for loop with using loop entity passing as identifier. 
Becasue the `for` statement yet only accept passing explicit array in the grammar.
To support it, cases between it is a explicit array and a identifier (trickyer if also want to support Attr expression) should be separately take care of.
Now, I simply don't allow to parsing following syntax: 
```oxiida
xs = [1, 2, 3, 4];
print xs;

for x in xs {
    print x + 1;
}
```

#### Do I need scope and closure? (TBD)

~For simplicity and for the moment I'd assume the workflow are small and fit for one page to review, using a shared global variable environment won't be a large issue.~
~there are not too many variables that may cause namespace confliction.~ 
~Anyway, it is not hard to implement, so let's see the use cases.~

It seems when I need nest workflow, the closure is unavoidable?
Meanwhile in the rlox practice, I still can not solve the memory leak of self-reference when implement the closure.
If it can be avoid I'll not add the feature.

The scope seems unavoidable when it comes to require independent local variable environment for each statment run in parallel.
Thus I may make every seq/para block a new scope, a new workflow has its own scope and every statement in the parallel block has a new scope.
See the para block design below where I describe why the scope can still be avoid if I just clone and pass the ownership to the parallel statements.
I think in the end it is a trade off between whether clone around a var_env become too expensive.
Use just one global environment has the advantage that the runtime implementation is simple without need to using reference counter and `RefCell` to manage the var_env for different scopes.
Or maybe there are much easy or proper way to implement var_env?

#### Coerce to limit the versatile of syntax

I don't want user who write workflow write super complex flows that make the DSL looks like a generic programming language.
Therefore, I'll make some assumptions and simply throw compile time unsupport error when unexpected syntax shows up.
parallel inside while is one of them, and I'll come up with more rules that users may think about but should not support.

Basic language primitives should all support, such as `while`, `for`, `if..else if.. else` and `break/continue`.
The unsupport list are for things like nested para block, or para block inside para while etc.

#### Type checking

Since the input/output are anyway serializable basic types (or composite from basic types), the type should be easy to conduct.
The type checking in the compile time by an extra traverse of syntex tree can also help to detect bugs such as accessing non-exist attrs of a identifier.

### Performance

The interpreter of oxiida is as tree-walk interpreter, it uses rust as the transpiler IR.
Because the performance of the oxiida language has different target compare to generic programing languages. 
The bottle neck of running workflows are mostly depends on the resources rather than the language itself.
Therefore when talking about the performance of workflow language, the performance is the throughput per CPU. 
It means the more remote (also include the local process not directly run with the engine) processes a CPU which runs the engine can handle, the more performance it has.
The codegen of oxiida language is to generate to rust code that can either directly run as script or passing to the running oxiida engine.

The "engine" is a fancy word that can scare future developers away because it seems hard to touch (or misused to show some crap design is fakely fancy).
It also not so much clear describe how oxiida running user defined processes and workflows.
The close jargon in programing language design should be the process virtual machine or runtime.

### Error handling

The workflow definition and the workflow execution are separated.
The interpreter should try its best to spot the syntax or workflow definited problem before it moved to runtime. 

## Foreign function interface (FFI) support

### Python

The `oxiida` workflow language can embed into python, and run from python with calling python-side function by passing local environment.
The `oxiida` is wrapped as a python library also named `oxiida`, and has function `run` that can get a string of workflow code and run it in the rust side runtime.
Python functions defined in the local scope will be used to store function declarations.
When there is a function call from oxiida side, it will first look into oxiida scope (the scope of the source workflow definition).
If nothing found in the oxiida scope, it will relay the call to the foreign function where the binding implemented, look up the function and call.
The function look up ad call is done by passing a message with function name and parameters. 
The parameters are from ox workflow source code.
Message can be listen by an actor can dispatch the python function call one thread per each message.
The actor is either started with local_set without daemon, or start with the daemon.
Every workflow has its own actor spawned in the daemon, the reason is to make workflow has independent ffi local environment to avoid mutation of variable across workflows.

The foreign function should be declared with `require` keywords before it can be called.
After declaration, it is used with function call syntax.

The design decision of using an actor to spawn blocking thread for running foreign function is because I want to avoid have any foreign language binding crate as dependencies for the core oxiida. 
It sort of like an rpc pattern but the message didn't go through tcp stream with requiring a broker middleware.
The messages are communicated over rust tokio channel and errors live in the runtime I have full control thus no burden to debug.
The actually foreign function call is through the binding in this case the pyo3 to have rust code call python in a `with_gil` block inside a `spawn_blocking` thread.
Using a dedicated new thread to make foreign function call make it possible to shift the load of managing CPU bound python functions to system scheduler.

### Julia/Lua (TDDO)

Look at python, how it should be done. 
Do I need to support having rust side can call all plugins?? 
How to make it possible to have plugin community around different language?? 
Can nvim an exampler?

### serde

serde_json used for persistence data format, msgpack used for ffi call.
In the future, when plugins need to be supported, also use msgpack to communicate.

## Misc

### SSH communication

- when upload files to remote, construct files/folders in local sandbox. Then create same folder tree structure. Then upload files. Finally verify the tree structure.
- metioned by Alex, how fine should the API exposed to allow different users to interact with the SSH? As a final user, the more abstract the better, as a plugin developer, the explicit the better. It is an API design problem so it is hard in general. I need to keep thinking about this.

### Relation with AiiDA

AiiDA has a lot of good design decisions and the community is one of the biggest treasure should not be ignored.

The calcjob and workchain has clear specification and clear type definition which make them well definied and can be generic.
Oxiida should provide native support to run them.
The class should be convert to syntax tree and modified into oxiida format and run through oxiida interpreter.

The plugin system (more like a dependency injection pattern in AiiDA) is good, it is the fundation of gathering a community.

Some desgins are just bad, so ignore them:

- A heavy and too much details CLI, need to replace with a TUI.
- orm is the thing I always want to avoid, it should be an excuse for not learning SQL (core developers should know SQL, users can use another DSL).
- workgraph is crap :< it has no design considerations, it has piles of ai-gen code, it duplicate aiida-core engine by copy plumpy codebase (thanks for this shit that push me away from AiiDA)
- the engine part (plumpy/kiwipy) is just complex and I am not proud to understand that complicity. It is not needed if multithreading is natively supported (thanks python's GIL, f*ck you!).
