# Chapter 7 - Data types

In this chapter we're going to review Go's built-in data types, often referred to as *primitives*.

Data types define what a value can be and are fixed at compile time. In Go conversions between types must be performed explicitly. 

We can split Go data types into several categories:- basic, aggregate, reference and interface, although the classifications are not strictly mutually exclusive, as we will see.

> The term *reference type* has been removed from official Go terminology, and instead aggregate types and reference types are often referred to collectively as *composite types*. We're going use the term anyway, and we'll see why soon.

## 7.1 Basic types

Basic types include types for numbers, strings and booleans.

### 7.1.1 Number

There are multiple ways to represent numbers depending on how big that number is, whether it is positive or negative and whether it's a whole number or a decimal.

Unless your program code needs to be optimised for memory usage, choose *int* for whole numbers and *float64* for decimals. Both types use 64 bits or 8 bytes in memory.

Don't use specific integer types such as *int8*, *int16*, *int32* or *int64* unless forced to by an external dependency which accepts or returns these types specifically.

Similarly, if you **do** choose to store decimals as *float32* in your program, to perform operations on these values, for example using the standard library [math package](https://pkg.go.dev/math), you'll need to convert them to *float64*, since the package accepts and returns *float64* types exclusively. 

Performing type conversions such as the above may complicate your syntax and in many cases, this trade-off won't be worth the memory saved by using types that occupy less memory. 

If memory constraints do become a concern, you can revisit the types used by your program of course. The point is this is a performance optimisation, which should come later. 

> Did you know *int* is actually an alias for *int32* or *int64*. Which, depends on whether your code is compiled for 32-bit or 64-bit architecture. No such alias exists for *float* although *float* was once a valid type but was removed from the language in 2011 before Go version 1.0 was released.

Go also supports unsigned integer types, where the value must be positive. For reasons already stated should you wish to use an unsigned integer use *uint* rather than a more specific type.

In addition to *int*, *uint* and *float64* types there are *complex* number types available in Go, but we're not going to cover those.

Variables of any number type are initialised to zero.

Before leaving number types, a note about working with floating point numbers. Below is a *Gotcha* relating to floating point number precision. 

**Example 23 - Floating point number addition problem**
```go
package main

import (
	"fmt"
)

func main() {

	a := 2.1
	b := 4.2

	c := a + b
	d := 2.1 + 4.2
	e := float64(2.1) + float64(4.2)

	fmt.Println(c == d)
	fmt.Println(c == e)
}

// Go Playground: https://go.dev/play/p/_uoAHQeoh9h
```

Output:
```
false
true

Program exited.
```

Looking at the simple additions in the example, we might expect both comparisons to be *true* but they are not. This is due to floating point number rounding precision. 

A real-world scenario in which we could encounter problems caused by different precision is in unit testing. An assertion that should pass, fails because the expected result is ever so slightly different than the result obtained. 

The takeaway is that comparisons between floating point numbers can be unreliable where their precision is different, so if you wish to add decimal numbers which are not already *float64* type it is worth performing that conversion explicitly.  

It's also worth ensuring the same level of precision when making assertion comparisons in your unit tests.

### 7.1.2 Boolean

The type *bool* stores true and false values and it uses one byte in memory. A variable of type *bool* is initialised to its *nil* value which is false.

### 7.1.3 String

Though a basic type, a *string* actually points to a backing slice of bytes. We cover the *byte* type next but think of a slice as an array for the moment, we'll also cover them soon.

To demonstrate, in the example below we print a portion of the string from index position 6 onwards by accessing the underlying byte slice:

**Example 24 - String cut using byte slice**
```go
package main

import "fmt"

func main() {
	myString := "Hello GolangAtSpeed"
	fmt.Println(myString[6:])
}

// Go Playground: https://go.dev/play/p/egXJA0gm97x
```

> In the example above we make a common mistake. The code could result in unexpected or buggy behaviour depending on the string literal we operate on. We'll point out the gotcha shortly when we get to the *rune* type.

Variables of type *string* have a value of an empty string at declaration.

### 7.1.4 Byte

A *byte* is an alias for *uint8*. It stores integers between 0 and 255 (the largest number we can represent in binary with 8 bits).  Therefore a *byte* can be used to represent any single ASCII character. 

As an alias for *uint8*, a variable of type *byte* is initialised to zero.

### 7.1.5 Rune

We'll start this section with a classic gotcha!  Check out the example program below in which we have two 'identical' strings but which appear to be of different length:

**Example 25 - Identical strings with different lengths**
```go
package main

import (
	"fmt"
)

func main() {
	greet1:= "hello"
	greet2:= "һello"

	fmt.Println(len(greet1))
	fmt.Println(len(greet2))
}

// Go Playground: https://go.dev/play/p/ujUnmx-LsWu
```

Don't worry, we'll explain what's happening here in our discussion of the *rune* type which follows.

To represent other characters beyond ASCII alone we need to use a *rune*.

A *rune* is an alias for *int32* which can represent a Unicode codepoint as a numerical value. As it's an *int32* it is large enough to represent any Unicode character in a sequence of almost 150,000 at the time of writing. 

ASCII is a subset of Unicode so ASCII character codes map directly to the same character codepoint in Unicode. 

A Unicode codepoint also has a UTF-8 hexadecimal representation.  Go will present this hex as a sequence of up to 4 bytes, how many bytes, depends on the UTF-8 hexadecimal code itself.

Let's illustrate with an example. In the program below we've a string which contains a single character - which happens to be our favorite Unicode character - the rocket. You can [inspect the character and its data here](https://codepoints.net/U+1F680).

**Example 26 - Unicode codepoint with UTF-8 code**
```go
package main

import (
	"encoding/hex"
	"fmt"
)

func main() {
	myString := "🚀"
	fmt.Println("Unicode codepoint represented by rune:", []rune(myString))
	fmt.Println("UTF-8 code represented by up to 4 bytes:", []byte(myString))
	fmt.Println("UTF-8 code represented as Hexadecimal:", hex.EncodeToString([]byte(myString)))
	fmt.Println("length", len(myString))
}

// Go Playground: https://go.dev/play/p/luDIj6DwPAG
```

Output:
```
Unicode codepoint represented by rune: [128640]
UTF-8 code represented by up to 4 bytes: [240 159 154 128]
UTF-8 code represented as Hexadecimal: f09f9a80
length 4

Program exited.
```

The rocket is a Unicode character encoded in the UTF-8 format as 4 bytes, so Go reports the length as 4, not 1.

Looking back at Example 25, we called the built-in *len()* function to print the length of our two "hello" strings. However, *len()* operates on the underlying byte slice, not on what we think of as the string length.

For most use cases if you only work with ASCII characters things will be fine and the results *len()* returns will be what you expect. 

> The mistake we often make as developers when working with strings is assuming that all characters are represented as single bytes and that *len()* returns a character count.

But if you need to work with Unicode, and get a "character" count, for example, from user input, you should first convert the string to a slice of runes.

**Example 27 - Getting the safe string length**
```go
package main

import "fmt"

func main() {
	greet1:= "hello"
	greet2:= "һello"
	rocket := "🚀"

	fmt.Println("greet1 length:", len([]rune(greet1)))
	fmt.Println("greet2 length:", len([]rune(greet2)))
	fmt.Println("rocket length:", len([]rune(rocket)))
}

// Go Playground: https://go.dev/play/p/K8m2fGOmgZt
```

Output:
```
greet1 length: 5
greet2 length: 5  
rocket length: 1

Program exited.
```

Variables of type *rune* are initialised to zero.

> Looking back at Example 25, the reason the string lengths are different is that the first character in the second string `greet2` is a Cyrillic character which requires 2 bytes to represent, whereas the ASCII equivalent in `greet1` requires only 1 byte.

## 7.2 Aggregate types

Aggregate types can contain aggregates of other data types. For example, the basic types we just reviewed, but also nested aggregate, reference, and interface types. 

Go's aggregate types are arrays and structs.

### 7.2.1 Array

An array is a set number of elements all of the same type. Once created, an array cannot be resized, its length is fixed and indeed the length forms part of the type itself.

All elements of an array are initialised to the nil value of the type.

**Example 28 - Array length, capacity and element initialisation**
```go
package main

import "fmt"

func main() {
	var myArray [5]int
	fmt.Println("Length:", len(myArray))
	fmt.Println("Capacity:", cap(myArray))
	fmt.Println("Contents:", myArray)
}

// Go Playground: https://go.dev/play/p/aPOdPxoQDy7
```

Output:
```
Length: 5
Capacity: 5
Contents: [0 0 0 0 0]

Program exited.
```

Array capacity is always equal to the array length, as arrays cannot be resized.

Arrays use zero-bound indexing, so the first element is at index position 0, the second at position 1, and so on.

Since the length of the array is part of its type, two arrays of different lengths are different types and cannot be assigned to one another, even if the element types are the same.

**Example 29 - Array length is part of type definition**
```go
package main

import "fmt"

func main() {
	var myArray1 [5]int
	var myArray2 [6]int
	
	myArray1 = myArray2 // This will not compile
	fmt.Println(myArray1)
}

// Go Playground: https://go.dev/play/p/xGVNv3xGLpU
```

Output:
```
./prog.go:9:13: cannot use myArray2 (variable of type [6]int) as type [5]int in assignment

Go build failed.
```

We can initialise an array using array literal syntax. If we want to let the compiler determine the array length from the number of elements provided, we can use the spread operator `...` in the type definition.

**Example 30 - Array initialisation**
```go
package main

import "fmt"

func main() {
	myArray1 := [5]int{1, 2, 3, 4, 5}
	myArray2 := [...]int{1, 2, 3, 4, 5}
	
	fmt.Println("myArray1:", myArray1)
	fmt.Println("myArray2:", myArray2)
	fmt.Printf("myArray1 type: %T\n", myArray1)
	fmt.Printf("myArray2 type: %T\n", myArray2)
}

// Go Playground: https://go.dev/play/p/N9Q2JpPbzCL
```

Arrays are rarely used in Go. As we will see, slices are generally preferred for their flexibility. Arrays may provide a small memory efficiency gain if you know the number of elements your code will work with and that number is fixed, but in practice, most Go code uses slices.

### 7.2.2 Struct

A struct holds collections of typed fields and provides a useful way of representing collections of data.

There are two approaches to creating struct variables. We can either create them as anonymous structs, where the properties of the struct are defined at the point we use them, or we can use the struct type as the basis for creating a named custom type.

Anonymous structs are useful for temporary data structures, whereas custom types are useful when the struct will be used in multiple places.

All fields of a struct are initialised to the nil value of their type, unless we explicitly provide values.

**Example 31 - Named custom struct type and anonymous struct**
```go
package main

import "fmt"

type User struct {
	Name string
	Age  int
}

func main() {
	// Named custom struct type
	user1 := User{Name: "Alice", Age: 30}
	
	// Anonymous struct
	user2 := struct {
		Name string
		Age  int
	}{Name: "Bob", Age: 25}
	
	fmt.Println("user1:", user1)
	fmt.Println("user2:", user2)
	
	// Anonymous structs can be assigned to named structs
	// if they have matching fields
	var user3 User
	user3 = user2 // This works
	fmt.Println("user3:", user3)
}

// Go Playground: https://go.dev/play/p/abc123def456
```

Note that whilst we can assign an anonymous struct to a custom struct type if the fields match, we cannot assign custom struct types to each other, even if they have identical fields. Each custom type has its own type identity.

Struct values can be compared using simple operations (`==`, `!=`) if they are the same type (or anonymous with matching fields) and all their field values are equal.

When creating struct values, we can omit the field names if we provide values for all fields in the order they are defined.

**Example 32 - Unnamed struct fields**
```go
package main

import "fmt"

type User struct {
	Name string
	Age  int
}

func main() {
	user1 := User{"Alice", 30}     // Field names omitted
	user2 := User{Name: "Bob"}     // Partial initialization
	
	fmt.Println("user1:", user1)
	fmt.Println("user2:", user2)
}

// Go Playground: https://go.dev/play/p/xyz789abc123
```

**Example 33 - Get and set struct fields**
```go
package main

import "fmt"

type User struct {
	Name string
	Age  int
}

func main() {
	user := User{Name: "Alice", Age: 30}
	
	// Get field values
	fmt.Println("Name:", user.Name)
	fmt.Println("Age:", user.Age)
	
	// Set field values
	user.Name = "Alice Smith"
	user.Age = 31
	
	fmt.Println("Updated user:", user)
}

// Go Playground: https://go.dev/play/p/def456ghi789
```

#### 7.2.2.1 Composition and Embedding

Structs can be embedded inside other structs. We call this *composition*. It's similar to inheritance in object-oriented languages, but more flexible.

Unlike many object-oriented languages, Go allows multiple inheritance - you can embed as many structs as you like inside another struct.

When we embed a struct, Go provides field promotion and receiver promotion, which means we can access the embedded struct's fields and call its receivers directly on the containing struct.

This promotes more code reusability than traditional OOP inheritance patterns.

**Example 34 - Composition and struct embedding**
```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

type Employee struct {
	Person        // Embedded struct
	Company string
	Salary  int
}

func main() {
	emp := Employee{
		Person:  Person{Name: "Alice", Age: 30},
		Company: "TechCorp",
		Salary:  50000,
	}
	
	// Access embedded fields directly
	fmt.Println("Name:", emp.Name)     // Promoted field
	fmt.Println("Age:", emp.Age)       // Promoted field
	
	// Or access via long form
	fmt.Println("Name:", emp.Person.Name)
	fmt.Println("Age:", emp.Person.Age)
	
	fmt.Println("Company:", emp.Company)
	fmt.Println("Salary:", emp.Salary)
}

// Go Playground: https://go.dev/play/p/ghi789jkl012
```

When we have embedded fields, we can access them directly (short form) or via the embedded struct name (long form). However, if we have named fields that conflict with embedded field names, we can only access the embedded fields via the long form.

#### 7.2.2.2 Memory Use, Alignment & Padding

If your struct has many fields, it's worth considering memory usage. On 64-bit architecture, data is stored in 8-byte slots in memory, and the ordering of struct fields can affect how much memory is used due to padding.

Suboptimal field ordering can significantly increase memory usage.

**Example 35 - Alignment and impact on memory**
```go
package main

import (
	"fmt"
	"unsafe"
)

type SuboptimalStruct struct {
	A bool    // 1 byte + 7 bytes padding
	B int64   // 8 bytes
	C bool    // 1 byte + 7 bytes padding  
	D int64   // 8 bytes
	E bool    // 1 byte + 7 bytes padding
}

type OptimalStruct struct {
	B int64   // 8 bytes
	D int64   // 8 bytes
	A bool    // 1 byte
	C bool    // 1 byte
	E bool    // 1 byte + 5 bytes padding
}

func main() {
	fmt.Printf("Suboptimal struct size: %d bytes\n", unsafe.Sizeof(SuboptimalStruct{}))
	fmt.Printf("Optimal struct size: %d bytes\n", unsafe.Sizeof(OptimalStruct{}))
	
	reduction := float64(unsafe.Sizeof(SuboptimalStruct{}) - unsafe.Sizeof(OptimalStruct{}))
	percentage := (reduction / float64(unsafe.Sizeof(SuboptimalStruct{}))) * 100
	fmt.Printf("Memory reduction: %.1f%%\n", percentage)
}

// Go Playground: https://go.dev/play/p/jkl012mno345
```

Output:
```
Suboptimal struct size: 40 bytes
Optimal struct size: 24 bytes
Memory reduction: 40.0%

Program exited.
```

As you can see, thoughtful field ordering can result in significant memory savings. This should be considered for structs with many fields in performance-critical scenarios.

## 7.3 Reference types

Reference types include maps, slices, and channels. We call them "reference types" because they behave as if they were pointers - they point to some backing data automatically, without us having to explicitly use pointer syntax.

### 7.3.1 Slice

The slice is the most versatile and commonly used Go data type. A slice points to an underlying backing array, which is managed by Go.

A slice has both a length and a capacity, and these two values can differ, unlike arrays where they are always the same.

> Length is the number of initialized elements in the slice. Capacity is the total number of elements available in the backing array.

**Example 36 - Creating slices**
```go
package main

import "fmt"

func main() {
	// Five ways to create slices
	var slice1 []int                    // nil slice
	slice2 := []int{}                   // empty slice
	slice3 := []int{1, 2, 3}           // slice literal
	slice4 := make([]int, 3)           // length 3, capacity 3
	slice5 := make([]int, 3, 5)        // length 3, capacity 5
	
	fmt.Printf("slice1: %v, len: %d, cap: %d, nil: %t\n", slice1, len(slice1), cap(slice1), slice1 == nil)
	fmt.Printf("slice2: %v, len: %d, cap: %d, nil: %t\n", slice2, len(slice2), cap(slice2), slice2 == nil)
	fmt.Printf("slice3: %v, len: %d, cap: %d, nil: %t\n", slice3, len(slice3), cap(slice3), slice3 == nil)
	fmt.Printf("slice4: %v, len: %d, cap: %d, nil: %t\n", slice4, len(slice4), cap(slice4), slice4 == nil)
	fmt.Printf("slice5: %v, len: %d, cap: %d, nil: %t\n", slice5, len(slice5), cap(slice5), slice5 == nil)
}

// Go Playground: https://go.dev/play/p/mno345pqr678
```

Note that only a nil slice equals `nil`. An empty slice with length 0 is not equal to `nil`.

Slices have a slice header that contains a pointer to the backing array, the length, and the capacity. This header can change in two situations:

1. When we add elements that exceed the current capacity (Go creates a new backing array)
2. When we add elements within capacity (Go updates only the length)

This leads to important behavior rules for reference types:
- If an operation doesn't modify the slice header → the slice behaves as a reference type
- If an operation modifies the slice header → Go creates a new value

**Example 37 - Slice not behaving as reference type**
```go
package main

import "fmt"

func appendToSlice(s []int) {
	s = append(s, 4) // This may create a new backing array
	fmt.Println("Inside function:", s)
}

func main() {
	slice := []int{1, 2, 3}
	fmt.Println("Before function:", slice)
	appendToSlice(slice)
	fmt.Println("After function:", slice)
}

// Go Playground: https://go.dev/play/p/pqr678stu901
```

Output:
```
Before function: [1 2 3]
Inside function: [1 2 3 4]
After function: [1 2 3]

Program exited.
```

To safely return a modified slice, we should return the new slice to the caller:

**Example 38 - Safely return new slice to caller**
```go
package main

import "fmt"

func appendToSlice(s []int) []int {
	s = append(s, 4)
	return s
}

func main() {
	slice := []int{1, 2, 3}
	fmt.Println("Before function:", slice)
	slice = appendToSlice(slice)
	fmt.Println("After function:", slice)
}

// Go Playground: https://go.dev/play/p/stu901vwx234
```

However, if we modify existing elements (without changing the slice header), the slice behaves as a reference type:

**Example 39 - Slice behaving as reference type**
```go
package main

import "fmt"

func modifySlice(s []int) {
	s[0] = 999 // Modifying existing element
	fmt.Println("Inside function:", s)
}

func main() {
	slice := []int{1, 2, 3}
	fmt.Println("Before function:", slice)
	modifySlice(slice)
	fmt.Println("After function:", slice)
}

// Go Playground: https://go.dev/play/p/vwx234yza567
```

Output:
```
Before function: [1 2 3]
Inside function: [999 2 3]
After function: [999 2 3]

Program exited.
```

> Best practice is to treat slices as values that are passed to and returned from functions, rather than trying to rely on reference type behavior. This makes your code clearer and less prone to bugs.

#### 7.3.1.1 Working with Slices

We can access slice elements by index, just like arrays. We can also reslice using the `:` operator to get subsets of a slice.

**Example 40 - Reslicing & working with slices**
```go
package main

import "fmt"

func main() {
	slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	
	fmt.Println("Full slice:", slice)
	fmt.Println("slice[2:5]:", slice[2:5])   // Elements 2, 3, 4
	fmt.Println("slice[:3]:", slice[:3])     // Elements 0, 1, 2
	fmt.Println("slice[7:]:", slice[7:])     // Elements 7, 8, 9
	fmt.Println("slice[:]:", slice[:])       // All elements
}

// Go Playground: https://go.dev/play/p/yza567bcd890
```

**Example 41 - Slices of slices**
```go
package main

import "fmt"

func main() {
	slice1 := []int{1, 2, 3, 4, 5}
	slice2 := slice1[1:4] // Elements 2, 3, 4
	
	fmt.Println("slice1:", slice1)
	fmt.Println("slice2:", slice2)
	
	// Modifying slice2 affects slice1 (shared backing array)
	slice2[0] = 999
	
	fmt.Println("After modifying slice2:")
	fmt.Println("slice1:", slice1)
	fmt.Println("slice2:", slice2)
}

// Go Playground: https://go.dev/play/p/bcd890efg123
```

> Be careful when creating slices from other slices - they share the same backing array. Use the `copy()` function if you need independent slices.

#### 7.3.1.2 Append

The `append` function is variadic, meaning it can accept multiple arguments. You can add a single element, multiple elements, or spread another slice.

**Example 42 - Using append variations**
```go
package main

import "fmt"

func main() {
	slice := []int{1, 2, 3}
	
	// Add single element
	slice = append(slice, 4)
	fmt.Println("After adding 4:", slice)
	
	// Add multiple elements
	slice = append(slice, 5, 6, 7)
	fmt.Println("After adding 5, 6, 7:", slice)
	
	// Add another slice (using spread operator)
	slice2 := []int{8, 9, 10}
	slice = append(slice, slice2...)
	fmt.Println("After adding slice2:", slice)
}

// Go Playground: https://go.dev/play/p/efg123hij456
```

**Example 43 - Reslicing to remove element**
```go
package main

import "fmt"

func main() {
	slice := []int{1, 2, 3, 4, 5}
	fmt.Println("Original:", slice)
	
	// Remove element at index 2 (value 3)
	index := 2
	slice = append(slice[:index], slice[index+1:]...)
	fmt.Println("After removing index 2:", slice)
}

// Go Playground: https://go.dev/play/p/hij456klm789
```

#### 7.3.1.3 Copy

The `copy()` function creates a copy of a slice with a new backing array, enabling mutation without affecting the source.

**Example 44 - Using copy to create slice with new backing array**
```go
package main

import "fmt"

func main() {
	source := []int{1, 2, 3, 4, 5}
	destination := make([]int, len(source))
	
	copied := copy(destination, source)
	
	fmt.Println("Source:", source)
	fmt.Println("Destination:", destination)
	fmt.Printf("Copied %d elements\n", copied)
	
	// Modify destination - source unaffected
	destination[0] = 999
	
	fmt.Println("After modifying destination:")
	fmt.Println("Source:", source)
	fmt.Println("Destination:", destination)
}

// Go Playground: https://go.dev/play/p/klm789nop012
```

#### 7.3.1.4 Resizing

Go automatically resizes slices when their capacity is exceeded. The resizing algorithm increases capacity based on the current size, but this can result in significant over-allocation.

**Example 45 - Slice backing array resizing**
```go
package main

import "fmt"

func main() {
	slice := make([]int, 0, 1)
	fmt.Printf("Initial: len=%d, cap=%d\n", len(slice), cap(slice))
	
	for i := 1; i <= 10; i++ {
		slice = append(slice, i)
		fmt.Printf("After adding %d: len=%d, cap=%d\n", i, len(slice), cap(slice))
	}
	
	// Calculate over-capacity percentage
	overCapacity := float64(cap(slice) - len(slice))
	percentage := (overCapacity / float64(len(slice))) * 100
	fmt.Printf("Over-capacity: %.1f%%\n", percentage)
}

// Go Playground: https://go.dev/play/p/nop012qrs345
```

If you know the exact size you need, consider using that capacity initially, or use arrays for fixed-size data to avoid over-allocation.

### 7.3.2 Map

Maps provide associative data storage using key-value pairs. Maps must be initialized before use - attempting to use a nil map will cause a panic.

**Example 46 - Trying to use nil map**
```go
package main

import "fmt"

func main() {
	var m map[string]int
	
	fmt.Println("Map:", m)
	m["key"] = 1 // This will panic!
}

// Go Playground: https://go.dev/play/p/qrs345tuv678
```

**Example 47 - Creating empty map**
```go
package main

import "fmt"

func main() {
	// Three ways to create maps
	map1 := make(map[string]int)
	map2 := map[string]int{}
	map3 := map[string]int{"a": 1, "b": 2}
	
	fmt.Println("map1:", map1)
	fmt.Println("map2:", map2)
	fmt.Println("map3:", map3)
}

// Go Playground: https://go.dev/play/p/tuv678wxy901
```

Maps exhibit reference type behavior - when passed to functions, changes to the map affect the original:

**Example 48 - Map passed by value but original mutated**
```go
package main

import "fmt"

func modifyMap(m map[string]int) {
	m["new"] = 100
	fmt.Println("Inside function:", m)
}

func main() {
	m := map[string]int{"a": 1, "b": 2}
	fmt.Println("Before function:", m)
	modifyMap(m)
	fmt.Println("After function:", m)
}

// Go Playground: https://go.dev/play/p/wxy901zab234
```

#### 7.3.2.1 Working with Maps

**Example 49 - Working with maps**
```go
package main

import "fmt"

func main() {
	m := make(map[string]int)
	
	// Add key-value pairs
	m["apple"] = 5
	m["banana"] = 3
	m["orange"] = 8
	
	fmt.Println("Map:", m)
	fmt.Println("Length:", len(m))
	
	// Iterate over map (order is non-deterministic)
	for key, value := range m {
		fmt.Printf("%s: %d\n", key, value)
	}
	
	// Delete a key
	delete(m, "banana")
	fmt.Println("After deleting banana:", m)
}

// Go Playground: https://go.dev/play/p/zab234cde567
```

#### 7.3.2.2 Safely Accessing Keys

Always check if a key exists before using its value to prevent bugs from zero values:

**Example 50 - Unsafe map access**
```go
package main

import "fmt"

func main() {
	m := map[string]int{"apple": 5}
	
	value := m["banana"] // Returns zero value for non-existent key
	fmt.Printf("banana: %d\n", value)
	
	if value > 0 {
		fmt.Println("We have bananas!")
	} else {
		fmt.Println("No bananas available") // This will print
	}
}

// Go Playground: https://go.dev/play/p/cde567fgh890
```

**Example 51 - Safe map access**
```go
package main

import "fmt"

func main() {
	m := map[string]int{"apple": 5}
	
	value, exists := m["banana"]
	fmt.Printf("banana: %d, exists: %t\n", value, exists)
	
	if exists && value > 0 {
		fmt.Println("We have bananas!")
	} else {
		fmt.Println("No bananas available")
	}
	
	// Or check existence directly
	if _, exists := m["apple"]; exists {
		fmt.Println("We have apples!")
	}
}

// Go Playground: https://go.dev/play/p/fgh890ijk123
```

### 7.3.3 Channel

Channels are the mechanism for communication between asynchronous code in Go. They are created using the `make()` function and can be unbuffered or buffered, and typed.

**Example 52 - Send and receive on unbuffered channel**
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)
	
	// Send to channel in a goroutine
	go func() {
		time.Sleep(1 * time.Second)
		ch <- "Hello from goroutine!"
	}()
	
	// Receive from channel
	message := <-ch
	fmt.Println("Received:", message)
}

