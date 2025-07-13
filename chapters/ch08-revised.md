# Chapter 8 - Managing program flow

Now that we've got many of the fundamentals in place, let's look at how we bring it all to life. In this section, we'll cover control structures and error handling: the glue of our program.

## 8.1 Control structures

Programs are dumb. They need to be told how to respond to every input and outcome. We call this *flow control* and we may employ three forms of *control structure* logic in our programs to facilitate this: *sequence*, *selection* and, *iteration*.

*Sequence* logic is linear. Statements are executed one after the other provided expectations are met.

*Selection* logic concerns itself with the conditional flow - what to do when one or more, of several possible outcomes, is satisfied.

*Iteration* logic covers the conditions under which a section of code should be repeatedly run, and of course, when that repetition should finish.

Let's look at how we implement flow control in Go.

### 8.1.1 Sequence logic

With *sequential* flow, execution continues statement by statement until a result is obtained and/or the function returns. Statements are ordered, such that the next builds on the last to achieve a result. Unless there is an error, panic or control signal to terminate, there is nothing to interrupt the execution of the logic.

Though we can call blocks of code asynchronously via a goroutine, the operations performed by such a goroutine would also execute in the same manner: top to bottom.

We may occasionally see the `goto` statement used, which allows for an unconditional jump to a label identifier **within** the same function. 

While it is unconditional, `goto` may be used to act on the result of some expectation or condition. 

In the example below we use `goto` to implement an endless loop which cycles every three seconds.

**Example 64 - Using goto to restart function execution**
```go
package main

import (
	"fmt"
	"time"
)

func main() {

START:

	fmt.Println("Timer running")
	time.Sleep(3 * time.Second)
	goto START
}

// Go Playground: https://go.dev/play/p/5dsPU41vPQ7
```

Output:
```
Timer running
Timer running
Timer running
...
```

This example is included only for completeness. The use of `goto` is not common in Go, nor recommended. It can make code harder to read and reason about. We have alternative control structures we can use to achieve the same as `goto`.

We should also mention *defer* here since it is another exception to the normal sequential flow. The `defer` keyword is used to link associated logic to aid readability and help prevent bugs. 

For instance, consider the operations of opening, reading and then closing a file. Between the opening and closing of the file, there could be many lines of code to process the file contents, the result being that the opening and close operations could become somewhat disassociated.

Failing to close the file would create a bug, but in long sequences of code, the omission of the close operation may be difficult to spot.

The `defer` keyword solves the problem by allowing us to position the open and close operations near one another. Any deferred statements are run before the function returns, if there are multiple deferred statements they are run on a LIFO basis.

```go
    // open file
    src, err := os.Open(filename)
    if err != nil {
        return
    }

    // defer closing file
    defer src.Close()

    // file processing operations
    ...
```

### 8.1.2 Selection logic

Go provides similar conditional statements to those found in other languages:

#### 8.1.2.1 If/else/elseif

If/else statements work as expected, with the addition that in Go, we don't need to wrap the boolean expression in parentheses.

**Example 65 - Basic if/else statement**
```go
package main

import "fmt"

func main() {
	age := 18
	
	if age >= 18 {
		fmt.Println("You are an adult")
	} else {
		fmt.Println("You are a minor")
	}
}

// Go Playground: https://go.dev/play/p/ifElseExample
```

#### 8.1.2.2 Short-form if statement

Go allows you to execute a statement before the condition. This is useful for error checking.

**Example 66 - Short-form if statement**
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	if num, err := strconv.Atoi("42"); err == nil {
		fmt.Printf("Converted number: %d\n", num)
	} else {
		fmt.Printf("Conversion failed: %v\n", err)
	}
}

// Go Playground: https://go.dev/play/p/shortFormIf
```

#### 8.1.2.3 Switch/case/default

In place of *elseif* statements, we can implement *case* statements. We can also use *default* to specify an equivalent to *else* if none of the *case* statement conditions match.

Go's implementation of *switch* is slightly different to other languages. It only runs the matched *case* (and not those that follow) and then implicitly breaks out of the switch statement although we can use the *fallthrough* keyword to emulate how other languages implement *switch*.

Another distinction is that in Go *switch* can operate on variables and constants of any type, and not only integers.

**Example 67 - Simple switch with default**
```go
package main

import "fmt"

func main() {
	myName := "Joe Blogs"

	switch myName {
	case "Joe Blogs":
		fmt.Println("Hi Joe!")
	case "Dave Blogs":
		fmt.Println("Hi Dave!")
	default:
		fmt.Println("Hi there!")
	}
}

