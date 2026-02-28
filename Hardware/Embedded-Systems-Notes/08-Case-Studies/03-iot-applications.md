# Chapter 8.3: IoT Embedded Systems

## Introduction

The Internet of Things (IoT) connects billions of embedded devices to the cloud, enabling smart homes, industrial automation, precision agriculture, and wearable technology. IoT embedded systems face unique challenges: ultra-low power consumption for battery operation, robust wireless connectivity, cloud integration, security, and over-the-air updates. This chapter explores the architecture, protocols, and design patterns for IoT applications.

---

## IoT System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│              IoT REFERENCE ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                      CLOUD LAYER                                │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │ │
│  │  │   IoT    │  │ Database │  │Analytics │  │Dashboard │       │ │
│  │  │ Platform │  │ Storage  │  │ & ML     │  │  & Apps  │       │ │
│  │  │(AWS IoT, │  │(TimeSeries│  │          │  │          │       │ │
│  │  │Azure IoT)│  │  InfluxDB)│ │          │  │          │       │ │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │ │
│  │       └──────────────┴────────────┴──────────────┘             │ │
│  └──────────────────────────────┬─────────────────────────────────┘ │
│                                 │                                    │
│                          ═══════╪═══════                             │
│                          INTERNET / WAN                              │
│                          ═══════╪═══════                             │
│                                 │                                    │
│  ┌──────────────────────────────┴─────────────────────────────────┐ │
│  │                     GATEWAY LAYER                               │ │
│  │  ┌─────────────────────────────────────────────────────────┐   │ │
│  │  │                   EDGE GATEWAY                           │   │ │
│  │  │   • Protocol translation (MQTT↔CoAP↔HTTP)               │   │ │
│  │  │   • Local data processing and filtering                 │   │ │
│  │  │   • Store and forward (network resilience)              │   │ │
│  │  │   • Security (TLS termination, authentication)          │   │ │
│  │  │   • Device management                                    │   │ │
│  │  └────────────────────────────┬────────────────────────────┘   │ │
│  └───────────────────────────────┼────────────────────────────────┘ │
│                                  │                                   │
│            ┌─────────────────────┼─────────────────────┐            │
│            │                     │                     │            │
│    ════════╪═══════     ════════╪═══════     ════════╪═══════      │
│    WiFi/Ethernet          LoRa/NB-IoT         Zigbee/BLE           │
│    ════════╪═══════     ════════╪═══════     ════════╪═══════      │
│            │                     │                     │            │
│  ┌─────────┴─────────────────────┴─────────────────────┴──────────┐ │
│  │                      DEVICE LAYER                               │ │
│  │                                                                 │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │ │
│  │  │ Smart    │  │ Industrial│  │  Agri    │  │ Wearable │       │ │
│  │  │ Sensor   │  │  Sensor  │  │ Sensor   │  │  Device  │       │ │
│  │  │          │  │          │  │          │  │          │       │ │
│  │  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │       │ │
│  │  │ │ MCU  │ │  │ │ MCU  │ │  │ │ MCU  │ │  │ │ MCU  │ │       │ │
│  │  │ │+Radio│ │  │ │+Radio│ │  │ │+Radio│ │  │ │+Radio│ │       │ │
│  │  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │       │ │
│  │  │  │ │ │   │  │  │ │ │   │  │  │ │ │   │  │  │ │ │   │       │ │
│  │  │  T H P   │  │  V I M   │  │  S M W   │  │  A G H   │       │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │ │
│  │                                                                 │ │
│  │  Legend: T=Temp, H=Humidity, P=Pressure, V=Vibration,          │ │
│  │          I=Current, M=Motor, S=Soil, W=Weather,                │ │
│  │          A=Accel, G=GPS, HR=Heart Rate                         │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Wireless Protocol Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│              IoT WIRELESS PROTOCOLS COMPARISON                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RANGE vs DATA RATE:                                                │
│                                                                      │
│  Data Rate                                                          │
│  (Mbps)                                                             │
│     │                                                                │
│  100┤                        ┌──────┐                               │
│     │                        │ WiFi │                               │
│   10┤                        └──────┘                               │
│     │                                                                │
│    1┤           ┌──────┐                                            │
│     │           │ BLE5 │                                            │
│  0.1┤           └──────┘     ┌──────┐                               │
│     │   ┌──────┐             │Zigbee│                               │
│ 0.01┤   │ LoRa │             └──────┘                               │
│     │   └──────┘                                                     │
│0.001┤───────┬──────────┬──────────┬──────────┬──────────┬───▶       │
│     │       10m       100m        1km       10km      Range         │
│                                                                      │
│  PROTOCOL COMPARISON TABLE:                                          │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ Protocol  │ Range   │Data Rate│ Power  │ Topology │ Use Case │  │
│  ├───────────┼─────────┼─────────┼────────┼──────────┼──────────┤  │
│  │ WiFi      │ 50-100m │ 100Mbps │ High   │ Star     │ Smart    │  │
│  │ (802.11)  │         │         │        │          │ home     │  │
│  ├───────────┼─────────┼─────────┼────────┼──────────┼──────────┤  │
│  │ BLE 5.0   │ 100-400m│ 2Mbps   │ Low    │ Star/    │ Wearables│  │
│  │           │         │         │        │ Mesh     │ beacons  │  │
│  ├───────────┼─────────┼─────────┼────────┼──────────┼──────────┤  │
│  │ Zigbee    │ 100m    │ 250kbps │ Very   │ Mesh     │ Home     │  │
│  │ (802.15.4)│         │         │ Low    │          │ auto     │  │
│  ├───────────┼─────────┼─────────┼────────┼──────────┼──────────┤  │
│  │ LoRa      │ 5-15km  │ 50kbps  │ Very   │ Star     │ Smart    │  │
│  │           │         │         │ Low    │          │ city     │  │
│  ├───────────┼─────────┼─────────┼────────┼──────────┼──────────┤  │
│  │ NB-IoT    │ 10km+   │ 250kbps │ Low    │ Star     │ Metering │  │
│  │           │         │         │        │(cellular)│ tracking │  │
│  ├───────────┼─────────┼─────────┼────────┼──────────┼──────────┤  │
│  │ Thread    │ 100m    │ 250kbps │ Low    │ Mesh     │ Smart    │  │
│  │           │         │         │        │          │ home     │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## MQTT Protocol

