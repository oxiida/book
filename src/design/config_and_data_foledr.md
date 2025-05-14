# Config and data folder

I don't want the ship the burden to user to setup where to store the config and where to store the db.
The default config is standard path from `directories-rs` (follow the standard of every OS).
The config will contain folders for multiple profiles, the `default` profile is created when the application first time runs, using LazyCell (non-thread safe but fine for Oxiida since the entry of cmd is single thread) to store the config parameters.

The default db when it runs with daemon mode, it should also goes with `ProjectDirs.data_dir` folder of directories-rs that is `$XDG_DATA_HOME/<project_path> or $HOME/.local/share/<project_path>` for Linux.
The data folder should be able to be changed easily through config.