// Go Playground: https://go.dev/play/p/3FiTf9lGEDd
```

Output:
```
Hi Joe!

Program exited.
```

**Example 68 - Expressionless switch statement**
```go
package main

import "fmt"

func main() {
	num := 12

	switch {
	case num >= 0 && num <= 10:
		fmt.Println("Between 0 and 10")
	case num >= 10:
		fmt.Println("Greater than 10")
	}
}

// Go Playground: https://go.dev/play/p/cx8jV4QXHRI
```

**Example 69 - Short-form switch with multiple match tests**
```go
package main

import "fmt"

func main() {
	switch letter := "z"; letter {
	case "a", "e", "i", "o", "u":
		fmt.Println("letter was a vowel")
	default:
		fmt.Println("letter was not a vowel")
	}
}

// Go Playground: https://go.dev/play/p/DjPtZbdigfm
```

**Example 70 - Fallthrough to execute the next case**
```go
package main

import "fmt"

func main() {
	day := "Mon"

	switch {
	case day == "Mon":
		fmt.Println("Monday")
		fallthrough
	case day == "Tue":
		fmt.Println("Tuesday")
	case day == "Wed":
		fmt.Println("Wednesday")
	}
}

// Go Playground: https://go.dev/play/p/hw1WLkjYPUb
```

Output:
```
Monday
Tuesday

Program exited.
```

> It is an often-held misconception that *fallthrough* moves to the next case statement and then checks for a match, executing only if the case test is satisfied. Though intuitively this seems sensible, it is not the case - the body of the next case statement is executed whether or not there is a match.

### 8.1.3 Iteration logic

Though many languages offer several constructs for implementing different types of loops, in Go we implement all loops using only `for` loops.

A single `for` keyword keeps matters simple and we're able to emulate all of the looping mechanisms like `do while`, `while` and `for each`, if we know how to construct the `for` loop.

So, let's look at all the different loop implementations we can create.

> Let's not be confused. Though we label the loops as their equivalents in other languages, we're using only `for` to create them. That's the only looping construct we have in Go!

#### 8.1.3.1 Infinite

Infinite loops are achieved using the most basic `for` loop syntax.

**Example 71 - Infinite loop with for**
```go
package main

import "fmt"

func main() {
	// this will timeout on the playground
	for {
		fmt.Println("infinite loop")
	}
}

// Go Playground: https://go.dev/play/p/ZBcHX0L11Qq
```

#### 8.1.3.2 Three Component

Common to most languages is the three-component loop which uses an *init statement*, a *loop condition* and a *post-loop statement* to specify the loop behaviour.

The three-component `for` loop is implemented in the same way in Go.

**Example 72 - Three component loop with for**
```go
package main

import "fmt"

func main() {
	for i := 0; i < 5; i++ {
		fmt.Println("iteration", i+1)
	}
}

// Go Playground: https://go.dev/play/p/IFE1-HGEGkQ
```

#### 8.1.3.3 While equivalent

*While* loops execute while a certain condition exists. In the below example, the condition is that n should be less than or equal to 5. The condition is checked before execution of a loop iteration. Notice how `n` finishes and prints at a value of `6`.

**Example 73 - While equivalent using for**
```go
package main

import "fmt"

func main() {
	n := 1
	for n <= 5 {
		fmt.Println("Iteration", n)
		n++
	}
	fmt.Printf("n finished at %d\n", n)
}

// Go Playground: https://go.dev/play/p/oC25p-YNNBx
```

#### 8.1.3.4 Do while equivalent

A `do while` style loop is very similar to `while` but the condition check is made after the loop iteration. In Go, using `for` alone the equivalent syntax would be like in the below example. Notice that `n` finishes at a value of `5`.

**Example 74 - Do while equivalent with for**
```go
package main

import "fmt"

func main() {
	n := 1
	for ok := true; ok; ok = n != 5 {
		fmt.Println("Iteration", n)
		n++
	}
	fmt.Printf("n finished at %d\n", n)
}

// Go Playground: https://go.dev/play/p/SZUMlRfW3Fn
```

#### 8.1.3.5 For Each (Enhanced in Go 1.23)

*For each* type implementations use the *range* keyword. We've seen *range* in many of the examples to date, and we can use it to iterate over collections such as *arrays*, *slices* and *maps* while getting convenient access to the data contained in the collection.

Go 1.23 enhanced the for-range semantics with cleaner iteration patterns and improved performance characteristics while maintaining full backward compatibility.

**Example 75 - For each performed using idiomatic for range**
```go
package main