```
┌─────────────────────────────────────────────────────────────────────┐
│              MQTT - MESSAGE QUEUING TELEMETRY TRANSPORT              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PUBLISH/SUBSCRIBE MODEL:                                            │
│                                                                      │
│  ┌──────────────┐            ┌──────────────┐                       │
│  │   SENSOR A   │            │   SENSOR B   │                       │
│  │              │            │              │                       │
│  │  Publishes:  │            │  Publishes:  │                       │
│  │  /home/temp  │            │  /home/humid │                       │
│  └──────┬───────┘            └──────┬───────┘                       │
│         │                           │                                │
│         │    PUBLISH                │    PUBLISH                    │
│         │    Topic: /home/temp      │    Topic: /home/humid         │
│         │    Payload: 23.5          │    Payload: 65                │
│         ▼                           ▼                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                        MQTT BROKER                           │    │
│  │                                                              │    │
│  │   Topic Tree:                                                │    │
│  │   └── home                                                   │    │
│  │       ├── temp ────────────────────▶ 23.5                   │    │
│  │       ├── humid ───────────────────▶ 65                     │    │
│  │       └── light ───────────────────▶ on                     │    │
│  │                                                              │    │
│  │   Subscribers:                                               │    │
│  │   • Dashboard: /home/#  (wildcard - all home topics)        │    │
│  │   • Logger: /home/temp                                      │    │
│  │   • HVAC: /home/temp, /home/humid                           │    │
│  │                                                              │    │
│  └───────────────────────────────┬─────────────────────────────┘    │
│                                  │                                   │
│         ┌────────────────────────┼────────────────────────┐         │
│         │                        │                        │         │
│         ▼                        ▼                        ▼         │
│  ┌──────────────┐         ┌──────────────┐        ┌──────────────┐ │
│  │  DASHBOARD   │         │    LOGGER    │        │     HVAC     │ │
│  │              │         │              │        │              │ │
│  │ Subscribes:  │         │ Subscribes:  │        │ Subscribes:  │ │
│  │ /home/#      │         │ /home/temp   │        │ /home/temp   │ │
│  └──────────────┘         └──────────────┘        │ /home/humid  │ │
│                                                   └──────────────┘ │
│                                                                      │
│  QoS LEVELS:                                                        │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ QoS │ Name            │ Description                           │ │
│  ├─────┼─────────────────┼───────────────────────────────────────┤ │
│  │  0  │ At most once    │ Fire and forget, no acknowledgment   │ │
│  │  1  │ At least once   │ Acknowledged, may duplicate          │ │
│  │  2  │ Exactly once    │ 4-step handshake, guaranteed single  │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### MQTT Client Implementation

```c
/* ================================================================
 * MQTT Client for ESP32 IoT Device
 * ================================================================ */

