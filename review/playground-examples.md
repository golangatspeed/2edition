# Go Playground Examples for Chapter 6 Built-in Functions

## Example 23 - Using min and max functions
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
```
**Playground URL**: [To be created when testing live]

## Example 24 - Using the clear function
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
```
**Playground URL**: [To be created when testing live]

## Example 25 - Practical use of built-in functions
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
```
**Playground URL**: [To be created when testing live]

## Notes for Author
These examples follow your established patterns:
- Simple, focused code that demonstrates specific concepts
- Clear comments explaining what's happening
- Practical examples that developers would actually use
- Consistent with your Go Playground integration approach

The examples need to be tested on Go Playground to get the actual URLs. The functions `min`, `max`, and `clear` are available in Go 1.21+.