import "fmt"

func main() {
	sl := []int{1, 2}
	mp := map[string]string{"key1": "value1", "key2": "value2"}

	// slice/array
	for index, value := range sl {
		fmt.Printf("index: %d, value: %d\n", index, value)
	}

	// map
	for key, value := range mp {
		fmt.Printf("key: %s, value: %s\n", key, value)
	}
}

// Go Playground: https://go.dev/play/p/CVjj4pUtc2c
```

Output:
```
index: 0, value: 1
index: 1, value: 2
key: key1, value: value1
key: key2, value: value2

Program exited.
```

> Beware of trying to mutate a slice element within a `for range` loop using the `value` variable. This won't work. The `value` variable is a copy of the slice element created at each iteration of the `for range` loop. Altering `value` will change it for that iteration only, and will not change the element in the slice itself. To mutate the slice element properly we should use the index position of the element in the slice, so using the example above if we wanted to add 1 to each element in the slice we would do the following:

```go
for index, value := range sl {
	fmt.Printf("index: %d, value: %d\n", index, value)
    sl[index]++ // and not value++
}
```

#### 8.1.3.6 Break & Continue

Go implements `break` and `continue` exactly as in C and similar languages. To exit out of the innermost loop and carry on execution we use `break`. To exit a single iteration and have the loop start the next iteration we use `continue`.

In this example we look for a value (3). When found we break out of the loop and print what we found.

**Example 76 - Using break to exit a loop**
```go
package main

import "fmt"

func main() {
	sl := []int{1, 2, 3, 4, 5}
	found := 0

	for _, v := range sl {
		if v == 3 {
			found = v
			break
		}
	}

	fmt.Println("found:", found)
}

// Go Playground: https://go.dev/play/p/o2vXkdAYNk3
```

In this example we use `continue` to advance to the next iteration if a number is even, so that we only print odd numbers.

**Example 77 - Using continue to advance to next loop iteration**
```go
package main

import "fmt"

func main() {
	sl := []int{1, 2, 3, 4, 5}

	for _, v := range sl {
		if v%2 == 0 {
			continue
		}
		fmt.Printf("%d is an odd number\n", v)
	}
}

// Go Playground: https://go.dev/play/p/qbVGN5S3hLY
```

> Note `break` is also used with `switch/case` statements and `select`, whereas `continue` is only relevant in `for` loops.

#### 8.1.3.7 Range over Functions (Go 1.23)

Go 1.23 introduced a revolutionary new iteration paradigm: the ability to range over functions. This extends Go's range capabilities beyond just data structures to include function-generated sequences, opening up powerful new patterns for iteration and data processing.

This feature allows you to create custom iterators that generate values on demand, providing a clean and efficient way to iterate over computed sequences, filtered data, or any custom iteration logic.

The key insight is that functions can now be "rangeable" if they follow a specific signature pattern: `func(func(T) bool)` where `T` is the type of values yielded during iteration.

**Example 78 - Basic range over function**
```go
package main

import "fmt"

// fibonacci returns a function that generates fibonacci numbers
func fibonacci(max int) func(func(int) bool) {
	return func(yield func(int) bool) {
		a, b := 0, 1
		for a <= max {
			if !yield(a) {
				return
			}
			a, b = b, a+b
		}
	}
}

func main() {
	fmt.Println("Fibonacci numbers up to 100:")
	for n := range fibonacci(100) {
		fmt.Printf("%d ", n)
	}
	fmt.Println()
}

// Go Playground: https://go.dev/play/p/[requires-go-1.23]
```

Output:
```
Fibonacci numbers up to 100:
0 1 1 2 3 5 8 13 21 34 55 89 

Program exited.
```

The `yield` function controls the iteration: returning `true` continues the iteration, while returning `false` terminates it early. This gives consumers complete control over when to stop iterating.

**Example 79 - Range over function with early termination**
```go
package main

import "fmt"

// counter generates numbers from start to end
func counter(start, end int) func(func(int) bool) {
	return func(yield func(int) bool) {
		for i := start; i <= end; i++ {
			if !yield(i) {
				fmt.Printf("\nIteration stopped early at %d\n", i)
				return
			}
		}
		fmt.Println("\nIteration completed normally")
	}
}

