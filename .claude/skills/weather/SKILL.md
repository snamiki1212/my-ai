---
name: weather
description: Search Google for current weather and display it. Use when the user asks about weather for a location.
argument-hint: "[location]"
allowed-tools: WebSearch, WebFetch
---

Search Google for the current weather in "$ARGUMENTS" (if no location is provided, ask the user for one).

## Steps

1. Use WebSearch with the query: "$ARGUMENTS weather today" (or the Japanese equivalent if appropriate)
2. If needed, use WebFetch to get more details from a weather page in the results
3. Display the weather in this format:

```
📍 Location
🌤 Condition
🌡 Temperature (high/low)
💧 Humidity
🌬 Wind
```

Keep the output concise and readable. Respond in the same language the user used.