// Go Playground: https://go.dev/play/p/ijk123lmn456
```

Variables created with `make()` are reference types - they automatically return a pointer when created, so you don't need to pass them by reference explicitly.

## 7.4 Interface types

Interfaces represent behavior - what something can do - via receiver function signatures with no implementation.

Go uses "duck typing": if something can do what a duck does (walk, quack), then it's a duck in Go's type system.

The empty interface `interface{}` defines no behavior and is therefore implemented by every type. It can substitute for any type. 

> Go 1.18 introduced the `Any` alias for `interface{}` to make the intent clearer when you want to accept any type.

A practical example is the `Stringer` interface from the `fmt` package:

```go
type Stringer interface {
    String() string
}
```

**Example 53 - The Stringer interface**
```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

// Implement the Stringer interface
func (p Person) String() string {
	return fmt.Sprintf("%s (%d years old)", p.Name, p.Age)
}

func main() {
	person := Person{Name: "Alice", Age: 30}
	fmt.Println(person) // Calls String() method automatically
}

// Go Playground: https://go.dev/play/p/lmn456opq789
```

**Example 54 - Implementing Stringer on custom type**
```go
package main

import "fmt"

type Temperature float64

func (t Temperature) String() string {
	return fmt.Sprintf("%.1f°C", float64(t))
}

