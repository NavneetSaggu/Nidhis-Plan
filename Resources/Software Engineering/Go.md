# Go

# Crash Course

## Basics

- Hello world looks like

```go
package main

import (
	"fmt"
	"math/rand"
	"math"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
	fmt.Println("It's way better than", math.Pi)
	fmt.Println("Anyways, Hello World")
}
```

- Execution begins from main package
- Imported packages are specified as shown
- Package name is the last element in the import statement
- `math` package does not include `math/rand` package
- There’s no semicolons
- All variables and functions have names, the ones which we want to be accessible from outside the package have to be exportable names
- All exportable names must start from capital letters, which is why you see things like

```go
rand.Intn(10) // function
fmt.Println.  // function
math.Pi       // variable
```

- Types come after in declaration in both variables and functions

```go
func add(x int, y int) int {
	return x + y
}
```

- Function can return any number of results, the bracket around return types is mandatory

```go
func swap(x, y string) (string, string) {
	return y, x
}
```

- You can also name the expected return variables, to make it easy to understand what the values are for

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

- var can be used to specify multiple variables in one go, at package or at function level

```go
var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```

- You can also have initializers as following

```go
var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```

- Inside functions,  `:=` can be used in place of `var`  when declaring new variables
Outside a function, every statement begins with a keyword (`var`, `func`, and so on) and so the `:=` construct is not available.

```go
func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"
	fmt.Println(i, j, k, c, python, java)
}
```

- The basic types are
    - bool
    - string
    - int  int8  int16  int32  int64
    - uint uint8 uint16 uint32 uint64 uintptr
    - byte // alias for uint8, it’s kinda like char?
    - rune // alias for int32, it represents a Unicode code point
    - float32 float64
    - complex64 complex128
- Variable declarations can be factored into blocks

```go
var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

- You have various data and meta data you can print about a variable
    - %T: Type
    - %v: Value
    - %t: Boolean
    - %d: Integer
    - %f: Floating point
- Variables declared without an explicit initial value are given their *zero value*. The zero value is:
    - `0` for numeric types,
    - `false` for the boolean type, and
    - `""` (the empty string) for strings

```go
func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}
```

- Unlike in C, in Go assignment between items of different type requires an explicit conversion.
- Type casting is done as follows. This code will break without typecasting

```go
func main() {
	var x, y int = 3, 4
	var f float64 = math.Sqrt(float64(x*x + y*y))
	var z uint = uint(f)
	fmt.Println(x, y, z)
}
```

- Constants use `const` keyword, no `var` over here
They cannot be declared using the `:=` syntax
They assume the type based on context

```go
const World = "世界"
```

- `const` s can also be declared inside blocks

```go
const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 100
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)
```

## For Loops

- Here’s what a for loop looks like

```go
  sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
```

- Here’s what a while loop looks like

```go
  sum := 1
	for sum < 1000 {
		sum += sum
	}