#include <stdint.h>
#include <stdbool.h>
#include <string.h>
#include "esp_mqtt_client.h"
#include "esp_log.h"
#include "cJSON.h"

static const char *TAG = "MQTT_CLIENT";

/* MQTT configuration */
#define MQTT_BROKER_URI     "mqtts://mqtt.example.com:8883"
#define MQTT_CLIENT_ID      "sensor_node_001"
#define MQTT_USERNAME       "iot_device"
#define MQTT_TOPIC_TELEMETRY    "devices/sensor_001/telemetry"
#define MQTT_TOPIC_COMMAND      "devices/sensor_001/command"
#define MQTT_TOPIC_STATUS       "devices/sensor_001/status"

/* Client handle */
static esp_mqtt_client_handle_t mqtt_client = NULL;
static bool mqtt_connected = false;

/* ================================================================
 * Sensor Data Structure
 * ================================================================ */
typedef struct {
    float temperature;
    float humidity;
    float pressure;
    uint32_t battery_mv;
    uint32_t timestamp;
} sensor_data_t;

/* ================================================================
 * MQTT Event Handler
 * ================================================================ */
static void mqtt_event_handler(void *handler_args, 
                                esp_event_base_t base,
                                int32_t event_id, 
                                void *event_data) {
    esp_mqtt_event_handle_t event = event_data;
    
    switch ((esp_mqtt_event_id_t)event_id) {
        case MQTT_EVENT_CONNECTED:
            ESP_LOGI(TAG, "Connected to broker");
            mqtt_connected = true;
            
            /* Subscribe to command topic */
            esp_mqtt_client_subscribe(mqtt_client, 
                                       MQTT_TOPIC_COMMAND, 1);
            
            /* Publish online status (Last Will was "offline") */
            esp_mqtt_client_publish(mqtt_client, 
                                     MQTT_TOPIC_STATUS,
                                     "{\"status\":\"online\"}", 0, 1, 1);
            break;
            
        case MQTT_EVENT_DISCONNECTED:
            ESP_LOGW(TAG, "Disconnected from broker");
            mqtt_connected = false;
            break;
            
        case MQTT_EVENT_DATA:
            ESP_LOGI(TAG, "Received: topic=%.*s", 
                     event->topic_len, event->topic);
            process_command(event->data, event->data_len);
            break;
            
        case MQTT_EVENT_ERROR:
            ESP_LOGE(TAG, "MQTT error occurred");
            break;
            
        default:
            break;
    }
}

/* ================================================================
 * Initialize MQTT Client with TLS
 * ================================================================ */
void mqtt_init(const char *ca_cert) {
    esp_mqtt_client_config_t mqtt_cfg = {
        .broker = {
            .address.uri = MQTT_BROKER_URI,
            .verification.certificate = ca_cert,
        },
        .credentials = {
            .username = MQTT_USERNAME,
            .client_id = MQTT_CLIENT_ID,
            .authentication = {
                .password = get_device_token(),  /* From secure storage */
            },
        },
        .session = {
            .last_will = {
                .topic = MQTT_TOPIC_STATUS,
                .msg = "{\"status\":\"offline\"}",
                .qos = 1,
                .retain = true,
            },
            .keepalive = 60,
        },
    };
    
    mqtt_client = esp_mqtt_client_init(&mqtt_cfg);
    esp_mqtt_client_register_event(mqtt_client, ESP_EVENT_ANY_ID,
                                    mqtt_event_handler, NULL);
    esp_mqtt_client_start(mqtt_client);
}

/* ================================================================
 * Publish Sensor Data as JSON
 * ================================================================ */
