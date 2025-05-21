# Storage for data persistence

One of the key feature to mark a scientific workflow is the ability to record the dynamic running logic. 
The data nodes as the input or output of the tasks will be generated and written into the persistent storage system where the runtime runs.
It can be a file, a database or even the remote object store. 
This requires the input/output data is serializable. 
Powered by the `serde` crate, combined with the type checking of the DSL I introduced in the syntax, it is clear what should be the basic type for the tasks' input/output.
The basic types are serializable values defined in: https://docs.rs/serde_json/latest/serde_json/enum.Value.html

(Now by default everything is recorded, do I need control the storage in a fine grid??) Whether to store input/output along with the process is controlled in the block statement level.
By default, everything is not stored for performance consideration and so that provenance can be an optional feature.
~~When the block is marked as "proof" by the leading `@` for `@{ stmts }`, it casade into the statements inside.~~
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

After support Sqlite as default persistent, I remove NullPersistent since it unusable.
The FilePersistence may still useful for just dumping things and will probably raise error if user trys to query from it.
I also try to keep two to have the trait because the persistent to PostgreSQL and surrealdb is something may needed when it goes to huge throughput and for distribute deployment.
The idea is also that the base version provide the sqlite and when it potentially go to commercial stage the other database support is additional services.

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

