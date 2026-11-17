# Honeyb MCP server

AI search visibility for agents. Honeyb asks ChatGPT, Google AI Overviews, Google AI Mode, Gemini, Claude, Perplexity, Grok and DeepSeek the questions your buyers ask, reads which brands and sources each answer names, and checks Google rankings.

This is a hosted remote server. There is no code to install or run. Add one URL to Claude Code, Cursor, Codex or any MCP client and send your Honeyb API key. This repository holds the setup guide and the registry entry, `server.json`.

| | |
| --- | --- |
| Endpoint | `https://platform.honeyb.ai/mcp` |
| Transport | Streamable HTTP, stateless, one JSON-RPC message per request |
| Auth | `Authorization: Bearer hb_live_...` |
| Registry name | `ai.honeyb/honeyb` |
| Docs | https://www.honeyb.ai/platform/api/mcp |

## Get a key

Apply for API access at https://www.honeyb.ai/platform/api and tell us your engines and volume. A person issues your key within one working day. Keys start with `hb_live_`.

Keep the key in the `HONEYB_API_KEY` environment variable. Every config below reads it from there, so the key never sits in a file.

```bash
export HONEYB_API_KEY="hb_live_..."
```

## Set it up

### Claude Code

Add this to `.mcp.json` in your project:

```json
{
  "mcpServers": {
    "honeyb": {
      "type": "http",
      "url": "https://platform.honeyb.ai/mcp",
      "headers": { "Authorization": "Bearer ${HONEYB_API_KEY}" }
    }
  }
}
```

### Cursor

Add this to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "honeyb": {
      "url": "https://platform.honeyb.ai/mcp",
      "headers": { "Authorization": "Bearer ${env:HONEYB_API_KEY}" }
    }
  }
}
```

Or install it in one click: [Add to Cursor](https://cursor.com/en/install-mcp?name=honeyb&config=eyJ1cmwiOiJodHRwczovL3BsYXRmb3JtLmhvbmV5Yi5haS9tY3AiLCJoZWFkZXJzIjp7IkF1dGhvcml6YXRpb24iOiJCZWFyZXIgJHtlbnY6SE9ORVlCX0FQSV9LRVl9In19). The link writes the config above, and Cursor reads the key from `HONEYB_API_KEY`.

### Codex

Add this to `~/.codex/config.toml`:

```toml
[mcp_servers.honeyb]
url = "https://platform.honeyb.ai/mcp"
bearer_token_env_var = "HONEYB_API_KEY"
```

### Any other MCP client

Connect to `https://platform.honeyb.ai/mcp` over Streamable HTTP and send `Authorization: Bearer <your key>` with every request. A request without a key gets a 401 with `WWW-Authenticate: Bearer realm="honeyb"`.

### Let your agent set it up

Paste this prompt into Claude Code, Cursor or Codex. It holds no key. The agent adds the server, reads the key from the environment and runs a first task.

```text
Set up the Honeyb API in this project and run a first task.
My Honeyb API key is in the HONEYB_API_KEY environment variable. Read it from there every time. Never write the key into a file, a commit or this chat.
1. Add the Honeyb MCP server at https://platform.honeyb.ai/mcp (streamable HTTP) with the header "Authorization: Bearer" followed by the value of HONEYB_API_KEY.
- Claude Code: add a "honeyb" server to .mcp.json with "type": "http", "url": "https://platform.honeyb.ai/mcp" and "headers": { "Authorization": "Bearer ${HONEYB_API_KEY}" }
- Cursor: add a "honeyb" server to .cursor/mcp.json with "url": "https://platform.honeyb.ai/mcp" and "headers": { "Authorization": "Bearer ${env:HONEYB_API_KEY}" }
- Codex: add [mcp_servers.honeyb] to ~/.codex/config.toml with url = "https://platform.honeyb.ai/mcp" and bearer_token_env_var = "HONEYB_API_KEY"
2. Call list_engines_and_prices and show me the engines.
3. Ask me for my brand name, my website and one question my buyers ask AI. Ask me to confirm, because it spends credits, then call create_visibility_task with that brand, that question as the prompt, and the engines chatgpt, google_ai_overviews and google_ai_mode.
4. Poll get_task until the status is completed or failed.
5. Call get_task_results and get_task_citations, then tell me which brands each engine named and which sources it cited.
```