void mqtt_publish_telemetry(const sensor_data_t *data) {
    if (!mqtt_connected) {
        ESP_LOGW(TAG, "Not connected, queuing message");
        queue_message(data);  /* Store for later */
        return;
    }
    
    /* Create JSON payload */
    cJSON *root = cJSON_CreateObject();
    cJSON_AddNumberToObject(root, "temperature", data->temperature);
    cJSON_AddNumberToObject(root, "humidity", data->humidity);
    cJSON_AddNumberToObject(root, "pressure", data->pressure);
    cJSON_AddNumberToObject(root, "battery_mv", data->battery_mv);
    cJSON_AddNumberToObject(root, "timestamp", data->timestamp);
    
    char *payload = cJSON_PrintUnformatted(root);
    
    /* Publish with QoS 1 (at least once delivery) */
    int msg_id = esp_mqtt_client_publish(
        mqtt_client,
        MQTT_TOPIC_TELEMETRY,
        payload,
        strlen(payload),
        1,      /* QoS 1 */
        0       /* Not retained */
    );
    
    ESP_LOGI(TAG, "Published msg_id=%d: %s", msg_id, payload);
    
    cJSON_free(payload);
    cJSON_Delete(root);
}

/* ================================================================
 * Process Incoming Commands
 * ================================================================ */
void process_command(const char *data, int len) {
    cJSON *root = cJSON_ParseWithLength(data, len);
    if (root == NULL) {
        ESP_LOGE(TAG, "Invalid JSON command");
        return;
    }
    
    cJSON *cmd = cJSON_GetObjectItem(root, "command");
    if (cJSON_IsString(cmd)) {
        if (strcmp(cmd->valuestring, "reboot") == 0) {
            ESP_LOGI(TAG, "Reboot command received");
            esp_restart();
        }
        else if (strcmp(cmd->valuestring, "set_interval") == 0) {
            cJSON *interval = cJSON_GetObjectItem(root, "value");
            if (cJSON_IsNumber(interval)) {
                set_report_interval(interval->valueint);
            }
        }
        else if (strcmp(cmd->valuestring, "ota_update") == 0) {
            cJSON *url = cJSON_GetObjectItem(root, "url");
            if (cJSON_IsString(url)) {
                start_ota_update(url->valuestring);
            }
        }
    }
    
    cJSON_Delete(root);
}
```

---

## Smart Home Sensor Node

```
┌─────────────────────────────────────────────────────────────────────┐
│              SMART HOME SENSOR NODE DESIGN                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     SENSOR NODE PCB                            │ │
│  │                                                                │ │
│  │  ┌─────────────────────────────────────────────────────────┐  │ │
│  │  │                    POWER SUPPLY                          │  │ │
│  │  │                                                          │  │ │
│  │  │   ┌──────────┐     ┌──────────┐     ┌──────────┐        │  │ │
│  │  │   │ 2xAA     │────▶│   LDO    │────▶│  3.3V    │        │  │ │
│  │  │   │ Battery  │     │ TPS7A02  │     │  Rail    │        │  │ │
│  │  │   │ 3.0V     │     │ (1µA Iq) │     │          │        │  │ │
│  │  │   └──────────┘     └──────────┘     └────┬─────┘        │  │ │
│  │  └──────────────────────────────────────────┼──────────────┘  │ │
│  │                                              │                 │ │
│  │  ┌──────────────────────────────────────────┴──────────────┐  │ │
│  │  │                   MCU (ESP32-C3)                         │  │ │
│  │  │                                                          │  │ │
│  │  │   ┌──────────────────────────────────────────────────┐  │  │ │
│  │  │   │  RISC-V Core @ 160MHz                            │  │  │ │
│  │  │   │  • 400KB SRAM                                    │  │  │ │
│  │  │   │  • 4MB Flash                                     │  │  │ │
│  │  │   │  • WiFi 802.11 b/g/n                            │  │  │ │
│  │  │   │  • BLE 5.0                                       │  │  │ │
│  │  │   │  • Deep sleep: 5µA                              │  │  │ │
│  │  │   └──────────────────────────────────────────────────┘  │  │ │
│  │  └────────────────────────────────────────────────────────┘  │ │
│  │         │           │            │            │               │ │
│  │         │ I2C       │ GPIO       │ ADC        │ PWM           │ │
│  │         ▼           ▼            ▼            ▼               │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │ │
│  │  │  BME280  │ │   PIR    │ │   LDR    │ │   RGB    │         │ │
│  │  │Temp/Humid│ │  Motion  │ │  Light   │ │   LED    │         │ │
│  │  │ Pressure │ │  Sensor  │ │  Sensor  │ │ Status   │         │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘         │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  SOFTWARE ARCHITECTURE:                                              │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │  ┌────────────────────────────────────────────────────────┐   │ │
│  │  │                    APPLICATION                          │   │ │
│  │  │   • Sensor reading (periodic)                          │   │ │
│  │  │   • Motion event handling                               │   │ │
│  │  │   • MQTT publish/subscribe                             │   │ │
│  │  │   • OTA update handling                                 │   │ │
│  │  └──────────────────────────┬─────────────────────────────┘   │ │
│  │                              │                                 │ │
│  │  ┌──────────────────────────┴─────────────────────────────┐   │ │
│  │  │                 MIDDLEWARE                              │   │ │
│  │  │   • MQTT client library                                │   │ │
│  │  │   • JSON parser (cJSON)                                │   │ │
│  │  │   • NVS (Non-Volatile Storage)                         │   │ │
│  │  │   • Power management                                    │   │ │
│  │  └──────────────────────────┬─────────────────────────────┘   │ │
│  │                              │                                 │ │
│  │  ┌──────────────────────────┴─────────────────────────────┐   │ │
│  │  │              ESP-IDF RTOS (FreeRTOS)                    │   │ │
│  │  │   • WiFi driver                                        │   │ │
│  │  │   • TCP/IP stack (lwIP)                                │   │ │
│  │  │   • TLS (mbedTLS)                                      │   │ │
│  │  │   • Deep sleep management                               │   │ │
│  │  └────────────────────────────────────────────────────────┘   │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  POWER BUDGET (Sleep + 1 report/minute):                           │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ State           │ Current │ Duration │ Charge per hour       │ │
│  ├─────────────────┼─────────┼──────────┼───────────────────────┤ │
│  │ Deep sleep      │ 5µA     │ 59s      │ 0.082 mAh             │ │
│  │ Wake + sense    │ 5mA     │ 50ms     │ 0.004 mAh             │ │
│  │ WiFi connect    │ 120mA   │ 500ms    │ 0.017 mAh             │ │
│  │ MQTT publish    │ 80mA    │ 200ms    │ 0.004 mAh             │ │
│  ├─────────────────┼─────────┼──────────┼───────────────────────┤ │
│  │ Total per hour  │         │          │ ~0.11 mAh             │ │
│  │ 2xAA (2500mAh)  │         │          │ ~2.5 years            │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Industrial IoT (IIoT)

