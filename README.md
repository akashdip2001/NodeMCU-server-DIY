# NodeMCU-server-DIY
Use NodeMCU esp8266 as a portable homemade server, which can host my website through internet.

To host an HTML, CSS, and JavaScript website using a NodeMCU (ESP8266) with a micro SD card reader, you'll need to follow these steps:

1. **Set Up Development Environment**:
    - Install the Arduino IDE.
    - Install the ESP8266 board package in the Arduino IDE.

2. **Connect the SD Card Reader to the NodeMCU**:
    - Connect the SD card module to the NodeMCU as follows:
        - SD card VCC to NodeMCU 3.3V
        - SD card GND to NodeMCU GND
        - SD card MISO to NodeMCU D6
        - SD card MOSI to NodeMCU D7
        - SD card SCK to NodeMCU D5
        - SD card CS to NodeMCU D8 (You can choose another pin if you prefer, just update your code accordingly)

3. **Prepare the Files**:
    - Place the HTML, CSS, and JavaScript files onto the micro SD card.
    - Ensure that files are in the root directory or in an organized folder structure.
    - if you want to use your custom folder path - follow the next code ⬇️

4. **Write the Arduino Sketch**:
    - Write a sketch to read the files from the SD card and serve them over WiFi using the ESP8266's web server capabilities.


```cpp
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <SD.h>
#include <SPI.h>

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

ESP8266WebServer server(80);
File fsUploadFile;  // a File object to temporarily store the received file

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
  
  if (!SD.begin(D8)) {
    Serial.println("Card Mount Failed");
    return;
  }
  
  server.on("/", HTTP_GET, handleRoot);
  server.onNotFound(handleNotFound);
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handleRoot() {
  File file = SD.open("/index.html");
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = "text/html";
  server.streamFile(file, fileType);
  file.close();
}

void handleNotFound() {
  String path = server.uri();
  if (path.endsWith("/")) {
    path += "index.html";
  }

  File file = SD.open(path.c_str());
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = getContentType(path);
  server.streamFile(file, fileType);
  file.close();
}

String getContentType(String filename) {
  if (server.hasArg("download")) return "application/octet-stream";
  else if (filename.endsWith(".htm")) return "text/html";
  else if (filename.endsWith(".html")) return "text/html";
  else if (filename.endsWith(".css")) return "text/css";
  else if (filename.endsWith(".js")) return "application/javascript";
  else if (filename.endsWith(".png")) return "image/png";
  else if (filename.endsWith(".gif")) return "image/gif";
  else if (filename.endsWith(".jpg")) return "image/jpeg";
  else if (filename.endsWith(".ico")) return "image/x-icon";
  else if (filename.endsWith(".xml")) return "text/xml";
  else if (filename.endsWith(".pdf")) return "application/x-pdf";
  else if (filename.endsWith(".zip")) return "application/x-zip";
  else if (filename.endsWith(".gz")) return "application/x-gzip";
  return "text/plain";
}
```
<h1> use custom folder 📂 ⬆️ </h1>

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <SD.h>
#include <SPI.h>

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

ESP8266WebServer server(80);
File fsUploadFile;

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  if (!SD.begin(D8)) {
    Serial.println("Card Mount Failed");
    return;
  }

  server.on("/", HTTP_GET, handleRoot);
  server.onNotFound(handleNotFound);
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handleRoot() {
  File file = SD.open("/my_website/index.html");
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = "text/html";
  server.streamFile(file, fileType);
  file.close();
}

void handleNotFound() {
  String path = "/my_website" + server.uri();
  if (path.endsWith("/")) {
    path += "index.html";
  }

  File file = SD.open(path.c_str());
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = getContentType(path);
  server.streamFile(file, fileType);
  file.close();
}

