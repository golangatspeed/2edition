# Chapter 6 - Variables and constants

Let's discuss variables, constants and scope next. A word of warning, things may feel a little 'chicken-and-egg' for a short time, as we'll have to briefly mention *type* which we haven't covered so far, and won't until the next chapter.

## 6.1 Variables

We've said that Go is opinionated. Usually, there is a specific way to do something - maybe a couple of ways - but one area where being opinionated would have been nice is variable declaration and initialisation. There are too many ways to do it.

> *Declaration* is the act of creating a variable with a name and type, *type* being what the variable can hold, such as a number or a string. *Initialisation* is the act of setting a variable equal to some value at the time it is declared. *Assignment* is the act of changing the value stored in a variable by assigning a new value at any point during program execution.

We'll look at the different styles shortly so that you know how to recognise them but, generally speaking, for clarity you should limit the styles you employ in your code to just three, two of which use the `var` keyword, and the third a short-form declaration and initialisation using `:=`.

Here they are:

**Example 9 - Recommended variable declaration styles**
```go
// Style 1. Declaration only in the global or function scope
var myVar int

// Style 2. Short form declaration and initialisation within the function scope only
myVar := 10

// Style 3. Multiple variables, omitting the repeating var keyword
var (
    myVar int
    myVar2 int
)
```

We'll cover scope soon, but to summarise, you may use all three styles inside functions, but you cannot use `Style 2` in the global scope. 

> Note that when we refer to function scope, we mean anything, *not* in the global scope, so variables declared in receivers would also have function scope.

You should probably move to the next section on constants in all honesty, but for completeness, the example program below shows all of the different styles which you may see, and indeed, can use. 

Some are declarations, others also initialise the variable. Some infer the type, while others explicitly state the type, but they're all valid ways to create variables.

**Example 10 - All variable declaration styles**
```go
package main

import (
	"fmt"
)

var myVar1 int

var (
	myVar2 int = 1
	myVar3 int
)

var myVar4 int = 4
var myVar5 = 5

func main() {
	var myVar6 int
    var myVar7, myVar8 int
	var myVar9, myVar10 = 9, 10

	myVar11 := 11
	myVar12, myVar13 := 12, 13

	fmt.Println("Go is not so opinionated on some things...")
	fmt.Println(myVar1, myVar2, myVar3, myVar4, myVar5)
	fmt.Println(myVar6, myVar7, myVar8, myVar9, myVar10, myVar11, myVar12, myVar13)
}

// Go Playground: https://go.dev/play/p/c8eFFqUfVTz
```

Let's break that example down.

A variable can be created using the `var` keyword in the global scope or function scope e.g. myVar1 and myVar6.

If we declare the variable and type it will be initialised to its *nil* value e.g. myVar1. 

We can initialise it ourselves and specify the type if we wish e.g. myVar4, or let the compiler infer it from the initialised value e.g. myVar5.

As with the `import` keyword, we can avoid repeating the `var` keyword by enclosing multiple declarations within braces. You can do this in the global or function scope.

Within the function scope - as well as the styles above - we can also use the short-form notation `:=` which declares and initialises a variable. Type is inferred based on the value e.g. myVar11.

Finally, we can declare multiple variables **of the same type** using standard or short form notation as a comma delimited list e.g. myVar7 and myVar8, myVar9 and myVar10 and myVar12 and myVar13.

A bit confusing yes?

> Remember, Go is compiled and not interpreted. We don't need clever optimisations in our syntax because the compiler will optimise our source code during a build. Our focus should be on writing clear code that the average Go developer can understand and limiting the styles we use to declare variables is a step towards that objective.

To wrap up variables we should mention *unused* variables. 

The Go compiler will complain if a variable within a function scope is not used by your program at build time.

It's nice that the compiler has our backs - leaving unused code hanging about could be a source of confusion, but, it's worth stating that the compiler will not complain about any unused global variables simply because they cannot be detected during compilation.

However, it's common to have temporary placeholder code and if you're not ready to use a variable in your program it can be frustrating to keep commenting it out to run your program.  

The solution is to use the variable in some way. A common approach is to print out the variable, but while this satisfies the compiler it can add noise when attempting to debug a program.

An alternative approach, which also keeps the compiler happy is to discard the unused variable using the *blank identifier*.

