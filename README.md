mood Package: Quick Start Guide
===============================

The `mood` package is a minimal framework for building command-line
interfaces (CLI) in Go. It handles argument parsing, command
dispatching, and provides a simple execution context.

Installation
------------

Install the `mood` package using the standard Go command:

```bash
go get github.com/phillip-england/mood
```

Basic Usage: Running a Default Command
--------------------------------------

Every `mood` application requires a default command to run when no other
subcommand is specified. Start by defining the `Cmd` interface and the
factory function.

The Command Interface (Cmd)
---------------------------

All commands must implement the `Cmd` interface:

```go
type Cmd interface {
  Execute(cli *Cli) error
}
```

Example: main.go
----------------

This example shows how to set up the default command and run the
application.

```go
package main

import (
    "fmt"
    "os"
    "github.com/phillip-england/mood"
)

// 1. Define the Default Command struct
type HelloCmd struct {}

// 2. Implement the Execute method
func (c *HelloCmd) Execute(cli *mood.Cli) error {
    // Access the command source (the executable name)
    fmt.Printf("Welcome to the %s CLI!\n", cli.Source)
    // Access the current working directory
    fmt.Printf("Current Directory: %s\n", cli.Cwd)

    // The Run method handles errors automatically
    return nil
}

// 3. Define the factory function
func HelloFactory(cli *mood.Cli) (mood.Cmd, error) {
    return &HelloCmd{}, nil
}

func main() {
    // Initialize the CLI with the default command factory
    cli, err := mood.New(HelloFactory)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Error initializing CLI: %v\n", err)
        os.Exit(1)
    }

    // Run the application
    if err := cli.Run(); err != nil {
        fmt.Fprintf(os.Stderr, "Execution Error: %v\n", err)
        os.Exit(1)
    }
}
```

Registering Subcommands
-----------------------

Use the `Cli.At` method to register commands that execute when a
specific first argument is provided (e.g., `app run` or `app config`).

Registering Commands Example
----------------------------

```go
// Inside your main function, after mood.New:
func main() {
  cli, err := mood.New(HelloFactory)
  if err != nil { /* error handling */ }

  // Register a new command named "run"
  cli.At("run", RunFactory) 

  // Register another command named "config"
  cli.At("config", ConfigFactory) 

  if err := cli.Run(); err != nil { /* error handling */ }
}
```

Accessing Arguments and Flags
-----------------------------

The `Cli` object provides methods to check for and retrieve arguments
and flags.

-   `FlagExists(flag string) bool`: Check for presence (e.g.,
    `cli.FlagExists("-v")`)
-   `ArgGetByPosition(position int) (string, bool)`: Get a non-flag
    argument by its index (1 for the first positional arg after the
    command, if any).
-   `ArgGetOrDefaultValue(arg string, defaultValue string) string`: Get
    an argument by its value string, or use a fallback.

Example: Getting the second positional argument (e.g., in
`app run VALUE`):

```go
// In the Execute method of the "run" command:
if val, exists := cli.ArgGetByPosition(2); exists {
  // val contains the VALUE string
}
```

Documentation generated for the `mood` package.
