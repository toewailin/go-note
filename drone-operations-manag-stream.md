To integrate the Fiber backend with a streaming server for real-time video, we’ll use FFmpeg (for video encoding/streaming) and WebRTC or HLS (HTTP Live Streaming) for low-latency video playback.

Architecture for Integration
	1.	Drone/Vessel Camera:
	•	Captures raw video footage.
	•	Streams video to the server via RTSP, RTMP, or WebRTC.
	2.	Streaming Server (FFmpeg/GStreamer):
	•	Encodes video to HLS or WebRTC formats.
	•	Provides video streams for the Fiber backend.
	3.	Fiber Backend:
	•	Serves HLS/WebRTC video URLs to the frontend.
	•	Integrates WebSocket for real-time telemetry alongside video.
	4.	Frontend:
	•	Displays the live video stream using a video player (e.g., hls.js or WebRTC API).

1. Streaming Server Configuration

We’ll configure FFmpeg to receive RTSP or RTMP streams from drones/vessels and convert them to HLS/WebRTC.

FFmpeg Command for HLS

This command converts a live RTSP stream to HLS format:

ffmpeg -i rtsp://<camera-stream-url> \
  -c:v libx264 -preset veryfast -crf 20 -g 50 \
  -hls_time 2 -hls_list_size 3 -hls_flags delete_segments \
  -f hls /path/to/output/stream.m3u8

	•	Input: RTSP stream URL.
	•	Output: HLS .m3u8 file (segments stored locally).

FFmpeg Command for WebRTC

To stream WebRTC:
	1.	Install ffmpeg with WebRTC support (libwebrtc).
	2.	Stream to WebRTC using an SDP file:

ffmpeg -i rtsp://<camera-stream-url> -c:v libx264 -f rtp rtp://<backend-ip>:<port>

2. Backend Integration with Fiber

Endpoint for Video URLs

Add an endpoint to provide the HLS/WebRTC video URL:

app.Get("/video/:id", getVideoStream)

func getVideoStream(c *fiber.Ctx) error {
    id := c.Params("id")
    // Replace with actual logic to fetch video URL by drone/vessel ID
    videoURL := "http://your-server.com/path/to/stream_" + id + ".m3u8"
    return c.JSON(fiber.Map{
        "id":       id,
        "videoURL": videoURL,
    })
}

WebSocket Integration for Video and Telemetry

Extend the /telemetry WebSocket handler to include real-time video stream info:

app.Get("/telemetry", websocket.New(func(c *websocket.Conn) {
    defer c.Close()

    for {
        // Simulate telemetry + video stream updates
        telemetry := map[string]interface{}{
            "id":        "drone123",
            "latitude":  37.7749,
            "longitude": -122.4194,
            "altitude":  100,
            "videoURL":  "http://your-server.com/path/to/stream_drone123.m3u8",
        }

        // Send JSON payload to client
        if err := c.WriteJSON(telemetry); err != nil {
            log.Println("Error writing WebSocket message:", err)
            break
        }
    }
}))

3. Frontend Integration

HLS Player Example

Use hls.js to display the live stream:

<video id="videoPlayer" controls autoplay></video>
<script src="https://cdn.jsdelivr.net/npm/hls.js"></script>
<script>
    const video = document.getElementById("videoPlayer");
    const hls = new Hls();

    // Replace with actual video URL fetched from the backend
    const videoURL = "http://your-server.com/path/to/stream_drone123.m3u8";

    if (Hls.isSupported()) {
        hls.loadSource(videoURL);
        hls.attachMedia(video);
    } else if (video.canPlayType("application/vnd.apple.mpegurl")) {
        video.src = videoURL;
    }
</script>

WebRTC Player Example

Use the WebRTC API to play streams:

<video id="videoPlayer" autoplay playsinline></video>
<script>
    const video = document.getElementById("videoPlayer");

    // Replace with your WebRTC SDP answer URL
    const webrtcURL = "webrtc://your-server.com/stream";

    const peerConnection = new RTCPeerConnection();
    peerConnection.ontrack = (event) => {
        video.srcObject = event.streams[0];
    };

    // Fetch SDP offer from server
    fetch(webrtcURL)
        .then((response) => response.json())
        .then((sdp) => peerConnection.setRemoteDescription(new RTCSessionDescription(sdp)));
</script>

4. Message Broker for Real-Time Communication

Use a message broker to handle telemetry and video stream metadata in real-time.

MQTT Integration

For real-time telemetry updates, integrate MQTT:

import (
	"github.com/eclipse/paho.mqtt.golang"
)

func connectMQTT() mqtt.Client {
	opts := mqtt.NewClientOptions().AddBroker("tcp://your-mqtt-broker:1883")
	opts.SetClientID("fiber-backend")

	client := mqtt.NewClient(opts)
	if token := client.Connect(); token.Wait() && token.Error() != nil {
		log.Fatalf("Failed to connect to MQTT: %v", token.Error())
	}
	return client
}

// Publish telemetry
func publishTelemetry(client mqtt.Client, topic string, payload string) {
	token := client.Publish(topic, 0, false, payload)
	token.Wait()
}

5. Complete Backend Code

Here’s a streamlined example combining Fiber, WebSocket, HLS, and MQTT:

package main

import (
	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/websocket/v2"
	"github.com/eclipse/paho.mqtt.golang"
	"log"
)

var mqttClient mqtt.Client

func main() {
	// Initialize Fiber and MQTT
	app := fiber.New()
	mqttClient = connectMQTT()

	// API endpoints
	app.Get("/video/:id", getVideoStream)
	app.Get("/telemetry", websocket.New(telemetryHandler))

	log.Fatal(app.Listen(":8080"))
}

func connectMQTT() mqtt.Client {
	opts := mqtt.NewClientOptions().AddBroker("tcp://your-mqtt-broker:1883")
	client := mqtt.NewClient(opts)
	if token := client.Connect(); token.Wait() && token.Error() != nil {
		log.Fatalf("Failed to connect to MQTT: %v", token.Error())
	}
	return client
}

func getVideoStream(c *fiber.Ctx) error {
	id := c.Params("id")
	videoURL := "http://your-server.com/path/to/stream_" + id + ".m3u8"
	return c.JSON(fiber.Map{"id": id, "videoURL": videoURL})
}

func telemetryHandler(c *websocket.Conn) {
	defer c.Close()

	for {
		telemetry := map[string]interface{}{
			"id":       "drone123",
			"latitude": 37.7749,
			"longitude": -122.4194,
			"videoURL": "http://your-server.com/path/to/stream_drone123.m3u8",
		}
		if err := c.WriteJSON(telemetry); err != nil {
			log.Println("Error sending telemetry:", err)
			break
		}
	}
}

Next Steps
	1.	Deploy Streaming Server:
	•	Configure FFmpeg/GStreamer for HLS/WebRTC.
	2.	Integrate Backend:
	•	Ensure Fiber serves video URLs dynamically.
	3.	Frontend Implementation:
	•	Embed HLS/WebRTC players into the UI.

Let me know if you need further assistance!