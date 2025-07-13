# Chapter 9 - Digging deeper

Now that we have a good grasp of Go fundamentals, next we will delve deeper into the language and explore more advanced concepts. We'll expand upon previously covered topics and introduce new techniques such as assertion, reflection, and advanced error handling patterns, tools which should enhance our abilities as Go programmers.

## 9.1 Developing with functions

We've used functions a few times already. We've called functions from standard library packages such as `log` and `fmt` and we've also created functions in the examples ourselves.

Next, we'll focus on some of the less common aspects of functions in programming languages, features which we do have in Go. Specifically, we'll focus on function signatures including variadic arguments, and return styles, including multiple returns.

> Note, that what we cover here, is equally applicable when working with *receivers*.

### 9.1.1 Function parameters

Functions often accept values passed to satisfy parameters in the function signature. 

In Go, when defining function parameters we need to specify both the variable name and its type.  To shorten the signature somewhat we can group parameters of the same type if it makes sense to do so, and it **isn't** detrimental to the API.

The snippets below show ungrouped and grouped parameters.  See how in this case, grouping makes the ordering less intuitive.

```go
// ungrouped parameters
func normalParams(name string, age int, houseNumber int, address1 string, address2 string){
    ...
}

// grouped parameters, which makes the API a bit awkward in this example
func groupedParams(name, address1, address2 string, houseNumber, age int){
    ...
}
```

Long function signatures can also be split with line breaks to improve readability. No escape character is needed when doing so.

```go
// split over lines
func normalParams(
	name string, 
	age int, 
	houseNumber int, 
	address1 string, 
	address2 string,
){
    ...
}
```

### 9.1.2 Variadic arguments

Go allows us to write functions that can take any number of arguments of the same type. These are called *variadic functions*.

To make a parameter variadic, we use the three dot `...` syntax followed by the type. The parameter becomes a slice of that type inside the function.

**Example 65 - Variadic arguments**
```go
package main

import "fmt"

func sum(numbers ...int) int {
	total := 0
	for _, num := range numbers {
		total += num
	}
	return total
}

func main() {
	fmt.Println(sum(1, 2, 3))
	fmt.Println(sum(1, 2, 3, 4, 5))
	
	// We can also pass a slice
	nums := []int{10, 20, 30}
	fmt.Println(sum(nums...)) // Note the ... when passing a slice
}

// Go Playground: https://go.dev/play/p/variadicExample
```

### 9.1.3 Multiple return values

Go functions can return more than one value. This is commonly used for error handling, where a function returns both a result and an error.

**Example 66 - Multiple return values**
```go
package main

import (
	"fmt"
	"strconv"
)

func divide(a, b float64) (float64, error) {
	if b == 0 {
		return 0, fmt.Errorf("division by zero")
	}
	return a / b, nil
}

func main() {
	result, err := divide(10, 2)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Result:", result)
}

// Go Playground: https://go.dev/play/p/multipleReturn
```

### 9.1.4 Named return values

Go allows you to name return values. When you do this, they are treated as variables defined at the top of the function.

**Example 67 - Named return values**
```go
package main

import "fmt"

func rectangleArea(length, width float64) (area float64) {
	area = length * width
	return // naked return - returns the named variable
}

func main() {
	area := rectangleArea(5, 3)
	fmt.Println("Area:", area)
}

// Go Playground: https://go.dev/play/p/namedReturn
```

### 9.1.5 Functions as types

In Go, functions are first-class citizens. This means they can be:
- Assigned to variables
- Passed as arguments to other functions
- Returned from functions

**Example 68 - Functions as types**
```go
package main

import "fmt"

// Define a function type
type operation func(int, int) int

func add(a, b int) int {
	return a + b
}

func multiply(a, b int) int {
	return a * b
}

func calculate(a, b int, op operation) int {
	return op(a, b)
}

func main() {
	// Assign functions to variables
	var op operation = add
	fmt.Println("Add:", calculate(5, 3, op))
	
	// Pass function directly
	fmt.Println("Multiply:", calculate(5, 3, multiply))
	
	// Anonymous function
	fmt.Println("Subtract:", calculate(5, 3, func(a, b int) int {
		return a - b
	}))
}

// Go Playground: https://go.dev/play/p/functionsAsTypes
```

## 9.2 Memory management

