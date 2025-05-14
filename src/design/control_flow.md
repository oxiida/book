# Control flow

- ~a while loop evaluates to an array containing each element that the body evaluated to, like CoffeeScript?~ no, this is quite unusual syntax, and while is then an expression. 

I support the for and while loop. 
The for loop is loop over a lenght fixed array thus it will always end the loop.
The while loop has the risk to use to not exit the loop, this will happend when the incremental expression is missed to run.
So first of all, recommend to use for loop over while loop.
Then for the while loop, it require to add a max iter to avoid the infinite loop, which is an outlandish syntax therefore to introduce it nicely I'll have a resolve time checker point to how to use this syntax when it is not provided in the first place.

There is an ambiguity when combine for loop with using loop entity passing as identifier. 
Becasue the `for` statement yet only accept passing explicit array in the grammar.
To support it, cases between it is a explicit array and a identifier (trickyer if also want to support Attr expression) should be separately take care of.
Now, I simply don't allow to parsing following syntax: 
```oxiida
xs = [1, 2, 3, 4];
print xs;

for x in xs {
    print x + 1;
}
```

