#include <WiFi.h>

const char* ssid = "ORSCOPE";
const char* password = "12345678";

WiFiServer server(80);

// Define motor control pins
const int in1Pin = 21;  // Input 1 pin of L298N
const int in2Pin = 22;  // Input 2 pin of L298N

bool movingUp = false;
bool movingDown = false;

void setup() {
  Serial.begin(115200);
  
  pinMode(in1Pin, OUTPUT);
  pinMode(in2Pin, OUTPUT);
  
  // Connect to Wi-Fi
  Serial.println();
  Serial.println("Connecting to WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
  
  // Start the server
  server.begin();
  Serial.println("Server started");
}

void loop() {
  WiFiClient client = server.available();  // Check for incoming clients
  
  if (client) {
    Serial.println("New Client connected");
    while (client.connected()) {
      if (client.available()) {
        String request = client.readStringUntil('\r');
        client.flush();
        
        // If the request is for IP address
        if (request.indexOf("/ip") != -1) {
          Serial.println("Sending IP address");
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/plain");
          client.println();
          client.println(WiFi.localIP());
          delay(100); // Adjust delay as needed
          client.stop();
          Serial.println("Client disconnected");
        }
        
        // If the request is to move up
        else if (request.indexOf("/up") != -1) {
          Serial.println("Moving Up");
          movingUp = true;
          movingDown = false;
        }
        
        // If the request is to move down
        else if (request.indexOf("/down") != -1) {
          Serial.println("Moving Down");
          movingDown = true;
          movingUp = false;
        }
        
        // If the request is to stop moving
        else if (request.indexOf("/stop") != -1) {
          Serial.println("Stopping");
          movingUp = false;
          movingDown = false;
        }
        
        // Move the actuator based on button state
        if (movingUp) {
          digitalWrite(in1Pin, LOW);
          digitalWrite(in2Pin, HIGH);
        } else if (movingDown) {
          digitalWrite(in1Pin, HIGH);
          digitalWrite(in2Pin, LOW);
        } else {
          digitalWrite(in1Pin, LOW);
          digitalWrite(in2Pin, LOW);
        }
        
        // Respond with control options
        client.println("HTTP/1.1 200 OK");
        client.println("Content-Type: text/html");
        client.println();
        client.println("<!DOCTYPE HTML>");
        client.println("<html>");
        client.println("<head><title>ESP32 Linear Actuator Control</title>");
        client.println("<style>");
        client.println("body { font-family: Arial, sans-serif; text-align: center; }");
        client.println(".button { display: inline-block; background-color: #4CAF50; color: white; padding: 20px 40px; text-align: center; font-size: 24px; margin: 20px; cursor: pointer; border: none; border-radius: 12px; text-decoration: none; }");
        client.println(".button:hover { background-color: #45a049; }");
        client.println("</style>");
        client.println("</head>");
        client.println("<body>");
        client.println("<h1>Ksqaure</h1>");
        
        // Create UP, DOWN, and STOP buttons
        client.println("<form>");
        client.println("<button class='button' type='button' onclick='moveUp()'>Up</button>");
        client.println("<button class='button' type='button' onclick='moveDown()'>Down</button>");
        client.println("<button class='button' type='button' onclick='stopMoving()'>Stop</button>");
        client.println("</form>");
        
        // JavaScript functions to move the actuator up, down, and stop
        client.println("<script>");
        client.println("function moveUp() {");
        client.println("  fetch('/up');");
        client.println("}");
        client.println("function moveDown() {");
        client.println("  fetch('/down');");
        client.println("}");
        client.println("function stopMoving() {");
        client.println("  fetch('/stop');");
        client.println("}");
        client.println("</script>");
        
        client.println("</body>");
        client.println("</html>");
        delay(100); // Adjust delay before closing client connection
        client.stop();
        Serial.println("Client disconnected");
      }
    }
  }
}