```
┌─────────────────────────────────────────────────────────────────────┐
│              INDUSTRIAL IoT ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    FACTORY FLOOR                               │ │
│  │                                                                │ │
│  │  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐         │ │
│  │  │   CNC       │   │  Robot      │   │  Conveyor   │         │ │
│  │  │   Machine   │   │  Arm        │   │  System     │         │ │
│  │  │             │   │             │   │             │         │ │
│  │  │  ┌───────┐  │   │  ┌───────┐  │   │  ┌───────┐  │         │ │
│  │  │  │ Vibr. │  │   │  │ Torque│  │   │  │ Speed │  │         │ │
│  │  │  │ Sensor│  │   │  │ Sensor│  │   │  │ Sensor│  │         │ │
│  │  │  └───┬───┘  │   │  └───┬───┘  │   │  └───┬───┘  │         │ │
│  │  │      │      │   │      │      │   │      │      │         │ │
│  │  │  ┌───┴───┐  │   │  ┌───┴───┐  │   │  ┌───┴───┐  │         │ │
│  │  │  │ PLC   │  │   │  │ PLC   │  │   │  │ PLC   │  │         │ │
│  │  │  └───┬───┘  │   │  └───┬───┘  │   │  └───┬───┘  │         │ │
│  │  └──────┼──────┘   └──────┼──────┘   └──────┼──────┘         │ │
│  │         │                 │                 │                 │ │
│  │         └─────────────────┼─────────────────┘                 │ │
│  │                           │                                   │ │
│  │                    Industrial Ethernet                        │ │
│  │                    (PROFINET/EtherNet/IP)                     │ │
│  │                           │                                   │ │
│  └───────────────────────────┼───────────────────────────────────┘ │
│                              │                                      │
│  ┌───────────────────────────▼───────────────────────────────────┐ │
│  │                      EDGE GATEWAY                              │ │
│  │                                                                │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐           │ │
│  │  │ OPC UA       │ │ Protocol    │ │ Local        │           │ │
│  │  │ Server       │ │ Converter   │ │ Analytics    │           │ │
│  │  │              │ │ (Modbus→MQTT│ │ (Edge AI)    │           │ │
│  │  └──────────────┘ └──────────────┘ └──────────────┘           │ │
│  │                                                                │ │
│  │  Key Functions:                                                │ │
│  │  • Data aggregation from multiple PLCs                        │ │
│  │  • Protocol translation (OPC-UA, Modbus, MQTT)                │ │
│  │  • Local buffering (network resilience)                       │ │
│  │  • Anomaly detection at edge                                  │ │
│  │  • Predictive maintenance ML inference                        │ │
│  │                                                                │ │
│  └────────────────────────────┬──────────────────────────────────┘ │
│                               │                                     │
│                        Secure VPN / MQTT                            │
│                               │                                     │
│  ┌────────────────────────────▼──────────────────────────────────┐ │
│  │                    CLOUD PLATFORM                              │ │
│  │                                                                │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │ │
│  │  │ Digital  │  │Predictive│  │   KPI    │  │  Alert   │      │ │
│  │  │  Twin    │  │Maintenanc│  │Dashboard │  │ Manager  │      │ │
│  │  │          │  │   e ML   │  │          │  │          │      │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Over-the-Air (OTA) Updates

```
┌─────────────────────────────────────────────────────────────────────┐
│              OTA UPDATE ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DUAL-PARTITION (A/B) UPDATE:                                       │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                       FLASH MEMORY                            │  │
│  │                                                               │  │
│  │  ┌───────────────────────────────────────────────────────┐   │  │
│  │  │  0x0000  BOOTLOADER                                   │   │  │
│  │  │          • Version check                              │   │  │
│  │  │          • Partition selection                        │   │  │
│  │  │          • Rollback handling                          │   │  │
│  │  ├───────────────────────────────────────────────────────┤   │  │
│  │  │  0x10000 PARTITION A (Running)          ◀─── ACTIVE   │   │  │
│  │  │          Firmware v1.2.0                              │   │  │
│  │  │          CRC: 0xABCD1234                              │   │  │
│  │  ├───────────────────────────────────────────────────────┤   │  │
│  │  │  0x110000 PARTITION B (OTA Target)     ◀─── UPDATING │   │  │
│  │  │          Firmware v1.3.0 (downloading...)             │   │  │
│  │  │          CRC: 0xEF567890                              │   │  │
│  │  ├───────────────────────────────────────────────────────┤   │  │
│  │  │  0x210000 NVS (Configuration)                         │   │  │
│  │  │          WiFi credentials, device ID, settings        │   │  │
│  │  ├───────────────────────────────────────────────────────┤   │  │
│  │  │  0x220000 OTA Data (Partition table)                  │   │  │
│  │  │          Active partition, boot count, flags          │   │  │
│  │  └───────────────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  UPDATE FLOW:                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   ┌──────────┐    ┌──────────┐    ┌──────────┐              │  │
│  │   │ Cloud    │───▶│ Device   │───▶│ Verify   │              │  │
│  │   │ Notify   │    │ Download │    │ Firmware │              │  │
│  │   └──────────┘    └──────────┘    └────┬─────┘              │  │
│  │                                        │                     │  │
│  │                                   ┌────▼─────┐               │  │
│  │                                   │Signature │               │  │
│  │                                   │  Valid?  │               │  │
│  │                                   └────┬─────┘               │  │
│  │                          Yes──────────┴──────────No          │  │
│  │                           │                       │          │  │
│  │                     ┌─────▼─────┐          ┌─────▼─────┐    │  │
│  │                     │  Write to │          │  Discard  │    │  │
│  │                     │Partition B│          │  Update   │    │  │
│  │                     └─────┬─────┘          └───────────┘    │  │
│  │                           │                                  │  │
│  │                     ┌─────▼─────┐                           │  │
│  │                     │  Set Boot │                           │  │
│  │                     │  Flag B   │                           │  │
│  │                     └─────┬─────┘                           │  │
│  │                           │                                  │  │
│  │                     ┌─────▼─────┐                           │  │
│  │                     │  Reboot   │                           │  │
│  │                     └─────┬─────┘                           │  │
│  │                           │                                  │  │
│  │                     ┌─────▼─────┐                           │  │
│  │                     │ Boot from │                           │  │
│  │                     │Partition B│                           │  │
│  │                     └─────┬─────┘                           │  │
│  │                           │                                  │  │
│  │                     ┌─────▼─────┐                           │  │
│  │                     │Self-Test  │                           │  │
│  │                     │ Passed?   │                           │  │
│  │                     └─────┬─────┘                           │  │
│  │          Yes──────────────┴──────────────No                 │  │
│  │           │                               │                  │  │
│  │     ┌─────▼─────┐                  ┌─────▼─────┐            │  │
│  │     │  Confirm  │                  │ Rollback  │            │  │
│  │     │  Update   │                  │    to A   │            │  │
│  │     └───────────┘                  └───────────┘            │  │
│  │                                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### OTA Update Code