**Example 11 - Discarding with the blank identifier**
```go
package main

import "fmt"

func main() {
	myVar := 1
	_ = myVar

	fmt.Println("compiler is happy")
}

// Go Playground: https://go.dev/play/p/NYjfpotoOtx
```

The example program above declares and initialises a variable then immediately discards it. The program outputs "compiler is happy" as expected, but if you comment out line 7 the compiler will not build the program and will output the following error:

```
./prog.go:6:2: myVar declared but not used

Go build failed.
```

Temporarily discarding the variable is my preferred approach, because it is clear what the intent is, we're saying we don't need it *at the moment*.  

Whether we choose to comment out, print or discard the variable, all should be considered temporary measures for use during early development only.

## 6.2 Constants

Constants are immutable values. They are declared with the `const` keyword and may be created in the same way as variables, with one exception, the short-form declaration `:=` is not valid for use with constants. 

Just like variables, a constant's type can be inferred if it not explicitly set. However, unlike variables this inference may be a loose typing. The type only becomes fixed at the point of conversion, or point of use.

Note, this fluidity applies only to numerical types.

Check out the example below which demonstrates this fluidity, then uncomment line 18 and run it again.

**Example 12 - Loose typing of number constants**
```go
package main

import (
	"fmt"
)

const myConst = 1
var myVar = 1

func needsAFloat(val float64) {
	fmt.Println("looks like we got a float:", val)
}

func main() {
	fmt.Printf("myConst is type '%T'\n", myConst)
	fmt.Printf("myVar is type '%T'\n", myVar)
	needsAFloat(myConst)
	//needsAFloat(myVar)
}

// Go Playground: https://go.dev/play/p/MC8HsTfnVz9
```

Output:
```
myConst is type 'int'
myVar is type 'int'
looks like we got a float: 1

Program exited.
```

### 6.2.1 Constants and the iota identifier

Sometimes the value stored by constant has meaning, but often that won't be the case, instead we simply want to differentiate between constant values, in fact, any value would do.

Typically, where the value doesn't have meaning and we simply want to tell between the constants we'll use integer values.

We can, if we wish, manage these values manually by maintaining the list ourselves, as in the example below.

**Example 13 - Managing number constants manually**
```go
package main

import (
	"fmt"
)

const (
	const1 = 1
	const2 = 2
	const3 = 3
	const4 = 4
	const5 = 5
)

func main() {
	fmt.Println("constants values are:", const1, const2, const3, const4, const5)
}

// Go Playground: https://go.dev/play/p/EVMSF3BCCn8
```

Output: 
```
constants values are: 1 2 3 4 5

Program exited.
```

Alternatively, we can use the *iota* identifier as a shorthand way of initialising all the constants. 

**Example 14 - Managing number constants with iota**
```go
package main

import (
	"fmt"
)

const (
	const1 = iota
	const2
	const3
	const4
	const5
)

func main() {
	fmt.Println("constants values are:", const1, const2, const3, const4, const5)
}

// Go Playground: https://go.dev/play/p/h4fNoptnCJZ
```

Output: 
```
constants values are: 0 1 2 3 4

Program exited.
```

Note *iota* is zero-based, if you wanted to start the sequence at 1 as in the previous example you would simply add 1 as shown below.

**Example 15 - Rebasing the numbering from 1**
```go
const (
	const1 = iota + 1
)
```

To use a non-linear sequence, we can employ the *blank identifier* once again to discard the specific iota values.

**Example 16 - Non-linear constant sequences**
```go
package main

import (
	"fmt"
)

const (
	const1 = iota
	_
	_
	const4
	const5
)

func main() {
	fmt.Println("constants values are:", const1, const4, const5)
}

// Go Playground: https://go.dev/play/p/8QypEyOoxT2
```

Output: 
```
constants values are: 0 3 4

Program exited.
```

Using *iota* can make constant management much easier, but can be prone to occasional problems if new constants are not added to the end of the list, but instead inserted into the list - say alphabetically - and the value itself has some meaning in your program. For example, the code below is fragile because we use the values, not the constants themselves in switch case comparisons.

**Example 17 - Values used in place of named constants**
```go
package main

import (
	"fmt"
)

const (
	blue = iota
	//red
	yellow
	purple
)

func main() {
	chosenColor := yellow
	switch chosenColor {
	case 0:
		fmt.Println("Chose blue")
	case 1:
		fmt.Println("Chose yellow")
	case 2:
		fmt.Println("Chose purple")
	}
}

// Go Playground: https://go.dev/play/p/n-POyR0-k7a
```

