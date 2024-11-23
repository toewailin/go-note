## **1. Setting Up Go**

### Installation
1. Download and install Go from [https://golang.org/dl/](https://golang.org/dl/).
2. Verify installation:
   ```bash
   go version
   ```
3. Set up your workspace:
   - The default workspace is `$HOME/go`.
   - Code is typically placed in `$HOME/go/src`.

4. Check environment variables:
   ```bash
   echo $GOPATH
   echo $GOROOT
   ```

---

## **2. Hello World**

### Code Structure
- Every Go program starts with the `main` package.
- The `main` function is the entry point.

### Example:
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

### Run the program:
```bash
go run main.go
```

---

## **3. Language Basics**

### Variables
Variables in Go are declared using the `var` keyword or the shorthand `:=`.

```go
package main

import "fmt"

func main() {
    // Explicit type declaration
    var name string = "Go"
    var version int = 1

    // Type inference
    var isCool = true

    // Shorthand declaration
    year := 2024

    fmt.Println(name, version, isCool, year)
}
```

### Constants
Use `const` for immutable values.
```go
const Pi = 3.14
const Greeting = "Hello, Go!"
```

### Control Structures
#### If-Else
```go
if x > 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is 10 or less")
}
```

#### For Loop
```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

#### Switch
```go
switch day := "Monday"; day {
case "Monday":
    fmt.Println("Start of the work week")
case "Friday":
    fmt.Println("End of the work week")
default:
    fmt.Println("It's just another day")
}
```

---

## **4. Functions**

Functions are first-class citizens in Go.

### Basic Functions
```go
func greet(name string) string {
    return "Hello, " + name
}

func main() {
    message := greet("Alice")
    fmt.Println(message)
}
```

### Multiple Return Values
```go
func divide(a, b int) (int, int) {
    return a / b, a % b
}

func main() {
    quotient, remainder := divide(10, 3)
    fmt.Println("Quotient:", quotient, "Remainder:", remainder)
}
```

---

## **5. Arrays, Slices, and Maps**

### Arrays
Fixed-size collection.
```go
var arr = [3]int{1, 2, 3}
```

### Slices
Dynamic arrays.
```go
slice := []int{1, 2, 3}
slice = append(slice, 4)
fmt.Println(slice) // [1 2 3 4]
```

### Maps
Key-value pairs.
```go
students := map[string]int{
    "Alice": 90,
    "Bob":   85,
}

fmt.Println(students["Alice"])
```

---

## **6. Structs**

Structs are custom data types.
```go
type Person struct {
    Name string
    Age  int
}

func main() {
    p := Person{Name: "Alice", Age: 30}
    fmt.Println(p.Name, p.Age)
}
```

---

## **7. Pointers**

Pointers allow you to reference memory addresses.
```go
func main() {
    x := 10
    p := &x

    fmt.Println("Address:", p)  // Prints memory address
    fmt.Println("Value:", *p)  // Dereference the pointer
}
```

---

## **8. Interfaces**

Interfaces define behavior.
```go
type Shape interface {
    Area() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14 * c.Radius * c.Radius
}

func main() {
    var s Shape = Circle{Radius: 5}
    fmt.Println("Area:", s.Area())
}
```

---

## **9. Concurrency**

Go's concurrency model is built around goroutines and channels.

### Goroutines
Lightweight threads.
```go
func printNumbers() {
    for i := 1; i <= 5; i++ {
        fmt.Println(i)
    }
}

func main() {
    go printNumbers() // Run concurrently
    fmt.Println("Main function")
}
```

### Channels
Channels are used to communicate between goroutines.
```go
func worker(ch chan string) {
    ch <- "Task completed"
}

func main() {
    ch := make(chan string)

    go worker(ch)

    message := <-ch
    fmt.Println(message)
}
```

---

## **10. Error Handling**

Go uses `error` values for error handling.
```go
import "errors"

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("cannot divide by zero")
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Result:", result)
}
```

---

## **11. Build a REST API**

Use **gorilla/mux** for building REST APIs.

### Example: Basic API
```go
package main

import (
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
)

type Equipment struct {
    ID   string `json:"id"`
    Name string `json:"name"`
}

var equipments = []Equipment{
    {ID: "1", Name: "Laptop"},
}

func getEquipments(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(equipments)
}

func main() {
    r := mux.NewRouter()
    r.HandleFunc("/equipments", getEquipments).Methods("GET")

    http.ListenAndServe(":8080", r)
}
```

---

## **12. Testing**

Write tests using Go's `testing` package.
```go
import "testing"

func TestDivide(t *testing.T) {
    result, err := divide(10, 2)
    if err != nil || result != 5 {
        t.Errorf("Expected 5, got %d", result)
    }
}
```

Run tests:
```bash
go test ./...
```

---

## **13. Build and Deploy**

### Build:
```bash
go build -o equipment_management
```

### Deploy:
- Use Docker:
  ```dockerfile
  FROM golang:1.19
  WORKDIR /app
  COPY . .
  RUN go build -o main .
  CMD ["./main"]
  ```
- Use Kubernetes for scaling.

---
