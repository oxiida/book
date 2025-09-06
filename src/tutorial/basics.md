# Oxiida language basics

## Overview

This tutorial covers only the most important language features, briefly discusses libraries, and at the end will direct you to reference material and resources on the other components.

## What will you learn?

How to use basic syntax to write a oxiida program and run it.

## How to run the examples?

Copy the code and store in the file, you can then execute it with `oxiida run <filename>`.

## Hello, world!

It’s traditional when learning a new language to write a little program that
prints the text `Hello, world!` to the screen, so we’ll do the same here!

> Note: This book assumes basic familiarity with the command line. Oxiida makes
> no specific demands about your editing or tooling or where your code lives, so
> if you prefer to use an integrated development environment (IDE) instead of
> the command line, feel free to use your favorite IDE. Unfortunatly I don't have 
> time to implement LSP for Oxiida. It is planned and might be only available after 
> basic language primitive constructs are fixed.

### Creating a Project Directory

You’ll start by making a directory to store your Oxiida code. It doesn’t matter
to Oxiida where your code lives, but for the exercises and projects in this book,
I suggest making a _projects_ directory in your home directory and keeping all
your projects there.

Open a terminal and enter the following commands to make a _projects_ directory
and a directory for the “Hello, world!” project within the _projects_ directory.

For Linux, macOS, and PowerShell on Windows, enter this:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

### Writing and Running an Oxiida script

Next, make a new source file and call it _main.ox_. Oxiida files always end with
the _.ox_ extension. If you’re using more than one word in your filename, the
convention is to use an underscore to separate them. For example, use
_hello_world.ox_ rather than _helloworld.ox_.

Now open the _main.ox_ file you just created and enter the code in Listing 1-1.

<Listing number="1-1" file-name="main.rs" caption="A program that prints `Hello, world!`">

```oxiida
print "Hello, world!";
```

</Listing>

Save the file and go back to your terminal window in the
_~/projects/hello_world_ directory. On Linux or macOS, enter the following
commands to compile and run the file:

```console
$ oxiida run main.ox
Hello, world!
```

Regardless of your operating system, the string `Hello, world!` should print to
the terminal. If you don’t see this output, refer back to the
["Troubleshooting"](./installation.md#troubleshooting)<!-- ignore --> part of the Installation
section for ways to get help.

If `Hello, world!` did print, congratulations! You’ve officially written a Rust
program. That makes you a Rust programmer—welcome!

The line in listing 1-1 does all the work in the little script: it prints text to the
screen. There are three important details to notice.

First, `print` keyword lead a print statement to print the evaluate value of the followed statement. 

Second, you see the `"Hello, world!"` string. It is a expression, thus the value returned from 
this evaluation to be printed by the `print` keyword on to the screen.

Third, the statement end the line with a semicolon (`;`), which indicates that this
expression is over and the next one is ready to begin. Most lines of Oxiida code
end with a semicolon.

## Basic syntax and semantic

In this language, the semicolon (`;`) serves as a statement terminator, marking the end of an expression when it is used as a statement.

### Variables and scopes

Variables types are inferred if possible otherwise it result in a compiler error.

```c
x = 5; // Ok!
```

The assignment and declaration are not distinguished, the first time the variable assigned in the scope, it get declared.

### Comments

We use c-style notation `//` to start comments.

### Types annotations

In the assignment statement, the type annotation is optional.

```c
x: Int = 4; // hardcoded precision i64
x = 4; // ok
y: Float = 4.0; // hardcoded precision f64 
y = 4.0; // ok
y: String = "hello world"; // Stored as a rust String.
y = "hello world"; // ok
```

### Function and closure

Functions allows to reuse and encapsulate code

```c
function foo(a: Int) -> Int {
  x: Int = ... 
  return x
}
```

Functions in this language may be defined recursively, as illustrated by the Fibonacci function below.

```c
function fibonacci(n: Int) -> Int {
    if (n == 0) {return 0;}
    if (n == 1) {return 1;}
    
    return fibonacci(n-1) + fibonacci(n-2);
}
```

Since we do not have mutable values, any scope can return values to communicate values back to the outer scope

```c
// (not yet supported)
y = 5
x: Int = function foo() {
  z = 5 + y // y passed by value
  return z // passed by value
}
```

TODO: Syntax not clear yet, since we imitate most likely python for for- and while-loops we should imitate the anonymous functions as well so the variable scopes are intuitively clear.

### Control flow

#### `if..else` branch

The condition is inside in the bracket.
The branch block is delimited by the `{}`.
The `if` can immediately follows `else` for multiple branches.

```c
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
```

#### `for`/`while` loop

See Jason RFC. Needed for sequential repeated actions.

What is important is that `while`/`for` loop does not start a new scope inside the block. 
This behavior align with Python and result the following output:

```c
x = 0;
while(x < 10) {
  x = x+1; 
}

print x;  // 10
```

Same for the `for` loop.

```c
x = 0;
for x in [1, 2, 3, 4] {
    x = x + 1;
}

print x; // 5
```

#### structured concurrency

- map, try_map, race, try_race

```
arr: Array = range(5)
struc_prefix: String = "metal_"
file: Array[Path, *] = map(arr, |struct_id| { // green threads spawned for each element in the arr (up to a limit)
  // struc_prefix is copied into this scope 
  run("python -c 'import utils; utils.create_structure({struc_prefix}{struct_id})'", path_new_from_cwd())
  return Path("{struc_prefix}{struct_id}.xyz")
})
```
I used Rust syntax, but whatever is simplest to write works.
In principle we could similarly implement filter and reduce functions.


## Example

Here is an example that uses basic syntax to represent a real world use case:

```c
use <std>;

// dataclass definition
dataclass MolStructure {
    name: String,
    positions: [[Float; 3]; *],
    kinds: [String; *], 
}

// define a function and customized constructor for CO
function CO(
    c_pos: [Float; 3], 
    o_pos: [Float; 3],
) -> MolStructure {
    mol = MolStructure("C monoxide", [c_pos, o_pos], ["C", "O"]);
    return mol;
}

// Example CO molecule
co_molecule = CO(
    [0.0000, 0.0000, 0.0000],  // Carbon at origin
    [1.1282, 0.0000, 0.0000],   // Oxygen along x-axis)
);

// Loop through atoms and print them
for idx in [0, 1] {
    idx_str = std::make_string_from_int(idx);
    print "Atom " .. idx_str .. ": kind=" .. co_molecule.kinds[idx] 
          .. ", position=" .. co_molecule.positions[idx];
}

// Extra: for loop over kinds
for kind in co_molecule.kinds {
    if (kind == "C") {
        print "Carbon atom! — I can do some quantum chemistry!";
    } else {
        print "A" .. kind .. " atom — check electronegativity?";
    }
}
// a tuple can mix of types
t: &(String, Float) = &("t1", 6.7);
```

## shell pipeline

Now, let's move to a real world case, orchestrate some shell commands into workflow.

The shell task is one of the basic built-in tasks provide by oxiida.
The raw shell call is construct as 

```oxiida
out = shell {"echo", ["-ne", "Hello, Oxiida!"]};
print out.stdout;
```

```oxiida
print "--shell pipeline sugar--";
out = shellpipe { 
    "echo" "-e" "apple\nbanana\napple\norange\nbanana\napple" | "sort" | "uniq" "-c" | "sort" "-nr" 
};
print out.stdout;
```
