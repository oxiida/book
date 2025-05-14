# Variable scope and shadowing

I personally think a proper generic programming language should separate the declaration and assignment a variable instead of mix them like python.
But as a small DSL, it might be a bit too verbose to have explicit declaration.
However, does not separate two statement means when the assignment appears the second time, it either a valid assignment or wrongly override the old value by overlook it is already used.
I can reject any reuse of the variable but it will end up having too many variables just for simple stuff.
To mitigate the problem, I apply the pre-runtime check to throw an error only when the variable is assigned with a different type of value.

For the variable shadowing, if I go with only have a global shared var_env, the shadowing will take the Lua way by using `local` keyword to declaring the variable to the local environment.

- ? how to deal with variable confliction?
- ? when there is also ffi var_env, how to manage it together?
- ? when ox module import introduce to allow workflow reuse, how to solve confliction?

## Do I need scope and closure? (TBD)

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