```

## If Statements

- Here’s what an if looks like, brackets may be skipped but not braces

```go
if x < 0 {
	return sqrt(-x) + "i"
}
```

- Like `for`, the `if` statement can start with a short statement that runs before `if` condition
Variables from here are accessible anywhere in the if block

```go
if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// can't use v here, though
```

## Switch

- switch in go looks very similar to other languages, except there’s no need for breaks

```go
switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
```

- You can also use them as substitute for long if else chains

```go
switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
```

## Defer

A defer statement defers the execution of a function until the surrounding function returns.
The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.
Deferred function calls are pushed onto a stack. These are evaluated in LIFO order when function returns.
The following code prints count in reverse.

```go
for i := 0; i < 10; i++ {
	defer fmt.Println(i)
}
```

## Pointers

Go has pointers. A pointer holds the memory address of a value.

The type `*T` is a pointer to a `T` value. Its zero value is `nil`.

```
var p *int
```

The `&` operator generates a pointer to its operand.

```
i := 42
p = &i
```

The `*` operator denotes the pointer's underlying value.

```
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```

This is known as "dereferencing" or "indirecting".

Unlike C, Go has no pointer arithmetic.

## Structs

- Collection of fields
- Constructors for these are called using {} to specify values
- dot `.` is used to access fields inside
- Pointers to structs are permitted
    - no need to do (*p).X or p→X
    - p.x is permitted

```go
type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	v.X = 4
	fmt.Println(v)
	fmt.Println(v.X)
	
	p := &v
	p.X = 1e9
	fmt.Println(v)
}
```

- They print a braced collection of values when printed

```go
var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
	fmt.Println(v1, v2, v3, p)
}
```

This prints

```go
{1 2} {1 0} {0 0} &{1 2}
```

## Arrays

- The type `[n]T` is an array of `n` values of type `T`
- This expression declares a variable `a` as an array of ten integers.

```
var a [10]int
```

- An array's length is part of its type, so arrays cannot be resized.

## Slice

- An array has a fixed size. A slice, on the other hand, is a dynamically-sized, flexible view into the elements of an array. In practice, slices are much more common than arrays.
- The type `[]T` is a slice with elements of type `T`.
- A slice is formed by specifying two indices, a low and high bound, separated by a colon:

```
a[low : high]
```

- This selects a half-open range which includes the first element, but excludes the last one.
- The following expression creates a slice which includes elements 1 through 3 of `a`:

```
a[1:4]
```

- Check this for more syntax

```go
func main() {
	primes := [6]int{2, 3, 5, 7, 11, 13} // [n]T so an array

	var s []int = primes[1:4]            // []T so a slice
	fmt.Println(s)
}
```

- A slice does not store any data, it just describes a section of an underlying array.
- Changing the elements of a slice modifies the corresponding elements of its underlying array.
- Other slices that share the same underlying array will see those changes.
- A slice literal is like an array literal without the length.
- This is an array literal:

```
[3]bool{true, true, false}
```

- And this creates the same array as above, then builds a slice that references it:

```
[]bool{true, true, false}
```

- Slice of structs

```go
func main() {
	q := []int{2, 3, 5, 7, 11, 13}
	fmt.Println(q)

	r := []bool{true, false, true, true, false, true}
	fmt.Println(r)

	s := []struct {
		i int
		b bool
	}{
		{2, true},
		{3, false},
		{5, true},
		{7, true},
		{11, false},
		{13, true},
	}
	fmt.Println(s)
}
```

### Length And Capacity

- A slice has both a length and a capacity.
- The length of a slice is the number of elements it contains.
- The capacity of a slice is the number of elements in the underlying array, counting from the first element in the slice.
- The length and capacity of a slice s can be obtained using the expressions len(s) and cap(s).
- You can extend a slice's length by re-slicing it, provided it has sufficient capacity. Try changing one of the slice operations in the example program to extend it beyond its capacity and see what happens.

```go
func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s)

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s)

	// Extend its length.
	s = s[:4]
	printSlice(s)

	// Drop its first two values.
	s = s[2:]
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

- If you reslice outside of capacity, you’ll get slice bounds out of range

### Zero Value

- The zero value of a slice is `nil`.
- A nil slice has a length and capacity of 0 and has no underlying array.

```go
func main() {
	var s []int
	fmt.Println(s, len(s), cap(s))
	if s == nil {
		fmt.Println("nil!")
	}
}
```

### Slice of Slice

```go
	// Create a tic-tac-toe board.
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// The players take turns.
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"

	for i := 0; i < len(board); i++ {
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}
```

### Appending To Slice

- Pass the slice, and the values to append

```go
func append(s []T, vs ...T) []T
```

- If the underlying array of `s` is too small, a bigger array will be allocated. 
The returned slice will point to the newly allocated array.
- This is like vectors

### Range Iteration

Can be done over map or slice, returns index and value

```go
func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}
```

You can skip the index or value by assigning to `_`.

```
for i, _ := range pow
for _, value := range pow
```

# Best Practices

## Logging And Stack Trace

Use debug stack to log the entire stack trace in case of a fatal error

```sql
string(debug.Stack())
```