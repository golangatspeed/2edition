# Chapter 14 - Modern Go Development

As Go continues to mature, it's exciting to see how the language has evolved while maintaining its core principles of simplicity and clarity. When I started working with Go, the language felt refreshingly straightforward compared to other languages I'd used. Now, several years on, Go has grown significantly more powerful while keeping that same approachable nature that attracted me initially.

In this chapter, we'll explore some of the modern features and practices that have emerged as Go has evolved from version 1.19 to 1.25. These aren't just incremental improvements – they represent thoughtful enhancements that solve real problems developers face daily.

## 14.1 An evolving standard library

One of the things I've come to appreciate about Go is how the language manages to evolve without breaking existing code. The Go team's commitment to backwards compatibility means your Go 1.0 code still compiles and runs with Go 1.25, but that doesn't mean the language has stagnated. Instead, we've seen a sophisticated approach to evolution through versioned packages and carefully designed additions.

### 14.1.1 The math/rand/v2 package

Go 1.22 introduced something quite significant: the first "v2" package in the standard library with `math/rand/v2`. This represents a major shift in how Go can evolve its APIs without breaking existing code.

The original `math/rand` package had some gotchas that caught many developers off guard. You had to remember to seed the random number generator, and the API had some inconsistencies that made it easy to make mistakes. The v2 package addresses these issues while providing better performance and a cleaner interface.

**Example 82 - Comparing math/rand vs math/rand/v2**
```go
package main

import (
	"fmt"
	oldrand "math/rand"
	"math/rand/v2"
	"time"
)

func main() {
	// Old way - requires manual seeding
	oldrand.Seed(time.Now().UnixNano())
	fmt.Println("Old math/rand:")
	fmt.Printf("Random int: %d\n", oldrand.Int())
	fmt.Printf("Random range [1-10]: %d\n", oldrand.Intn(10)+1)
	
	// New way - auto-seeded, better API
	fmt.Println("\nNew math/rand/v2:")
	fmt.Printf("Random int: %d\n", rand.Int())
	fmt.Printf("Random range [1-10]: %d\n", rand.IntN(10)+1)
}

// Go Playground: https://go.dev/play/p/mathRandV2Compare
```

Notice the improvements: no manual seeding required, and `IntN(n)` instead of `Intn(n)` which follows better naming conventions. These changes might seem small, but they eliminate common sources of bugs.

**Example 83 - Practical random data generation**
```go
package main

import (
	"fmt"
	"math/rand/v2"
)

func generateRandomUser() map[string]interface{} {
	firstNames := []string{"Alice", "Bob", "Charlie", "Diana"}
	lastNames := []string{"Smith", "Johnson", "Williams", "Brown"}
	
	firstName := firstNames[rand.IntN(len(firstNames))]
	lastName := lastNames[rand.IntN(len(lastNames))]
	
	return map[string]interface{}{
		"id":    rand.Uint64(),
		"name":  fmt.Sprintf("%s %s", firstName, lastName),
		"age":   rand.IntN(50) + 18, // Age between 18-67
		"score": rand.Float64() * 100,
	}
}

func main() {
	for i := 0; i < 3; i++ {
		user := generateRandomUser()
		fmt.Printf("User %d: %v\n", i+1, user)
	}
}

// Go Playground: https://go.dev/play/p/randomUserGeneration
```

### 14.1.2 The json/v2 package evolution

While `json/v2` is still in development as of Go 1.25, it represents the next major evolution in JSON handling. The current `encoding/json` package has served us well, but years of real-world usage have revealed areas for improvement.

The new package promises better performance, more intuitive semantics, and cleaner handling of edge cases. When it becomes available, it will follow the same versioned approach as `math/rand/v2`, allowing gradual migration without breaking existing code.

This pattern – introducing v2 packages that coexist with their v1 counterparts – represents Go's sophisticated approach to evolution. You get the stability of backwards compatibility alongside access to improved APIs.

> This versioned package approach shows Go's maturity as a language ecosystem. Rather than forcing major version bumps that break the world, Go provides a path for API evolution that respects existing codebases while enabling progress.

## 14.2 Structured logging with slog

If you've been writing Go applications for a while, you've probably struggled with logging consistency across your codebase. Different packages use different logging approaches, making it difficult to maintain coherent log output. Go 1.21 introduced the `slog` package to address this challenge.

### 14.2.1 Why structured logging matters

When I first started building Go applications, I used `fmt.Printf` statements for debugging and maybe `log.Printf` for more serious logging. This works fine for small programs, but as applications grow, unstructured logs become difficult to search, parse, and analyse.

Structured logging means treating log entries as data rather than just text. Each log entry contains key-value pairs that can be easily processed by log analysis tools.

**Example 84 - Basic structured logging with slog**
```go
package main

import (
	"log/slog"
	"os"
)

func main() {
	// Create a structured logger
	logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
	
	// Basic logging with structure
	logger.Info("User login attempt",
		"user_id", "user123",
		"ip_address", "192.168.1.100",
		"success", true)
	
	logger.Warn("High memory usage",
		"memory_percent", 85.5,
		"service", "web-server")
	
	logger.Error("Database connection failed",
		"error", "connection timeout",
		"database", "users_db",
		"retry_count", 3)
}

// Go Playground: https://go.dev/play/p/basicSlogUsage
```

### 14.2.2 Custom log handlers

The power of `slog` comes from its handler system. You can create custom handlers that format logs exactly how you need them.

