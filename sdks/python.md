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
    # Optional: Override base URL (must include /api/v1). Default: https://api.pranthora.com/api/v1
    # base_url="http://localhost:5050/api/v1"
)
```

## Real-time Voice Calls (Outbound)

Start an **outbound phone call** from your application. The backend uses your attached Twilio number to call the given phone number and connects the call to the specified agent. The person you call hears and talks to the agent over the phone.

### Prerequisites

- A Pranthora account with an API key.
- At least one Twilio phone number configured for your user (used as the caller ID for outbound calls).

### Start a Call

```python
from pranthora import Pranthora

# Initialize client (use base_url for local dev, e.g. "http://localhost:5050/api/v1")
client = Pranthora(api_key="YOUR_API_KEY", base_url="https://api.pranthora.com/api/v1")

# Start an outbound call: call to_phone_number using the given agent (your Twilio number is used as caller ID)
result = client.start(
    agent_id="YOUR_AGENT_ID",
    to_phone_number="+1234567890",
)
print(f"Call started: {result['call_sid']}, from: {result['from_phone_number']}")
```

Optional **assistant overrides** (e.g. variables for the agent) — reserved for future use:

```python
assistant_overrides = {
    "variableValues": {
        "name": "John",
        "company": "Acme Corp"
    }
}
result = client.start(
    agent_id="YOUR_AGENT_ID",
    to_phone_number="+1234567890",
    assistant_overrides=assistant_overrides,
)
```

### Stop a Call

Hang up an active call. If you omit `call_sid` and `from_phone_number`, the client uses the last call from `start()`.

```python
# Stop the last call you started
client.stop()

# Or stop a specific call (use values returned from start())
client.stop(call_sid="CAxxxx...", from_phone_number="+19876543210")
```

### Minimal Example

```python
from pranthora import Pranthora

client = Pranthora(api_key="YOUR_API_KEY", base_url="https://api.pranthora.com/api/v1")

result = client.start(
    agent_id="YOUR_AGENT_ID",
    to_phone_number="+1234567890",
)
print(f"Call SID: {result['call_sid']}")

# Later: hang up
client.stop()
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

## Voice Calls (Telephony) — Lower-level API

You can also use the **calls** resource directly to create and stop calls. `client.start()` and `client.stop()` are convenience wrappers that use this under the hood.

### Create an Outbound Call

```python
# With a specific agent (recommended for outbound)
call = client.calls.create(
    phone_number="+1234567890",
    agent_id="YOUR_AGENT_ID",
)
print(f"Call initiated: {call['call_sid']}, from: {call['from_phone_number']}")

# Without agent_id: uses the agent mapped to your Twilio number
call = client.calls.create(phone_number="+1234567890")
```

### Stop (Hang Up) a Call

```python
# Requires call_sid and from_phone_number (from create() or start() response)
client.calls.stop(
    call_sid="CAxxxx...",
    from_phone_number="+19876543210",
)
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
