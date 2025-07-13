# Chapter 10a - Generics

*Generics* are a feature in most statically typed programming languages. Generics, or generic programming, allows us to write more reusable code by abstracting types away. It provides a way to reduce duplication, which would otherwise arise when writing functionality compatible with more than just a single type.

Go wasn't designed with support for generics but after much debate over several years, generics support was added to the language in version 1.18.

Mechanically, generics enable us to write code with values whose type can be specified later at the point of use, while maintaining type safety using syntax to place constraints on which types may be accepted.

In this chapter we will examine generics in detail, and through several examples, we'll learn how to do generic programming in Go.

But first we need to understand the problem, which is essentially a side-effect of *static typing*.

## 10a.1 Before generics

Consider the simple `sumInt64()` function in the following example. It's designed to add two integers of the *int64* type. An unremarkable, trivial function.

**Example 77 - Adding two *int64* integers**
```go
package main

import "fmt"

func sumInt64(x, y int64) int64 {
	return x + y
}

func main() {
	fmt.Println(sumInt64(1, 2))
}

// Go Playground: https://go.dev/play/p/XDU49bgxRs_j
```

The problem is that its parameter and return typing mean it can only add *int64* types. 

But, we can't count on all integers being of this type, so what do we do when we need to add *int8*, *int16*, or *int32* typed integers? 

One obvious approach is to duplicate the same function three more times, one for each integer type that we'll need to add. 

In the following example, we implement those new functions. Notice, that the function body is always the same, only the *parameter* and *return* types are different.

**Example 78 - Adding other integer types**
```go
package main

import "fmt"

func sumInt8(x, y int8) int8 {
	return x + y
}

func sumInt16(x, y int16) int16 {
	return x + y
}

func sumInt32(x, y int32) int32 {
	return x + y
}

func sumInt64(x, y int64) int64 {
	return x + y
}

func main() {
	fmt.Println(sumInt8(1, 2))
	fmt.Println(sumInt16(1, 2))
	fmt.Println(sumInt32(1, 2))
	fmt.Println(sumInt64(1, 2))
}

// Go Playground: https://go.dev/play/p/Q3SzagyF-CS
```

With the function duplicated, we can now add any integer type, which is great, but it does feel a bit awkward. All that duplication just to add two numbers?

Perhaps there is another way to solve the problem.

Well, there is/was. Before Go 1.18 we could already use a *form* of generic programming with *interfaces*. By using the empty interface specifically, rather than duplicating the function as we did in the previous example, we could write a single implementation that accepts the `interface{}` type as parameters and then performs a *type switch* on the arguments to determine how to handle them. 

This approach has drawbacks too, which we will come to, but first, let's look at how we could write a single function that could add any pair of same typed integers.

**Example 79 - A single sumIntAny implementation**
```go
package main

import (
	"errors"
	"fmt"
	"reflect"
)

func sumIntAny(x, y interface{}) (interface{}, error) {
	if reflect.TypeOf(x) != reflect.TypeOf(y) {
		return nil, errors.New("mismatched types")
	}

	switch v := x.(type) {
	case int8:
		return v + y.(int8), nil
	case int16:
		return v + y.(int16), nil
	case int32:
		return v + y.(int32), nil
	case int64:
		return v + y.(int64), nil
	}

	return nil, nil
}

func main() {
	res, err := sumIntAny(int8(1), int8(2))
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("Result:", res)
	//fmt.Println("Addition:", res+res)
}

// Go Playground: https://go.dev/play/p/GZ0JOBSwu9_n
```

One problem is evident immediately. The function is now significantly more complex.  

We need to call it by explicitly stating which integer types we are passing. We need to check that whatever values are passed are the same concrete type so we can add them, and we have an error return value in case they are not.

We're using a *type switch* to handle the different types that could be represented by `interface{}` before performing the integer addition. But what if a string is passed, we don't handle strings currently, and we should probably return an error?

