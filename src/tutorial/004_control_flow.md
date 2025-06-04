# Control Flow

Blocks introduced by `for` and `if`..`else` use braces `{ ... }`. 
A brace block **always starts a new local scope**, so any variable you assign inside it becomes a fresh local binding even when a variable with the same name already exists outside. 

A typical if..else block looks like this,

```oxi
x = 11;

if (x > 10) {
    x = x + 1;
    y = 0;
    print "1st if branch";
    print x; // output: 12.0
} else if (x < 2) {
    seq {
        print "else if branch";
        print x;
    }
} else {
    print "else branch";
    print x;
}

// error, y not defined. different from python/julia
// print y;
// x is the outter scope x
print x;  // output: 11.0
```

An example for loop like this,

```oxi
x = 0;
for x in [1, 2, 3, 4] {
    print x + 1;
    y = x + 1;
    print y;
}

print "-------";
print x; // output: 0.0
// y is not defined in the outer scope.
// print y;
```

A `while` loop, by contrast, is closed with the keyword end and does not create a new scope. 
This lets you declare the loop variable before the loop, update it inside the loop body, and keep using it afterward.

The `while` loop looks like so,

```oxi
x = 0;

while (x < 10)
    x = x + 1;
    print x;
end

print "----";
print x;  // print 10.0
```

The language has no separate declaration keywords like `let` or `var`.
A simple assignment both declares and assigns if the variable was not found in the scope. 
Because of this, block delimiters themselves signal scope changes: braces mean "new scope", `while`..`end` means "stay in the current one."

With this scheme, a `for` loop behaves like Julia (the loop variable is local), whereas a `while` loop behaves like Python (the variable remains shared). 
The result is a minimal grammar that makes scope boundaries visually obvious and mutation rules predictable.
