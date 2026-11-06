# open-ai-agent-multi-agent

A small multi-agent demo built with the [OpenAI Agents SDK for JavaScript](https://github.com/openai/openai-agents-js) (`@openai/agents`). It models customer support for an internet broadband company. It runs against any **OpenAI-compatible local model server**, such as LM Studio, Ollama or vLLM, so you don't need an OpenAI API key.

## How it works

```
User query
    │
    ▼
┌──────────────┐   fetch_available_plans   ┌───────────────────┐
│ Sales Agent  │ ────────────────────────▶ │ list of plans     │
│              │                           └───────────────────┘
│              │   refund_expert (agent as tool)
│              │ ────────────────────────▶ ┌──────────────┐  process_refund  ┌──────────────┐
└──────────────┘                           │ Refund Agent │ ───────────────▶ │ refunds.txt  │
                                           └──────────────┘                  └──────────────┘
```

- **Sales Agent** is the entry point. It talks to the user, looks up plans and hands refund requests to the Refund Agent.
- **Refund Agent** is exposed to the Sales Agent as a tool (`refundAgent.asTool(...)`). It owns the refund logic.
- **Tools**
  - `fetch_available_plans` returns a hard-coded list of plans (₹399 / 30MB/s, ₹999 / 100MB/s, ₹1499 / 200MB/s).
  - `process_refund` takes a `customerId` and `reason` and appends a refund record to `./refunds.txt`.

The SDK is configured for local servers:

- `setDefaultOpenAIClient` points the SDK at `baseURL` from `.env`.
- `setOpenAIAPI('chat_completions')` uses the Chat Completions API, because most local servers don't implement the Responses API.
- `setTracingDisabled(true)` stops the SDK from uploading traces to OpenAI.

## Prerequisites

- Node.js 18 or newer (the project uses ES modules)
- A running OpenAI-compatible server whose model supports **tool/function calling**

## Setup

```bash
npm install
```

Create a `.env` file in the project root:

```env
# OpenAI-compatible endpoint of your local server, e.g. LM Studio
baseURL=http://localhost:1234/v1
```

`.env` is git-ignored. The API key is set to a placeholder in code because local servers ignore it.

## Run

```bash
npm start
```

This runs a sample query against the Sales Agent:

> I had a plan 399. I need a refund right now. my cus id is cust123 because of I am shifting to a new place

The agent's final answer is printed to the console, and the refund is recorded in `refunds.txt`.

To try other queries, change the argument passed to `runAgent(...)` at the bottom of [index.js](index.js).

## Project structure

```
.
├── index.js        # agents, tools and entry point
├── package.json
└── .env            # local config (not committed)
```

## Tech stack

- [`@openai/agents`](https://www.npmjs.com/package/@openai/agents): agents, tools and the run loop
- [`openai`](https://www.npmjs.com/package/openai): client for the OpenAI-compatible endpoint
- [`zod`](https://zod.dev): tool parameter schemas
- [`dotenv`](https://www.npmjs.com/package/dotenv): loads `.env`
