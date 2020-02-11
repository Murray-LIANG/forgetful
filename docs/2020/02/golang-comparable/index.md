# Golang Comparable `==`


## Golang中的类型
- 基本类型：整型（`int`/`uint`/`int8`/`uint8`/`int16`/`uint16`/`int32`/`uint32`/`int64`/`uint64`/`byte`/`rune`等）、浮点数（`float32`/`float64`）、复数类型（`complex64`/`complex128`）、字符串（`string`）。

- 复合类型（又叫聚合类型）：数组和结构体类型。

- 引用类型：切片（slice）、`map`、`channel`、指针。

- 接口类型：如`error`。

## `==`操作的两个操作数类型必须相同，否则编译报错。

## 复合类型：数组和结构体

**数组的长度为类型的一部分，长度不同意味着类型不同，不能比。**
- 数组：比每个元素，对于不同类型的元素再根据类型递归比下去。
- 结构体：比每个字段。


## 引用

比两个引用变量是否指向同一份数据，并不比较实际指向的数据内容。

**引用类型中的切片类型和`map`类型不可比较，只能和`nil`比较**

### 切片
```golang
a := [] interface {}{ 1 , 2.0 }
a[1] = a  // 切片间接指向自己
fmt.Println(a)

// runtime: goroutine stack exceeds 1000000000-byte limit
// fatal error: stack overflow
```

为什么切片不能比较？

可能因为：
1. 切片和数组类似，但是切片是引用，比较的是是否指向同一份数据，而这种比较方式与数组比较元素的方式完全不一样，容易让程序员混淆。
2. 如果将切片做成比较元素的方式，长度和容量是切片的隐藏字段，长度和容量不一样的切片会采用与数组一样的方式，被认为是类型不一样而编译错误。这样，在大部分情况下，切片的比较都会编译失败。
3. 即使忽略长度和容量不一样，切片元素可以是自身的引用（即切片本身），这样比较会导致堆栈溢出。

所以，golang不让你比较切片。

同理，`map`中的value也可能是不可比较的切片，所以也不让比较。

## 接口类型

接口类型的值，为接口值。每个接口值由两个部分组成：动态类型 - 具体类型和动态值 - 该类型的一个值。
```golang
var a interface{} = 1
```
`a`为接口类型，`a`的接口值包括：动态类型 - `int`，和动态值 - `1`。比较两个接口类型的变量，必须两个变量的动态类型和动态值都相同。

如果两个接口类型变量的动态类型是不可比的类型，则会在**运行时`panic`**。

值得注意的是：两个接口类型的变量的类型不一样，并不一定会报类型不一致的错。只要一个接口能转换为另一个接口，就可以比较。比如：
```golang
var f *os.File

var r io.Reader = f
var rc io.ReadCloser = f
fmt.Println(r == rc)  // true

var w io.Writer = f
// invalid operation: r == w (mismatched types io.Reader and io.Writer)
fmt.Println(r == w)
```
因为实现了`io.ReadCloser`接口的类型便实现了`io.Reader`接口，所以，可以比较这两个类型不一样的接口类型的两个变量。
```golang
type ReadCloser interface {
    Reader
    Closer
}
```

## 不可比性的传递

如果一个结构体由于含有切片字段不可比较，那么将它作为元素的数组不可比较，将它作为字段类型的结构体不可比较。

## map
`map`的`key`使用`==`来判断相等的，所以不可比的类型不能作为`map`的`key`。



