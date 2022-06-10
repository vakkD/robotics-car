This document includes prototype and the three statements.

<br><br>

At the start we wanted to use a RF module but we did not have immediate access to one. We thought of using infrared but quickly turned that idea down due to the many restrictions.
Then we started looking into a bluetooth module, I tried experimenting with a module from the school but could not get it to work.
We had the idea of using WiFi and immediately this seemed like a good solution as it is much faster and has a larger range than bluetooth.
We thought this would be a future proof solution that would allow the use of more components so we decided to controll using the <a href="https://www.jaycar.com.au/mega-with-wi-fi/p/XC4421">Mega with WiFi</a> (module called ESP8266). 
On this we will setup a webserver for a phone to connect to, controlling the car. 
When connected to server the user will have a joystick to make the car move, similar to a real life joystick.
Data needs to be uploaded for the server to display the controlls (html layout) which we did not finish creating, but successfully got the server to work.
Learning HTML I thought was a bit challenging because it is quite different from C, still there is more .


This is the code for the server, I added comments where needed.
```
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>
#include <ESP8266mDNS.h>
#include <FS.h>

const char *networkName = "WiFi Rover";
const char *password = "test";

ESP8266WebServer server(80);

String parser;

void setup(void)
{
//    WiFi.mode(WIFI_STA);
//    WiFi.begin(ssid, password);

    // Creating a WiFi network with the name "WiFi Rover" and password "test".
    WiFi.mode(WIFI_AP); 
    WiFi.softAP(networkName, password);

    Serial.begin(115200);

    SPIFFS.begin();

    // This is just printing the IP address of the ESP8266 to the serial monitor.
    delay(2000);
    Serial.print("%"); 
    Serial.println(WiFi.softAPIP());

    // Reserving 128 bytes of memory for the parser string.
    parser.reserve(128);

    Serial.println("Wait for connection");

    // This is a callback function that is called when the ESP8266 receives a POST request on the /data path. It checks if the request has the left and right arguments, and if it does, it saves them to the parser string.
    server.on("/data", HTTP_POST, []() {
        if (!(server.hasArg("left") && server.hasArg("right"))) {
            server.send(500, "text/html", "bad");
            return;
        }
        parser = server.arg("left") + ":" + server.arg("right") + "\n";
        server.send(200, "text/html", "OK");
    });

    // This is a function that is called when the ESP8266 receives a request on a path that is not defined. It checks if the requested file exists on the SPIFFS, and if it does, it sends it to the client.
    server.onNotFound([]() {
        if (!fileRead(server.uri()))
        {
            server.send(404, "text/html",
                        "<h1>404 File not found on SPIFFS</h1>");
        }
    });
    
    MDNS.begin("rover");
    
    server.begin();
}

void loop(void)
{
    // Handling the client and updating the MDNS.
    server.handleClient();
    MDNS.update();
    
    Serial.print(parser);
    parser = "";
    Serial.flush();
    delay(100);
}

bool fileRead(String filepath)
{
    // If the filepath ends with a slash, it adds index.html to the end of the filepath.
    if (filepath.endsWith("/"))
        filepath += "index.html";

    // Checking if the file exists on the SPIFFS, and if it does, it opens it, streams it to the client, and closes it.
    if (SPIFFS.exists(filepath))
    {

        File f = SPIFFS.open(filepath, "r");
        server.streamFile(f, contentType(filepath));
        f.close();
        return true;
    }

    return false;
}

// A function that returns the content type of a file based on its extension.
String contentType(String filepath)
{
    if (filepath.endsWith(".html"))
        return "text/html";
    else if (filepath.endsWith(".css"))
        return "text/css";
    else if (filepath.endsWith(".js"))
        return "text/javascript";
    else
        return "text/plain";
}
```

</br></br>

Below is the diagram for the car, with measurements of all the components.</br>

![car diagram](https://github.com/vakkD/robotics-car/blob/main/52B5EE9D-5BC2-4ABD-8478-D297F068B930.jpeg)</br>

<a href="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a0/Ackermann_turning.svg/1200px-Ackermann_turning.svg.png">Here</a> is a diagram of the ackermann steering.
Using the servo to move the bar.

</br></br>

The first prototype for the car is made out of cardboard and components held down using zipties, to ensure that the componets can be changed and moved around easily.
Still in this first prototype we decided to not include the ackermann steering system because of time restraints and chose to instead turn using wheel speed.
We firstly wanted to 3D print the car shell. <a href="https://github.com/vakkD/robotics-car/blob/0085ab7558148f43ecdcb71a4b3425b9330547cf/car%20robotics%20v3.obj">This</a> is the rough test model I made for the car. 
I tried to print this at home, but was not able to due to printer issues. We settled to instead use cardboard to create the prototype.
I connected the Arduino Mega to the motor controller and connected one motor, with one 9V battery connected, this is a video of the left motor spinning. </br>

<img src="https://github.com/vakkD/robotics-car/blob/main/leftwheelspin.gif" width="35%"></br>
I was able to get both of the motors to work using the motor controller (seen by teacher) but twice the battery died straight after and I did not have enough time to fix it to the power supply, I got one motor to work but not both.

The power issue was what caused most of the problems and delays. I tried using the 5 battery pack and the 9V, both failed. 
I think I could get the power supply working but would not be suitable for a remote control car, having a big box needing to be carried around.
Once we figure out a solution to the power problem, it should be much easier from there as the hard parts (server, power and motor) have been done.

<br><br>

So far the soldering tutorial I made has not come into use, but I think it definitely will after the prototype stage.