func main() {
	temp := Temperature(23.5)
	fmt.Println("Temperature:", temp)
}

// Go Playground: https://go.dev/play/p/opq789rst012
```

> Be careful with custom Stringer implementations - they can sometimes produce unexpected output when printing, especially during debugging.

## 7.5 Creating custom types

So far we've looked at Go's built-in types. We've also used named custom struct types.

We've said that types and the statically-typed nature of Go enable compile-time checking which makes our programs safer at runtime. 

But, built-in types alone won't protect us in all circumstances. Consider the example below.

**Example 55 - Simple transposition error**
```go
package main

import "fmt"

func divider(d, n int) float64 {
	return float64(n) / float64(d)
}

func main() {
	var n = 1
	var d = 4

	result := divider(n, d)
	fmt.Println(result) // we think we will get 0.25
}

// Go Playground: https://go.dev/play/p/XhTIz_A-dxd
```

Output: 
```
4

Program exited.
```

We have a basic function which expects two integers, a denominator and numerator. It performs a division calculation and returns the result.

But, when we call the function we make a simple transposition error when passing the numerator and denominator. 

We got the parameters backwards. As both are integers, the compiler has no issue with our code, which builds and runs fine, but, instead of the `0.25` result we expected, we got a result of `4`, which is incorrect. This kind of mistake is very easy to make and can be hard to debug.

But, we can help ourselves a little here. We can employ Go's type system to ensure this kind of error is something the Go compiler will flag during compilation.  And we do this by creating custom types.

Custom types are based on built-in types but they have their own type identity.  A custom type which can hold a *string* value is not the same as the *string* type, for example. It's possible to convert it to a string, but unless it is converted it cannot be used as a *string*.

Let's use custom types to rewrite the code in Example 55 and see if we can't catch that transposition error when the code is compiled.

**Example 56 - Type safety using custom types**
```go
package main

