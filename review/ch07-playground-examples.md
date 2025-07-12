# Go Playground Examples for Chapter 7 New Features

## Generic Type Aliases Examples

### Example 58 - Generic type aliases
```go
package main

import "fmt"

// Traditional generic type definition
type Map[K comparable, V any] map[K]V

// Generic type alias (Go 1.24+)
type StringMap[V any] = map[string]V
type IntSlice[T any] = []T

func main() {
	// Using generic type alias for string-keyed maps
	users := StringMap[string]{
		"user1": "Alice",
		"user2": "Bob",
		"user3": "Charlie",
	}
	
	// Using generic type alias for typed slices
	numbers := IntSlice[int]{1, 2, 3, 4, 5}
	names := IntSlice[string]{"Alice", "Bob", "Charlie"}
	
	fmt.Println("Users:", users)
	fmt.Println("Numbers:", numbers)
	fmt.Println("Names:", names)
}
```
**Playground URL**: [To be tested on Go 1.24+]

### Example 59 - Complex generic type aliases
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
```
**Playground URL**: [To be tested on Go 1.24+]

## math/rand/v2 Examples

### Example 65 - Comparing math/rand vs math/rand/v2
```go
package main

import (
	"fmt"
	oldrand "math/rand"
	"math/rand/v2"
	"time"
)

func main() {
	// Old way (math/rand) - requires manual seeding
	oldrand.Seed(time.Now().UnixNano())
	fmt.Println("Old math/rand package:")
	fmt.Printf("Random int: %d\n", oldrand.Int())
	fmt.Printf("Random float: %.3f\n", oldrand.Float64())
	fmt.Printf("Random range [1-10]: %d\n", oldrand.Intn(10)+1)
	
	fmt.Println("\nNew math/rand/v2 package:")
	// New way (math/rand/v2) - auto-seeded, better API
	fmt.Printf("Random int: %d\n", rand.Int())
	fmt.Printf("Random float: %.3f\n", rand.Float64())
	fmt.Printf("Random range [1-10]: %d\n", rand.IntN(10)+1)
	
	// v2 provides direct range methods
	fmt.Printf("Random uint32: %d\n", rand.Uint32())
	fmt.Printf("Random uint64: %d\n", rand.Uint64())
}
```
**Playground URL**: [To be tested on Go 1.22+]

### Example 66 - Practical usage with math/rand/v2
```go
package main

import (
	"fmt"
	"math/rand/v2"
)

// generateRandomUser creates a user with random data
func generateRandomUser() map[string]interface{} {
	firstNames := []string{"Alice", "Bob", "Charlie", "Diana", "Eve", "Frank"}
	lastNames := []string{"Smith", "Johnson", "Williams", "Brown", "Jones", "Garcia"}
	domains := []string{"example.com", "test.org", "demo.net"}
	
	firstName := firstNames[rand.IntN(len(firstNames))]
	lastName := lastNames[rand.IntN(len(lastNames))]
	domain := domains[rand.IntN(len(domains))]
	
	return map[string]interface{}{
		"id":       rand.Uint64(),
		"name":     fmt.Sprintf("%s %s", firstName, lastName),
		"email":    fmt.Sprintf("%s.%s@%s", firstName, lastName, domain),
		"age":      rand.IntN(50) + 18, // Age between 18-67
		"score":    rand.Float64() * 100, // Score 0-100
		"active":   rand.IntN(2) == 1, // Random boolean
	}
}

func main() {
	fmt.Println("Generated random users:")
	for i := 0; i < 3; i++ {
		user := generateRandomUser()
		fmt.Printf("\nUser %d:\n", i+1)
		for key, value := range user {
			fmt.Printf("  %s: %v\n", key, value)
		}
	}
}
```
**Playground URL**: [To be tested on Go 1.22+]

### Example 67 - Deterministic randomness for testing
```go
package main

import (
	"fmt"
	"math/rand/v2"
)

func rollDice(rng *rand.Rand) int {
	return rng.IntN(6) + 1
}

func main() {
	// Create a deterministic random source for testing
	source := rand.NewPCG(12345, 67890) // Fixed seeds
	rng := rand.New(source)
	
	fmt.Println("Deterministic dice rolls (for testing):")
	for i := 0; i < 5; i++ {
		roll := rollDice(rng)
		fmt.Printf("Roll %d: %d\n", i+1, roll)
	}
	
	// Reset with same seeds - will produce identical sequence
	source = rand.NewPCG(12345, 67890)
	rng = rand.New(source)
	
	fmt.Println("\nSame sequence with reset:")
	for i := 0; i < 5; i++ {
		roll := rollDice(rng)
		fmt.Printf("Roll %d: %d\n", i+1, roll)
	}
}
```
**Playground URL**: [To be tested on Go 1.22+]

## Notes for Author

### Integration Approach
- **Generic Type Aliases**: Added as subsection 7.5.2 in "Creating custom types" - natural progression after basic custom types
- **math/rand/v2**: Added as new section 7.7 "Modern standard library packages" - sets precedent for future v2 packages

### Style Consistency
- Maintained your conversational, mentoring tone throughout
- Used same example numbering sequence (continuing from existing examples)
- Followed your pattern of practical, real-world demonstrations
- Included information blurbs with key insights
- Preserved your encouraging teaching approach

### Technical Accuracy
- Generic type aliases require Go 1.24 (stable release)
- math/rand/v2 requires Go 1.22
- All examples demonstrate practical usage patterns
- Migration guidance provided for real-world adoption

### Testing Notes
- Generic type alias examples need Go 1.24+ environment
- math/rand/v2 examples need Go 1.22+ environment  
- Some features may not work in older Go Playground versions
- Consider noting Go version requirements in final examples