```c
/* ================================================================
 * OTA Update Handler for ESP32
 * ================================================================ */

#include "esp_ota_ops.h"
#include "esp_http_client.h"
#include "esp_https_ota.h"
#include "esp_log.h"

static const char *TAG = "OTA";

/* Firmware signature verification key (public key) */
extern const uint8_t server_cert_pem_start[] asm("_binary_ca_cert_pem_start");

/* ================================================================
 * Perform OTA Update
 * ================================================================ */
esp_err_t perform_ota_update(const char *url) {
    esp_err_t err;
    
    ESP_LOGI(TAG, "Starting OTA update from: %s", url);
    
    /* Configure HTTP client for firmware download */
    esp_http_client_config_t http_config = {
        .url = url,
        .cert_pem = (const char *)server_cert_pem_start,
        .timeout_ms = 30000,
        .keep_alive_enable = true,
    };
    
    /* Configure OTA */
    esp_https_ota_config_t ota_config = {
        .http_config = &http_config,
    };
    
    /* Get running partition info */
    const esp_partition_t *running = esp_ota_get_running_partition();
    ESP_LOGI(TAG, "Running partition: %s", running->label);
    
    /* Initialize OTA */
    esp_https_ota_handle_t ota_handle = NULL;
    err = esp_https_ota_begin(&ota_config, &ota_handle);
    if (err != ESP_OK) {
        ESP_LOGE(TAG, "OTA begin failed: %s", esp_err_to_name(err));
        return err;
    }
    
    /* Get update partition */
    const esp_partition_t *update_partition = 
        esp_ota_get_next_update_partition(NULL);
    ESP_LOGI(TAG, "Writing to partition: %s", update_partition->label);
    
    /* Download and write firmware */
    int binary_file_length = 0;
    while (1) {
        err = esp_https_ota_perform(ota_handle);
        if (err != ESP_ERR_HTTPS_OTA_IN_PROGRESS) {
            break;
        }
        
        /* Report progress */
        binary_file_length = esp_https_ota_get_image_len_read(ota_handle);
        ESP_LOGI(TAG, "Downloaded %d bytes", binary_file_length);
    }
    
    if (err != ESP_OK) {
        ESP_LOGE(TAG, "OTA failed: %s", esp_err_to_name(err));
        esp_https_ota_abort(ota_handle);
        return err;
    }
    
    /* Verify and complete */
    err = esp_https_ota_finish(ota_handle);
    if (err != ESP_OK) {
        ESP_LOGE(TAG, "OTA finish failed: %s", esp_err_to_name(err));
        return err;
    }
    
    ESP_LOGI(TAG, "OTA successful, rebooting...");
    esp_restart();
    
    return ESP_OK;  /* Never reached */
}

/* ================================================================
 * Rollback Check on Boot
 * ================================================================ */
void ota_rollback_check(void) {
    const esp_partition_t *running = esp_ota_get_running_partition();
    esp_ota_img_states_t ota_state;
    
    if (esp_ota_get_state_partition(running, &ota_state) == ESP_OK) {
        if (ota_state == ESP_OTA_IMG_PENDING_VERIFY) {
            ESP_LOGI(TAG, "First boot after OTA, running self-test");
            
            /* Perform self-test */
            if (perform_self_test()) {
                ESP_LOGI(TAG, "Self-test passed, confirming update");
                esp_ota_mark_app_valid_cancel_rollback();
            } else {
                ESP_LOGE(TAG, "Self-test failed, rolling back");
                esp_ota_mark_app_invalid_rollback_and_reboot();
            }
        }
    }
}

/* ================================================================
 * Self-Test After Update
 * ================================================================ */
bool perform_self_test(void) {
    /* Test 1: Check sensor connectivity */
    if (!sensor_init_test()) {
        ESP_LOGE(TAG, "Sensor test failed");
        return false;
    }
    
    /* Test 2: Check network connectivity */
    if (!network_connectivity_test()) {
        ESP_LOGE(TAG, "Network test failed");
        return false;
    }
    
    /* Test 3: Check MQTT connection */
    if (!mqtt_connection_test()) {
        ESP_LOGE(TAG, "MQTT test failed");
        return false;
    }
    
    /* Test 4: Verify critical configuration */
    if (!config_integrity_test()) {
        ESP_LOGE(TAG, "Config test failed");
        return false;
    }
    
    ESP_LOGI(TAG, "All self-tests passed");
    return true;
}
```