import "fmt"

type numer int
type denom int

func divider(d denom, n numer) float64 {
	return float64(n) / float64(d)
}

func main() {
	var n numer = 1
	var d denom = 4

	result := divider(n, d)
	fmt.Println(result)
}

// Go Playground: https://go.dev/play/p/6UQcR54_wYX
```

Output:
```
./prog.go:16:20: cannot use n (variable of type numer) as type denom in argument to divider
./prog.go:16:23: cannot use d (variable of type denom) as type numer in argument to divider

Go build failed.
```

Perfect! Now the compiler catches our transposition error at build time.

Custom types give us:
1. An extra layer of type checking and safety
2. The ability to catch more errors at compile time  
3. The ability to have receivers bound to them
4. The ability to implement interfaces

**Example 57 - Implementing Stringer interface on custom type**
```go
package main

import "fmt"

type UserID int

func (u UserID) String() string {
	return fmt.Sprintf("User-%d", int(u))
}

func main() {
	userID := UserID(12345)
	fmt.Println("User ID:", userID)
}

// Go Playground: https://go.dev/play/p/rst012uvw345
```

### 7.5.1 Type aliases

Since Go 1.9, you can also create type aliases using the `=` syntax:

```go
type MyString = string
```

Type aliases create an alternative name for an existing type but don't create a new type. They're useful for gradual code migration and creating shorter names for complex types.

### 7.5.2 A look ahead: Generic types

Go 1.18 introduced Generics, which allow you to write code that works with multiple types while maintaining type safety. This enables you to create more flexible and reusable custom types without sacrificing Go's compile-time guarantees.

Generics are particularly powerful when combined with custom types, allowing you to create type-safe collections, data structures, and algorithms that work across different types. They help eliminate code duplication while maintaining the performance and safety benefits of Go's type system.

We'll explore generics in detail in Chapter 9, where you'll learn how to create generic functions, types, and use new features like generic type aliases that build upon the foundation of custom types we've covered here.

## 7.6 Converting between types

To complete our discussion of data types, for now, we're going to look at how we convert values from one type to another.

### 7.6.1 Type conversion

Go does not support *type casting*, only type conversion. What's the difference?

Type casting can often be used with non-compatible data types as well as compatible types, while *type conversion* is restricted to compatible data types, and this is the case with Go. You can't convert a *string* to an *int* using type conversion, for example.

In other languages *type conversion* is often implicit, but in Go because of its strong type system the conversion is explicit. This requires we state what we want the compiler to convert the value to using a wrapper function. The compiler will perform the conversion if the types are compatible, or fail to build with an error if they are not.

The example below demonstrates several type conversions between compatible data types. Generally, type conversion between number types is fine. As is type conversion back and forth between *[]byte* and *string*.

The commented code also shows type conversions between incompatible data types, try removing the comments and running the program again. It won't build, instead, we will receive compilation errors.

**Example 60 - Type conversion examples**
```go
package main

