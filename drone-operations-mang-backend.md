Here’s a sample implementation of all API endpoints for your drone and vessel operations management system using Fiber:

Complete Fiber Backend Code

package main

import (
	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/websocket/v2"
	"log"
	"sync"
)

// In-memory data storage for simplicity
var drones = make(map[string]map[string]interface{})
var vessels = make(map[string]map[string]interface{})
var mutex = &sync.Mutex{} // To handle concurrent updates

func main() {
	app := fiber.New()

	// Routes
	app.Get("/drones", getDrones)          // Get all drones
	app.Post("/drones", addDrone)          // Add a new drone
	app.Put("/drones/:id", updateDrone)    // Update drone by ID

	app.Get("/vessels", getVessels)        // Get all vessels
	app.Post("/vessels", addVessel)        // Add a new vessel
	app.Put("/vessels/:id", updateVessel)  // Update vessel by ID

	app.Get("/telemetry", websocket.New(telemetryHandler)) // Real-time telemetry via WebSocket
	app.Post("/control", controlHandler)                  // Control commands via HTTP POST

	// Start the server
	log.Fatal(app.Listen(":8080"))
}

///// Drone Handlers /////

// Get all drones
func getDrones(c *fiber.Ctx) error {
	mutex.Lock()
	defer mutex.Unlock()
	return c.JSON(drones)
}

// Add a new drone
func addDrone(c *fiber.Ctx) error {
	mutex.Lock()
	defer mutex.Unlock()

	var drone map[string]interface{}
	if err := c.BodyParser(&drone); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error": "Invalid input"})
	}
	id := drone["id"].(string)
	drones[id] = drone
	return c.Status(fiber.StatusCreated).JSON(drone)
}

// Update a drone by ID
func updateDrone(c *fiber.Ctx) error {
	mutex.Lock()
	defer mutex.Unlock()

	id := c.Params("id")
	if _, exists := drones[id]; !exists {
		return c.Status(fiber.StatusNotFound).JSON(fiber.Map{"error": "Drone not found"})
	}

	var updates map[string]interface{}
	if err := c.BodyParser(&updates); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error": "Invalid input"})
	}

	for key, value := range updates {
		drones[id][key] = value
	}
	return c.JSON(drones[id])
}

///// Vessel Handlers /////

// Get all vessels
func getVessels(c *fiber.Ctx) error {
	mutex.Lock()
	defer mutex.Unlock()
	return c.JSON(vessels)
}

// Add a new vessel
func addVessel(c *fiber.Ctx) error {
	mutex.Lock()
	defer mutex.Unlock()

	var vessel map[string]interface{}
	if err := c.BodyParser(&vessel); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error": "Invalid input"})
	}
	id := vessel["id"].(string)
	vessels[id] = vessel
	return c.Status(fiber.StatusCreated).JSON(vessel)
}

// Update a vessel by ID
func updateVessel(c *fiber.Ctx) error {
	mutex.Lock()
	defer mutex.Unlock()

	id := c.Params("id")
	if _, exists := vessels[id]; !exists {
		return c.Status(fiber.StatusNotFound).JSON(fiber.Map{"error": "Vessel not found"})
	}

	var updates map[string]interface{}
	if err := c.BodyParser(&updates); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error": "Invalid input"})
	}

	for key, value := range updates {
		vessels[id][key] = value
	}
	return c.JSON(vessels[id])
}

///// WebSocket and Control Handlers /////

// WebSocket handler for real-time telemetry
func telemetryHandler(c *websocket.Conn) {
	defer c.Close()

	for {
		// Receive telemetry data from drones/vessels
		messageType, msg, err := c.ReadMessage()
		if err != nil {
			log.Println("Error reading WebSocket message:", err)
			break
		}

		// Log telemetry data or broadcast to other clients
		log.Printf("Received telemetry: %s", string(msg))

		// Send acknowledgment
		if err := c.WriteMessage(messageType, []byte("Telemetry received")); err != nil {
			log.Println("Error writing WebSocket message:", err)
			break
		}
	}
}

// HTTP handler for sending control commands
func controlHandler(c *fiber.Ctx) error {
	var command map[string]interface{}
	if err := c.BodyParser(&command); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error": "Invalid input"})
	}

	// Process the control command (e.g., forward to a message broker or drone)
	log.Printf("Control command: %v", command)

	return c.Status(fiber.StatusOK).JSON(fiber.Map{"status": "Command sent"})
}

API Endpoints and Use
	1.	Drone APIs:
	•	GET /drones: Retrieve all registered drones.
	•	POST /drones: Add a new drone. Requires JSON body:

{
  "id": "drone123",
  "name": "Alpha Drone",
  "status": "active",
  "location": {"lat": 37.7749, "lng": -122.4194}
}


	•	PUT /drones/:id: Update a drone’s details. Requires JSON body with fields to update.

	2.	Vessel APIs:
	•	GET /vessels: Retrieve all registered vessels.
	•	POST /vessels: Add a new vessel. Requires JSON body:

{
  "id": "vessel123",
  "name": "Alpha Vessel",
  "status": "active",
  "location": {"lat": 37.7749, "lng": -122.4194}
}


	•	PUT /vessels/:id: Update a vessel’s details. Requires JSON body with fields to update.

	3.	Telemetry API:
	•	GET /telemetry: WebSocket endpoint for real-time telemetry. Streams data to/from drones/vessels.
	4.	Control Command API:
	•	POST /control: Send control commands. Example JSON body:

{
  "id": "drone123",
  "command": "move",
  "params": {"direction": "north", "speed": "10"}
}

How to Run
	1.	Install Fiber:

go get -u github.com/gofiber/fiber/v2
go get -u github.com/gofiber/websocket/v2


	2.	Run the Server:

go run main.go


	3.	Test the Endpoints:
	•	Use tools like Postman or cURL for REST APIs.
	•	Use a WebSocket client (e.g., browser extensions or wscat) for the telemetry endpoint.

Let me know if you need help integrating this with the streaming server or frontend!