**Example 85 - Using different log handlers**
```go
package main

import (
	"log/slog"
	"os"
)

func main() {
	// Text handler for development
	textLogger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelDebug,
	}))
	
	// JSON handler for production
	jsonLogger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelInfo,
	}))
	
	textLogger.Debug("Development debug message", "component", "auth")
	jsonLogger.Info("Production info message", "transaction_id", "txn_456")
}

// Go Playground: https://go.dev/play/p/slogHandlers
```

The structured approach makes logs much more useful for monitoring and debugging production systems. You can easily filter, aggregate, and alert on specific log attributes.

## 14.3 WebAssembly improvements

WebAssembly support in Go has been steadily improving, and recent versions have introduced features that make it much more practical for real-world use. While WASM might not be something you use every day, it's worth understanding because it opens up new deployment possibilities.

### 14.3.1 Export functions from Go to WASM

Go 1.24 introduced the `go:wasmexport` directive, which allows you to expose Go functions to the WebAssembly host environment. This makes it much easier to create Go libraries that can be called from JavaScript.

**Example 86 - Exporting Go functions to WebAssembly**
```go
//go:build wasm

package main

import (
	"fmt"
	"syscall/js"
)

//go:wasmexport calculate
func calculate(a, b int) int {
	return a * b + 10
}

//go:wasmexport greet
func greet(name string) string {
	return fmt.Sprintf("Hello, %s from Go!", name)
}

func main() {
	// Keep the program running
	select {}
}

// This compiles to WASM with: GOOS=js GOARCH=wasm go build -o app.wasm
```

This improvement makes Go a more viable option for web-based applications where you need high-performance computations but want to run in the browser.

## 14.4 Modern API patterns

As Go has matured, certain patterns have emerged for building robust APIs. These aren't language features per se, but they represent best practices that have evolved from years of Go development in production.

### 14.4.1 Context-aware APIs

One pattern I've seen become standard in modern Go APIs is the consistent use of context. Nearly every public function that might block or perform I/O should accept a context parameter.

**Example 87 - Context-aware API design**
```go
package main

import (
	"context"
	"fmt"
	"time"
)

type UserService struct {
	// Implementation details
}

func (s *UserService) GetUser(ctx context.Context, userID string) (*User, error) {
	// Check if context is cancelled
	select {
	case <-ctx.Done():
		return nil, ctx.Err()
	default:
	}
	
	// Simulate database operation with context
	return s.fetchUserFromDB(ctx, userID)
}

func (s *UserService) fetchUserFromDB(ctx context.Context, userID string) (*User, error) {
	// Simulate work that respects context cancellation
	done := make(chan *User, 1)
	
	go func() {
		time.Sleep(100 * time.Millisecond) // Simulate DB query
		done <- &User{ID: userID, Name: "Alice"}
	}()
	
	select {
	case user := <-done:
		return user, nil
	case <-ctx.Done():
		return nil, ctx.Err()
	}
}

type User struct {
	ID   string
	Name string
}

func main() {
	service := &UserService{}
	ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
	defer cancel()
	
	user, err := service.GetUser(ctx, "user123")
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		return
	}
	fmt.Printf("User: %+v\n", user)
}

// Go Playground: https://go.dev/play/p/contextAwareAPI
```

### 14.4.2 Options pattern evolution

The options pattern has become a standard way to handle functions with many optional parameters. Modern implementations often use generic constraints to make them more type-safe.

**Example 88 - Modern options pattern**
```go
package main

import "fmt"

type Server struct {
	host    string
	port    int
	timeout int
	debug   bool
}

type Option func(*Server)

func WithHost(host string) Option {
	return func(s *Server) {
		s.host = host
	}
}

func WithPort(port int) Option {
	return func(s *Server) {
		s.port = port
	}
}

func WithTimeout(timeout int) Option {
	return func(s *Server) {
		s.timeout = timeout
	}
}

func WithDebug(debug bool) Option {
	return func(s *Server) {
		s.debug = debug
	}
}

func NewServer(opts ...Option) *Server {
	// Default values
	server := &Server{
		host:    "localhost",
		port:    8080,
		timeout: 30,
		debug:   false,
	}
	
	// Apply options
	for _, opt := range opts {
		opt(server)
	}
	
	return server
}

func main() {
	// Create server with custom options
	server := NewServer(
		WithHost("0.0.0.0"),
		WithPort(3000),
		WithDebug(true),
	)
	
	fmt.Printf("Server config: %+v\n", server)
}

// Go Playground: https://go.dev/play/p/modernOptionsPattern
```

## 14.5 Looking forward

What I find exciting about Go's evolution is how it maintains the language's core philosophy while adding genuinely useful features. The additions we've covered – versioned packages, structured logging, improved WebAssembly support, and refined patterns – all solve real problems without adding unnecessary complexity.

The versioned package approach, in particular, represents a sophisticated solution to the problem of API evolution. Rather than breaking changes that force ecosystem-wide updates, Go provides a path for gradual improvement that respects existing code while enabling progress.

As you build modern Go applications, these features and patterns can help you write more maintainable, observable, and robust software. The key is to adopt them thoughtfully – not because they're new, but because they solve real problems in your specific context.

> Modern Go development isn't about using every new feature available. It's about understanding the tools at your disposal and choosing the right ones for your specific challenges. The language's continued evolution gives us better tools, but the principles of clear, simple code remain as important as ever.