import "fmt"

func main() {
	var myInt = 1
	var myFloat = 0.0
	var myString = "Hello World"
	var myBytes = []byte{240, 159, 154, 128}

	fmt.Printf("Before conversion, myInt is: %T, myFloat is: %T, myString is: %T, myBytes is: %T\n",
		myInt, myFloat, myString, myBytes)

	convertedInt := float64(myInt)
	convertedFloat := int(myFloat)
	convertedBytes := string(myBytes)
	convertedString := []byte(myString)

	fmt.Printf("After conversion, myInt is: %T, myFloat is: %T, myString is: %T, myBytes is: %T\n",
		convertedInt, convertedFloat, convertedString, convertedBytes)

	// incompatible data types
	//convertedFloat2 := string(myFloat)
	//convertedBool := int(true)
	//convertedString2 := int(myString)
}

// Go Playground: https://go.dev/play/p/FOkyY6XJp1q
```

Output:
```
Before conversion, myInt is: int, myFloat is: float64, myString is: string, myBytes is: []uint8
After conversion, myInt is: float64, myFloat is: int, myString is: []uint8, myBytes is: string

Program exited.
```

> Note a simple gotcha.  The Go compiler will **not** error if you try to convert an *int* to a *string* using type conversion, but you will not get a string representation of the integer. The conversion to string will take place, but the integer is treated as a Unicode codepoint. In the following example we don't get a string type of value "128640", we get the Unicode rocket character.

**Example 61 - This won't print what you expect**
```go
package main