Understanding how Go manages memory is crucial for writing efficient programs. Go uses a garbage collector to automatically manage memory, but it's still important to understand the concepts of stack and heap allocation.

### 9.2.1 Stack vs heap

The stack is used for local variables and function parameters. It's fast and automatically managed. The heap is used for dynamically allocated memory that may need to persist beyond the function that created it.

Go's compiler performs escape analysis to determine whether a variable should be allocated on the stack or heap.

### 9.2.2 Escape analysis

Escape analysis determines whether variables can be allocated on the stack or must be moved to the heap. You can see the compiler's decisions using the `-gcflags="-m"` flag.

**Example 69 - Escape analysis**
```go
package main

import "fmt"

func createPointer() *int {
	x := 42 // This will escape to heap because we return a pointer to it
	return &x
}

func noEscape() {
	x := 42 // This stays on stack
	fmt.Println(x)
}

func main() {
	ptr := createPointer()
	fmt.Println(*ptr)
	noEscape()
}

// To see escape analysis: go run -gcflags="-m" main.go
// Go Playground: https://go.dev/play/p/escapeAnalysis
```

## 9.3 Interfaces and type assertions

Interfaces are one of Go's most powerful features. They define a contract that types must fulfill, enabling polymorphism and loose coupling.

### 9.3.1 Interface basics

An interface is defined by a set of method signatures. Any type that implements all the methods of an interface automatically satisfies that interface.

**Example 70 - Basic interfaces**
```go
package main

import "fmt"

type Speaker interface {
	Speak() string
}

type Dog struct {
	Name string
}

func (d Dog) Speak() string {
	return d.Name + " says woof!"
}

type Cat struct {
	Name string
}

func (c Cat) Speak() string {
	return c.Name + " says meow!"
}

func makeSound(s Speaker) {
	fmt.Println(s.Speak())
}

func main() {
	dog := Dog{Name: "Buddy"}
	cat := Cat{Name: "Whiskers"}
	
	makeSound(dog)
	makeSound(cat)
}

// Go Playground: https://go.dev/play/p/basicInterface
```

### 9.3.2 Type assertions

Type assertions allow you to extract the underlying concrete type from an interface value.

**Example 71 - Type assertions**
```go
package main

import "fmt"

func checkType(i interface{}) {
	// Type assertion with safety check
	if str, ok := i.(string); ok {
		fmt.Printf("It's a string: %s\n", str)
	} else if num, ok := i.(int); ok {
		fmt.Printf("It's an int: %d\n", num)
	} else {
		fmt.Printf("Unknown type: %T\n", i)
	}
}

func main() {
	checkType("hello")
	checkType(42)
	checkType(3.14)
}

// Go Playground: https://go.dev/play/p/typeAssertion
```

### 9.3.3 Type switches

Type switches provide a cleaner way to handle multiple type assertions.

**Example 72 - Type switches**
```go
package main

import "fmt"

func describe(i interface{}) {
	switch v := i.(type) {
	case string:
		fmt.Printf("String: %s (length: %d)\n", v, len(v))
	case int:
		fmt.Printf("Integer: %d\n", v)
	case bool:
		fmt.Printf("Boolean: %t\n", v)
	default:
		fmt.Printf("Unknown type: %T\n", v)
	}
}

func main() {
	describe("hello")
	describe(42)
	describe(true)
	describe(3.14)
}

// Go Playground: https://go.dev/play/p/typeSwitch
```

## 9.4 Reflection

Reflection allows a program to examine its own structure at runtime. Go's reflection is provided by the `reflect` package.

### 9.4.1 Basic reflection

**Example 73 - Basic reflection**
```go
package main

import (
	"fmt"
	"reflect"
)

func examineValue(x interface{}) {
	v := reflect.ValueOf(x)
	t := reflect.TypeOf(x)
	
	fmt.Printf("Value: %v\n", v)
	fmt.Printf("Type: %v\n", t)
	fmt.Printf("Kind: %v\n", v.Kind())
}

func main() {
	examineValue(42)
	examineValue("hello")
	examineValue([]int{1, 2, 3})
}

// Go Playground: https://go.dev/play/p/basicReflection
```

### 9.4.2 Reflection with structs

Reflection is particularly useful when working with structs, allowing you to examine fields and tags.

