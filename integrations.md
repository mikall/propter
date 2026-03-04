# Integrations

This file tells Claude Code how to connect to your agent. Fill in each section for your setup.

---

## Agent Input

How to call the agent with a test message.

```
Method:     [POST / MCP tool call / ...]
Endpoint:   [https://your-n8n.com/webhook/your-agent]
Headers:    [{"Content-Type": "application/json", "Authorization": "Bearer ..."}]
Body:       [{"message": "{{test_message}}", "sessionId": "prompt-optimizer-{{run_id}}"}]
```

**Notes:**
- Use a unique `sessionId` per run to avoid conversation state leaking between tests.
- If your agent requires additional fields (e.g., `userId`, `context`), add them to the body.

## Agent Output

Where to find the response in the agent's reply.

```
Response field:   [response.body.output]
Format:           [string / JSON / markdown]
```

## Tool Inspection

How to check which tools the agent used and measure latency.

```
Available:        [yes / no]
Method:           [execution log API / sub-workflow output / response metadata]
Tools field:      [response.body.tools_used]
Latency field:    [response.body.latency_s]
```

If tool inspection is not available, set `Available: no` and the `tool_usage` scoring dimension will be skipped.

## Prompt Storage & Update

Where the system prompt lives and how to update it.

```
Storage:          [Supabase table / file / environment variable / n8n node / ...]
Location:         [public.system_prompt → column "prompt" where agent_id = "your-agent"]
Read method:      [MCP tool: supabase SQL select / file read / API call]
Update method:    [MCP tool: supabase SQL update / file write / API call]
```

**Important:** The agent must load the prompt dynamically (not hardcoded) so updates take effect immediately.

---

## Example: n8n + Supabase

Below is a filled example for a typical n8n agent with prompts stored in Supabase.

### Agent Input
```
Method:     POST
Endpoint:   https://your-n8n.cloud/webhook/agent-chat
Headers:    {"Content-Type": "application/json"}
Body:       {"message": "{{test_message}}", "sessionId": "prompt-optimizer-{{run_id}}"}
```

### Agent Output
```
Response field:   response.body.output
Format:           string
```

### Tool Inspection
```
Available:        yes
Method:           n8n execution log API
Tools field:      (extracted from execution data)
Latency field:    (computed from execution start/end timestamps)
```

### Prompt Storage & Update
```
Storage:          Supabase
Location:         public.system_prompt → column "prompt" where agent_id = "my-agent"
Read method:      SELECT prompt FROM public.system_prompt WHERE agent_id = 'my-agent'
Update method:    UPDATE public.system_prompt SET prompt = '{{new_prompt}}', updated_at = now() WHERE agent_id = 'my-agent'
```