Another problem is the return type. Without performing a type conversion, we have to return the same type as was passed. Since this can be one of four integer types, we can only represent this with the empty interface.

We could choose to convert to an arbitrary integer type and return that type, but which one? If the caller passes *int8* values, does it make sense to return an *int64* result?

So we stick with the empty interface return, but that creates friction for the caller. The calling code, must introspect the interface, probably with another *type switch*.

The `fmt.Println()` performs that type switch internally, which is why it can print the value held in the interface, but try uncommenting line 35 to perform the simple addition. The program won't compile.

```
./prog.go:35:27: invalid operation: operator + not defined on res (variable of type interface{})

Go build failed.
```

So both solutions are clunky when you stop and think, but as Go developers, we've never known any different. This is just how it is - the price of type safety!

But, for other developers coming from languages which support generic programming, this is not how we should do it at all. They may find the above approaches quite naive compared to a solution based on generics.

Their frustration is genuine. Perhaps there is a better way?

## 10a.2 Solving the problem with generics

In this section, we're going to tackle the same problem using generics. 

We'll start with a simple but naive example, and build on that over several more examples until we understand everything that the current implementation of generics gives us, plus what might come in later versions of Go.

> To work with generics examples locally, you'll need a Go version of 1.18 or above. At the time of writing the Go Playground has Go 1.19 available, so all the examples included here, will run over there.

One way a generic function differs from a normal function is that we can pass additional type parameters with constraints. 

Think of these as placeholders for type. The concrete type will be assigned by the compiler when the function is called. The constraint provides some hints to the compiler about what that type is allowed to be. This means some checks are possible during compilation, and we'll see this in action shortly.

The type parameters together with their constraints are passed in square braces which must immediately follow the function's name. 

In the snippet below we illustrate with a single type parameter and constraint. This is sufficient for our solution, but we're not restricted to a single type parameter/constraint, and we'll show an example of that later on.

```go
func SumAny[T any] (x, y T) T {
    ...
}
```

So how do we reason about this unusual syntax? Let's walk through it.

`T`, is a placeholder or *alias* for a *type* which has the constraint `any` which means it can be any type. We could have passed the empty interface type, `interface{}` instead. 

Both `x` and `y` parameters use the type *alias* for their type. The single return value is also using the alias. So, whatever type is passed for `x` and `y` - which must be the same - will also be the return type.

It's like a template, with real types being substituted for the placeholders when they become known. 

The capitalization of the type parameter `T` is not a requirement, but things get a little (very) confusing if we mix lowercase type parameters with variable parameters. We're adopting the capitalization convention to improve readability. 

Finally, `T` has no special significance, we could have used any name for the type parameter. Single letter or a semantic name. Once again, convention is good. We'll use the single letter, just as we'll use capitalisation.

So, now that we understand the semantics a little better, let's implement that function body of the `SumAny()` function. 

It should be straightforward enough.

**Example 80 - A generic implementation of SumAny**
```go
package main

import (
	"fmt"
	"reflect"
)

func SumAny[T any](x, y T) T {
	return x + y
}

func main() {
	res := SumAny(int16(1), int16(2))
	fmt.Println(res)
	fmt.Println(reflect.TypeOf(res))
}

// Go Playground: https://go.dev/play/p/jbzggpV7uM8
```

Output:
```
./prog.go:9:9: invalid operation: operator + not defined on x (variable of type T constrained by any)

Go build failed.
```

It doesn't compile. 

The constraint we specified allows this function to accept *any* type, but not all types can be summed with the addition operator, since not all types are numbers. 

The compiler recognises this during compilation, so this doesn't introduce a panic risk fortunately. It looks like what we need instead is a constraint that accepts only *any* integer type?

There are two ways we can achieve this. We can use a type parameter constraint list or we can create our own constraint using an interface. 

We'll implement the same function again using both approaches.

First, let's use a type constraint list. We need to specify all valid types for our function and we do this using a *pipe* delimited list.

