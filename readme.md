# Packages and Modules


# Variables
## Declaration and Definition
Variables can be declared with 
```go
var name type
// or 
var name [type] = "value"
// or
name := "value"
```
With the first option, you declare but don't define and therefore have to specify a type. If using the var keyword, you can define and declare a variable as expected, with or without type declaration.
With the third option, you are implicitly defining a variable, so you must use the walrus operator. This is also only available within functions.

To redefine a variable, just use the `=` operator.

Variables declared but not defined are given a zero value. This is 0 for numerics, false for booleans, and "" for strings.

There are also constants that can be defined. 
```go
const name type = "value"
```
These can be of any type, but cannot be declared using the walrus operator.

Numeric constants are high precision. This means for something like an int, you can store more than what you would be able to with a generic int. These values can't be used in memory or they may cause an overflow/underflow which panics.

## Types
Some types available
```go
string
bool
int
int{8,16,32,64}
uint
uint{8,16,32,64,ptr}
byte // alias for uint8
rune // alias for int32
float32, float64
complex64, complex128 // more functionality with math/cmplx
```

## Conversion
You can convert to a different type by using that type as a function. This must be done, or Go will panic. 
```go
var i int = 20
var u uint = uint(i)
```

## Pointers
```go
// p is a pointer, zerod to nil
var p *int
p == nil // true

i := 42
p = &i // p now points to i

fmt.Println(*p) // read i through pointer p :: 42
*p = 21 // set i through pointer p :: dereference
```

## Structs
```go
type Vertex struct {
	// these can be lowercase and will still be exposed
	// unlike functions in a package
	X int
	Y int
}

v := Vertex{1, 2}
v.X = 4

p := &v
v.X = 5 // no dereferencing needed for struct pointers

//specifically define struct fields
v2 := Vertex{Y: 1} // X is implicitly 0
v3 := &Vertex{} // pointer to vertex where X and Y are 0
```

## Arrays
```go
// template
var a [num]type
// ex
var nums [10]int
```
Arrays cannot be resized

## Slices
A slice is dynamically sized view into an array, the zero value is `nil`
```go
a[low : high]
// as usual indices start at low and don't include high
nums[1 : 10]

// slice literal
// creates an array then references it
[]bool{true, true, false}
```
Changes to a slice changes the array it references

Slicing indices can be omitted for 0 on the low end and the length of the slice on the high end
```go
var a [10]int
// all of the following are equivalent
a[0:10]
a[:10]
a[0:]
a[:]
```

Slices have both lengths and capacities where
- length is the number of elements in the slice
- capacity is the number of elements in the array the slice references
```go
a := []int{1, 2, 3, 4, 5}
s[1:3] // [2, 3]
len(s) // 2
cap(s) // 5
```

It is also possible to re-slice a slice, the indices used are in reference to the original array.

A slice can also be created with the `make` function
```go
a := make([]int, 5) // len = 5, cap = 5
b := make([]int, 5, 6) // len = 5, cap = 6
```

Slices can be nested
```go
a := [][]int{
	[]int{1, 2, 3},
	[]int{4, 5, 6},
}
```

Appending to slices
```go
a := []int{1, 2, 3}
a = append(a, 4, 5, 6) // returns, does not append in place
```

## Maps
Maps are hashmaps with key/value pairs of a specified type, the zero value is `nil`
Maps can be made very similarly to slices
```go
var a map[key]value
var a_ex map[string]int

a = make(map[string]int)

// map literal
a := map[string]int{
	"key": 1337,
}

type Vertex struct {
	Lat, Long float64
}

// map literal with complex types
b := map[string]Vertex{
	"Somewhere": Vertex{
		40.3, 30.6
	},
}
// or
b := map[string]Vertex{
	"Somwehere": {40.3, 30.6},
}
```

Making updates to the map
```go
m := make(map[string]int)

m['answer'] = 42
m['answer'] = 9
// deleting reverts to a zero value
delete(m, 'answer') // -> m['answer'] is 0

// v -> value, zero value if ok is false
// ok -> if it exists in the map
// if both v and ok are already defined, omit :
v, ok := m['answer'] 
```

## Interfaces
Interfaces are a set of method signatures and can be any type that implements those methods.
Unlike methods, interfaces require the correct typing, so calling a method on a pointer vs value can and most likely will differ if that method is not implemented for both the pointer and value.
```go
type InterfaceName interface {
	MethodName() type // type is the return type
}
// any type that implements MethodName can be held by the interface
```
Interfaces are implemented implicitly, if a type implements the methods that are scoped by the interface, that type implements the interface.

