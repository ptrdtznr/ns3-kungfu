ns-3 is a discrete-event network simulator for Internet systems, targeted primarily for research and educational use. The following lines might help setting up things easier. 

1. [Installation/-Build](#installation/-build)
2. [Running](#running)
3. [Debugging](#debugging)
4. [Profiling](#profiling)


# Installation/ Build

Installation is straight forward. Just follow the [Installation instructions from ns-3](https://www.nsnam.org/wiki/Installation). However, using *bake* makes life much easier. 

## Non-Optimized
``` 
./waf configure --enable-examples --disable-tests
``` 
## Optimized
``` 
./waf configure -d optimized --enable-examples --disable-tests
``` 

Once you have configured, make sure to build it:
``` 
./waf build
``` 

# Running
``` 
./waf --run "YOUR-SCRIPT"
``` 


# Debugging

I am working with [vscode](https://code.visualstudio.com/), a super code editor. However, once you are familiar with it, you need to create a debug configuration and to have these files in your workspace of your project (`ns-3/contrib/your-module`):
```
── .vscode
│   ├── c_cpp_properties.json
│   ├── launch.json 
│   └── tasks.json
```
## c_cpp_properties.json
```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "~/bake/source/ns-3.31/build"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "gcc-x64",
            "compileCommands": "~/bake/source/ns-3.31/build/compile_commands.json"
        }
    ],
    "version": 4
}
```

## launch.json
Launch config emulates waf environment, which calls gdb directly. My build folder is `${workspaceFolder}/build`, substitute it with yours. Replace the values inside `environment` below with those found from running by executing in your ns-3 folder:

```
./waf shell
env 
```

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) nbaton-ns3",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/../../build/contrib/*TO_YOUR_DEBUG_FILE",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [
                {
                    "Name": "PYTHONPATH",
                    "Value": "REPLACE_THIS_WITH_YOUR_*PYTHONPATH*_VALUES"
                },
                {
                    "Name": "LD_LIBRARY_PATH",
                    "Value": "REPLACE_THIS_WITH_YOUR_*LD_LIBRARY_PATH*_VALUES"
                },
                {
                    "Name": "NS3_MODULE_PATH",
                    "Value": "REPLACE_THIS_WITH_YOUR_*NS3_MODULE_PATH*_VALUES"
                },
                {
                    "Name": "NS3_EXECUTABLE_PATH",
                    "Value": "REPLACE_THIS_WITH_YOUR_*NS3_EXECUTABLE_PATH*_VALUES"
                },
                {
                    "Name": "PATH",
                    "Value": "REPLACE_THIS_WITH_YOUR_*PATH*_VALUES"
                }
            ],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "logging": {
                "engineLogging": true,
                "trace": true
            }
        }
    ]
}
```

## tasks.json

Since your module is located in `ns-3/contrib/your-module` you can simply call the `./waf build` command with a relativ path like this:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "echo",
            "type": "shell",
            "command": "../.././waf",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

# Profiling

First you need to install [valgrind](https://valgrind.org/) and [kcachegrind@SourceForge](http://kcachegrind.sourceforge.net/html/Home.html) resp. [kcachegrind@Github](https://github.com/KDE/kcachegrind).
```
sudo apt-get install valgrind kcachegrind
```

Caution: It is important to `--enable-static`, otherwise you will end up profiling your main program only and nothing else (especially not your code).

```
./waf configure -d optimized --enable-examples --disable-tests --enable-static
./waf build
```
After few minutes ;-) you are able to profile your code like this:

```
./waf --run "YOUR-SCRIPT" --command-template="valgrind --tool=callgrind --trace-children=yes --collect-jumps=yes %s"
kcachegrind callgrind.out.*
```

Be passioned, it takes some time but it helps ;-) 