# Performance

The interpreter of oxiida is as tree-walk interpreter, it uses rust as the transpiler IR.
Because the performance of the oxiida language has opinionate goal compare to generic programing languages. 
The bottleneck of running workflows are mostly depends on the resources rather than the language itself.
Therefore when talking about the performance of workflow language, the performance is the throughput per CPU. 
It means the more remote (also include the local process not directly run with the engine) processes a CPU which runs the engine can handle, the more performance it has.
The codegen of oxiida language is to generate to rust code that can either directly run as script or passing to the running oxiida engine.

The "engine" is a fancy word that can scare future developers away because it seems hard to touch (or misused to show some crap design is fakely fancy).
It also not so much clear describe how oxiida running user defined processes and workflows.
The close jargon in programing language design should be the process virtual machine or runtime.

