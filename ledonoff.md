#include <WiFi.h>
#include <PubSubClient.h>

#define WIFI_STA_NAME "VisawaHouse_2.4G"
#define WIFI_STA_PASS "mag0859603510"

#define MQTT_SERVER   "test.mosquitto.org"
#define MQTT_PORT     1883
#define MQTT_NAME     "iot-design"

#define LED_BUILTIN 2  // Use GPIO2 for ESP32 built-in LED
#define LED_PIN 4      // Single LED on GPIO4

WiFiClient client;
PubSubClient mqtt(client);

void callback(char* topic, byte* payload, unsigned int length) {
  payload[length] = '\0';
  String payload_str = (char*)payload;
  Serial.println("[" + String(topic) + "]: " + payload_str);

  if (String(topic) == "iot/led") {
    if (payload_str == "ON") {
      digitalWrite(LED_PIN, HIGH); // Turn on LED
    } else if (payload_str == "OFF") {
      digitalWrite(LED_PIN, LOW);  // Turn off LED
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);

  Serial.println("\nConnecting to WiFi...");
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_STA_NAME, WIFI_STA_PASS);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    digitalWrite(LED_BUILTIN, !digitalRead(LED_BUILTIN));
  }

  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("\nWiFi connected");
  Serial.println("IP address: " + WiFi.localIP().toString());

  mqtt.setServer(MQTT_SERVER, MQTT_PORT);
  mqtt.setCallback(callback);
}

void loop() {
  if (!mqtt.connected()) {
    Serial.print("MQTT connection... ");
    if (mqtt.connect(MQTT_NAME)) {
      Serial.println("connected");
      mqtt.subscribe("iot/led");
    } else {
      Serial.println("failed");
      delay(5000);
      return;
    }
  }

  mqtt.loop();
}