Under the hood, interfaces are a tuple of the value and its concrete type, so calling a method on an interface finds the method associated with that specific type. This can be seen with the describe function.
```go
type I interface {
	M()
}
type T struct {
	S string
}
func (t *T) M() {
	fmt.Println(t.S)
}
var i I
i = &T{"hello"}
describe(i) // -> (&{"Hello"}, *package.T)

// interfaces can also hold nil values
// these cases must be handled in the methods
// or the method will panic
var t *T
i = t
describe(i) // -> (<nil>, *package.T)

// interfaces can also be nil values
var i I
describe(i) // -> (<nil>, <nil>)
// calling a method on a nil interface will panic
```

### Empty Interface
There is also something called the empty interface which every type implements. For example the Println function takes in any number of arguments of the empty interface type.
```go
var i interface{}
describe(i) // -> (<nil>, <nil>)
i = 42
i = "hello"
// you can set i to be any type
```

You can also assert that an interface uses a specific type
```go
v := i.(T) // v is value, i is interface, T is type

var i interface{} = "hello"
s := i.(string) // s becomes "hello"
s := i.(float64) // panics

// you can also do this with an error check to prevent panics
// very similar to map syntax
s, ok := i.(string) // s->"hello" ok->true
s, ok := i.(float64) // s->0 ok->false
```

### Type Switching
you can use a switch to perform different logic on an interface depending on its type
```go
switch v := i.(type) {
case T:
	... //v has type T
case S:
	... //v has type S
default:
	... //v has type i
}
```

### Stringer
There is an interface within the `fmt` package which defines a type that can be printed as a string. To add a type to the interface, define a `String()` method.
```go
type Person struct {
	Name string
	Age  int
}
func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}
a := Person{"Sam", 21}
fmt.Println(a) // -> Sam (21 years)
```


# Functions
Defining functions
```go
func funcname(param1 type, param2 type) type type {
	// functions can have multiple parameters and returns
	// the returns return in a tuple, typically used for errors
}
```

Functions are considered values in Go. Go functions can also be closures.
```go
// functions like these are bound to variables
// for example, each return from adder has its own sum var
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		// returns a variable defined outside its body
		return sum
	}
}
```

## Methods
No classes in Go, but you can still define methods on types
```go
type Vertex struct {
	X, Y float64
}

// state the type and the name you want to give it
// in parens before the name of the variable
func (v Vertex) Abs() float {
	return math.sqrt(v.X*v.X + v.Y*v.Y)
}

v := Vertex{3, 4}
absoluteValue := v.Abs()

// you can also use pointer receivers
func (v *Vertex) Scale(f float64) {
	// these methods can modify the pointed to object
	v.X = v.X * f
	v.Y = v.Y * f
}
// you can still call this method without referencing
v.Scale(10)
// the same is true with pointers and dereferencing
p := &v
p.Abs() // compiles and returns

// this works on non struct types as well
// but you need to define a local type
type NewFloat float64
func (f NewFloat) MethodName float64 {
	...
}
f := NewFloat(5.0).MethodName()
```

# Conditionals
If statements
```go
if condition {
	// some block
} else {
	
}

// can use this dec variable in any block in conditional
if dec := val; condition {
	return dec
} else {
	return dec
}
```

Switch statements
```go
// like with if statements we can declare a variable
// that is only accessible within the conditional
switch v := val; v {
	case "case":
		...
		// no need for a break here, go does this automatically
	case "another":
		...
	// you can also use variables in switch statements
	case someVariable:
		...
	// default is also optional, will just skip over all blocks
	default ""
}
```
These are evaluated from top to bottom until one matches, then it stops checking.

You can also write blank switch statements, which function as switch true
```go
switch {
	case false:
		// doesn't get called
	case true:
		// does get called
}
```

# Printing and Formatting
To gain access to printing functions included in the standard library, you import "fmt"
```go
import (
	"fmt"
)
...
fmt.Printf("Hello, world")
```

# Loops
There is only a for loop in go
```go
for i := 0 i < 10 i++ {
	...
}
```

To simulate a while loop, go makes the init and post statements optional
```go
for ; i < 10 ; {
	... // i must already be declared
}
// you can also omit the semicolons
for i < 10 {
	...
}
```

A loop forever can be even further simplified
```go
for {
	...
}
```

## Range in Loops
```go
a := []int{1, 2, 3, 4, 5}
for i, v := range a {
	// range operates on slices or maps
	// it returns the index and value (or key and value)
	...
}

// it's possible to omit either variable
for _, v := range a 
for i, _ := range a
// or just use the index
for i := range a 

```

# Async
You can defer statements to be called at the end of the current function. 
```go
func main() {
	defer fmt.Println("world")
	fmt.Println("hello")
}
```
this will print `hello( \n )world`. Deferred statements will print in the reverse order they were called, so defer statements at the bottom of the function will be called before those stated at the top of the function.
Basically a stack using LIFO
