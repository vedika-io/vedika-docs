---
title: "Streaming API | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/streaming
canonical: https://vedika.io/docs/sdk/streaming
---

# Streaming API

Real-time AI responses with Server-Sent Events

POST `/api/v1/astrology/query/stream` **$0.01-0.05/query

## Why Use Streaming?

#### Faster Perceived Response

Users see text immediately instead of waiting for full response

#### Better Chat Experience

Natural typing effect like human conversation

#### Handle Long Responses

No timeout issues for detailed readings

## Basic Streaming

```
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

// Stream a query response
const stream = await vedika.queryStream({
  question: 'What does my birth chart reveal about career?',
  birthDetails: {
    datetime: '1990-05-15T10:30:00+05:30',
    latitude: 28.6139,
    longitude: 77.2090
  }
});

// Handle streaming events
for await (const chunk of stream) {
  if (chunk.type === 'text') {
    process.stdout.write(chunk.text);  // Print without newline
  } else if (chunk.type === 'done') {
    console.log('\n\nStream complete!');
  }
}
```

## Stream Event Types

| Event Type | Description | Data |
| --- | --- | --- |
| `start` | Stream has started | `{ streamId: string }` |
| `text` | Text chunk received | `{ text: string }` |
| `chart_data` | Birth chart calculated | `{ chart: BirthChart }` |
| `thinking` | AI is processing | `{ stage: string }` |
| `done` | Stream complete | `{ usage: { tokens, cost } }` |
| `error` | Error occurred | `{ error: string, code: string }` |

## Advanced Event Handling

```
const stream = await vedika.queryStream({
  question: 'Analyze my career and relationships',
  birthDetails: birthDetails
});

let fullResponse = '';
let chartData = null;

for await (const event of stream) {
  switch (event.type) {
    case 'start':
      console.log('Stream started:', event.streamId);
      break;

    case 'thinking':
      console.log('Processing:', event.stage);
      // Stages: 'analyzing_chart', 'consulting_ai', 'generating_response'
      break;

    case 'chart_data':
      chartData = event.chart;
      console.log('Chart calculated. Sun sign:', chartData.sunSign);
      break;

    case 'text':
      fullResponse += event.text;
      process.stdout.write(event.text);
      break;

    case 'done':
      console.log('\n--- Stream Complete ---');
      console.log('Tokens used:', event.usage.tokens);
      console.log('Cost:', `$${event.usage.cost.toFixed(4)}`);
      break;

    case 'error':
      console.error('Stream error:', event.error);
      break;
  }
}

// Full response is now available
console.log('Full response length:', fullResponse.length);
```

## React Integration

```
import { useState, useCallback } from 'react';
import { VedikaClient } from '@vedika-io/sdk';

function AstrologyChat() {
  const [messages, setMessages] = useState([]);
  const [isStreaming, setIsStreaming] = useState(false);
  const [currentResponse, setCurrentResponse] = useState('');

  const vedika = new VedikaClient();

  const sendMessage = useCallback(async (question, birthDetails) => {
    setIsStreaming(true);
    setCurrentResponse('');

    // Add user message
    setMessages(prev => [...prev, { role: 'user', content: question }]);

    try {
      const stream = await vedika.queryStream({ question, birthDetails });

      let response = '';

      for await (const event of stream) {
        if (event.type === 'text') {
          response += event.text;
          setCurrentResponse(response);
        } else if (event.type === 'done') {
          // Finalize the message
          setMessages(prev => [...prev, {
            role: 'assistant',
            content: response
          }]);
          setCurrentResponse('');
        }
      }
    } catch (error) {
      console.error('Stream error:', error);
      setMessages(prev => [...prev, {
        role: 'error',
        content: 'Failed to get response. Please try again.'
      }]);
    } finally {
      setIsStreaming(false);
    }
  }, [vedika]);

  return (
    <div className="chat-container">
      {messages.map((msg, i) => (
        <div key={i} className={`message ${msg.role}`}>
          {msg.content}
        </div>
      ))}

      {isStreaming && currentResponse && (
        <div className="message assistant streaming">
          {currentResponse}
          <span className="cursor">|</span>
        </div>
      )}

      {/* Input form here */}
    </div>
  );
}
```

## Browser EventSource (Low-Level)

```
// Direct SSE connection (for custom implementations)
async function streamWithEventSource(question, birthDetails, apiKey) {
  const response = await fetch('https://api.vedika.io/api/v1/astrology/query/stream', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`
    },
    body: JSON.stringify({ question, birthDetails })
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = JSON.parse(line.slice(6));
        handleEvent(data);
      }
    }
  }
}

function handleEvent(event) {
  switch (event.type) {
    case 'text':
      document.getElementById('response').textContent += event.text;
      break;
    case 'done':
      console.log('Complete');
      break;
  }
}
```

## Cancel Streaming

```
// Create an AbortController
const controller = new AbortController();

// Start streaming with abort signal
const stream = await vedika.queryStream({
  question: 'Tell me everything about my chart',
  birthDetails,
  signal: controller.signal
});

// Cancel after 5 seconds
setTimeout(() => {
  console.log('Cancelling stream...');
  controller.abort();
}, 5000);

try {
  for await (const event of stream) {
    console.log(event.type, event.text?.slice(0, 50));
  }
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Stream was cancelled');
  } else {
    throw error;
  }
}
```

## Multi-Turn Conversation

```
// Stream with conversation history
const stream = await vedika.queryStream({
  question: 'What about my love life?',
  birthDetails,
  conversationId: 'conv_abc123',  // Continue existing conversation
  history: [
    { role: 'user', content: 'Analyze my birth chart' },
    { role: 'assistant', content: 'Based on your chart...' }
  ]
});

for await (const event of stream) {
  // AI has context from previous messages
  if (event.type === 'text') {
    console.log(event.text);
  }
}
```

## Related Topics

[ **

#### React Integration

Build chat UIs

](https://vedika.io/docs/sdk/react-integration) [ **

#### Error Handling

Handle stream errors

](https://vedika.io/docs/sdk/error-handling) [ **

#### Horoscope App

Build complete apps

](https://vedika.io/docs/sdk/horoscope-app)

---
_Full page (always current): [https://vedika.io/docs/sdk/streaming](https://vedika.io/docs/sdk/streaming) · [Vedika docs](https://vedika.io/docs)_
