# MQTT HTTP Subscribe Example

This Node-RED flow acts as the opposite of the MQTT HTTP Bridge. While the bridge allows sending MQTT messages via HTTP requests, this flow lets you subscribe to MQTT topics via HTTP requests - similar to using `mosquitto_sub -C 1` command line.

## How It Works

1. When you make an HTTP GET request to `/mqtt/subscribe`, you can specify:
   - `topic`: The MQTT topic to subscribe to
   - `timeout`: How long to wait for a message (in milliseconds) before timing out
   - `host`: MQTT broker host (defaults to localhost)
   - `port`: MQTT broker port (defaults to 4140)

2. The HTTP request executes a `mosquitto_sub` command with the `-C 1` option to receive a single message, and `-W` to set a timeout.

3. The response is returned when either:
   - A message is received on the specified topic (success)
   - The timeout period expires (error)

## Usage Examples

### Using curl

Subscribe to a topic with default timeout (30 seconds):
```bash
curl "http://localhost:1880/mqtt/subscribe?topic=my/test/topic"
```

Subscribe with custom timeout (10 seconds):
```bash
curl "http://localhost:1880/mqtt/subscribe?topic=my/test/topic&timeout=10000"
```

Subscribe to a specific MQTT broker:
```bash
curl "http://localhost:1880/mqtt/subscribe?topic=my/test/topic&host=mqtt.example.com&port=1883"
```

### Successful response example:
```json
{
  "topic": "my/test/topic",
  "payload": "Hello MQTT World!",
  "timestamp": "2023-07-25T15:30:45.123Z"
}
```

### Timeout response example:
```json
{
  "error": "Timeout waiting for MQTT message",
  "topic": "my/test/topic",
  "timeout_ms": 30000
}
```

### Error response example:
```json
{
  "error": "Error executing MQTT subscription",
  "details": "Error message from mosquitto_sub",
  "topic": "my/test/topic"
}
```

## Integration with Shell Scripts

You can use this in shell scripts similar to using `mosquitto_sub -C 1` directly:

```bash
# Wait for a message and process it
RESPONSE=$(curl -s "http://localhost:1880/mqtt/subscribe?topic=system/status")
PAYLOAD=$(echo $RESPONSE | jq -r '.payload')
echo "Received: $PAYLOAD"
```

## Technical Implementation Notes

- This implementation uses the `exec` node to run `mosquitto_sub` command directly.
- The command is run with the `-C 1` flag (capture only one message) and `-W` for timeout.
- The HTTP response is sent directly using the HTTP response node, avoiding any issues with accessing response objects.
- Required dependencies: The `mosquitto_sub` client needs to be installed on the Node-RED server.