func main() {
	fmt.Println("Counting 1-10, but stopping at first even number > 5:")
	for n := range counter(1, 10) {
		fmt.Printf("%d ", n)
		if n > 5 && n%2 == 0 {
			break // This causes yield to return false
		}
	}
}

// Go Playground: https://go.dev/play/p/[requires-go-1.23]
```

This pattern is particularly powerful for creating custom data processing pipelines and lazy evaluation scenarios:

**Example 80 - Practical iterator for filtered data**
```go
package main

import (
	"fmt"
	"strings"
)

// filterLines yields lines from a text that match a predicate
func filterLines(text string, predicate func(string) bool) func(func(string) bool) {
	return func(yield func(string) bool) {
		lines := strings.Split(text, "\n")
		for _, line := range lines {
			line = strings.TrimSpace(line)
			if line != "" && predicate(line) {
				if !yield(line) {
					return
				}
			}
		}
	}
}

// containsWord returns a predicate that checks if a line contains a word
func containsWord(word string) func(string) bool {
	return func(line string) bool {
		return strings.Contains(strings.ToLower(line), strings.ToLower(word))
	}
}

func main() {
	text := `
	Go is a programming language
	Python is also popular
	Go has excellent concurrency
	JavaScript runs in browsers
	Go compiles to native code
	`

	fmt.Println("Lines containing 'Go':")
	for line := range filterLines(text, containsWord("Go")) {
		fmt.Printf("- %s\n", line)
	}
}

// Go Playground: https://go.dev/play/p/[requires-go-1.23]
```

**Example 81 - Two-value iteration with range over functions**
```go
package main

import "fmt"

// enumerate yields (index, value) pairs for a slice
func enumerate[T any](slice []T) func(func(int, T) bool) {
	return func(yield func(int, T) bool) {
		for i, v := range slice {
			if !yield(i, v) {
				return
			}
		}
	}
}

// batch yields slices of a specified size from the input
func batch[T any](slice []T, size int) func(func([]T) bool) {
	return func(yield func([]T) bool) {
		for i := 0; i < len(slice); i += size {
			end := i + size
			if end > len(slice) {
				end = len(slice)
			}
			if !yield(slice[i:end]) {
				return
			}
		}
	}
}

func main() {
	fruits := []string{"apple", "banana", "cherry", "date", "elderberry", "fig"}
	
	fmt.Println("Enumerated fruits:")
	for i, fruit := range enumerate(fruits) {
		fmt.Printf("%d: %s\n", i, fruit)
	}
	
	fmt.Println("\nFruits in batches of 3:")
	for batch := range batch(fruits, 3) {
		fmt.Printf("Batch: %v\n", batch)
	}
}

// Go Playground: https://go.dev/play/p/[requires-go-1.23]
```

Range over functions represents a fundamental shift in how we can approach iteration in Go. It enables:

- **Lazy evaluation**: Values are generated only when needed
- **Memory efficiency**: No need to create entire collections in memory
- **Composability**: Iterators can be chained and combined
- **Custom logic**: Any iteration pattern can be implemented
- **Early termination**: Consumers control when iteration stops

This feature bridges the gap between Go's simplicity and the powerful iteration patterns found in functional programming languages, while maintaining Go's characteristic performance and clarity.

> **Best Practice**: Use range over functions when you need custom iteration logic, lazy evaluation, or when working with large or infinite sequences. The pattern is particularly useful for data processing pipelines, filtering operations, and generating computed sequences on demand.

## 8.2 Error handling

Go's error-handling capabilities have earned it a reputation as one of the most reliable languages for production-level applications.

We said right at the beginning when introducing Go that errors are just values. In Go, there are no exceptions and no *try/catch* type operations.

We're able to create error values, and, decide how we handle the error values we receive, in our program flow. Generally, we'll either log the error if execution should continue, or return from the function/receiver with the error if it should not. As a rule, we shouldn't do both.

The default, zero value of an error value is nil. Only non-nil values are considered to be errors. Function signatures should be designed with the error as the last return value, and it is generally expected that when returning a non-nil error value, any other return values should be set to their *nil* or *empty* representation rather than include data. Error values themselves, should usually be written all in lowercase.

In Go, errors are represented by the built-in type *error*. This type is an interface that defines the behavior of an error, which is any value that can describe itself as a string via an `error()` receiver.

```go
type error interface {
    Error() string
}
```

For many use cases, when we don't need to create custom error types, we'll simply leverage Go's built-in packages for handling errors, and we'll look at how we use those first.

### 8.2.1 Error helpers

The most commonly used error helper package is `errors`. This package provides a simple way to create and manipulate errors. It contains helpers for creating errors, checking for errors, and formatting errors.

We can create an error using `errors.New(string)` this is factory type function for a built-in custom data struct type, `errorString` which implements the `error` interface.

We can see what is happening under the hood by inspecting the package.

```go
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}