## Tools

Eight tools. Two spend credits: `create_visibility_task` asks the engines your prompts, and `create_rank_check` checks Google positions. The other six are free. `list_engines_and_prices` lets the agent quote the cost before it spends anything. Every tool runs the same operation as the REST API, on the same key.

| Tool | What it does | Credits |
| --- | --- | --- |
| `list_engines_and_prices` | Lists the engines a task can run on, whether each searches the web, and every live operation with its price and limits. | Free |
| `create_visibility_task` | Asks the engines you pick a set of prompts about a brand and reads every answer: whether the brand is named, the competitors beside it, the sentiment and the pages cited. | Spends credits |
| `get_task` | Returns a task's status, its answer counts and the credits held and charged. The agent polls it until the task finishes. | Free |
| `get_task_results` | Returns every answer, one row per prompt and engine: the answer text, the brand and competitor mentions and the sentiment. | Free |
| `get_task_citations` | Returns every page the engines cited, with its domain, page classification, position and whether the citing answer named the brand. | Free |
| `create_rank_check` | Checks where a domain ranks on Google for up to 1,000 keywords: position, ranking URL, SERP features and the AI Overview citation. | Spends credits |
| `get_rank_check` | Returns a rank check's status and keyword counts, and collects the keywords Google has finished. The agent polls it once a minute. | Free |
| `get_rank_check_results` | Returns one row per keyword: position, ranking URL and title, SERP features and whether the AI Overview cited the domain. | Free |

Tool annotations: the six read tools are `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: false`. The two create tools are `readOnlyHint: false`, `destructiveHint: false`, `idempotentHint: false`, `openWorldHint: true`.

## Engines and prices

ChatGPT, Google AI Overviews and Google AI Mode are captured from the live apps, with no memory and no personalization. The other engines answer through their model APIs, and every row says which engine it came from.

Prices in US dollars per 1,000 answers, as listed at https://www.honeyb.ai/platform/api.

| Engine | Slug | Collected from | Raw | Enriched |
| --- | --- | --- | --- | --- |
| ChatGPT | `chatgpt` | Live app | $1.10 | $3.00 |
| Google AI Overviews | `google_ai_overviews` | Live app | $1.10 | $3.00 |
| Google AI Mode | `google_ai_mode` | Live app | $1.10 | $3.00 |
| Gemini | `gemini` | Model API with web search | $1.10 | $3.00 |
| Claude | `claude` | Model API with web search | $6.00 | $8.00 |
| Perplexity | `perplexity` | Model API with web search | On request | On request |
| Grok | `grok` | Model API | On request | On request |
| DeepSeek | `deepseek` | Model API | On request | On request |

Google rank checks are $1.00 per 1,000 checks, read from the live Google results page, with the AI Overview citation.

Raw is the answer text and every source it cited. Enriched is the same answer read into the brands detected, the competitors named, the sentiment and every source classified.

Tool calls spend the same prepaid credits at the same prices as the REST API. Listing, polling and reading are free. Failed answers and unmeasured keywords are never charged, and credits do not expire. Every price: https://www.honeyb.ai/platform/api/pricing

## Limits

- Up to 300 prompts a task, each up to 500 characters, on any of the eight engines.
- Up to 10 competitors a task, always counted when named.
- Up to 1,000 keywords a rank check, at any Google location code, from a country down to a city.
- AI answers are collected in the United States.
- 3,000 requests an hour per key, shared with the REST API.
- Results are kept 30 days after submission.

## Links

- MCP server docs: https://www.honeyb.ai/platform/api/mcp
- Honeyb API: https://www.honeyb.ai/platform/api
- Pricing: https://www.honeyb.ai/platform/api/pricing
- Privacy policy: https://www.honeyb.ai/privacy-policy
- Terms of service: https://www.honeyb.ai/terms-of-service
- Contact: hello@honeyb.ai

## About this repository

The server runs at `platform.honeyb.ai`, so this repository holds no server code. `server.json` is the entry Honeyb publishes to the official MCP Registry under `ai.honeyb/honeyb`. Open an issue here or write to hello@honeyb.ai with questions.