String getContentType(String filename) {
  if (server.hasArg("download")) return "application/octet-stream";
  else if (filename.endsWith(".htm")) return "text/html";
  else if (filename.endsWith(".html")) return "text/html";
  else if (filename.endsWith(".css")) return "text/css";
  else if (filename.endsWith(".js")) return "application/javascript";
  else if (filename.endsWith(".png")) return "image/png";
  else if (filename.endsWith(".gif")) return "image/gif";
  else if (filename.endsWith(".jpg")) return "image/jpeg";
  else if (filename.endsWith(".ico")) return "image/x-icon";
  else if (filename.endsWith(".xml")) return "text/xml";
  else if (filename.endsWith(".pdf")) return "application/x-pdf";
  else if (filename.endsWith(".zip")) return "application/x-zip";
  else if (filename.endsWith(".gz")) return "application/x-gzip";
  return "text/plain";
}
```

### Explanation:
- **WiFi Setup**: Connects to your WiFi network.
- **SD Card Initialization**: Initializes the SD card reader.
- **Server Setup**: Defines endpoints and starts the server.
- **File Serving**: Reads files from the SD card and serves them with the correct MIME type.

### Upload and Test:
1. Upload the sketch to your NodeMCU.
2. Insert the SD card into the reader.
3. Open the serial monitor to check the status of the connection and the server.
4. Once connected, open a web browser and go to the IP address of your NodeMCU (you can find it in the serial monitor output).

Your website should now be accessible via the NodeMCU! Make sure to replace `"your_SSID"` and `"your_PASSWORD"` with your actual WiFi credentials.

-----------

<h1> Make it portable using 3.7v battery 🔋+ Dynamic DNS </h1>


To make NodeMCU-based website portable and accessible over the internet,follow these steps:

1. **Powering the NodeMCU with Batteries**:
    - **Lithium Battery Connection**: Use a lithium battery (3.7V) and a battery charging module (such as a TP4056 module) to power the NodeMCU. Ensure the battery output is connected to the VIN and GND pins of the NodeMCU. The VIN pin can accept a range of voltages, typically from 4.5V to 10V, so make sure your battery setup provides the appropriate voltage.

2. **Setting Up Internet Access**:
    - **WiFi Configuration**: The NodeMCU should be configured to connect to a WiFi network with internet access.
    - **Port Forwarding**: Set up port forwarding on your router to make the NodeMCU accessible from the internet. This involves:
        - Assigning a static IP address to the NodeMCU on your local network.
        - Configuring your router to forward requests on a specific port (e.g., port 80) to the NodeMCU's IP address.
    - **Dynamic DNS**: Since home IP addresses can change, you might want to use a Dynamic DNS (DDNS) service to get a consistent domain name that maps to your home's dynamic IP address.

Here is a step-by-step guide:

### Powering the NodeMCU

1. **Connect the Battery**:
    - Connect a 3.7V lithium battery to a TP4056 charging module.
    - Connect the output of the TP4056 module to the VIN and GND pins of the NodeMCU.

### Setting Up the Internet Access

1. **Configure WiFi**:
    Ensure the NodeMCU connects to your WiFi network by updating the WiFi credentials in the Arduino sketch.

```cpp
const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";
```

2. **Port Forwarding**:
    - Access your router's settings (usually via a web interface, e.g., http://192.168.1.1).
    - Find the port forwarding section.
    - Forward external port 80 to the internal IP address of the NodeMCU on port 80.

3. **Dynamic DNS**:
    - Sign up for a DDNS service (such as No-IP, DynDNS, or DuckDNS).
    - Configure your router to update the DDNS service with your current IP address, or use a DDNS update client on a device connected to your network.

### Code

Here's the complete Arduino sketch that serves files from the SD card and can be accessed via the internet:

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <SD.h>
#include <SPI.h>

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

ESP8266WebServer server(80);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  if (!SD.begin(D8)) {
    Serial.println("Card Mount Failed");
    return;
  }

  server.on("/", HTTP_GET, handleRoot);
  server.onNotFound(handleNotFound);
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handleRoot() {
  File file = SD.open("/my_website/index.html");
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = "text/html";
  server.streamFile(file, fileType);
  file.close();
}

void handleNotFound() {
  String path = "/my_website" + server.uri();
  if (path.endsWith("/")) {
    path += "index.html";
  }

  File file = SD.open(path.c_str());
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = getContentType(path);
  server.streamFile(file, fileType);
  file.close();
}

String getContentType(String filename) {
  if (server.hasArg("download")) return "application/octet-stream";
  else if (filename.endsWith(".htm")) return "text/html";
  else if (filename.endsWith(".html")) return "text/html";
  else if (filename.endsWith(".css")) return "text/css";
  else if (filename.endsWith(".js")) return "application/javascript";
  else if (filename.endsWith(".png")) return "image/png";
  else if (filename.endsWith(".gif")) return "image/gif";
  else if (filename.endsWith(".jpg")) return "image/jpeg";
  else if (filename.endsWith(".ico")) return "image/x-icon";
  else if (filename.endsWith(".xml")) return "text/xml";
  else if (filename.endsWith(".pdf")) return "application/x-pdf";
  else if (filename.endsWith(".zip")) return "application/x-zip";
  else if (filename.endsWith(".gz")) return "application/x-gzip";
  return "text/plain";
}
```

### Testing

1. **Upload the Sketch**: Upload the sketch to your NodeMCU using the Arduino IDE.
2. **Insert SD Card**: Insert the SD card containing your website files in the `my_website` directory.
3. **Connect to Power**: Connect the NodeMCU to your lithium battery setup.
4. **Verify Connection**: Open the serial monitor to check the WiFi connection and the IP address assigned to your NodeMCU.
5. **Access Locally**: Initially, test the website locally using the IP address of the NodeMCU.
6. **Access Remotely**: Use your public IP address or DDNS domain to access the website from the internet.

