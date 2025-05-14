# Runtime

Runtime is a huge topic requires a lot mental efforts to make the design correct.
I need to come back many times to re-think how to make it well implemented to get both performance and maintainability.
There are hard concept such as `Pin`/`Unpin` need to go throught again and again and have a well understand on deep native tokio runtime implementation to borrow ideas and to make the pattern consistent.

The inner state machine is called "Process", it hold states and return handler for the runtime controller to communicate from outside of runtime.
The "Task" is a wrapper of the "Process", it contains also the input and output of the runable target. 
Inside the process, it defines generic running logic which in the context of workflow usually require involving the resource management. 
The process running logic is essentially how the input port being used to modified the output port.

Therefore the "Task" interface require `task.new` as the constructer (will builder pattern fit better??) and require `task.output` to access the output later when task completed.
It also require the `task.execute` to define how to use inputs to modify the mutated output port. 

## concurrent processes

The reserved keyword `para` is introduced to decorate the block (should also for expression evaluation??) to run in concurrent manner.
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