import "fmt"

func main() {
	var myInt = 128640
	convertedInt := string(myInt)
	
	fmt.Printf("The int is now a string: '%s', of type '%T'\n", convertedInt, convertedInt)
}

// Go Playground: https://go.dev/play/p/GHVwfyD-mlH
```

Output:
```
The int is now a string: '🚀', of type 'string'

Program exited.
```

### 7.6.2 Other conversion mechanisms

If you do want to convert an integer to its string representation, you can use `fmt.Sprintf`:

**Example 62 - Using fmt.Sprintf for int to string**
```go
package main

import "fmt"

func main() {
	var myInt = 128640
	convertedInt := fmt.Sprintf("%d", myInt)
	
	fmt.Printf("The int is now a string: '%s', of type '%T'\n", convertedInt, convertedInt)
}

// Go Playground: https://go.dev/play/p/abc123
```

For more complex conversions between strings and numeric types, the `strconv` package provides useful functions:

**Example 63 - Package strconv examples**
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	myIntString := "10"
	myFloatString := "3.1415926535"
	myInt := 10

	// int to string
	resStr := strconv.Itoa(myInt)
	fmt.Printf("resStr is '%T', of '%s'\n", resStr, resStr)

	// string to int
	if resInt, err := strconv.Atoi(myIntString); err == nil {
		fmt.Printf("resInt is '%T', '%v'\n", resInt, resInt)
	}

	// string to float
	if resFloat, err := strconv.ParseFloat(myFloatString, 64); err == nil {
		fmt.Printf("resFloat is '%T', '%v'\n", resFloat, resFloat)
	}
}

// Go Playground: https://go.dev/play/p/5kW2FyxBGgU
```

Output:
```
resStr is 'string, of '10'
resInt is 'int', '10'
resFloat is 'float64', '3.1415926535'

Program exited.
```

Finally, to conclude this section, what should we do if *type conversion* won't work since the types are incompatible, and there is no built-in helper function that does what we need, for example, to convert a *bool* to an *int*?

In these rare circumstances, we as developers get an opportunity to step up and devise a conversion helper of our own, rather than rely on the Go standard library for all of our needs ;)

In the example below we create a simple `boolToI()` helper function to perform just such a conversion.

**Example 64 - A custom BoolToI helper function**
```go
package main

import "fmt"

func boolToI(b bool) int {
	var bToI int
	if b {
		bToI = 1
	}
	return bToI
}

func main() {
	myBool := true
	myBool2 := false

	boolAsInt := boolToI(myBool)
	boolAsInt2 := boolToI(myBool2)

	fmt.Printf("boolAsInt: %d, boolAsInt2: %d\n", boolAsInt, boolAsInt2)
}

// Go Playground: https://go.dev/play/p/gseJ2vzyDSc
```

Output:
```
boolAsInt: 1, boolAsInt2: 0

Program exited.
```

