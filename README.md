# Scaler Agent

A Next.js + Gemini conversational agent that runs a strict
START -> THINK -> TOOL -> OBSERVE -> OUTPUT reasoning loop and clones the
Scaler Academy site into a self-contained `index.html`.

## Stack

- Next.js 16 (App Router) + React 19 + TypeScript + Tailwind v4
- Google Gemini (`gemini-2.5-flash-lite` by default — lowest tier, highest free daily quota) via `@google/generative-ai`
- Server-Sent Events stream from `/api/agent` to a React chat UI
- Sandboxed file output under `./generated/`

## Setup

```bash
npm install
cp .env.local.example .env.local
# put your key in .env.local:  GEMINI_API_KEY=...
# (optional) override the model:  GEMINI_MODEL=gemini-2.5-flash
npm run dev
```

Open http://localhost:3000.

## How it works

- The browser POSTs `{ message, history }` to `/api/agent`.
- The route runs the agent loop server-side and streams each step over SSE.
- `TOOL` steps execute one of: `getTheWeatherOfCity`, `getGithubDetailsAboutUser`,
  `executeCommand` (allow-listed: mkdir/ls/cat/echo/pwd, sandboxed to
  `./generated/`), `writeFile` (also sandboxed to `./generated/`).
- The generated `scaler_clone/index.html` is served by `/api/preview` and
  rendered in an `<iframe>` next to the chat.

## Project layout

```
src/
  app/
    api/agent/route.ts     # SSE agent loop
    api/preview/route.ts   # serves files from ./generated/
    page.tsx
    layout.tsx
  components/
    Chat.tsx
    StepBubble.tsx
    Preview.tsx
  lib/
    llm.ts                 # Groq client factory + safe JSON parse
    prompt.ts              # SYSTEM_PROMPT
    types.ts
    tools/
      index.ts             # tool_map
      weather.ts
      github.ts
      exec.ts
      writeFile.ts
generated/                 # runtime output (gitignored)
```

## Notes

- Filesystem-based output is intended for local dev. On read-only serverless
  hosts, switch `writeFile`/`exec` to in-memory and pipe content via SSE.
- Conversation state is held client-side: the server returns the updated
  message log on `done`, the client sends it back on the next turn.