---

## IoT Security

```
┌─────────────────────────────────────────────────────────────────────┐
│              IoT SECURITY ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DEFENSE IN DEPTH:                                                   │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Layer 1: DEVICE SECURITY                                      │ │
│  │   • Secure boot (verified bootloader chain)                   │ │
│  │   • Hardware crypto (AES, SHA, ECDSA)                         │ │
│  │   • Secure element for key storage                            │ │
│  │   • Flash encryption                                          │ │
│  │   • Fuse protection (disable JTAG)                           │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│  ┌───────────────────────────▼───────────────────────────────────┐ │
│  │ Layer 2: COMMUNICATION SECURITY                               │ │
│  │   • TLS 1.3 for all connections                              │ │
│  │   • Mutual authentication (mTLS)                              │ │
│  │   • Certificate rotation                                      │ │
│  │   • Encrypted payload (additional layer)                      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│  ┌───────────────────────────▼───────────────────────────────────┐ │
│  │ Layer 3: ACCESS CONTROL                                       │ │
│  │   • Device authentication tokens                              │ │
│  │   • Role-based access control (RBAC)                         │ │
│  │   • API rate limiting                                         │ │
│  │   • Command authorization                                     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│  ┌───────────────────────────▼───────────────────────────────────┐ │
│  │ Layer 4: UPDATE SECURITY                                      │ │
│  │   • Code signing (ECDSA P-256)                               │ │
│  │   • Version rollback protection                               │ │
│  │   • Secure OTA with integrity verification                   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  SECURE BOOT CHAIN:                                                  │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐    │ │
│  │   │   ROM       │────▶│  1st Stage  │────▶│  2nd Stage  │    │ │
│  │   │ Bootloader  │     │ Bootloader  │     │ Bootloader  │    │ │
│  │   │ (Immutable) │     │ (Signed)    │     │ (Signed)    │    │ │
│  │   └─────────────┘     └─────────────┘     └──────┬──────┘    │ │
│  │         │                    │                   │           │ │
│  │         ▼                    ▼                   ▼           │ │
│  │   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐    │ │
│  │   │ Verify hash │     │ Verify sig  │     │ Verify sig  │    │ │
│  │   │ of stage 1  │     │ of stage 2  │     │ of app      │    │ │
│  │   │ (eFuse key) │     │ (public key)│     │ (public key)│    │ │
│  │   └─────────────┘     └─────────────┘     └──────┬──────┘    │ │
│  │                                                  │           │ │
│  │                                           ┌──────▼──────┐    │ │
│  │                                           │ Application │    │ │
│  │                                           │  (Verified) │    │ │
│  │                                           └─────────────┘    │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **IoT Architecture** | Device → Gateway → Cloud layers with multiple protocols |
| **MQTT** | Lightweight pub/sub messaging protocol for IoT |
| **QoS Levels** | 0=fire-forget, 1=at-least-once, 2=exactly-once |
| **LoRa** | Long-range, low-power wireless for kilometers range |
| **BLE Mesh** | Bluetooth mesh for smart home device networks |
| **Edge Gateway** | Protocol translation, local processing, buffering |
| **IIoT** | Industrial IoT with PLC integration, OPC-UA |
| **OTA Update** | Dual-partition (A/B) update with rollback capability |
| **Secure Boot** | Verified boot chain from ROM to application |

---

## Quick Revision Questions

1. **Compare MQTT and CoAP protocols. When would you choose each for an IoT application?**

2. **Explain the publish/subscribe model in MQTT. What are the three QoS levels?**

3. **Design a power budget for a battery-powered sensor node that reports every 5 minutes. Target 2-year battery life.**

4. **What is the role of an edge gateway in industrial IoT? List five key functions.**

5. **Describe the A/B partition OTA update mechanism. How does rollback protection work?**

6. **What security measures should be implemented for an IoT device? Explain secure boot chain.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 8.2: Medical Devices](02-medical-devices.md) | [Unit 8: Case Studies](README.md) | [Chapter 8.4: Consumer Electronics](04-consumer-electronics.md) |

---
