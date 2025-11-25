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

Start a voice session directly from your Python application, similar to the Vapi SDK. This connects your local microphone and speaker to the agent.

### Start a Call

```python
# Start a call with an existing agent
client.start(agent_id="YOUR_AGENT_ID")

# Or start with overrides
assistant_overrides = {
    "variableValues": {
        "name": "John"
    }
}
client.start(agent_id="YOUR_AGENT_ID", assistant_overrides=assistant_overrides)
```

### Stop a Call

```python
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
    transcriber="deepgram_nova_3"
)

print(f"Created agent: {agent['id']}")
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
