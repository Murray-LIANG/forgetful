# Golang init


## Initializing Packages on Import

```go
func init() {
    rand.Seed(time.Now().UnixNano())
}
```

Above `init` codes ensure the initialization are performed prior to the package being used.

## Multiple Instances of `init()`

The `init()` function can be declared multiple times throughout a package. 

In most cases, `init()` functions will execute in the order that you encounter them.

You can put `init()` in several files. They will be compiled in the alphabetical order of file names.

## Using `init()` for Side Effects

```go
// --snip--
import _ "image/png"

// --snip--
```
Because GO doesn't allow you to import packages that are not used throughout the program. This `import` line will only run the `init()` function of `image/png` package and discard the value of import itself. This is called importing for side effect.

