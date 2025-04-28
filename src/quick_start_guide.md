# Quick start guide

In the quick start guide, you can see two examples:

- [Perform an arithmetic operation](#perform-an-arithmetic-operation)
- [Perform a pipleline of shell tasks](#perform-a-pipeline-of-shell-tasks)
- [Perform tasks with control flow](#perform-tasks-with-control-flow)

Tasks are executed as part of a workflow, which can be constructed just as intuitively as building arithmetic expressions or chaining commands in a shell pipeline.
Additionally, every workflow automatically records data processing, enabling backtracing and review in the future.

For both examples, the workflow can be embeded into the python, julia and lua.

- [Example in Python](#example-in-python)
- [(TODO) Example in Julia]()
- [(TODO) Example in Lua]()

Before you start, make sure you have Oxiida installed.

- To simply run workflows with Oxiida, [download the Oxiida binary here]() and place it somewhere in your system’s PATH.
- If using Oxiida from Python, Julia, or Lua, install the `oxiida` library for your language of choice.

For detailed installation instructions, see the [installation guide](getting_started/installation.md).

## Perform an arithmetic operation

Suppose you want to perform an arithmetic expression evaluation, with the requirement you want to record the full data flow for future provenance.

```
(6 + 4) * -5 / 2^2
```

In oxiida you do exactly the same, writing following code to a file, named it `arithmetic.ox` (or name what every you like, it is just a text file).

```oxiida
x = (6 + 4) * -5 / 2^2;

print x;
```

You run it through the oxiida binary

```sh
oxiida run --storage file arithmetic.ox
```

You then get the result of `-12.5` and a file `provenance_dump.csv` file for how data nodes connected with operations.

`--storage file` is to tell Oxiida to write the data flow into a file (by default named `provenance_dump.csv`) in the same directory you running the command.
If the option not provided, the information will be dumpped to stdout.

> [!NOTE]
> Persistence the data flow to csv file is only for prototype demostration purpose. 
> It is mainly to show I can easily implement a persistence approach and have a multiple writer without using mutex because the actor pattern is used here.
> The planned feature is to support SQLite as default data storage backend.

## Perform a pipeline of shell tasks

Suppose you want to perform two shell pipeline operation on multiple shell commands, with the requirement you want to record the full data flow for future provenance.
Moreover, two pipeline are independent and may tasks a bit longer to finish but you don't want to wait one to finish before start the other.

The example shell piple is like this 

```sh
echo -e apple\nbanana\napple\norange\nbanana\napple | sort | uniq -c | sort -nr 
echo \n
sleep 2

echo -e rust\npython\njulia\nrust\nrust\njulia\nlua\nlua | sort | uniq -c | sort -nr 
echo \n
sleep 2
```

you wait for 4 seconds and you see the output

```
      3 apple
      2 banana
      1 orange


      3 rust
      2 lua
      2 julia
      1 python
```

Let's save 2 seconds of your life with some records of what you are doing

```oxiida
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

```sh
oxiida run para_pipe_shell.ox
```

You get the same output, but depend on which `seq` (for sequence) block finish first you may get results of the language ranking before the fruit ranking.

## Perform tasks with control flow

Oxiida has native support for the control flow (it is a language so sure it support).
The loop control flow has `while` and `for` syntax support and the regular `if`..`else` with `else if` **without** sugaring it as `elif`.

Here is an made up shell call tasks that use all of them.

```
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

## Example in Python

Similar as using the arithmetic example as showed above, now I need to run a function (let's say I want to do high school match with `math.sin(x)`, and a customize function to compute some nonsense stuff).
And this function is defineded or imported in python.
I can run it from Oxiida and also surpass the GIL!

```python
import oxiida
import math
from time import sleep, time

def super_complex_operation(x: float, y: float, z: float) -> float:
    intermediate1 = math.sin(x) * math.log1p(abs(y))
    intermediate2 = math.exp(-z) + x ** 2
    # Sleep 2 sec to demostrat two of them can run concurrently without GIL limitation
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

The powerfulness of running python function within Oxiida is you are not limited by the python GIL anymore.
Using `para` block syntax allow to call both functions to run concurrently in separate threadings.