By following these steps, NodeMCU should be able to host my website and make it accessible both locally and over the internet, powered by a portable lithium battery setup.

----------

<h1> No router - use MIFI (4G LTE module) </h1>

To make your NodeMCU (ESP8266) server portable and accessible over the internet without relying on a fixed router setup, you'll need a way to provide both power and mobile internet connectivity. Here are the components and steps required to achieve this:

### Components Required
1. **NodeMCU (ESP8266)**
2. **Micro SD Card Reader Module**
3. **Micro SD Card (with your website files)**
4. **Lithium Battery (3.7V) and TP4056 Charging Module**
5. **4G LTE Module or Portable Mobile WiFi Hotspot (MiFi)**
6. **Power Bank (optional, for extended battery life)**

### Setup Steps

#### 1. Powering the NodeMCU
Use a 3.7V lithium battery with a TP4056 charging module to power the NodeMCU:
- **TP4056 Module Connection**:
  - B+ and B-: Connect to the lithium battery.
  - OUT+: Connect to the VIN pin of the NodeMCU.
  - OUT-: Connect to the GND pin of the NodeMCU.

#### 2. Providing Internet Access
Option A: **Using a 4G LTE Module**
- A 4G LTE module can provide internet access directly to the NodeMCU.
- Modules like the SIM7600, SIM7000, or similar can be used.
- Connect the 4G module to the NodeMCU using UART or USB interface.

Option B: **Using a Portable Mobile WiFi Hotspot (MiFi)**
- A MiFi device creates a mobile WiFi hotspot using a 4G SIM card.
- Connect the NodeMCU to this WiFi network.

#### 3. Preparing the SD Card
- Place your website files in a directory named `my_website` on the SD card.

### Example Code for NodeMCU with SD Card and 4G LTE Module (or MiFi)
Here’s the code to serve files from the SD card over the internet using the NodeMCU. This example assumes you're using a WiFi hotspot for internet access.

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <SD.h>
#include <SPI.h>

const char* ssid = "your_MiFi_SSID";   // Replace with your MiFi SSID
const char* password = "your_MiFi_password"; // Replace with your MiFi password

ESP8266WebServer server(80);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  if (!SD.begin(D8)) {
    Serial.println("Card Mount Failed");
    return;
  }

  server.on("/", HTTP_GET, handleRoot);
  server.onNotFound(handleNotFound);
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handleRoot() {
  File file = SD.open("/my_website/index.html");
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = "text/html";
  server.streamFile(file, fileType);
  file.close();
}

void handleNotFound() {
  String path = "/my_website" + server.uri();
  if (path.endsWith("/")) {
    path += "index.html";
  }

  File file = SD.open(path.c_str());
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = getContentType(path);
  server.streamFile(file, fileType);
  file.close();
}

