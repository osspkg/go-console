# Console application

## Creating console application

```go
import "go.osspkg.com/console"

// creating an instance of the application, 
// specifying its name and description for flag: --help 
root := console.New("tool", "help tool")
// adding root command
root.RootCommand(...)
// adding one or more commands
root.AddCommand(...)
// launching the app
root.Exec()
```

## Creating a simple command

```go
import "go.osspkg.com/x/console"
// creating a new team with settings
console.NewCommand(func(setter console.CommandSetter) {
	// passing the command name and description
    setter.Setup("simple", "first-level command")
    // description of the usage example
    setter.Example("simple aa/bb/cc -a=hello -b=123 --cc=123.456 -e")
    // description of flags
    setter.Flag(func(f console.FlagsSetter) {
    	// you can specify the flag's name, default value, and information about the flag's value.
        f.StringVar("a", "demo", "this is a string argument")
        f.IntVar("b", 1, "this is a int64 argument")
        f.FloatVar("cc", 1e-5, "this is a float64 argument")
        f.Bool("d", "this is a bool argument")
    })
    // argument validation: specifies the number of arguments, 
    // and validation function that should return 
    // value after validation and validation error
    setter.ArgumentFunc(func(s []string) ([]string, error) {
        if !strings.Contains(s[0], "/") {
            return nil, fmt.Errorf("argument must contain /")
        }
        return strings.Split(s[0], "/"), nil
    })
    // command execution function
    // first argument is a slice of arguments from setter.Argument
    // all subsequent arguments must be in the same order and types as listed in setter.Flag
    setter.ExecFunc(func(args []string, a string, b int64, c float64, d bool) {
        fmt.Println(args, a, b, c, d)
    })
}),
```

### example of execution results

**go run main.go --help**

```text
NAME:
        tool - help tool
SYNOPSIS:
        tool   [arg]
COMMANDS:
        one   first level

```

**go run main.go simple --help**

```text
NAME
        tool - help tool
SYNOPSIS
        tool simple [arg]
DESCRIPTION
        first-level command
ARGUMENTS
        -a       this is a string argument (default: demo)
        -b       this is a int64 argument (default: 1)
        --cc     this is a float64 argument (default: 1e-05)
        -e       this is a bool argument (default: false)

```

## Creating multi-level command tree

To create a multi-level command tree,
you need to add the child command to the parent via the `AddCommand` method.

At the same time, in the parent command, it is enough to
specify only the name and description via the `Setup` method.

```go
root := console.New("tool", "help tool")

simpleCmd := console.NewCommand(func(setter console.CommandSetter) {
    setter.Setup("simple", "third level")
    ....
})

twoCmd := console.NewCommand(func(setter console.CommandSetter) {
    setter.Setup("two", "second level")
    setter.AddCommand(simpleCmd)
})

oneCmd := console.NewCommand(func(setter console.CommandSetter) {
    setter.Setup("one", "first level")
    setter.AddCommand(twoCmd)
})

root.AddCommand(oneCmd)
root.Exec()
```