func New(text string) error {
    return &errorString{text}
}
```

In the code snippet, `New(string)` creates a pointer of type `errorString`, sets the `s` field of the type and returns `errorString` which is an error because it satisfies the *error* interface.

We can also use `fmt.Errorf()` to generate formatted errors which include data. Ultimately this implementation also wraps `errors.New()`.

#### 8.2.1.1 Predefined errors

We can create predefined errors (often referred to as sentinel errors) in our code using either of the above approaches. We can then use the errors package to inspect the type of error returned using `errors.Is()`. This gives us options about how we handle different classes of error perhaps based on severity: can we log and proceed or should we halt and return. Predefined errors are also useful in unit testing when performing assertions.

In Example 82, we use predefined errors to check what kind of error was received.

**Example 82 - Using errors.Is to handle different error values**
```go
package main

import (
	"errors"
	"fmt"
)

var ErrScoreLessThanMin = errors.New("score is less than minimum")
var ErrScoreOverMax = errors.New("score is greater than maximum")

func checkRating(score int) (int, error) {
	min := 0
	max := 5

	if score < min {
		return 0, ErrScoreLessThanMin
	}
	if score > max {
		return 0, ErrScoreOverMax
	}

	return score, nil
}

func main() {
	rating, err := checkRating(-1)
	if err != nil {
		switch {
		case errors.Is(err, ErrScoreLessThanMin):
			fmt.Println("rating score too low")
		case errors.Is(err, ErrScoreOverMax):
			fmt.Println("rating score too high")
		default:
			fmt.Printf("unexpected error: %s\n", err)
		}
		return
	}
	fmt.Printf("Rating : %d\n", rating)
}

// Go Playground: https://go.dev/play/p/gb2eikJ9Hp6
```

### 8.2.2 Custom error types

Custom error types allow us to convey more specific information about the error.

To create a custom error type, we need to create a new type that implements the error interface. This can be achieved by simply defining a new type that has an `Error() string` receiver.

**Example 83 - Custom error type**
```go
package main

import "fmt"

type ValidationError struct {
	Field   string
	Message string
}

func (e ValidationError) Error() string {
	return fmt.Sprintf("validation failed on field '%s': %s", e.Field, e.Message)
}

func validateAge(age int) error {
	if age < 0 {
		return ValidationError{Field: "age", Message: "cannot be negative"}
	}
	if age > 150 {
		return ValidationError{Field: "age", Message: "unrealistic value"}
	}
	return nil
}

func main() {
	if err := validateAge(-5); err != nil {
		fmt.Println("Error:", err)
		
		// Type assertion to access custom fields
		if ve, ok := err.(ValidationError); ok {
			fmt.Printf("Field: %s\n", ve.Field)
			fmt.Printf("Message: %s\n", ve.Message)
		}
	}
}

// Go Playground: https://go.dev/play/p/customErrorType
```

### 8.2.3 Error wrapping

Go 1.13 introduced error wrapping, which allows you to add context to errors while preserving the original error for inspection.

**Example 84 - Error wrapping with fmt.Errorf**
```go
package main

import (
	"errors"
	"fmt"
)

func readConfig(filename string) error {
	err := openFile(filename)
	if err != nil {
		return fmt.Errorf("failed to read config from %s: %w", filename, err)
	}
	return nil
}

func openFile(filename string) error {
	return errors.New("file not found")
}

func main() {
	err := readConfig("config.yaml")
	if err != nil {
		fmt.Println("Error:", err)
		
		// Unwrap to get the original error
		originalErr := errors.Unwrap(err)
		if originalErr != nil {
			fmt.Println("Original error:", originalErr)
		}
		
		// Check if the wrapped error is a specific type
		if errors.Is(err, errors.New("file not found")) {
			fmt.Println("This is a file not found error")
		}
	}
}

// Go Playground: https://go.dev/play/p/errorWrapping
```

These error handling patterns provide a robust foundation for building reliable Go applications. The combination of simple error values, custom types, and error wrapping gives you the flexibility to handle errors appropriately for your specific use case while maintaining code clarity and debuggability.