---
title: Python SDK
subtitle: >-
  Build voice interfaces and backend integrations using Pranthora's Python SDK
slug: sdks/python
---

## Overview

The Pranthora Python SDK provides a simple and powerful way to interact with the Pranthora Voice Assistant Platform. It allows you to manage agents, initiate calls, and handle real-time voice interactions directly from your application.

## Installation

Install the package via pip:

```bash
pip install pranthora
```

For real-time voice features (microphone/speaker support), you also need `pyaudio` and `websockets`. On macOS, install `portaudio` first:

```bash
brew install portaudio
pip install pyaudio websockets
```

## Initialization

Initialize the client with your API key.

```python
from pranthora import Pranthora

client = Pranthora(
    api_key="YOUR_API_KEY",
    # Optional: Override base URL for development
    # base_url="http://localhost:5050" 
)
```

## Real-time Voice

Start a voice session directly from your Python application. This connects your local microphone and speaker to the agent for real-time conversation.

### Prerequisites

Install required audio libraries:

```bash
pip install pyaudio websockets
```

**Note**: On macOS, you may need to install PortAudio first:
```bash
brew install portaudio
```

### Start a Call

```python
from pranthora import Pranthora

# Initialize client
client = Pranthora(api_key="YOUR_API_KEY", base_url="https://api.pranthora.ai")

# Start a call with an existing agent
client.start(agent_id="YOUR_AGENT_ID")

# Or start with assistant overrides (variables)
assistant_overrides = {
    "variableValues": {
        "name": "John",
        "company": "Acme Corp"
    }
}
client.start(agent_id="YOUR_AGENT_ID", assistant_overrides=assistant_overrides)
```

### Stop a Call

```python
# Stop the current voice session
client.stop()
```

### Complete Example with Event Handlers

```python
from pranthora import Pranthora
import time

client = Pranthora(api_key="YOUR_API_KEY", base_url="https://api.pranthora.ai")

# Set up event callbacks
def on_connected():
    print("✅ Connected to voice session")

def on_disconnected():
    print("❌ Disconnected from voice session")

def on_first_response(message: str):
    print(f"✨ First response: {message}")

def on_transcript(role: str, text: str):
    print(f"📝 [{role}]: {text}")

def on_interruption():
    print("⚡ Interruption detected")

def on_agent_speaking_start():
    print("🤖 Agent started speaking")

def on_agent_speaking_stop():
    print("🤖 Agent stopped speaking")

def on_error(error: str):
    print(f"❌ Error: {error}")

# Attach callbacks
voice_client = client._voice_client
voice_client.on_connected = on_connected
voice_client.on_disconnected = on_disconnected
voice_client.on_first_response = on_first_response
voice_client.on_transcript = on_transcript
voice_client.on_interruption = on_interruption
voice_client.on_agent_speaking_start = on_agent_speaking_start
voice_client.on_agent_speaking_stop = on_agent_speaking_stop
voice_client.on_error = on_error

# Start the call
client.start(agent_id="YOUR_AGENT_ID")

# Keep the session running
try:
    while voice_client.is_running:
        time.sleep(1)
        # You can check statistics
        stats = voice_client.get_stats()
        print(f"Messages received: {stats['messages_received']}")
except KeyboardInterrupt:
    print("\nStopping call...")
    client.stop()
```

### Audio Format

The SDK sends audio as **raw PCM bytes** (16-bit, 24kHz, mono) directly to the WebSocket. The backend processes these bytes for voice activity detection, transcription, and response generation.

### Call Statistics

Get real-time statistics about the call:

```python
stats = client._voice_client.get_stats()
print(f"Duration: {stats.get('duration_seconds', 0)}s")
print(f"Messages received: {stats['messages_received']}")
print(f"Audio sent: {stats['audio_bytes_sent']} bytes")
print(f"Audio received: {stats['audio_bytes_received']} bytes")
print(f"First response received: {stats['first_response_received']}")
```

### Message Logs

Access detailed logs of the call:

```python
logs = client._voice_client.get_logs()
for log in logs:
    print(f"[{log['timestamp']}] {log['type']}: {log['message']}")
```

## Agents

Manage your voice agents programmatically.

### Create an Agent

Create a fully configured agent in a single step using friendly names.

```python
agent = client.agents.create(
    name="Sales Assistant",
    description="Handles outbound sales calls",
    model="gpt-4.1", 
    system_prompt="You are a friendly sales representative.",
    voice="thalia",
    transcriber="deepgram_nova_3",
    first_response_message="Hello! How can I help you today?"
)

print(f"Created agent: {agent['agent']['id']}")
```

### List All Agents

Get all agents for the current user.

```python
# Get all agents
agents = client.agents.list()

for agent in agents:
    agent_data = agent.get('agent', {})
    print(f"Name: {agent_data.get('name')}, ID: {agent_data.get('id')}")
    print(f"Status: {'Active' if agent_data.get('is_active') else 'Inactive'}")
```

### Get Agent by ID

Retrieve a specific agent with all its configurations.

```python
# Get a specific agent by ID
agent = client.agents.get(agent_id="YOUR_AGENT_ID")

print(f"Agent Name: {agent['agent']['name']}")
print(f"Status: {'Active' if agent['agent']['is_active'] else 'Inactive'}")

# Access configurations with friendly names
if 'configurations' in agent:
    configs = agent['configurations']
    if 'model' in configs:
        print(f"Model: {configs['model'].get('model_name', 'N/A')}")
    if 'tts' in configs:
        print(f"Voice: {configs['tts'].get('voice_name_friendly', 'N/A')}")
    if 'transcriber' in configs:
        print(f"Transcriber: {configs['transcriber'].get('transcriber_name', 'N/A')}")
```

### Update an Agent

Update agent properties and configurations.

```python
# Update agent name and description
updated_agent = client.agents.update(
    agent_id="YOUR_AGENT_ID",
    name="Updated Agent Name",
    description="Updated description"
)

# Update with configuration changes
updated_agent = client.agents.update(
    agent_id="YOUR_AGENT_ID",
    name="New Name",
    voice="darla",
    temperature=0.8,
    system_prompt="You are a helpful customer support agent."
)

print(f"Updated agent: {updated_agent['agent']['name']}")
```

### Delete an Agent

Delete an agent (force_delete=True by default).

```python
# Delete an agent
client.agents.delete(agent_id="YOUR_AGENT_ID")

# Or explicitly set force_delete
client.agents.delete(agent_id="YOUR_AGENT_ID", force_delete=True)
```

### Supported Models & Providers

The SDK supports a wide range of models and providers. You can use these friendly names directly in the `create` method.

<Card title="View All Models" icon="bot" href="/sdks/supported-models">
  Browse the complete list of supported LLM models, voices, and transcribers with their friendly names.
</Card>

## Voice Calls (Telephony)

Initiate and manage outbound telephone calls.

### Make an Outbound Call

Trigger an outbound call to a phone number.

```python
call = client.calls.create(
    phone_number="+1234567890"
)

print(f"Call initiated: {call.get('call_sid')}")
```

## Webhooks

Handle real-time events from Pranthora.

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/webhook/pranthora', methods=['POST'])
def handle_webhook():
    payload = request.get_json()
    
    # Process the webhook payload
    print(f"Received event: {payload}")
    
    return jsonify({"received": True})

if __name__ == '__main__':
    app.run(port=3000)
```
