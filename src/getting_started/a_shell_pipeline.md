# A shell pipeline

Now, let's move to a real world case, orchestrate some shell commands into workflow.

The shell task is one of the basic built-in tasks provide by oxiida.
The raw shell call is construct as 

```oxiida
out = shell {"echo", ["-ne", "Hello, Oxiida!"]};
print out.stdout;
```

```oxiida
print "--shell pipeline sugar--";
out = shellpipe { "echo" "-e" "apple\nbanana\napple\norange\nbanana\napple" | "sort" | "uniq" "-c" | "sort" "-nr" };
print out.stdout;
```
