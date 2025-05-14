# Error handling (syntax error)

The workflow definition and the workflow execution are separated.
The interpreter should try its best to spot the syntax or workflow definited problem before it moved to runtime. 
The check is done in the syntax tree resolve phase to spot errors such as misuse of tokens and undefined variables.