Output:
```
Chose yellow

Program exited.
```

Uncomment line 9 and and now the program thinks we chose a different colour.

Output:
```
Chose purple

Program exited.
```

> If you use *iota* to simplify constant management, be sure to use constant names throughout your code and never perform comparisons on actual values, which could change as new constants are introduced into your codebase.

## 6.3 Scope

Scope is an important concept, especially for variables, so let's review what we mean by *scope*.

A variable declared in the global scope may be accessed and mutated by any function in the same package. Equally, an exported package variable declared in the global scope may be mutated by any code that imports the package.

This is not the case for constants, since they cannot be mutated after first initialisation. For this reason, there's a strong argument for only using constants in the global scope.

A variable of the same name can be declared in multiple scopes, as each scope has a separate namespace. Avoid this if possible, it can lead to something we call *variable shadowing* where it can become unclear what value is stored in the variable we are using in a specific scope.

Consider the example below where a variable is declared in both the global and function scope.

**Example 18 - Global and function scope**
```go
package main

import "fmt"

var myVar = 10

func main() {
	myVar += 5
	var myVar int
	myVar -= 5
	fmt.Println("myVar is:", myVar)
}

// Go Playground: https://go.dev/play/p/V7OT8HX_XMP
```

Go will use the locally scoped variable in preference to any parent scoped variable. In the example above we printed the `myVar` variable declared in `main()` as that was the locally scoped variable. 

The same is also true for declarations made inside control structures like loop and conditional statements. Go will use the variable inside the control structure, in preference to the function or globally scoped variable.

**Example 19 - Local scope in control structure**
```go
package main

import (
	"fmt"
)

var myVar = 10

func main() {
	// global scope
	fmt.Println(myVar)

	myVar := 5

	for i := 0; i < 1; i++ {
		//control structure scope
		myVar := 0
		fmt.Println(myVar)
	}

	// function scope
	fmt.Println(myVar)
}

// Go Playground: https://go.dev/play/p/edNkZtJuCPk
```

Output:
```
10
0
5

Program exited.
```

For this reason, duplicated variable names should be avoided for all but the most generic of values, such as errors.

## 6.4 Variable semantics. Pointers and values

When working with variables we often need to pass them around as arguments and return values in functions and receivers. 

In Go, we can share variables as *values* or *pointers*.  A value is a copy of the contents of the variable, whereas a pointer is a copy of the address of the variable in memory.

As stated in an earlier chapter, both are values, just different kinds of values, but to avoid confusion here, when we say *pass by value* we mean sharing a copy of the variable, whereas with *pass by reference* we mean sharing the memory address of the variable.

When we pass something to a function by value, the function receives a copy of that value. This could be ideal in a function which prints simple output, like a username for example, but, passing a copy of a value may not be ideal if we are passing a large struct around in our program, with each copy duplicating that value in memory. 

The fact that we have a copy of the original value, and not the original value itself means we cannot mutate the original. In many cases, this change protection will be exactly what we want in code which should be able to read but not amend a value.

By contrast, if we pass by reference, the value we pass is the address in memory of the variable. It is not a copy of the variable, nor is it the original variable, it is simply the address of the original variable in hexadecimal form, which could look like this `0xc0000ac018`.

Using the memory address we can get the original variable stored in memory at that address by *dereferencing* the value and we may, if we wish, mutate that value using the same approach.

Two examples follow which show the syntax used to pass by value, pass by reference and deference the value.

In the first example, we have a function that accepts a value of type string. Although we attempt to change it, we are mutating a copy of the value and not the original, so the final output is unchanged.

**Example 20 - Pass by value**
```go
package main

import (
	"fmt"
)

func SayHello(name string) {
	fmt.Printf("Hello %s\n", name)
	name = "Dave Blogs"
}

func main() {
	name := "Joe Blogs"
	SayHello(name)
	fmt.Println(name)
}

// Go Playground: https://go.dev/play/p/uESKRz6r-tS
```

Output:
```
Hello Joe Blogs
Joe Blogs

Program exited.
```

Moving to the second example, the `SayHello()` function signature has been modified to accept a pointer to a value of type string. We use `*string` in the signature to denote this, and this function will no longer accept the value itself.

