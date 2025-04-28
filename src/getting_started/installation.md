# Installation

Before you start, make sure you have **Oxiida** installed.

## Pre-compiled binaries

Download the Oxiida binary from [SourceForge](https://sourceforge.net/projects/oxiida/files/) and place it somewhere in your system’s `PATH`.

## From a programming language 

If you intend to call Oxiida from Python, Julia, or Lua, install the corresponding `oxiida` library:

| Language | Package manager command |
| -------- | ----------------------- |
| Python   | `pip install oxiida`    |
| Julia (not yet avail)  | `import Pkg; Pkg.add("Oxiida")` |
| Lua (not yet avail)    | `luarocks install oxiida` |

### Troubleshooting

To check whether you have Oxiida installed correctly, open a shell and enter this
line:

```console
$ oxiida --version
```

You should see the version number.

```text
oxiida x.y.z
```

If you see this information, you have installed Oxiida successfully! If you don’t
see this information, check that Rust is in your `PATH` system variable as
follows.

In Linux and macOS, use:

```console
$ echo $PATH
```

If that’s all correct and Oxiida still isn’t working, please join the Zulip chat and ask questions directly

- [Join oxiida Zulip chat](http://bit.ly/44wK00z)
