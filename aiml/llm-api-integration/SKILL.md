# LLM API Integration

Integrate OpenAI, Claude, Gemini, and other LLM APIs into production applications.

## Rules

### 1. Provider Abstraction
```ts
interface LLMProvider {
  chat(messages: Message[], options?: ChatOptions): Promise<ChatResponse>;
  stream(messages: Message[], options?: ChatOptions): AsyncIterable<ChatChunk>;
}

class OpenAIProvider implements LLMProvider { ... }
class AnthropicProvider implements LLMProvider { ... }
class GeminiProvider implements LLMProvider { ... }
```
Abstract behind interface. Swap providers without changing application code.

### 2. Streaming
- ALWAYS use streaming for user-facing chat. Non-streaming for background/batch.
- Handle stream errors: reconnect, fallback to non-streaming, or show error mid-response.
- Use `AbortController` to cancel in-flight streams on unmount/navigation.
- Show incremental text as tokens arrive. Don't wait for full response.

### 3. Prompt Engineering Best Practices
- System prompt: set role, tone, rules, output format.
- User prompt: clear instruction + context + constraints.
- Few-shot examples: 2-3 examples for complex tasks.
- Output format instruction: "Respond in JSON with keys: { ... }".
- Token limits: track usage, truncate context window when near limit.

### 4. Function Calling
```ts
const tools = [{
  name: 'get_weather',
  description: 'Get current weather for a city',
  parameters: {
    type: 'object',
    properties: {
      city: { type: 'string', description: 'City name' },
    },
    required: ['city'],
  },
}];
const response = await llm.chat(messages, { tools });
if (response.toolCalls) {
  const result = await executeTool(response.toolCalls[0]);
  messages.push({ role: 'tool', content: JSON.stringify(result), toolCallId: response.toolCalls[0].id });
  const final = await llm.chat(messages); // Continue with tool result
}
```
- Validate tool inputs (Zod schema).
- Timeout tools (5s default).
- Sanitize tool outputs before sending back.
- Log all tool calls for debugging.

### 5. Error Handling
- 429 Rate Limit: exponential backoff (1s → 2s → 4s → 8s). Max 3 retries.
- 401 Invalid API Key: alert admin, not the user.
- 500 Server Error: retry once, then fallback to "AI is temporarily unavailable".
- Token limit exceeded: truncate oldest messages, retry with shorter context.
- Timeout: 30s for chat, 60s for streaming.

### 6. Cost Optimization
- Track token usage per request and per user.
- Cache identical prompts (same system + user message) with TTL.
- Use cheaper model for simple tasks: `gpt-4o-mini` for classification, `gpt-4o` for complex reasoning.
- Truncate conversation history to last N messages or last X tokens.
- Set hard monthly budget alerts.

### 7. Security
- NEVER expose API keys in client-side code. Proxy through backend.
- Sanitize user input: strip injection attempts, limit input length.
- Rate limit per user: 20 requests/min.
- Content filtering: scan input/output for PII, harmful content.
