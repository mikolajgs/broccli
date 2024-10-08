# broccli

[![Go Reference](https://pkg.go.dev/badge/github.com/mikolajgs/broccli.svg)](https://pkg.go.dev/github.com/mikolajgs/broccli) [![Go Report Card](https://goreportcard.com/badge/github.com/mikolajgs/broccli)](https://goreportcard.com/report/github.com/mikolajgs/broccli) ![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/mikolajgs/broccli?sort=semver)

----

The `mikolajgs/broccli` package simplifies command line interface management. It allows you to define commands complete with arguments and flags, and attach handlers to them. The package handles all the parsing automatically.

----

### Install

Ensure you have your
[workspace directory](https://golang.org/doc/code.html#Workspaces) created and
run the following:

```
go get -u github.com/mikolajgs/broccli
```

### Example

Let's start with an example covering everything. First, let's create main
`CLI` instance and commands:

```
func main() {
    myCLI := broccli.NewCLI("Example", "App", "Author <a@example.com>")

    print := myCLI.AddCmd("print", "Prints out flags", func(c *CLI) int {
        fmt.Fprintf(os.Stdout, "Printing on the screen:\n%s\n%s\n\n", c.Flag("text1"), c.Arg("text2"))
        return 2
    })
    template := myCLI.AddCmd("template", "Process template", func(c *CLI) int {
        fmt.Fprintf(os.Stdout, "Do something here")
        return 0
    })
    start := myCLI.AddCmd("start", "Start the game", func(c *CLI) int {
        fmt.Fprintf(os.Stdout, "Do something here")
        return 0
    })
}
```

Next, let's add flags, args and required environments variables to our commands:

```
    print.AddFlag("text1", "a", "Text", "Text to print", TypeString, IsRequired)
    print.AddFlag("text2", "b", "Alphanum with dots", "Can have dots", TypeAlphanumeric, AllowDots)
    // If '-r' is passed, the '--text2'/'-b' flag is required
    print.AddFlag("make-text2-required", "r", "", "Make alphanumdots required", TypeBool, 0, OnTrue(func(c *Cmd) {
        c.flags["text2"].flags = c.flags["text2"].flags | IsRequired
    }))

    template.AddFlag("template", "t", "filepath", "Path to template file", TypePathFile, IsExistent|IsRequired)
    template.AddFlag("file-output", "o", "filepath", "Output to a specific file instead of stdout", TypePathFile, 0)
    template.AddFlag("number", "n", "int", "Number necessary for initialisation", TypeInt, IsRequired)

    start.AddFlag("verbose", "v", "", "Verbose mode", TypeBool, 0)
    start.AddFlag("username", "u", "username", "Username", TypeAlphanumeric, AllowDots|AllowUnderscore|IsRequired)
    start.AddFlag("threshold", "", "1.5", "Threshold, default 1.5", TypeFloat, 0)
    start.AddArg("input", "FILE", "Path to a file", TypePathFile, IRequired)
    start.AddArg("difficulty", "DIFFICULTY", "Level of difficulty (1-5), default 3", TypeInt, 0)
```

One of the arguments to AddFlag, AddArg or AddEnvVar is type of the value.  It can be one of the Type* consts, eg.
TypeInt, TypeBool, TypeString, TypePathFile etc.

Just next to the type, there is an argument that can contain additional validation flags, such as:

* IsRequired when flag/arg is required;
* AllowMultipleValues if flag/arg can have have multiple values, eg. 1,2,3, separated by comma or another character (there are flags for that such as SeparatorSemiColon etc.);
* IsExistent which will cause a flag/arg to be checked if it exists;
* IsRegularFile, IsDirectory, IsValidJSON and so on...

Check flags.go for more information on flag types.

And in the end of `main()` func:

```
    os.Exit(myCLI.Run())
```
