Sure! Let’s dive into the Fiber framework and learn it professionally from scratch. We’ll cover everything from installation to advanced topics like middleware, WebSocket integration, and building RESTful APIs.

1. What is Fiber?

Fiber is a web framework for Go, inspired by Express.js. It’s built on top of fasthttp, the fastest HTTP engine for Go, making Fiber extremely lightweight and performant.

2. Setting Up Fiber

Install Fiber
	1.	Install Go if you haven’t already: Go Installation Guide
	2.	Create a new project:

mkdir fiber-app
cd fiber-app
go mod init fiber-app


	3.	Install Fiber:

go get github.com/gofiber/fiber/v2

3. Hello World Example

Create a file named main.go:

package main

import "github.com/gofiber/fiber/v2"

func main() {
    // Create a new Fiber app
    app := fiber.New()

    // Define a route
    app.Get("/", func(c *fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })

    // Start the server
    app.Listen(":3000")
}

Run the app:

go run main.go

Visit http://localhost:3000 in your browser, and you’ll see “Hello, World!”.

4. Core Concepts

Routing

Fiber’s routing system is simple and powerful. Here’s an example:

app.Get("/hello", func(c *fiber.Ctx) error {
    return c.SendString("Hello!")
})

app.Post("/submit", func(c *fiber.Ctx) error {
    name := c.FormValue("name")
    return c.SendString("Name: " + name)
})

app.Put("/update/:id", func(c *fiber.Ctx) error {
    id := c.Params("id")
    return c.JSON(fiber.Map{"id": id, "status": "updated"})
})

	•	Get, Post, Put, and Delete map HTTP methods to routes.
	•	Use Params for dynamic segments in routes.

Middleware

Middleware in Fiber processes requests before they reach your route handlers.

Example: Logging Middleware

import "github.com/gofiber/fiber/v2/middleware/logger"

app.Use(logger.New()) // Logs each request

Example: Custom Middleware

app.Use(func(c *fiber.Ctx) error {
    c.Set("X-Custom-Header", "MyApp")
    return c.Next() // Pass control to the next handler
})

Request Handling

Fiber provides convenient methods for working with requests:

app.Post("/data", func(c *fiber.Ctx) error {
    name := c.FormValue("name")   // Get form data
    id := c.Params("id")          // Get route params
    query := c.Query("search")    // Get query params
    return c.JSON(fiber.Map{
        "name":  name,
        "id":    id,
        "query": query,
    })
})

Static File Serving

Serve static files (e.g., HTML, CSS, JS):

app.Static("/", "./public")

Access files in the public directory via http://localhost:3000/filename.

5. Building a RESTful API

Here’s an example of a CRUD API for managing books:

Full Code

package main

import (
	"github.com/gofiber/fiber/v2"
)

type Book struct {
	ID     string `json:"id"`
	Title  string `json:"title"`
	Author string `json:"author"`
}

var books = []Book{}

func main() {
	app := fiber.New()

	// Get all books
	app.Get("/books", func(c *fiber.Ctx) error {
		return c.JSON(books)
	})

	// Add a new book
	app.Post("/books", func(c *fiber.Ctx) error {
		book := new(Book)
		if err := c.BodyParser(book); err != nil {
			return c.Status(400).JSON(fiber.Map{"error": "Cannot parse JSON"})
		}
		books = append(books, *book)
		return c.Status(201).JSON(book)
	})

	// Get a book by ID
	app.Get("/books/:id", func(c *fiber.Ctx) error {
		id := c.Params("id")
		for _, book := range books {
			if book.ID == id {
				return c.JSON(book)
			}
		}
		return c.Status(404).JSON(fiber.Map{"error": "Book not found"})
	})

	// Update a book by ID
	app.Put("/books/:id", func(c *fiber.Ctx) error {
		id := c.Params("id")
		for i, book := range books {
			if book.ID == id {
				if err := c.BodyParser(&book); err != nil {
					return c.Status(400).JSON(fiber.Map{"error": "Cannot parse JSON"})
				}
				books[i] = book
				return c.JSON(book)
			}
		}
		return c.Status(404).JSON(fiber.Map{"error": "Book not found"})
	})

	// Delete a book by ID
	app.Delete("/books/:id", func(c *fiber.Ctx) error {
		id := c.Params("id")
		for i, book := range books {
			if book.ID == id {
				books = append(books[:i], books[i+1:]...)
				return c.SendStatus(204)
			}
		}
		return c.Status(404).JSON(fiber.Map{"error": "Book not found"})
	})

	app.Listen(":3000")
}

Testing the API
	1.	Start the server: go run main.go.
	2.	Use Postman or curl to interact with the API:
	•	GET /books: Retrieve all books.
	•	POST /books: Add a book with JSON body:

{
  "id": "1",
  "title": "Golang Basics",
  "author": "John Doe"
}


	•	PUT /books/:id: Update a book.
	•	DELETE /books/:id: Delete a book.

6. Advanced Features

1. WebSocket Support

import "github.com/gofiber/websocket/v2"

app.Get("/ws", websocket.New(func(c *websocket.Conn) {
	for {
		// Read message from client
		msgType, msg, err := c.ReadMessage()
		if err != nil {
			break
		}
		// Echo message back to client
		c.WriteMessage(msgType, msg)
	}
}))

2. JWT Authentication

import "github.com/gofiber/jwt/v3"

// Protected routes
app.Use("/api", jwt.New(jwt.Config{
    SigningKey: []byte("secret"),
}))

app.Get("/api/dashboard", func(c *fiber.Ctx) error {
    return c.SendString("Welcome to the dashboard!")
})

3. Middleware for Security

import "github.com/gofiber/fiber/v2/middleware/cors"

app.Use(cors.New(cors.Config{
    AllowOrigins: "http://example.com",
    AllowHeaders: "Content-Type,Authorization",
}))

7. Deployment
	1.	Build the app:

go build -o app


	2.	Run the app:

./app


	3.	Use a reverse proxy like NGINX for production.

8. Key Tips for Professional Development
	•	Use Environment Variables: Use github.com/joho/godotenv for managing configuration.
	•	Testing: Use the fiber/testing package for unit testing routes.
	•	Error Handling: Centralize error handling with custom middlewares.

With this guide, you now have a comprehensive understanding of Fiber! Let me know if you’d like deeper insights into any specific feature or help with a project setup.