```go
package main

import (
	"fmt"
	"reflect"
)

func SumAny[T int8 | int32 | int64 | int](x, y T) T {
	return x + y
}

func main() {
	res := SumAny(int16(1), int16(2))
	fmt.Println(res)
	fmt.Println(reflect.TypeOf(res))
}

// Go Playground: https://go.dev/play/p/yZemwfvY6ST
```

Output:
```
./prog.go:13:15: int16 does not implement int8|int32|int64|int (int16 missing in int8 | int32 | int64 | int)

Go build failed.
```

Again it doesn't build. This time, because we are passing `int16` types to the function, and we deliberately didn't include that type in the constraint list. 

Add it to the list and run the example again - don't forget the pipe separator! 

You should now see this output, and notice the return type used was `int16` too, because the type placeholder `T` became `int16` when the function was called.

```
3
int16

Program exited.
```

We've just written our first generic function in Go! The function signature is *bit* harder to read, but it's avoided us duplicating the function three or four times. Neither did we need to perform complex type switch style assertions in this single implementation. In fact the function body is the same as we started with.

However, we can do better. We can create our own constraints, which are interfaces basically, and specify the types allowed for that constraint. This single act will restore order in our codebase.

> Go has two built-in constraints *any*, which we've seen, and *comparable*, which allows any type which supports equality comparisons with the `==` and `!=` operators.
>
> In later releases it's probable that the standard library will include a new [constraints package](https://cs.opensource.google/go/x/exp/+/a68e582f:constraints/constraints.go), which could include an `Integer` constraint, but for now, if we need to go further than *comparable* we must build the constraint ourselves.

In this next example, we'll a create custom constraint *Numeric* by moving the constraint list into a new interface type.

**Example 81 - Creating a custom constraint**
```go
package main

import (
	"fmt"
	"reflect"
)

type Numeric interface {
	int8 | int16 | int32 | int64 | int
}

func SumAny[T Numeric](x, y T) T {
	return x + y
}

func main() {
	res := SumAny(int16(1), int16(2))
	fmt.Println(res)
	fmt.Println(reflect.TypeOf(res))
}

// Go Playground: https://go.dev/play/p/IxpNJBWPkZo
```

The output is identical to before, and the `SumAny()` function signature is simple to read again. As a bonus *Numeric* is reusable, we can avoid the constraint list in the next generic function that performs addition or subtraction, and instead use the *Numeric* constraint.

As you might expect, custom constraints, since they are interfaces, are composable. This allows us to build less restrictive constraints by composing from restrictive constraints.

In the next example, we'll amend the code so that `SumAny()` can also perform addition on unsigned integers, a glaring omission. But, rather than use one big constraint list, we will use composition to create the *Numeric* constraint, by creating and embedding a *Signed* and *Unsigned* constraint.

**Example 82 - Using composition with constraints**
```go
package main

import (
	"fmt"
	"reflect"
)

type Numeric interface {
	Signed | Unsigned
}

type Unsigned interface {
	uint8 | uint16 | uint32 | uint64 | uint
}

type Signed interface {
	int8 | int16 | int32 | int64 | int
}

func SumAny[T Numeric](x, y T) T {
	return x + y
}

func main() {
	res := SumAny(int16(1), int16(2))
	fmt.Println(res)
	fmt.Println(reflect.TypeOf(res))
}

// Go Playground: https://go.dev/play/p/2yARr7xNBG8
```

Kudos to the Go development team! That is incredibly elegant in its simplicity, and the way it builds on what we already know.

But what about custom types?

We can use custom types in constraint lists and constraints, just like built-in types, but Go provides a modifier we can apply to built-in types, so that any user-defined type, which is based on one of the built-in types can be allowed. 

Let's take our example in a slightly different direction to illustrate this.

