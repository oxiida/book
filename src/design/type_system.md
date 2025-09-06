# Type system

Since the input/output are anyway serializable basic types (or composite from basic types), the type should be easy to conduct.
The type checking in the compile time by an extra traverse of syntex tree can also help to detect bugs such as accessing non-exist attrs of a identifier.

Since the cost of mistakes in the runtime is so expensive, the typechecker should be lean to a very restrict genre. 
If there are infered type seems suspecious, raise error and ask for explicit type annotation.

