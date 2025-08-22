# Notification System

A simple notification system for ESP32 applications using string keys for inter-task communication.

## Overview

This notification system provides:
- Thread-safe communication between tasks
- String-based notification keys
- Lightweight design with minimal overhead
- Manual memory management for flexibility

Perfect for inter-task communication in ESP32 projects.

## Features

- **Thread-safe**: Safe concurrent access with mutexes
- **Lightweight**: Simple design, minimal overhead
- **Key-based**: String keys for easy identification
- **Manual casting**: You control type casting and memory

## Basic Usage

### Include the Header

```cpp
#include "Notification.h"
```

### Create a Notification Instance

```cpp
Notification* notification = new Notification();
```

### Send Notifications

```cpp
// Send pointer to static data
static int temperature = 25;
bool success = notification->send("temperature", &temperature);

// Send pointer to string
static const char* status = "READY";
success = notification->send("status", (void*)status);

// Send pointer to struct
static struct {
    int id;
    float value;
} sensor = {1, 23.4f};
success = notification->send("sensor", &sensor);

// Send dynamically allocated data
uint8_t* buffer = malloc(100);
// ... fill buffer ...
success = notification->send("buffer", buffer);
// Note: You manage the buffer memory!
```

### Consume Notifications

```cpp
// Consume and cast manually
void* data = notification->consume("temperature");
if (data) {
    int* temp = (int*)data;
    printf("Temperature: %d\n", *temp);
}

// Consume string
data = notification->consume("status");
if (data) {
    const char* status = (const char*)data;
    printf("Status: %s\n", status);
}

// Consume with timeout (in milliseconds)
data = notification->consume("sensor", 1000);
if (data) {
    struct { int id; float value; }* sensor = data;
    printf("Sensor ID: %d, Value: %.1f\n", sensor->id, sensor->value);
}

// Consume dynamic data
data = notification->consume("buffer");
if (data) {
    uint8_t* buffer = (uint8_t*)data;
    // Use buffer...
    free(buffer);  // Free if it was malloc'd
}
```

## API Reference

### Core Methods

#### `bool send(const char* key, void* data)`
Send a notification with a void* pointer.
- `key`: String identifier
- `data`: Pointer to your data
- Returns: true if successful
- **Note**: You manage the memory

#### `void* consume(const char* key, uint32_t timeout_ms = 100)`
Consume a notification and get the void* pointer.
- `key`: String identifier
- `timeout_ms`: Timeout in milliseconds
- Returns: void* pointer (cast it yourself) or nullptr
- **Note**: You handle the casting

### Management Methods

#### `bool has(const char* key)`
Check if a notification exists.

#### `bool remove(const char* key)`
Remove a notification without consuming it.

#### `void clear()`
Remove all notifications.

#### `size_t count()`
Get number of pending notifications.

#### `bool wait(const char* key, uint32_t timeout_ms = 0)`
Wait for a notification to arrive (blocking). Use 0 for no timeout.

## Usage Patterns

### Static Data Communication

```cpp
// Producer
static int counter = 0;
void producerTask() {
    while (1) {
        counter++;
        notification->send("counter", &counter);
        delay(1000);
    }
}

// Consumer  
void consumerTask() {
    while (1) {
        void* data = notification->consume("counter", 0); // No timeout
        if (data) {
            int* count = (int*)data;
            printf("Counter: %d\n", *count);
        }
    }
}
```

### Dynamic Data Management

```cpp
// Producer allocates
void producer() {
    char* message = malloc(50);
    strcpy(message, "Hello World");
    notification->send("message", message);
    // You still own 'message' - don't free yet!
}

// Consumer takes ownership
void consumer() {
    void* data = notification->consume("message");
    if (data) {
        char* message = (char*)data;
        printf("Message: %s\n", message);
        free(message);  // Now free it
    }
}
```

### Event System

```cpp
// Define events
typedef enum {
    EVENT_BUTTON_PRESS,
    EVENT_SENSOR_READY,
    EVENT_WIFI_CONNECTED
} event_type_t;

// Send event
static event_type_t event = EVENT_BUTTON_PRESS;
notification->send("system_event", &event);

// Handle events
void* data = notification->consume("system_event");
if (data) {
    event_type_t* event = (event_type_t*)data;
    switch (*event) {
        case EVENT_BUTTON_PRESS:
            // Handle button press
            break;
        case EVENT_SENSOR_READY:
            // Handle sensor ready
            break;
        case EVENT_WIFI_CONNECTED:
            // Handle WiFi connection
            break;
    }
}
```

## Why This Approach?

1. **Simple**: Straightforward API with minimal complexity
2. **Flexible**: You control memory and casting decisions
3. **Efficient**: Lightweight implementation with string keys
4. **Safe**: Thread-safe operations with mutex protection
5. **Familiar**: Standard void* pointer pattern

## Thread Safety

Fully thread-safe using mutexes. Safe to use from:
- Multiple tasks running concurrently
- Different CPU cores on ESP32
- Various execution contexts

## Examples

See [`NotificationExample.cpp`](example/NotificationExample.cpp) for comprehensive usage examples including:

- Basic send/consume patterns with different data types
- Producer/consumer task examples
- Memory management strategies
- Event system implementation
- Advanced notification management

The example demonstrates real-world usage patterns for inter-task communication in ESP32 applications.
