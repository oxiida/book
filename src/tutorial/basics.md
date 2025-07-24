# Oxiida language basics

## Overview

TBA

This tutorial only covers the most important language features, briefly discusses libraries, and at the end will direct you to reference material and resources on the other components.

## What will you learn?

TBA

## How to run the examples?

TBA

## Hello, world!

It’s traditional when learning a new language to write a little program that
prints the text `Hello, world!` to the screen, so we’ll do the same here!

> Note: This book assumes basic familiarity with the command line. Oxiida makes
> no specific demands about your editing or tooling or where your code lives, so
> if you prefer to use an integrated development environment (IDE) instead of
> the command line, feel free to use your favorite IDE. Unfortunatly I don't have 
> time to implement LSP for Oxiida. It is planned and will be one available after 
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

## Data types

TBA

## Control flows

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

The `while` loop looks like this,

```oxi
x = 0;

while (x < 10) {
    x = x + 1;
    print x;
}

print "----";
print x;  // print 10.0
```

The language has no separate declaration keywords like `let` or `var`.
A simple assignment both declares and assigns if the variable was not found in the scope. 

With this scheme, a `for` loop behaves like Julia (the loop variable is local), whereas a `while` loop behaves like Python (the variable remains shared). 
The result is a minimal grammar that makes scope boundaries visually obvious and mutation rules predictable.

## Function and workflow

TBA

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