**Example 21 - Pass by reference**
```go
package main

import (
	"fmt"
)

func SayHello(name *string) {
	fmt.Println(name)
	fmt.Printf("Hello %s\n", *name)
	*name = "Dave Blogs"
}

func main() {
	name := "Joe Blogs"
	SayHello(&name)
	fmt.Println(name)
}

// Go Playground: https://go.dev/play/p/7Ne6OPnCslN
```

Output:
```
0xc00009e210
Hello Joe Blogs
Dave Blogs

Program exited.
```

On line 15 we take a pointer to the `name` variable by prefixing it with the *&* character and pass that as the single argument to `SayHello()`.

On line 8 we print the pointer stored in the `name` parameter which outputs a memory address, and on line 9 we dereference the value using the asterisk prefix - as in we ask for the value stored at that memory address - so that we may print it. 

Finally, on line 10 we assign a new value to the dereferenced `name` variable and we can see that we have mutated the original variable as when we print it again at line 16, the new value is displayed.

## 6.5 Value initialisation

Building on the information in the previous section we're going to look at how we can create values and pointers.  

Values are simple, we've seen them many times in this section already. Whenever we create a variable and share it, we ordinarily pass it by value. Of course, we can obtain a pointer to it after creation if needs be, as we did at line 15 in the previous example.

So what if we want to work with pointers exclusively and we don't want the value which can be copied, but a single representation of the thing in memory. To do this we need to take a pointer directly - the memory address - and share that.

There are three ways to achieve this. We can use the short-form *&* prefix,the *new* function, and *make*. All give us a pointer to a value in memory. 

The example below shows two approaches relevant to creating a *struct* type. We'll cover structs in the next section, and later we'll also look at the third approach, *make* when we consider reference types.

**Example 22 - Obtaining a pointer directly**
```go
package main

import "fmt"

type User struct {
	Name string
}

func ChangeName(name string, user *User) {
	user.Name = name
}

func main() {
	u1 := new(User)
	u2 := &User{
		Name: "Joe Blogs",
	}

	ChangeName("Dave Blogs", u1)
	ChangeName("Trevor Blogs", u2)

	fmt.Println("u1:", u1)
	fmt.Println("u2:", u2)
}

// Go Playground: https://go.dev/play/p/t3iJJ47laIt
```

Output:
```
u1: &{Dave Blogs}
u2: &{Trevor Blogs}

Program exited.
```

On line 14 we use the new keyword to obtain a pointer to a *nil* value of *User* and on line 15 we use the *&* prefix to obtain another pointer to a *User* using struct literal syntax, which allows us to initialise it to a non-nil value.

> Did you notice anything different in this example compared to Example 21?  

We didn't need to dereference the structs either to assign a new value or to print them, when previously we did?

Go performs automatic dereferencing on struct types for us which results in a much cleaner syntax.

Consider the `changeName()` function in the previous example. The syntax below is equivalent to that in Example 22. Although it is perfectly valid, and the compiler will not complain, it is less readable, so we should let Go perform the dereferencing on our behalf:

```go
func ChangeName(name string, user *User) {
	(*user).Name = name
}
```

> Note that when we print the structs at lines 26 & 27, although Go performs the dereferencing automatically and prints the struct values it reminds us that we are working with pointers by prefixing the output with the *&* character.

## 6.6 Built-in utility functions

Go 1.21 introduced three new built-in functions that work with basic types and can simplify common operations with variables and values. These functions - `min`, `max`, and `clear` - are now part of the language itself, so you don't need to import any packages to use them.

Let's explore each of these utility functions and see how they can make our code more concise and readable.

### 6.6.1 Finding minimum and maximum values

Before Go 1.21, finding the minimum or maximum value between two or more numbers required writing your own comparison logic or importing external packages. The new `min` and `max` functions eliminate this need entirely.

Both functions accept multiple arguments of the same comparable type and return the smallest or largest value respectively.

