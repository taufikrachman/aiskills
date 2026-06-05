# AI Agent Architecture

Build autonomous AI agents with tools, memory, and planning.

## Rules

### 1. Agent Loop Pattern
```
while (not done):
  1. Observe → current context + tool results
  2. Think   → LLM decides next action
  3. Act     → execute tool call
  4. Reflect → evaluate result, update memory
```

### 2. Tool Definition
```ts
interface Tool {
  name: string;
  description: string;           // LLM uses this to decide when to call
  parameters: z.ZodSchema;       // Validate inputs
  execute: (params: any) => Promise<ToolResult>;
  requiresApproval?: boolean;    // High-risk tools need human approval
}

type ToolResult = {
  success: boolean;
  data?: any;
  error?: string;
};
```
- Tools are simple, focused functions. One responsibility each.
- File operations: read, write, edit. Terminal: exec. Web: search, fetch.
- High-risk tools (delete, exec destructive): require human approval.
- Tool timeout: 30s default. Tools that hang block the agent.

### 3. Tool Calling Strategy
- Max 3 sequential tool calls per turn. Prevent infinite loops.
- Validate tool inputs with Zod BEFORE execution.
- If tool fails: return error to LLM, let it recover or try alternative.
- NEVER pass raw tool output to LLM — format/summarize/truncate.
- Log all tool calls: timestamp, input, output, duration.

### 4. Memory System
```
Working Memory  — Current conversation context (short-term)
Short-term      — Last N messages in session
Long-term       — Persistent memory store (vector DB + summaries)
```
- Summarize old messages when context window is full.
- Use embeddings for semantic search over long-term memory.
- Memory retrieval: "search memory for: topic" before generating.

### 5. Planning & Reasoning
```
Simple task:     direct LLM call (no planning needed)
Multi-step task: planner → executor → verifier
Complex task:    decompose → sub-agent per subtask → aggregate
```
- Explicit planning step for multi-step tasks: "Plan: 1. Search flight 2. Check weather 3. Suggest itinerary".
- Re-plan on failure: if step 2 fails, adjust plan.
- Sub-agents for parallelizable tasks (faster, cheaper for simple subtasks).

### 6. Safety Guardrails
- Input filter: reject harmful/injection prompts.
- Output filter: scan for PII, harmful content, prompt leakage.
- Budget cap: max N tool calls per session, max $X cost.
- Human-in-the-loop for: file deletion, external messaging, payments.
- Audit log: every decision, tool call, and output.

### 7. System Prompt Structure
```markdown
## Identity
You are X. Your role is Y.

## Tools
[List of available tools with descriptions]

## Rules
- [Safety rules]
- [Output format rules]
- [Error handling rules]

## Current Context
[Memory, user info, environment]
```
Keep system prompt under 2000 tokens. Update context section dynamically.