**Example 83 - Allow any type with underlying type *int***
```go
package main

import (
	"fmt"
	"reflect"
)

type OnlyInt interface {
	int
	// ~int
}

type myInt int

func SumAny[T OnlyInt](x, y T) T {
	return x + y
}

func main() {
	var myInt myInt = 1
	res := SumAny(myInt, 2)
	fmt.Println(res)
	fmt.Println(reflect.TypeOf(res))
}

// Go Playground: https://go.dev/play/p/b8jNMrEtUcL
```

Output:
```
./prog.go:21:15: myInt does not implement OnlyInt (possibly missing ~ for int in constraint OnlyInt)

Go build failed.
```

Once again, this code fails to compile, but the compiler message is very helpful. We are trying to pass a value of type *myInt*, a user-defined type which has the underlying type of *int*. 

If you uncomment line 10, and comment out line 9, this tilde modifier tells the compiler that we will accept any type which has the underlying type of *int*.

The code should now compile and run, producing the following output.

```
3
main.myInt

Program exited.
```

The last example we're going to look at demonstrates how we can use multiple type parameters each with its own constraint. It's quite contrived but serves to illustrate the point.

**Example 84 - Multiple type parameters**
```go
package main

import (
	"fmt"
	"reflect"
)

func MakeAddToMap[K comparable, V any](k K, v V) map[K]V {
	mp := make(map[K]V)
	mp[k] = v
	return mp
}

func main() {
	mp := MakeAddToMap("key", 1)
	fmt.Println(reflect.TypeOf(mp))
	fmt.Println(mp)
}

// Go Playground: https://go.dev/play/p/DFeMUaFqBzG
```

Output: 
```
map[string]int
map[key:1]

Program exited.
```

Notice that we use the built-in *comparable* constraint on the `K` type parameter, as not every type can be used for a map key. If we had specified *any* the program would have failed to build.

The syntax is again a little confusing, but I suspect as we start to see more generic programming in Go, we'll get used to it.

## 10a.3 Generic type aliases

Building on our understanding of generics, Go 1.24 introduced full support for generic type aliases, a powerful feature that allows you to create aliases for generic types with type parameters. This feature was previewed in Go 1.23 but is now stable and ready for production use.

Generic type aliases are particularly useful when working with complex generic types that appear frequently in your code. They can make your code more readable and maintainable by providing cleaner, more expressive syntax for complex generic relationships.

Think of generic type aliases as a way to create shortcuts for complex generic types, similar to how regular type aliases work but with the added power of generics.

**Example 85 - Generic type aliases**
```go
package main

import "fmt"

// Traditional generic type definition
type Map[K comparable, V any] map[K]V

// Generic type alias (Go 1.24+)
type StringMap[V any] = map[string]V
type TypedSlice[T any] = []T

func main() {
	// Using generic type alias for string-keyed maps
	users := StringMap[string]{
		"user1": "Alice",
		"user2": "Bob",
		"user3": "Charlie",
	}
	
	// Using generic type alias for typed slices
	numbers := TypedSlice[int]{1, 2, 3, 4, 5}
	names := TypedSlice[string]{"Alice", "Bob", "Charlie"}
	
	fmt.Println("Users:", users)
	fmt.Println("Numbers:", numbers)
	fmt.Println("Names:", names)
}

// Go Playground: https://go.dev/play/p/[requires-go-1.24]
```

Notice the difference between the traditional generic type `Map[K comparable, V any]` and the generic type alias `StringMap[V any] = map[string]V`. The alias uses the `=` symbol and provides a more specific, constrained version of the generic map type.

Generic type aliases are especially powerful when working with complex nested types. They help break down complexity into manageable, reusable components:

