# Quick start guide

In the quick start guide, you can see two examples:

- [Perform an arithmetic operation](#perform-an-arithmetic-operation)
- [Perform a pipleline of shell tasks](#perform-a-pipeline-of-shell-tasks)
- [Perform tasks with control flow](#perform-tasks-with-control-flow)

Tasks are executed as part of a workflow, which can be constructed just as intuitively as building arithmetic expressions or chaining commands in a shell pipeline.
Additionally, every workflow automatically records data processing, enabling backtracing and review in the future.

For both examples, the workflow can be embeded into the python, julia and lua.

- [Example of running workflow in Python](#example-of-running-workflow-in-python)
- [(TODO) Example in Julia]()
- [(TODO) Example in Lua]()

Before you start, make sure you have Oxiida installed.

- To simply run workflows with Oxiida, download the Oxiida binary [**here**](https://sourceforge.net/projects/oxiida/files/) and place it somewhere in your system’s PATH.
- If using Oxiida from Python, Julia, or Lua, install the `oxiida` library for your language of choice.

For detailed installation instructions, see the [installation guide](tutorial/installation.md).

> #### Command‑Line Notation
> 
> In this guide, lines that you **should type** in a terminal start with `$`.
> You don’t need to type the `$` character itself; it’s the prompt.
> Lines that don’t start with `$` show the _output_ of the previous command.
> PowerShell‑specific examples use `>` rather than `$`.


## Perform an arithmetic operation

Suppose you want to perform an arithmetic expression evaluation, with the requirement you want to record the full data flow for future provenance.

```c
(6 + 4) * -5 / 2^2
```

In oxiida you do exactly the same, writing following code to a file, named it `arithmetic.ox` (or name what every you like, it is just a text file).

```c
x = (6 + 4) * -5 / 2^2;

print x;
```

You run it through the oxiida binary

```console
$ oxiida run arithmetic.ox
```

## Perform a pipeline of shell tasks

Suppose you want to perform two shell pipeline operation on multiple shell commands, with the requirement you want to record the full data flow for future provenance.
Moreover, two pipeline are independent and may tasks a bit longer to finish but you don't want to wait one to finish before start the other.

The example shell piple is like this 

```console
$ echo -e apple\nbanana\napple\norange\nbanana\napple | sort | uniq -c | sort -nr 
$ echo \n
$ sleep 2

$ echo -e rust\npython\njulia\nrust\nrust\njulia\nlua\nlua | sort | uniq -c | sort -nr 
$ echo \n
$ sleep 2
```

you wait for 4 seconds and you see the output

```text
      3 apple
      2 banana
      1 orange

      3 rust
      2 lua
      2 julia
      1 python
```

Let's save 2 seconds of your life with some records of what you are doing

```c
para {
    seq {
        out = shellpipe { 
            "echo" "-e" "apple\nbanana\napple\norange\nbanana\napple" | "sort" | "uniq" "-c" | "sort" "-nr" 
        };
        print out.stdout;
        // `shellpipe` is actually a syntax sugar of combination of `shell` call
        shell {"sleep", ["2"]};
    }

    seq {
        out = shellpipe { 
            "echo" "-e" "rust\npython\njulia\nrust\nrust\njulia\nlua\nlua" | "sort" | "uniq" "-c" | "sort" "-nr" 
        };
        print out.stdout;
        shell {"sleep", ["2"]};
    }
}
```

Save it in the `para_pipe_shell.ox` and run it with

```console
$ oxiida run para_pipe_shell.ox
```

You get the same output, but depend on which `seq` (for sequence) block finish first you may get results of the language ranking before the fruit ranking.

## Perform tasks with control flow

Oxiida has native support for the control flow (it is a language so sure it support).
The loop control flow has `while` and `for` syntax support and the regular `if`..`else` with `else if` **without** sugaring it as `elif`.

Here is an made up shell call tasks that use all of them.

```c
out = shellpipe {"shuf" "-n1" "-e" "good day" "bad day" "soso day" | "tr" "-d" "'\n'"};
print out.stdout;
if (out.stdout == "good day") {
    print "I can do some scientific stuff.";
} else if (out.stdout == "soso day") {
    print "I need get a beer.";
} else {
    // it is a bad day man
    x = 0;
    while (x < 5) {
        print "I need to do some rust!";
        x = x + 1;
    }

    for uhr in ["23:00", "24:00", "01:00", "02:00"] {
        print "I keep on rust at:" + uhr;
    }
}
```

## Example of running workflow in Python

Oxiida is published in pypi, install it by 

```console
$ pip install oxiida
```

Similar as using the arithmetic, now I need to run a function (let's say I want to do high school match with `math.sin(x)`, and a customize function to compute some nonsense stuff).
And this function is defineded or imported in python.
I can run it from Oxiida with multithreading support.

Create a python script `wf.py` with following text.

```python
import oxiida
import math
from time import sleep, time

def super_complex_operation(x: float, y: float, z: float) -> float:
    intermediate1 = math.sin(x) * math.log1p(abs(y))
    intermediate2 = math.exp(-z) + x ** 2
    # Sleep 2 sec to demostrat two of them can run concurrently without writing multithreading control code.
    sleep(2);
    result = (intermediate1 + intermediate2) / (1 + abs(z - y))
    print("time:", time.strftime("%Y-%m-%d %H:%M:%S"))
    return result

# language = oxiida
workflow = """
require super_complex_operation;
require time, sleep;

para {
    print "--anchor--";
    seq {
        print(super_complex_operation(10, 3, 6));
    }

    seq {
        print(super_complex_operation(5.4, 3, 7));
    }
}
"""

if __name__ == '__main__':
    oxiida.run(workflow)
```

Then run it by 

```console
$ python wf.py
```

The powerfulness of running python function within Oxiida is you do not need to write multithreading code or async code that need to bridge with synchronous python code.
Using `para` block syntax allow to call both functions to run concurrently in separate threadings (althrough the heavy CPU load tasks are still dragged by the GIL).

Once the Python remove GIL limit, the Oxiida will levarage the no-gil CPython and automatically get full power.

