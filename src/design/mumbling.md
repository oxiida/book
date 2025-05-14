# Mumbling

## Relation with AiiDA

The name Oxiida means the oxidize (rusty) AiiDA. 
Things get oxidized things get stable. 
AiiDA never stable because it never ever had clear boundaries on the internal and public APIs. 

Have to say, AiiDA has a lot of good design decisions and the community is one of the biggest treasure should not be ignored.
Among the material simulation community it is the most unique one that try to cover and use advance technology to reach a high throughput on running scientific workflows.
Using RMQ + PSQL to leverage multiple processes to interact with large amount of remote resources is a nice workaround for python GIL back to 2016 when asyncio is not yet a python's stdlib.
Especially, when 6 years ago when I need a workflow to support the large database storage it is the only thing I can find in the field.

The calcjob and workchain has clear specification and clear type definition which make them well definied and can be generic.
Oxiida should provide native support to run them.
The class should able to be convert to syntax tree and modified into oxiida format and run through oxiida interpreter.

The plugin system (more like a dependency injection pattern in AiiDA) is good, it is the fundation of gathering a community.

Some desgins are just bad, so ignore them:

- A heavy and too much details CLI, need to replace with a TUI.
- orm is the thing I always want to avoid, it should not be an excuse for not learning SQL (core developers should know SQL, users can have another DSL i.e query builder).
- workgraph is crap :< it has no design considerations, it has piles of ai-gen code, it duplicate aiida-core engine by copy plumpy codebase (thanks for this shit that push me away from AiiDA)
- the engine part (plumpy/kiwipy) is just complex and I am not proud to understand that complicity. It is not needed if multithreading is natively supported (however, before 3.13 still not possible to opt-in with real multithreading performance, thanks python's GIL, f*ck you!).
