# jelly: the JSON build system

# SUMMARY

A basic build system cobbled together from the jq JSON utility and a handful of POSIX standard commands.

# EXAMPLE

```console
$ cd example

$ ./jelly
Pass

$ ./jelly test
Pass

$ ./jelly lint
./good.json
{
  "count": 3
}
./bad.json
parse error: Expected another key-value pair at line 3, column 1

$ ./jelly help
Usage: ./jelly [<task>]

Tasks:

help
lint
test
```

# CONFIGURATION

The build system is configured with a jq script named `jelly` (no file extension). The file should feature `chmod +x` permissions.

## Create a task

jelly tasks are created with shell command strings, indexed by task name in a single JSON document.

A task may specify multiple shell commands through the conventional semicolon (`;`) shell command delimiter.

Note that additional quoting may be required for certain characters, according to traditional C-style backslash syntax.

```sh
#!/bin/sh
unset IFS
set -euf

jq -r '.[($ARGS.positional[0] // "test")]' --args "$@" <<HERE | sh
{
    "test": "echo 'Pass'",
    "lint": "for f in \$(find . -iname '*.json' -print); do echo \\"\$f\\" && jq . \\"\$f\\"; done"
}
HERE
```

Above, we see a stripped down jelly build system, with just two tasks, `test` and `lint`.

The three opening lines `#!/bin/sh`, `unset IFS`, `set -euf` setup a robust, POSIX compliant shell script wrapper around an inner jq script. This allows jelly to automatically process task commands with `| sh`.

Note that the JSON configuration resides inside of a shell heredoc. In addition to any JSON string escapes, certain special characters like backslash (`\`) and dollar (`$`) in your task shell commands, will require additional backslash escapes.

Naturally, you will want to replace uninformative `echo 'Pass'` test command with some real commands relevant to your specific testing needs.

The jq expression `// "test"` declares the `test` task as the default, when no task is named when jelly runs.

## Usage menu

```sh
#!/bin/sh
unset IFS
set -euf

jq -r '.[($ARGS.positional[0] // "test")]' --args "$@" <<HERE | sh
{
    "test": "echo 'Pass'",
    "lint": "for f in \$(find . -iname '*.json' -print); do echo \\"\$f\\" && jq . \\"\$f\\"; done",
    "help": "printf \\"Usage: ./jelly [<task>]\\\\n\\\\nTasks:\\\\n\\\\n\\"; tail -r ./jelly | tail -n +2 | tail -r | tail -n +6 | sed 's/\\\\\\\\\\\\\\\\/\\\\\\\\/g' | jq -r 'keys | .[]'"
}
HERE
```

The canned `help` task generates a usage menu. It works by extracting the task names from its own jelly script.

## Dry Run

For debugging, temporarily replace `| sh` with `| cat` to see the command strings that jelly would have executed.

## Further Research

There is probably a way for json to perform the simple text substitution duties on raw string text input, which would remove the need for sed in usage menu generation.

We should improve validation so that typos are caught and transformed into a request for the usage menu. Currently, invalid task names sent to `./jelly` result in empty output. Strange output like that is ver bad to send to `sh`. Perhaps POSIX has something to say about what `sh` should even do with null output. That may not be well defined behavior, or present a security risk to some operating systems. In any case, the user deserves a more specific error message.

A looping mechanism would add support for processing multiple targets in a single `./jelly <task> <task> <task>`... invocation.

GNU `head` features support for `-n` with negative values, removing the need for inefficient `tail -r` file content reversal.

Much JSON related work will be simplified when the format officially adopts a comment syntax, preferably one that integrates well with UNIX shebangs.

# REQUIREMENTS

* a UNIX environment with [coreutils](https://www.gnu.org/software/coreutils/) / [base](http://ftp.freebsd.org/pub/FreeBSD/releases/) / [macOS](https://www.apple.com/macos) / [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) / etc.
* [jq](https://jqlang.github.io/jq/) 1.6+
* [sed](https://pubs.opengroup.org/onlinepubs/009695299/utilities/sed.html)

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
* [Rebar3](https://www.rebar3.org/), a build system for Erlang projects
* [rez](https://github.com/mcandre/rez) builds C/C++ projects
* [Shake](https://shakebuild.com/), a task runner for Haskell projects
* [ShellCheck](https://www.shellcheck.net/), a shell script linter with a rich collection of rules for promoting safer scripting
* [slick](https://github.com/mcandre/slick), a linter to enforce stricter, unextended POSIX sh syntax compliance
* [stank](https://github.com/mcandre/stank), a collection of POSIX-y shell script linters
* [tinyrick](https://github.com/mcandre/tinyrick) for Rust projects

🍮
