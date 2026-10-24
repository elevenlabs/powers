# Agents Client Tools

Use this file only when the agent must call webhook tools, browser client tools, or built-in system tools.

## Tool Types

| Type | Runs where | Use for |
|------|------------|---------|
| Webhook | Your backend over HTTP | Secure API calls, database access, private business logic |
| Client | The browser or app client | UI changes, navigation, local state |
| System | Built into ElevenLabs | End call or transfer actions |

## Webhook Tools

```python
agent = client.conversational_ai.agents.create(
    name="Weather Assistant",
    tools=[{
        "type": "webhook",
        "name": "get_weather",
        "description": "Get current weather for a city. Use when the user asks about weather.",
        "webhook": {
            "url": "https://api.example.com/weather",
            "method": "POST",
            "headers": {
                "Authorization": "Bearer {{API_KEY}}"
            }
        },
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "City name"
                }
            },
            "required": ["city"]
        }
    }],
)
```

### Incoming Webhook Payload

```json
{
  "tool_call_id": "call_abc123",
  "tool_name": "get_weather",
  "parameters": {
    "city": "San Francisco"
  },
  "conversation_id": "conv_xyz789"
}
```

### Webhook Response

```json
{
  "result": "The weather in San Francisco is 68 F and sunny."
}
```

## Client Tools

Register client tools when the conversation starts:

```javascript
import { Conversation } from "@elevenlabs/client";

const conversation = await Conversation.startSession({
  agentId: "your-agent-id",
  clientTools: {
    show_product: async ({ productId }) => {
      return { success: true, shown: productId };
    },
    navigate_to: async ({ page }) => {
      window.location.href = `/${page}`;
      return { success: true };
    },
  },
});
```

Also describe those tools in the agent config so the model knows when to use them.

## System Tools

### End Call

```python
tools=[{"type": "system", "name": "end_call"}]
```

### Transfer To Number

```python
tools=[{
    "type": "system",
    "name": "transfer_to_number",
    "phone_number": "+1234567890",
    "description": "Transfer to human support",
}]
```

### Transfer To Agent

```python
tools=[{
    "type": "system",
    "name": "transfer_to_agent",
    "agent_id": "other-agent-id",
    "description": "Transfer to sales specialist",
}]
```

## Best Practices

- Write explicit descriptions that tell the model when a tool should be used.
- Make parameter descriptions concrete and example-driven.
- Keep webhook latency low and set reasonable timeouts.
- Return structured, usable errors so the model can recover gracefully.
- Put secure or authoritative actions in webhook tools, not client tools.
