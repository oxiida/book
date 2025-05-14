# Workflow definition

The anology can be made by mapping the concept of remote HPC as the CPUs and the database as the RAM (optional if the runtime keep on running and do not need to recover tasks from persistent storage, then just use RAM as "RAM").

It is unavoid to have a domain specific language to write workflow, since workflows can be regard as programming to control the remote resource.

Therefore, `oxiida` has an async runtime based on the rust tokio and has its own syntax (the standard procedures of a programming language design with lexing/parsing/interpreting) to mark as a small language for workflow construction.