**Example 74 - Struct reflection**
```go
package main

import (
	"fmt"
	"reflect"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func examineStruct(x interface{}) {
	v := reflect.ValueOf(x)
	t := reflect.TypeOf(x)
	
	if v.Kind() != reflect.Struct {
		fmt.Println("Not a struct")
		return
	}
	
	for i := 0; i < v.NumField(); i++ {
		field := v.Field(i)
		fieldType := t.Field(i)
		
		fmt.Printf("Field: %s, Type: %s, Value: %v, Tag: %s\n",
			fieldType.Name, field.Type(), field.Interface(), fieldType.Tag.Get("json"))
	}
}

func main() {
	p := Person{Name: "Alice", Age: 30}
	examineStruct(p)
}

// Go Playground: https://go.dev/play/p/structReflection
```

## 9.5 Error handling patterns

Error handling is a crucial aspect of Go programming. Let's explore some advanced patterns for dealing with errors.

### 9.5.1 Custom error types

**Example 75 - Custom error types**
```go
package main

import (
	"fmt"
)

type ValidationError struct {
	Field   string
	Message string
}

func (e ValidationError) Error() string {
	return fmt.Sprintf("validation error in field '%s': %s", e.Field, e.Message)
}

func validateAge(age int) error {
	if age < 0 {
		return ValidationError{Field: "age", Message: "cannot be negative"}
	}
	if age > 150 {
		return ValidationError{Field: "age", Message: "seems unrealistic"}
	}
	return nil
}

func main() {
	if err := validateAge(-5); err != nil {
		// Type assertion to access custom error fields
		if ve, ok := err.(ValidationError); ok {
			fmt.Printf("Field: %s, Message: %s\n", ve.Field, ve.Message)
		}
		fmt.Println("Error:", err)
	}
}

// Go Playground: https://go.dev/play/p/customError
```

### 9.5.2 Error wrapping

Go 1.13 introduced error wrapping, which allows you to add context to errors while preserving the original error.

**Example 76 - Error wrapping**
```go
package main

import (
	"errors"
	"fmt"
)

func processFile(filename string) error {
	err := readFile(filename)
	if err != nil {
		return fmt.Errorf("failed to process file %s: %w", filename, err)
	}
	return nil
}

func readFile(filename string) error {
	// Simulate file reading error
	return errors.New("file not found")
}

func main() {
	err := processFile("data.txt")
	if err != nil {
		fmt.Println("Error:", err)
		
		// Unwrap to get the original error
		originalErr := errors.Unwrap(err)
		fmt.Println("Original error:", originalErr)
		
		// Check if the error is a specific type
		if errors.Is(err, errors.New("file not found")) {
			fmt.Println("This is a file not found error")
		}
	}
}

// Go Playground: https://go.dev/play/p/errorWrapping
```

### 9.5.3 Joining multiple errors

Go 1.20 introduced `errors.Join()`, a powerful function that allows you to combine multiple errors into a single error. This is particularly useful when you need to collect multiple validation errors or when operations can fail in multiple ways simultaneously.

The joined error implements the `Unwrap() []error` method, which means you can use `errors.Is()` and `errors.As()` to check for any of the wrapped errors.

**Example 77 - Basic error joining**
```go
package main

import (
	"errors"
	"fmt"
)

func validateUser(name string, age int, email string) error {
	var errs []error
	
	if name == "" {
		errs = append(errs, errors.New("name cannot be empty"))
	}
	
	if age < 0 || age > 150 {
		errs = append(errs, errors.New("age must be between 0 and 150"))
	}
	
	if email == "" {
		errs = append(errs, errors.New("email cannot be empty"))
	}
	
	if len(errs) > 0 {
		return errors.Join(errs...)
	}
	
	return nil
}

func main() {
	err := validateUser("", -5, "")
	if err != nil {
		fmt.Println("Validation failed:")
		fmt.Println(err)
	}
}

// Go Playground: https://go.dev/play/p/errorsJoinBasic
```

Output:
```
Validation failed:
name cannot be empty
age must be between 0 and 150
email cannot be empty
```

Notice how `errors.Join()` creates a single error that contains all the individual error messages, each on its own line. This provides a clean way to present multiple validation failures to users.