**Example 23 - Using min and max functions**
```go
package main

import "fmt"

func main() {
	// Finding minimum values
	minInt := min(10, 5, 8, 3, 15)
	minFloat := min(3.14, 2.71, 1.41)
	minString := min("zebra", "apple", "banana")

	fmt.Println("Minimum integer:", minInt)
	fmt.Println("Minimum float:", minFloat)
	fmt.Println("Minimum string:", minString)

	// Finding maximum values
	maxInt := max(10, 5, 8, 3, 15)
	maxFloat := max(3.14, 2.71, 1.41)
	maxString := max("zebra", "apple", "banana")

	fmt.Println("Maximum integer:", maxInt)
	fmt.Println("Maximum float:", maxFloat)
	fmt.Println("Maximum string:", maxString)
}

// Go Playground: https://go.dev/play/p/[placeholder]
```

Output:
```
Minimum integer: 3
Minimum float: 1.41
Minimum string: apple
Maximum integer: 15
Maximum float: 3.14
Maximum string: zebra

Program exited.
```

As you can see, these functions work with integers, floats, and even strings (using lexicographical ordering). The functions require at least one argument and all arguments must be of the same comparable type.

### 6.6.2 Clearing collections

The `clear` function provides a convenient way to remove all elements from slices, maps, and channels. This is particularly useful when you want to reuse a collection without reallocating memory.

**Example 24 - Using the clear function**
```go
package main

import "fmt"

func main() {
	// Clearing a slice
	numbers := []int{1, 2, 3, 4, 5}
	fmt.Println("Before clear:", numbers, "len:", len(numbers))
	
	clear(numbers)
	fmt.Println("After clear:", numbers, "len:", len(numbers))

	// Clearing a map
	colours := map[string]string{
		"red":   "#FF0000",
		"green": "#00FF00",
		"blue":  "#0000FF",
	}
	fmt.Println("Before clear:", colours, "len:", len(colours))
	
	clear(colours)
	fmt.Println("After clear:", colours, "len:", len(colours))

	// The underlying capacity is preserved for slices
	numbers = append(numbers, 10, 20, 30)
	fmt.Println("After append:", numbers, "len:", len(numbers), "cap:", cap(numbers))
}

// Go Playground: https://go.dev/play/p/[placeholder]
```

Output:
```
Before clear: [1 2 3 4 5] len: 5
After clear: [0 0 0 0 0] len: 5
Before clear: map[blue:#0000FF green:#00FF00 red:#FF0000] len: 3
After clear: map[] len: 0
After append: [10 20 30] len: 3 cap: 5

Program exited.
```

Notice how `clear` behaves differently with slices versus maps. For slices, it sets all elements to their zero value but preserves the length and capacity. For maps, it removes all key-value pairs, reducing the length to zero.

### 6.6.3 Practical applications

These built-in functions are particularly useful in everyday programming scenarios. Here's a practical example that demonstrates how they might be used together:

**Example 25 - Practical use of built-in functions**
```go
package main

import "fmt"

func processScores(scores []int) (int, int, []int) {
	if len(scores) == 0 {
		return 0, 0, scores
	}

	// Find the highest and lowest scores
	highest := max(scores...)
	lowest := min(scores...)

	// Clear the slice to prepare for filtered results
	filtered := make([]int, 0, len(scores))
	
	// Add scores that are above the average of min and max
	threshold := (highest + lowest) / 2
	for _, score := range scores {
		if score > threshold {
			filtered = append(filtered, score)
		}
	}

	return highest, lowest, filtered
}

func main() {
	testScores := []int{85, 92, 78, 96, 88, 74, 91, 83}
	
	high, low, aboveAverage := processScores(testScores)
	
	fmt.Printf("Original scores: %v\n", testScores)
	fmt.Printf("Highest score: %d\n", high)
	fmt.Printf("Lowest score: %d\n", low)
	fmt.Printf("Scores above threshold: %v\n", aboveAverage)
}

// Go Playground: https://go.dev/play/p/[placeholder]
```

Output:
```
Original scores: [85 92 78 96 88 74 91 83]
Highest score: 96
Lowest score: 74
Scores above threshold: [85 92 96 88 91]

Program exited.
```

Note the syntax `max(scores...)` on line 9. The three dots (`...`) are the *spread operator* which unpacks the slice elements as individual arguments to the function. This is necessary because `max` expects individual values, not a slice.

These built-in functions represent Go's continued evolution towards providing developers with essential utilities while maintaining the language's philosophy of simplicity. They eliminate the need for custom implementations or external dependencies for these common operations, making your code more readable and maintainable.

> When you're working with collections and need to find extremes or reset their contents, consider whether these built-in functions can simplify your approach. They're not only more concise but also likely to be more efficient than custom implementations.