---
layout: post
title: "How We Let AI Search Real Hotels: A Simple MCP Server, Explained"
date: 2026-08-04 08:00:00 +0800
categories: [mcp, ai, travel]
---

AI assistants are great at *recommending* hotels. They're terrible at *finding you a real one you can actually book tonight*.

That gap — between a plausible answer and a real transaction — is exactly what [MCP (Model Context Protocol)](https://modelcontextprotocol.io) was built to close. (New to MCP? Start with my [plain-English intro](/what-is-mcp/).)

This post walks through a small but real example: a simple MCP server that exposes **2M+ hotels in 200+ countries** to any AI assistant (Claude, Cursor, Codex, …) through three easy tools: `searchHotels`, `getHotelDetail`, and `getHotelSearchTags`.

## The problem

An AI travel agent built on memory alone has the same three failures:

- **Old hotels.** A hotel that closed last year still shows up.
- **Fake prices.** The AI "remembers" a rate that was last true in 2024.
- **No way to book.** Even a great recommendation is a dead end if there's no real price or real inventory behind it.

The fix is simple in spirit: let the AI call a real tool that checks real data. MCP is the standard way to do that.

## The design: keep it simple

The server is deliberately **thin**. It's a middleman:

1. An AI sends a request over MCP.
2. The server passes it to the hotel API.
3. The real result comes back to the AI.

```
┌──────────────┐  MCP request   ┌──────────────────────┐  HTTPS   ┌──────────────────┐
│ AI assistant │ ─────────────▶│ RollingGo Hotel MCP  │ ───────▶ │ Upstream API      │
│ (Claude, …)  │ ◀─────────────│  (this server)       │ ◀─────── │ mcp.rollinggo.ai  │
└──────────────┘  real result   └──────────────────────┘          └──────────────────┘
```

Two decisions keep it simple:

**1. No secrets stored on the server.** The API key travels with each request in the header. So any number of users can share one deployment safely.

**2. One shared connection.** All requests reuse a single connection pool, so it's fast and doesn't waste resources per call.

## The three tools

Each tool is a typed question the AI can ask:

| Tool | What it does |
|---|---|
| `searchHotels` | Finds matching hotels with a live lowest price |
| `getHotelDetail` | Drills into one hotel: room types, taxes, cancellation policy |
| `getHotelSearchTags` | Turns fuzzy words ("gym", "free WiFi") into real filter tags |

Together they form a natural flow: search → check the real price → look at the details → decide.

## How to connect it

Once you have a free API key from the [partner center](https://global.rollinggo.store/), any MCP client connects to `https://mcp.rollinggo.ai/mcp` with a small config:

```json
{
  "mcpServers": {
    "RollingGo-Hotel": {
      "url": "https://mcp.rollinggo.ai/mcp",
      "type": "streamable-http",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

Then "find me a five-star hotel near the Shanghai Bund" becomes a real search — no custom backend needed on your side.

## Things to watch out for

The useful part of any technical post is the sharp edges:

- **Price is a package, not a number.** The search result wraps the rate like `{"hasPrice": true, "currency": "USD", "lowestPrice": 648.0}`. Flatten it to a plain number and you lose the currency — and some fields can legitimately be empty.
- **Search price is a reference, not a quote.** Real-time rates are confirmed at the detail step. Treat search results as candidates.
- **Plain cURL fails with 400.** The endpoint needs `Accept: application/json, text/event-stream`. Forget it and every request fails with a cryptic error.
- **Keep upstream responses server-side.** Log them internally, but only send a status back to the caller — don't leak backend details.

## The honest version

This version **searches and quotes only**. Booking and payment are part of the OAuth-based enterprise tier, not the simple API-key path shown here. And the data is only as current as the upstream feed — this server adds no intelligence. It simply makes real hotel data speak the AI's language.

## The takeaway

A simple MCP server is a low-friction way to make a real, transactional domain — hotel inventory — usable by AI. The AI does the talking, the tool does the checking, and neither side has to guess. That's the whole game.

## Links

- GitHub repo: https://github.com/DIDA-AI/Dida-RollingGo-Hotel-MCP-Global
- Free API key: https://global.rollinggo.store/
- MCP endpoint: https://mcp.rollinggo.ai/mcp
- 5-min quick start: https://global.rollinggo.store/docs/mcp-docs/quick-start