**Example 78 - Error joining with checking**
```go
package main

import (
	"errors"
	"fmt"
)

var (
	ErrInvalidName  = errors.New("invalid name")
	ErrInvalidAge   = errors.New("invalid age")
	ErrInvalidEmail = errors.New("invalid email")
)

func validateUserTyped(name string, age int, email string) error {
	var errs []error
	
	if name == "" {
		errs = append(errs, fmt.Errorf("name validation failed: %w", ErrInvalidName))
	}
	
	if age < 0 || age > 150 {
		errs = append(errs, fmt.Errorf("age validation failed: %w", ErrInvalidAge))
	}
	
	if email == "" || !contains(email, "@") {
		errs = append(errs, fmt.Errorf("email validation failed: %w", ErrInvalidEmail))
	}
	
	return errors.Join(errs...)
}

func contains(s, substr string) bool {
	for i := 0; i <= len(s)-len(substr); i++ {
		if s[i:i+len(substr)] == substr {
			return true
		}
	}
	return false
}

func main() {
	err := validateUserTyped("", 200, "invalid-email")
	if err != nil {
		fmt.Println("Validation errors occurred:")
		fmt.Println(err)
		fmt.Println()
		
		// Check for specific error types
		if errors.Is(err, ErrInvalidName) {
			fmt.Println("❌ Name validation failed")
		}
		if errors.Is(err, ErrInvalidAge) {
			fmt.Println("❌ Age validation failed")
		}
		if errors.Is(err, ErrInvalidEmail) {
			fmt.Println("❌ Email validation failed")
		}
	}
}

// Go Playground: https://go.dev/play/p/errorsJoinChecking
```

This example demonstrates how `errors.Join()` preserves the ability to check for specific error types using `errors.Is()`. Even though the errors are joined, you can still detect individual error conditions.

**Example 79 - Practical file processing with error joining**
```go
package main

import (
	"errors"
	"fmt"
)

type FileProcessor struct {
	files []string
}

func NewFileProcessor(files []string) *FileProcessor {
	return &FileProcessor{files: files}
}

func (fp *FileProcessor) ProcessFiles() error {
	var errs []error
	
	for _, file := range fp.files {
		if err := fp.processFile(file); err != nil {
			errs = append(errs, fmt.Errorf("failed to process %s: %w", file, err))
		}
	}
	
	if len(errs) > 0 {
		return fmt.Errorf("file processing completed with errors: %w", errors.Join(errs...))
	}
	
	return nil
}

func (fp *FileProcessor) processFile(filename string) error {
	// Simulate different types of file processing errors
	switch filename {
	case "missing.txt":
		return errors.New("file not found")
	case "corrupted.txt":
		return errors.New("file corrupted")
	case "locked.txt":
		return errors.New("file locked by another process")
	default:
		return nil // Success
	}
}

func main() {
	files := []string{"good.txt", "missing.txt", "corrupted.txt", "another-good.txt", "locked.txt"}
	processor := NewFileProcessor(files)
	
	err := processor.ProcessFiles()
	if err != nil {
		fmt.Println("Processing completed with errors:")
		fmt.Println(err)
		fmt.Println()
		
		// You could also iterate through individual errors
		var joinedErr interface{ Unwrap() []error }
		if errors.As(err, &joinedErr) {
			fmt.Println("Individual errors:")
			for i, individualErr := range joinedErr.Unwrap() {
				fmt.Printf("%d. %v\n", i+1, individualErr)
			}
		}
	} else {
		fmt.Println("All files processed successfully!")
	}
}

// Go Playground: https://go.dev/play/p/errorsJoinPractical
```

This practical example shows how `errors.Join()` is particularly useful when processing multiple items where you want to:

1. **Continue processing** even when some items fail
2. **Collect all errors** rather than stopping at the first failure  
3. **Provide comprehensive feedback** about what went wrong
4. **Maintain the ability** to check for specific error types

The `errors.Join()` function represents a significant improvement in Go's error handling capabilities, making it easier to handle scenarios where multiple errors can occur and you need to present them as a cohesive unit while preserving the ability to inspect individual errors.

> **Best Practice**: Use `errors.Join()` when you need to collect multiple related errors, especially in validation scenarios or when processing collections of items. This approach provides better user experience by showing all issues at once rather than requiring multiple round-trips to fix problems one at a time.