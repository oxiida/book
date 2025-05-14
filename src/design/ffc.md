# Foreign Function Call (FFC) support

## Python

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
Using a dedicated new thread to make foreign function call make it possible to shift the load of managing IO bound python functions to system scheduler through multithreading.

However python before 3.13 has the notorious global interperet lock (GIL), so for CPU bound functions even they are spawned in the different thread, when running through pyo3, it has to hold the gil.
From python 3.13, the no-gil is opt-in therefore can release the power of multithreading when executing the cpu-bound tasks.
But it is not yet the default for the python community and not lots of community maintained libraries are compiled with no-gil.
Therefore in Oxiida python binding, when the actor get message to run the python function, it can spawn a forked process.
As a result, for running N CPU-bounded when it paralleled with multiprocessing, with syntax `=fnname=` in the para block, all CPUs used can reach 100% usage.

In the daemon mode, it requires a python function actor to run python functions that available from current environment. 
It requires two ffi, one is when python specialized daemon start it capture the current environment, the other is the closure of the script being run.
The python environment for daemon is a tricky problem in AiiDA, when new package added or when the editable installed package is changed with source code, it requires to restart daemon but it is a silent rule AiiDA users have to know.
In a mature system, the ergonomic way is to notify user to restart the daemon, otherwise responsible for the unexpected results when the environment has changed.
The solution I had in mind is to have a file system monitor start with daemon to look at the site-packages folder of the environment. 
When file changed, daemon attach a warning in front of every client call ask to restart the daemon.

## Julia/Lua (TDDO)

Look at python, how it should be done. 
Do I need to support having rust side can call all plugins?? 
How to make it possible to have plugin community around different language?? 
Can nvim an exampler?
