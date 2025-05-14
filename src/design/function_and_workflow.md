# Function and workflow

I define two types of reusable block, function and workflow.
Function is defined with the keywords `function` while the workflow is declared with `workflow`.
The different is that the function is one of the fundamental building blocks of workflow, itself is represented as a data processor task.
The workflow on the other hand can be reuse but it is more for the orchastra purpose. 
From usage point of view, running function will result in spit out a task data with input and output, while the workflow will contains the tasks and represented as the upper layer that encapsulate its callee's tasks.
On the other hand, workflow can call workflow and function, while function can only call function and only the function called from the workflow will be treated as a task.

It is a lot overhead if every granular expressions are being recorded with data into storage.
The function introduce a block that the intermiddle steps are not recorded but only the function it self as an abstract data logging node in the persistence storage.
On the contrary, the workflow keep track of all details of intermiddle steps.

This bring the idea that I may also want to introduce anonymous function/workflow.
But that require more details of design how to manage closure etc, not plan before v1.
