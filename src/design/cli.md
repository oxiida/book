# CLI

The command line interface design of AiiDA was complexed because of the tangling of the inner operation with the interface layer.
That is to say, no layer and abstraction between what the inner system should do and hwat the commands are send to the interface ask for what to do.
An typical example is the process control, which need to communicate with process queuing system (in aiida the queue is managed by the rabbitmq and event loop).
The command directly interact with the inner API that connect to the rabbitmq.
This makes multi-users support and AiiDAlab communication hard to implement and there is no security gaurenteed once user get cli access to the port.
There were effort to have standard RestAPI to talk to the engine part to communicate but it is just hard because the public APIs in aiida is a mess.

In Oxiida, the abstraction of the interface is natively under consideration.
The CLI is not directly manipulate any inner state but send message over tcpstream to communicate with the running daemon or with the persistent system (storage, or database).
This makes it easy to add authentication in the handshake when tcp connection is created either through CLI or through other interface (I am thinking about protocol buffers and using gRPC to standardlize the communication to realize the language agnostic frontend inerface design).

The message is send to an actor that responsible for manage the manipulation of inner state of the runtime system, include persistent actor and ffi tasks actor. 

The persistent actor has a single connection to the storage. 
There is a caveat here need more thinking about the lifetime of the actor. 
If there is one such actor attached with daemon and it is writing to the storage, for the `run` command, it should reuse this actor which require mutex or use a new actor which result in racing?
It leads to more question such as can multiple daemon runs at the same time and connect to the same persistence system?
The catch is always about data racing when the filesystem are shared to be accessed from more than one entity. 
No idea can be borrowed from AiiDA since it just igore such design consideration by shift the responsibility to PostgreSQL which support multi-writer.
In the design of Oxiida, the sqlite is targetted as the production database and it accept single writer by default.
The decision here is, I will have `dry_run` which always start its own task actor and write things the DB in local; `submit` by default submit to the daemon and `submit --wait` will submit and wait for workflow to finish. 
The `submit --wait` is the command which `print` will send the message back through the channel and show the print info to the user space.
If no `--wait` is passing, then the `print` statement will just log into the tracing.

The process control actor is per runtime, every actor has its own oxiida language runtime to interpret the source code. 
I can do this based on the assumption that the workflows are independent drive to the terminated status.
Every workflow get its variable environment either by closure from ffi or from where it is launched, and can then run independently in what every oxiida runtime it can run.

