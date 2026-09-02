# Makefile as the single point of entry for the project

Developing any project eventually involves a multitude of commands, ranging from running tests and checking linting to building images and applying migrations. These commands end up in the README and in the developers’ minds, inevitably becoming out of sync with reality. As a result, a new team member might spend hours figuring out how to run the project locally, while a veteran of the project might find themselves in the same situation after a long hiatus.

The `make` utility can help solve this problem by gathering all the commands needed to work in one place, called a `Makefile`. The `Makefile` isn’t just a list of shortcuts; it’s living documentation that never becomes outdated, because if a command stops working, it will be fixed. Unlike a README, which may not be kept up to date, the `Makefile` is updated as needed, ensuring it remains current throughout the project’s lifecycle.

Cases where `make` isn't necessary:
- The project is small and has 1–2 working commands
- There is already a well-established tool, such as `npm` (although in terms of automation, `make` has advantages over `npm`)

The official `make` documentation can be found at [this link](https://www.gnu.org/software/make/manual/make.html).

## What is a Makefile?

`make` is a command-line utility. It was originally created to automate the compilation of programs and to solve the problem of having to recompile an entire project when only a single file had changed, but later on, this concept of describing tasks and the dependencies between them found more general applications.

A `Makefile` is a file containing rules for the `make`. It is located in the project root and describes a set of named commands (targets) that can be run with a single command:
```bash
make build # Run the build
make test # Run the tests
make run # Run the application
```

You can also call targets together, in which case they will be processed in the order in which they are listed at the time of the call:
```bash
make build test run
```

Under the hood, these are simply wrappers (rules) around regular terminal commands.

> People who have used `npm` before may notice some similarities. In the context of JavaScript/TypeScript, `make` is like `npm`, and `Makefile` is like the `scripts` section in `package.json`—the logic is the same.

More information about `make` and `Makefile` can be found [here](https://www.gnu.org/software/make/manual/make.html#Overview).


## Installing make

There are several ways to install `make`.

### Windows

Prerequisites:
- Windows 10/11
- PowerShell running as an administrator

The easiest and recommended way to install `make` on Windows is to use the [Chocolatey](https://chocolatey.org/) package manager. Instructions for installing `Chocolatey` can be found [here](https://chocolatey.org/install).

To install `make`, run:
```bash
choco install make
```

To verify that the installation was successful, run:
```bash
make --version
```

If the installation was successful, the installed version of `make` should be displayed.

### Linux

> Depending on the distribution, different package managers may be preinstalled; be sure to use the correct one. (Debian/Ubuntu - `apt`, Alpine - `apk`).

To install `make`, run:
```bash
apt install make # For Debian/Ubuntu
apk add make # For Alpine
```

To verify that the installation was successful, run:
```bash
make --version
```

### MacOS

The easiest way to install `make` on macOS is to install the Command Line Tools (CLT) (Apple's lightweight developer toolkit).

To install `make`, run:
```bash
xcode-select --install
```

To verify that the installation was successful, run:
```bash
make --version
```


## Makefile Syntax

For the most part, a Makefile consists of a set of rules—text structures that describe the sequence of actions and, if necessary, dependencies on other rules.

The structure of a rule is very simple:
```makefile
target: prerequisites
    recipe
```

Recalling that `make` was originally conceived as a tool for automating compilation, the **target** or target file is usually a filename, but it can also be the name of an action to be performed when referring to **utility** (phony) rules.

A **recipe** is a set of actions performed by `make`. A single recipe can contain multiple commands, which can be on the same line or on separate lines. Note that each line of a recipe must begin with a tab character.

Sometimes, before a set of actions can be executed, other actions must be performed first; in other words, a recipe may have **prerequisites** on other rules. Such prerequisites are specified after the target of the current rule and the `:` character, taking the form of a list of targets (separated by ` `) on which the current rule depends.

Here is an example of a rule that, when invoked, will print `"Hello, world!"` to the terminal:
```makefile
# Makefile

hello:
    echo "Hello, world!"
```

To run the rule, you must call `make` in the terminal, specifying the target of the desired rule:
```bash
make hello

# Terminal:
# echo "Hello, world!"
# Hello, world!
```

By default, `make` prints the command before executing it. To disable this, add the `@` symbol before the command:
```makefile
# Makefile

hello:
    @echo "Hello, world!"
```

Then, when executed, the following will be printed:
```bash
make hello

# Terminal:
# Hello, world!
```

More information about rules and their structure can be found [here](https://www.gnu.org/software/make/manual/make.html#Rule-Introduction).


### Prerequisites for the targets

Sometimes it may be necessary to run one target before executing another; in other words, one target may depend on another, which in turn depends on a third, and so on (think of compiling files). To handle this, you can specify dependencies immediately after the target name, followed by the `:` symbol:
```makefile
# Makefile

build:
    @echo "Building the project..."

test: build
    @echo "Running tests..."
```

`make` will resolve dependencies automatically:
```makefile
make test

# Terminal:
# Building the project...
# Running tests...
```

There is no limit to the number of dependencies.

More information about dependencies can be found [here](https://www.gnu.org/software/make/manual/make.html#Rule-Introduction).


### Phony targets

By default, `make` assumes that a target is a filename. If a file with a name identical to that of a target is found in the project, `make` will not run the corresponding target, since it will assume that the file is already present.

A phony target is not an actual filename. In fact, it is simply a name for the actions that should be performed on request.

You can declare phony targets using the `.PHONY` helper target:
```makefile
# Makefile

.PHONY: build test

build:
    @echo "Building the project..."

test: build
    @echo "Running tests..."
```

> A single `Makefile` can contain multiple `.PHONY` targets, but it is recommended to specify only one that contains all phony targets, as this makes them easier to track.

More information about phony targets can be found [here](https://www.gnu.org/software/make/manual/make.html#Phony-Targets).


### Variables

`Makefile` supports the definition of variables, which are strings of text. Variables can be used in various parts of the `Makefile`, such as commands, dependencies, conditions, and so on.

Variable name requirements:
- May consist of any sequence of characters except: `:`, `,`, `#`, `=`, and **spaces**
- Case-sensitive; `port` and `Port` are **different variables**
- Names beginning with a `.` or an uppercase letter are reserved by `make` (although they can be defined, their functionality is not guaranteed with future updates to `make`)

You can define a variable as follows:
```makefile
# Makefile

APP_HOST = 127.0.0.1
APP_PORT := 8080
```

There are four types of variables: **recursive** (`=`), **simple** (`:=`/`::=`), **conditional** (`?=`), and **special**.

#### Recursive Type

Variable definition:
```makefile
# Makefile

APP_HOST = 127.0.0.1
```

The value is set at the time of **use**; the order **does not** matter:
```makefile
# Makefile

ENV_WIZARD_MESSAGE = Current environment: $(ENV_TYPE)
ENV_TYPE = production

show-environment-wizard:
    @echo $(ENV_WIZARD_MESSAGE) # `ENV_WIZARD_MESSAGE` is generated at this moment
```

```bash
make show-environment-wizard

# Terminal:
# "Current environment: production"
```

**Infinite loops** are possible:
```makefile
# Makefile

FILES = $(FILES) main.cpp # Error
```
    
#### Simple Type

```makefile
# Makefile

APP_HOST := 127.0.0.1
APP_PORT ::= 8080 # POSIX format, identical to `:=`
```

The value of a variable is set at the time of **declaration**; the order **matters**:
```makefile
# Makefile

ENV_WIZARD_MESSAGE = Current environment: $(ENV_TYPE) # `ENV_WIZARD_MESSAGE` is generated at this point
ENV_TYPE = production

show-environment-wizard:
    @echo $(ENV_WIZARD_MESSAGE)
```

```bash
make show-environment-wizard # "Current environment: "
```

You can **safely** expand a variable using itself, as shown in the example with `pytest` (a Python testing library) below:
```makefile
# Makefile

PYTEST_ARGS := tests/

ifdef VERBOSE
PYTEST_ARGS := $(PYTEST_ARGS) -v
endif

ifdef COVERAGE
PYTEST_ARGS := $(PYTEST_ARGS) --cov=src --cov-report=html
endif

test:
    @pytest $(PYTEST_ARGS)
```

```bash
make test # test/
make test VERBOSE=1 # pytest test/ -v
make test COVERAGE=1 # pytest test/ --cov=src --cov-report=html
make test VERBOSE=1 COVERAGE=1 # pytest test/ -v --cov=src --cov-report=html
```

To use a variable, use the `$` symbol with parentheses or curly braces:
```makefile
# Makefile

run:
    @my-api \
        --host $(APP_HOST) \
        --port ${APP_PORT}
```

> If you need to escape the `$` symbol, use `$$`.

#### Conditional Assignment

There is another assignment operator `?=` (conditional assignment) which works only when the variable has not yet been defined.

This operator can be used to set default values:
```makefile
#  Makefile

ENV_TYPE ?= development

ifeq ($(ENV_TYPE), production)
LOG_LEVEL := ERROR
else
LOG_LEVEL := DEBUG
endif

run:
    @echo "Current environment: $(ENV_TYPE)"
    @python main.py --log-level $(LOG_LEVEL)
```

```bash
make run # "Current environment: development"
make run ENV_TYPE=production # "Current environment: production"
```

#### Service Variables

##### .DEFAULT_GOAL

By default, when you run `make`, the first tagret in `Makefile` will be executed, with the exception of utility target. To override the default target, use the `.DEFAULT_GOAL` variable:
```makefile
# Makefile

.DEFAULT_GOAL := help

help:
    @echo "List of available goals:"
    @echo "help - Displays a list of available goals"
    @echo "run - Runs the project"
    @echo "run-tests - Runs the tests"

run:
    @python main.py

run-tests:
    @pytest
```

```bash
make

# Terminal:
# List of available goals:
# help - Displays a list of available goals
# run - Runs the project
# run-tests - Runs the tests
```

#### Importing Variables from .env

Sometimes it can be useful to use environment variables defined in the `.env` file within a `Makefile`. To import variables from `.env` into a `Makefile`, use:
```makefile
# Makefile

include .env
export
```

In the previous example, if the `.env` file is missing, `make` will throw an exception. To handle this case, you can use the built-in `wildcard` function, which checks for the file’s existence and returns an empty string if it doesn’t exist.
```makefile
# Makefile

ifneq ($(wildcard .env),)
include .env
export
endif

ENV_TYPE ?= development

ifeq ($(ENV_TYPE), production)
LOG_LEVEL := ERROR
else ifeq ($(ENV_TYPE), staging)
LOG_LEVEL := WARNING
else
LOG_LEVEL := DEBUG
endif

run:
    @echo "Current environment: $(ENV_TYPE)"
    @python main.py --log-level $(LOG_LEVEL)
```

```shell
# .env

ENV_TYPE=production
```

```bash
make run

# Terminal:
# Current environment: production

make run ENV_TYPE=staging # The argument has the highest priority

# Terminal:
# Current environment: staging
```

More information about variables can be found [here](https://www.gnu.org/software/make/manual/make.html#Using-Variables).


### Passing Arguments

Rules in a `Makefile` support the ability to retrieve data from outside that is, at the time a specific target is invoked by defining arguments. Essentially, arguments are identical to variables. An argument is always of type string.

An argument is used in the same way as a variable:
```makefile
# Makefile

run:
    @python main.py --port $(port)
    @echo "The application is running on port $(port)"
```

```bash
make run port=8080

# Terminal:
# The application is running on port 8080
```

If a target has dependencies, the argument’s value will also be available when the dependency is executed:
```makefile
# Makefile

build:
    @echo "Building for the $(env-type) environment"
    if [ "$(env-type)" = "production" ]; then \
        @conan build . --settings:host="build_type=Release" \
    else \
        @conan build . --settings:host="build_type=Debug" \
    fi

run: build
    @echo "Running the application in the $(env-type) environment"
    if [ "$(env-type)" = "production" ]; then \
        ./build/Release/my-api \
    else \
        ./build/Debug/my-api \
    fi
```

```bash
make run env-type=production

# Terminal:
# Build for the production environment
# Run the application in the production environment

make run env-type=development

# Terminal:
# Build for the development environment
# Run the application in the development environment
```

More information on passing arguments can be found [here](https://www.gnu.org/software/make/manual/make.html#Overriding).

## Using Multiple Makefiles

`Makefile` rules can be grouped—for example, by topic—and split into multiple files. These files must have the `.mk` extension and be specified when calling `make` using the `-f` (`--file`) or `--makefile` option:
```makefile
# deploy-nestjs.mk

deploy-with-nestjs-api:
    @export DEPLOY_DATABASE=false && \
    helmfile cache cleanup && helmfile --environment local --namespace local -f deploy/helmfile.yaml.gotmpl apply
```

```makefile
# deploy-cpp.mk

deploy-with-cpp-api:
    @export DEPLOY_DATABASE=true && \
    export TO_DOS_API_REPO=to-dos-api-cpp && \
    helmfile cache cleanup && helmfile --environment local --namespace local -f deploy/helmfile.yaml.gotmpl apply
```

```bash
make -f deploy-nestjs.mk deploy-with-nestjs-api
make -f deploy-cpp.mk deploy-with-cpp-api
```

Such files can also be imported into the main Makefile using the `include` directive:
```makefile
# Makefile

include deploy-nestjs.mk
include deploy-cpp.mk

destroy:
    @export DEPLOY_DATABASE=true && \
    helmfile cache cleanup && helmfile --environment local --namespace local -f deploy/helmfile.yaml.gotmpl destroy
```

```bash
make deploy-with-nestjs-api
make deploy-with-cpp-api
```

> When called without arguments, `make` automatically searches for `GNUmakefile`, `makefile`, and `Makefile` files in that order.

You can find more information about splitting a `Makefile` [here](https://www.gnu.org/software/make/manual/make.html#Include) and [here](https://www.gnu.org/software/make/manual/make.html#Makefile-Arguments).