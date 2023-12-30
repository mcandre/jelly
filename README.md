# jelly: the JSON build system

# SUMMARY

A basic build system cobbled together from the jq JSON utility and a handful of POSIX standard commands.

# EXAMPLE

```console
$ cd example

$ ./jelly | sh
Pass

$ ./jelly test | sh
Pass

$ ./jelly lint | sh
./bad.json
parse error: Expected another key-value pair at line 3, column 1

$ ./jelly help | sh
Usage: ./jelly <task> | sh

Tasks:

help
lint
test
```

# CONFIGURATION

The build system is configured with a jq script named `jelly` (no file extension). The file should feature `chmod +x` permissions.

## Shebang!

```console
#!/usr/bin/env jq -nrf --args
```

`#!` is the classic UNIX marker that signals the shell to apply a different interpreter to execute the rest of the script. `/usr/bin/env jq` informs the shell where to find the jq interpreter, whose absolute path will differ between operating systems.

The `-n` flag informs jq to not stall and expect instructions from stdin.

The `-r` flag informs jq to emit output in raw form, so that the shell commands it emits will not be wrapped in JSON document braces.

The `-f` flag informs jq to process the rest of the script as a series of jq instructions.

The `--args` flag informs jq to receive all successive command line arguments supplied (to `./jelly`) in the standard jq `$ARGS.positional` array variable.

## Create a task

jelly tasks are created with shell command strings, indexed by task name in a single JSON document.

A task may specify multiple shell commands through the conventional semicolon (`;`) shell command delimiter.

Note that additional quoting may be required for certain characters, according to traditional C-style backslash syntax.

```sh
#!/usr/bin/env jq -nrf --args
{
    "test": "echo 'Pass'"
}
| .[($ARGS.positional[0] // "test")]

```

Here, we see an concise jelly build system with a single `test` task.

`// "test"` declares the `test` task as the default, when no task is named when jelly runs.

Naturally, you will want to replace uninformative `echo 'Pass'` commands with some real commands relevant to your specific testing needs.

```sh
#!/usr/bin/env jq -nrf --args
{
    "test": "echo 'Pass'",
    "lint": "find . -iname '*.json' -print -exec jq . '{}' \\;"
}
| .[($ARGS.positional[0] // "test")]
```

Here, we see another task, `lint`, defined in a context for common JSON validation requirements. jq can serve as quick a linter of basic JSON document integrity, aside from any schema validation requirements. However, the wrapping in UNIX `find` will *not* preserve failing exit codes in the event of a linter warning. Pick some more robust linting commands for use in a larger CI/CD pipeline. Also, find's `-print` flag creates a lot of noise.

## Usage Menu

```sh
#!/usr/bin/env jq -nrf --args
{
    "test": "echo 'Pass'",
    "lint": "find . -iname '*.json' -print -exec jq . '{}' \\;",
    "help": "printf \"Usage: ./jelly <task> | sh\\n\\nTasks:\\n\\n\"; tail -r ./jelly | tail -n +2 | tail -r | tail -n +2 | jq -r 'keys | .[]'"
}
| .[($ARGS.positional[0] // "test")]
```

Finally, we have a `help` task devoted to generating a usage menu. This is valuable for anyone running the script, to remind them of the valid names of the tasks. The more tasks configured, the more useful the menu becomes.

This menu operates by extracting the task names from the script and generating a corresponding help message. Note that both `tail -r` reversal subcommands are used purely for portability, and are not expected to scale with the number of tasks configured in the build system. But this is not a serious build system. (Neither is NPM run-script, or make.)

This is an experiment in what is possible to do with minimal tools available, while promoting same-programming-language development scripts for more projects.

## Dry Run

For debugging, temporarily drop `| sh` to see a dry run of the commands that jelly would have executed.

## Further Research

We should improve validation so that typos are caught and transformed into a request for the usage menu. Currently, invalid task names sent to `./jelly` result in empty output. Strange output like that is ver bad to send to `sh`. Perhaps POSIX has something to say about what `sh` should even do with null output. That may not be well defined behavior, or present a security risk to some operating systems. In any case, the user deserves a more specific error message.

A looping mechanism would add support for processing multiple targets in a single `./jelly <task> <task> <task>`... invocation.

GNU `head` features support for `-n` with negative values, removing the need for inefficient `tail -r` file content reversal.

A shell function shadowing `jelly` would alleviate the need to manually append `| sh` to each run. One day, jq may even implement a syntax for executing shell commands on its own.

Much JSON related work will be simplified when the format officially adopts a comment syntax, preferably one that integrates well with UNIX shebangs.

# REQUIREMENTS

* a UNIX environment with [coreutils](https://www.gnu.org/software/coreutils/) / [base](http://ftp.freebsd.org/pub/FreeBSD/releases/) / [macOS](https://www.apple.com/macos) / [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) / etc.
* [jq](https://jqlang.github.io/jq/) 1.6+

# SEE ALSO

* Inspiration from [nobuild](https://github.com/tsoding/nobuild), a convention for C/C++ build systems
* [bashate](https://github.com/openstack/bashate), a shell script style linter
* [bb](https://github.com/mcandre/bb), a build system for (g)awk projects
* [beltaloada](https://github.com/mcandre/beltaloada), a guide to writing build systems for (POSIX) sh
* [Gradle](https://gradle.org/), a build system for JVM projects
* [lake](https://luarocks.org/modules/steved/lake), a Lua task runner
* [lichen](https://github.com/mcandre/lichen), a sed task runner
* [Mage](https://magefile.org/), a task runner for Go projects
* [npm](https://www.npmjs.com/), [Grunt](https://gruntjs.com/), Node.js task runners
* [POSIX make](https://pubs.opengroup.org/onlinepubs/009695299/utilities/make.html), a task runner standard for C/C++ and various other software projects
* [Rake](https://ruby.github.io/rake/), a task runner for Ruby projects
* [rez](https://github.com/mcandre/rez) builds C/C++ projects
* [Shake](https://shakebuild.com/), a task runner for Haskell projects
* [ShellCheck](https://www.shellcheck.net/), a shell script linter with a rich collection of rules for promoting safer scripting
* [slick](https://github.com/mcandre/slick), a linter to enforce stricter, unextended POSIX sh syntax compliance
* [stank](https://github.com/mcandre/stank), a collection of POSIX-y shell script linters
* [tinyrick](https://github.com/mcandre/tinyrick) for Rust projects

🍮
