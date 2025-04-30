# Oh! gil! Un-painful parallelism

## Multi-threading for IO-bounded tasks

In python, you can use multithreading to unlock some performance for IO-bounded instructions.
For instance look at the example below, the `io_work()` calls `time.sleep()` function which is an IO-bounded operation.
However, if python runs in sequence, this sleep will block other operations there until it finished.
Using multithreading can release the CPU power by not blocking such type of IO tasks.

```python
import oxiida
from time import time, perf_counter, sleep

def count_duration(t0: float) -> float:
    return perf_counter() - t0

def io_work():
    sleep(1)

# How long does two run take?
pt0 = perf_counter()
io_work()
io_work()
duration = count_duration(pt0)
print(f"run in sequence: {duration:.2f} s")
```

It runs in 2s in total, not very good, things can sleep together.

Let's then use multi-threading

```python
# extra import of threading from python stdlib
import threading

pt0 = perf_counter()
t1 = threading.Thread(target=io_work)
t2 = threading.Thread(target=io_work)
t1.start()
t2.start()
t1.join()
t2.join()

duration = count_duration(pt0)
print(f"triple threads: {duration:.2f} s")
```

Nice, it only takes 1s now, all good.

In Oxiida, if the python ffi calls are run inside `para` block, they are spawned in multithreadings.

```python
import oxiida
from time import time, perf_counter, sleep

def count_duration(t0: float) -> float:
    return perf_counter() - t0

def io_work():
    sleep(1)

# language = oxiida
workflow = """
require io_work, count_duration, perf_counter;
require time;

print "-- multiple tasks (run in sequence) --";
t0 = perf_counter();
seq {
    io_work();
    io_work();
}
print count_duration(t0);

print "-- multiple tasks (multithreading with gil) --";
t0 = perf_counter();
para {
    io_work();
    io_work();
}
print count_duration(t0);
"""

if __name__ == '__main__':
    print("-- running workflow --")
    pt0 = perf_counter()
    oxiida.run(workflow)
    # you can get the time count and print it in python as well
    _ = count_duration(pt0)
```

If `io_work` called from `seq` block, they run in sequence, if they called from `para` block, they run concurrently in multithreading manner. 
Pretty neat, no? You can avoid to use concurrent programming language primitive such as manage join handlers. 
It is fit well for the workflow language.
Because not like generic programming language, the workflow language is target to run the task to the end instead of mutate the middle state of a task.
That is to say, at the end of scope the handlers are joined automatically to make sure the tasks run to the end.

## Multi-processing for CPU-bounded task

However, the elephant in the room is Python’s notorious Global Interpreter Lock (GIL).
For CPU-bound code, a single process can run only one thread at a time, so extra threads can’t step in to share the work.

```python
# Heavy CPU call to show the GIL still hold and cannot be surpass
def cpu_work():
    total = 0
    for i in range(10_000_0000):     # this is purely CPU, no I/O, no sleep, takes 3s in my intel-i7 CPU
        total += i * i

# 1) How long does one run take?
pt0 = perf_counter()
_ = cpu_work()
duration = count_duration(pt0)
print(f"single thread: {duration:.2f} s")

# 2) Now run two of exactly the same job in “parallel” threads
pt0 = perf_counter()

t1 = threading.Thread(target=cpu_work)
t2 = threading.Thread(target=cpu_work)
t1.start()
t2.start()
t1.join()
t2.join()

duration = count_duration(pt0)
print(f"triple threads: {duration:.2f} s")
```

Multithreading not only fails to accelerate the workload, it can actually slow the program down compared with running the tasks sequentially. 
The frequent context switches that threads introduce add overhead.
You can fall back on Python’s built-in multiprocessing, but other workflow engines can’t simply toggle it on—they’re still restricted by the expressiveness of Python’s own language constructs.

By embedding an Oxiida workflow in your Python script, you can build a CPU-bound pipeline that sidesteps the GIL: the Oxiida runtime transparently forks and orchestrates multiple OS processes through its Rust backend.

```python
import oxiida
from time import time, perf_counter, sleep

def count_duration(t0: float) -> float:
    return perf_counter() - t0

def cpu_work(idx: str):
    total = 0
    for i in range(10_000_0000):
        total += i * i
    print(f"finish! task #{idx}")

# language = oxiida
workflow = """
require cpu_work, count_duration, perf_counter;
require time;

print "-- multiple tasks (multiprocessing) --";
t0 = perf_counter();
para {
    =cpu_work=("1st");
    =cpu_work=("2nd");

    // or add more if you have more cores, it still finished with same amount of time.
    //=cpu_work=("more");
    //=cpu_work=("and more");
    //=cpu_work=("and mor more");
}
print count_duration(t0);
"""

if __name__ == '__main__':
    print("-- running workflow --")
    pt0 = perf_counter()
    oxiida.run(workflow)
    _ = count_duration(pt0)
```

The only different here is the "cat-like" syntax I introduced, which flags a function is send to run in a forked process.

If you open `top`/`htop` to monitoring the CPU/threads usage, the contrast is clear: with threads, the GIL keeps each process at rouphly 50% of CPU usage.
But with process forked , the CPU usage is 100% and the whole workload finishes in essentially the same time as a single task.