String getContentType(String filename) {
  if (server.hasArg("download")) return "application/octet-stream";
  else if (filename.endsWith(".htm")) return "text/html";
  else if (filename.endsWith(".html")) return "text/html";
  else if (filename.endsWith(".css")) return "text/css";
  else if (filename.endsWith(".js")) return "application/javascript";
  else if (filename.endsWith(".png")) return "image/png";
  else if (filename.endsWith(".gif")) return "image/gif";
  else if (filename.endsWith(".jpg")) return "image/jpeg";
  else if (filename.endsWith(".ico")) return "image/x-icon";
  else if (filename.endsWith(".xml")) return "text/xml";
  else if (filename.endsWith(".pdf")) return "application/x-pdf";
  else if (filename.endsWith(".zip")) return "application/x-zip";
  else if (filename.endsWith(".gz")) return "application/x-gzip";
  return "text/plain";
}
```

### Steps to Deploy
1. **Upload the Sketch**: Upload the provided Arduino sketch to your NodeMCU.
2. **Insert SD Card**: Insert the SD card with your website files into the SD card reader module.
3. **Connect Power**: Connect the NodeMCU to the lithium battery setup.
4. **Internet Access**:
   - If using a 4G LTE module, ensure it is powered and connected correctly.
   - If using a MiFi device, turn it on and ensure it has internet connectivity.
5. **Test Locally**: Verify the NodeMCU is connected to the MiFi network by checking the serial monitor for the assigned IP address.
6. **Access Remotely**: Use the public IP address provided by your mobile internet service provider to access your NodeMCU server over the internet.

By following these steps, you can create a portable NodeMCU server setup that is both battery-powered and capable of serving your website over the internet using mobile data.

----------

<h1> use Mobile Hotspot - wifi </h1>

To create a portable NodeMCU server that uses your mobile WiFi hotspot for internet access and is accessible to all internet users, you need to follow these steps. Additionally, I'll explain how to use a Dynamic DNS (DDNS) service to provide a memorable hostname instead of an IP address.

### Components Required
1. **NodeMCU (ESP8266)**
2. **Micro SD Card Reader Module**
3. **Micro SD Card (with your website files)**
4. **Lithium Battery (3.7V) and TP4056 Charging Module**
5. **Mobile Phone with WiFi Hotspot Capability**
6. **Dynamic DNS (DDNS) Service**

### Steps to Setup

#### 1. Powering the NodeMCU
1. **Connect the Battery**:
    - Connect a 3.7V lithium battery to a TP4056 charging module.
    - Connect the output of the TP4056 module to the VIN and GND pins of the NodeMCU.

#### 2. Setting Up the NodeMCU for Mobile Hotspot
1. **Configure WiFi in the Arduino Sketch**:
    - Update your NodeMCU sketch with your mobile hotspot SSID and password.

```cpp
const char* ssid = "your_MobileHotspot_SSID";
const char* password = "your_MobileHotspot_password";
```

#### 3. Preparing the SD Card
1. **Prepare Your Files**:
    - Place your website files in a directory named `my_website` on the SD card.

#### 4. Write and Upload the Arduino Sketch
1. **Arduino Sketch**:

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <SD.h>
#include <SPI.h>

const char* ssid = "your_MobileHotspot_SSID";
const char* password = "your_MobileHotspot_password";

ESP8266WebServer server(80);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  if (!SD.begin(D8)) {
    Serial.println("Card Mount Failed");
    return;
  }

  server.on("/", HTTP_GET, handleRoot);
  server.onNotFound(handleNotFound);
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handleRoot() {
  File file = SD.open("/my_website/index.html");
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = "text/html";
  server.streamFile(file, fileType);
  file.close();
}

void handleNotFound() {
  String path = "/my_website" + server.uri();
  if (path.endsWith("/")) {
    path += "index.html";
  }

  File file = SD.open(path.c_str());
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  String fileType = getContentType(path);
  server.streamFile(file, fileType);
  file.close();
}

String getContentType(String filename) {
  if (server.hasArg("download")) return "application/octet-stream";
  else if (filename.endsWith(".htm")) return "text/html";
  else if (filename.endsWith(".html")) return "text/html";
  else if (filename.endsWith(".css")) return "text/css";
  else if (filename.endsWith(".js")) return "application/javascript";
  else if (filename.endsWith(".png")) return "image/png";
  else if (filename.endsWith(".gif")) return "image/gif";
  else if (filename.endsWith(".jpg")) return "image/jpeg";
  else if (filename.endsWith(".ico")) return "image/x-icon";
  else if (filename.endsWith(".xml")) return "text/xml";
  else if (filename.endsWith(".pdf")) return "application/x-pdf";
  else if (filename.endsWith(".zip")) return "application/x-zip";
  else if (filename.endsWith(".gz")) return "application/x-gzip";
  return "text/plain";
}
```

2. **Upload the Sketch**: Connect the NodeMCU to your computer and upload the sketch using the Arduino IDE.

#### 5. Setup Dynamic DNS (DDNS)
1. **Sign Up for a DDNS Service**:
    - Register for a free DDNS service like No-IP, DynDNS, or DuckDNS.

2. **Configure the DDNS Service**:
    - Follow the instructions provided by the DDNS service to set up your domain name (e.g., `mynodemcu.ddns.net`).

3. **Update DDNS with Mobile Hotspot IP**:
    - Install a DDNS update client on your mobile device or another device that can regularly update the DDNS service with your mobile hotspot's IP address.
    - Some DDNS services have mobile apps that can do this.

#### 6. Testing and Deployment
1. **Power the NodeMCU**: Connect the NodeMCU to the lithium battery.
2. **Activate Mobile Hotspot**: Turn on your mobile phone's WiFi hotspot.
3. **Connect NodeMCU**: The NodeMCU should connect to your mobile hotspot. Check the serial monitor to confirm the IP address assigned.
4. **Access Locally**: Test accessing your website locally using the IP address assigned by the hotspot.
5. **Access Remotely**: Use your DDNS domain name (e.g., `mynodemcu.ddns.net`) to access your NodeMCU server over the internet.

By following these steps, you can create a completely portable NodeMCU server that is powered by a lithium battery and accessible via the internet using a mobile WiFi hotspot and DDNS. This setup allows anyone on the internet to access your website using the DDNS domain name.
