# Miscellaneous (no categorized)

## How much APIs exposed by Process?

Implementing a certain type of `Process` is quite sophisticate, it may result in infinite loop or putting functions in wrong mode (async v.s. sync).
Thus, the core part should provide all the implemented types processes to try to cover some generic cases, any new process type will regard as new features.
Provided process types are:
- `SchedulerProcess`: similar to AiiDA `CalcJob`, bundle the remote file operations through transport and perform scheduler (slurm first and see what requirement it has, make it plugable) operations through transport.
- `LocalProcess`: no remote communication required, but use the resources where the service is running. The process spawn using spawn-blocking in other threads for performance, but require tense monitoring to avoid draining the resource and starving the service.
- `KubernetesProcess`: communicate to a k8s cluster and start customized pod on demand, i.e. cloud processes.
- `HyperequeueProcess`: or sub-scheduler using hyperqueue (rust crate, therefore can call through native APIs) as native sub-scheduler once a large abstract resource is available. 

See if the categories above needed, some might be categorize under same abstraction.

## Paused state

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

## Resource actors pool

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

## run process in local thread

The Process does not need to be `Send + Sync`, since I made it only run in `local_set` theread. 
The reason is send over can be expensive when process carry a lot of data.
Meanwhile, process is `Send` means the type of its inner data it carries needs to be `Send`, for instance the input and output. 
This make it cumbersome to having explicitly write `Send` trait bound for its data.

## The create state

The created state seems a bit redundant since the first running state can be the entry for launching.
Maybe it is useful when need to write persistent proc info intot disk i.e. DB.

## SSH communication

- when upload files to remote, construct files/folders in local sandbox. Then create same folder tree structure. Then upload files. Finally verify the tree structure.
- metioned by Alex, how fine should the API exposed to allow different users to interact with the SSH? As a final user, the more abstract the better, as a plugin developer, the explicit the better. It is an API design problem so it is hard in general. I need to keep thinking about this.