**Example 86 - Complex generic type aliases**
```go
package main

import "fmt"

// Without generic type aliases - complex and hard to read
type ComplexResult1[T any] map[string][]map[string]T

// With generic type aliases - much cleaner
type NestedMap[T any] = map[string]T
type SliceOfMaps[T any] = []NestedMap[T]
type ComplexResult[T any] = map[string]SliceOfMaps[T]

func processData[T any](data ComplexResult[T]) {
	for category, sliceOfMaps := range data {
		fmt.Printf("Category: %s\n", category)
		for i, nestedMap := range sliceOfMaps {
			fmt.Printf("  Item %d: %v\n", i, nestedMap)
		}
	}
}

func main() {
	data := ComplexResult[int]{
		"group1": SliceOfMaps[int]{
			NestedMap[int]{"a": 1, "b": 2},
			NestedMap[int]{"c": 3, "d": 4},
		},
		"group2": SliceOfMaps[int]{
			NestedMap[int]{"e": 5, "f": 6},
		},
	}
	
	processData(data)
}

// Go Playground: https://go.dev/play/p/[requires-go-1.24]
```

This example demonstrates how generic type aliases can transform a complex, hard-to-read type signature into a series of clear, understandable components. Each alias serves a specific purpose and can be reused throughout your codebase.

You'll find generic type aliases particularly useful when:

- Working with deeply nested generic structures
- Creating domain-specific abstractions over generic types
- Migrating existing codebases to use generics gradually
- Building libraries that expose simplified interfaces to complex generic types
- Reducing cognitive load when reading and maintaining code

**Example 87 - Practical domain-specific aliases**
```go
package main

import "fmt"

// Domain-specific generic type aliases for a web application
type UserID = string
type ProductID = string
type OrderID = string

type Repository[ID comparable, T any] = map[ID]T
type UserRepository = Repository[UserID, User]
type ProductRepository = Repository[ProductID, Product]
type OrderRepository = Repository[OrderID, Order]

type User struct {
	ID   UserID
	Name string
	Age  int
}

type Product struct {
	ID    ProductID
	Name  string
	Price float64
}

type Order struct {
	ID     OrderID
	UserID UserID
	Items  []ProductID
}

func main() {
	// Clean, readable repository usage
	users := UserRepository{
		"u1": User{ID: "u1", Name: "Alice", Age: 30},
		"u2": User{ID: "u2", Name: "Bob", Age: 25},
	}
	
	products := ProductRepository{
		"p1": Product{ID: "p1", Name: "Laptop", Price: 999.99},
		"p2": Product{ID: "p2", Name: "Mouse", Price: 29.99},
	}
	
	orders := OrderRepository{
		"o1": Order{ID: "o1", UserID: "u1", Items: []ProductID{"p1", "p2"}},
	}
	
	fmt.Printf("User: %+v\n", users["u1"])
	fmt.Printf("Product: %+v\n", products["p1"])
	fmt.Printf("Order: %+v\n", orders["o1"])
}

// Go Playground: https://go.dev/play/p/[requires-go-1.24]
```

This feature helps bridge the gap between complex generic code and readable, maintainable software. By providing meaningful names for complex generic types, you make your code more approachable for both current and future maintainers.

> Generic type aliases maintain full type safety while providing a cleaner, more expressive syntax for complex generic relationships. They represent the evolution of Go's type system toward greater expressiveness without sacrificing the language's core principles of simplicity and clarity.

The introduction of generic type aliases in Go 1.24 demonstrates the language's continued commitment to making complex features accessible and practical for everyday use. As you work with generics in your projects, consider how type aliases can improve code readability and reduce the cognitive burden on your development team.

## 10a.4 Summary

We should now have a fairly good grasp of generics. Let's recap on what we've learned.

We know how to apply generic programming to solve some of the types of problems where traditionally we've just written more code, or settled for some type assertion. 

We've seen the complexity generics can introduce into our syntax, and, while accepting that novelty and our unfamiliarity are factors, we've discovered ways we can mitigate this with custom constraints and composition, for example.

We've looked at how the generics implementation has evolved with the addition of aliases in Go 1.24 and future versions of Go will likely add more features.

But of course, not everything requires us to apply generics. Often a little duplication is absolutely fine. Such code is often simple to read, and simple to reason about. We must recognise that maintaining code is often the responsibility of a team of developers so we should be considerate of them, and their abilities, especially when applying features which are relatively new, and that not everybody has experience with.