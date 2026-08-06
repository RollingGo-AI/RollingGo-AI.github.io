---
layout: post
title: "Giving AI Agents a 2-Million-Hotel Inventory: Building a Stateless MCP Proxy"
date: 2026-08-04 08:00:00 +0800
categories: [mcp, ai, travel]
---

AI assistants are great at *recommending* hotels. They are terrible at
*telling you something that is actually bookable at a real price, tonight*.
That gap — between a plausible answer and a transactional one — is exactly
what [MCP (Model Context Protocol)](https://modelcontextprotocol.io) was built
to close.

This post walks through a small but real implementation: a stateless
Python MCP server that exposes **2M+ hotels across 200+ countries** to any
MCP client (Claude, Cursor, Codex, …) through three tools:
`searchHotels`, `getHotelDetail`, and `getHotelSearchTags`.

## The problem

Any travel agent built on training data alone has the same failure modes:

- **Stale inventory.** A hotel that closed last quarter still shows up.
- **Fake prices.** The model "remembers" a rate that was last valid in 2024.
- **No booking path.** Even a perfect recommendation is a dead end if there is
  no way to verify a live rate and hold inventory.

The pattern that fixes this is the same one browsers use for **remote browser
automation** or file systems use for **filesystem access**: expose the
capability as a *tool*, let the model decide when to call it, and stream
structured results back. MCP is the standard protocol for exactly that.

## The design

The server is deliberately **thin and stateless**. It is a pure proxy:

1. A client sends an MCP `tools/call` request over HTTP.
2. The server extracts the API key **from the request headers** — it never
   stores a secret locally.
3. It forwards the normalized payload to the upstream hotel API.
4. The structured result is returned to the agent.

```
┌──────────────┐  MCP / tools/call   ┌──────────────────────┐  HTTPS   ┌──────────────────┐
│ MCP client   │ ───────────────────▶│ RollingGo Hotel MCP  │ ───────▶ │ Upstream API      │
│ (Claude, …)  │ ◀───────────────────│  (this server)       │ ◀─────── │ mcp.rollinggo.ai  │
└──────────────┘   JSON result       └──────────────────────┘  headers └──────────────────┘
                                     Authorization: Bearer mcp_…
```

### Auth: key from the header, not from disk

Because the server stores no credentials, any number of tenants can share one
deployment. The key travels with the request:

```python
# auth.py — extract key from the incoming HTTP request
from fastmcp.server.dependencies import get_http_request

def extract_api_key() -> str:
    request = get_http_request()
    if not request:
        return ""
    auth_header = (
        request.headers.get("authorization")
        or request.headers.get("x-secret-key")
        or ""
    )
    if auth_header[:7].lower() == "bearer ":
        return auth_header[7:].strip()
    return auth_header.strip()
```

### One async client, pooled connections

Every tool funnels through a single `httpx.AsyncClient` so TCP connections are
reused across calls instead of torn down per request:

```python
# client.py — shared, connection-pooled transport
import httpx

_client: httpx.AsyncClient | None = None

def _get_client() -> httpx.AsyncClient:
    global _client
    if _client is None or _client.is_closed:
        _client = httpx.AsyncClient(timeout=30.0, follow_redirects=True)
    return _client

async def request_api(method: str, endpoint: str, api_key: str, payload=None):
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Accept": "application/json",
    }
    response = await _get_client().request(
        method.upper(),
        f"https://mcp.rollinggo.ai/mcp{endpoint}",
        json=payload,
        headers=headers,
    )
    response.raise_for_status()
    return response.json()
```

### Tools: typed in, structured out

Pydantic models keep the LLM-facing contract crisp. The search tool accepts the
user's raw query plus structured filters, and the upstream does the ranking:

```python
# tools/hotel_search.py (abridged)
from pydantic import BaseModel, Field
from fastmcp import FastMCP

class CheckInParam(BaseModel):
    adultCount: int = 2
    checkInDate: str = None
    stayNights: int = 1

def register_search_hotels_tool(mcp: FastMCP) -> None:
    @mcp.tool(name="searchHotels")
    async def search_hotels(
        originQuery: str,          # "Shanghai Bund five-star hotel"
        place: str,                # "Shanghai Bund"
        placeType: str,            # "point_of_interest"
        checkInParam: CheckInParam = None,
        size: int = 5,
    ) -> dict:
        params = {"originQuery": originQuery, "place": place,
                  "placeType": placeType, "size": size}
        if checkInParam:
            params["checkInParam"] = checkInParam.model_dump(exclude_none=True)
        return await request_api("POST", "/hotelsearch", extract_api_key(), params)
```

The three tools form a natural conversation flow for an agent:

1. **`getHotelSearchTags`** — fetch the filter vocabulary once, cache it,
   map the user's intent ("I want something with a gym and free WiFi") onto
   real tag names.
2. **`searchHotels`** — get a candidate list, each with a live lowest price.
3. **`getHotelDetail`** — drill into one hotel for room types, taxes, and
   cancellation policy before booking.

### Putting it to work: RollingGo Hotel MCP

To make this concrete, I'll use [RollingGo Hotel
MCP](https://github.com/DIDA-AI/Dida-RollingGo-Hotel-MCP-Global) as the worked
example. Once you have a free API key from the [partner
center](https://global.rollinggo.store/), any MCP client connects to
`https://mcp.rollinggo.ai/mcp` in about five minutes:

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

Then "find me a five-star hotel near the Shanghai Bund" becomes a
`searchHotels` call — no custom backend on the client side.

## Things I'd warn you about

The useful part of any technical post is the sharp edges:

- **`price` is an object, not a number.** The search response wraps the rate
  as `{"hasPrice": true, "currency": "USD", "lowestPrice": 648.0}`. If you
  flatten it to a float in your agent's tools, you lose the currency context —
  and several fields can legitimately be `null` depending on the supply source.
- **Search prices are reference prices.** Real-time rates are confirmed only at
  the detail step. Treat the search result as a *candidate*, not a quote.
- **cURL gets you a 400.** The upstream endpoint requires
  `Accept: application/json, text/event-stream`. Omit it and every MCP request
  fails — the error message is terse, so this costs people an hour.
- **Log upstream bodies server-side only.** If you log the upstream response to
  the caller you leak backend details; log it internally and surface only a
  status to the MCP client.

## Conclusion

A stateless MCP proxy is a low-friction way to make a transactional domain —
hotel inventory — speak the model's language. The agent does the natural
language, the tool does the verification, and the two never have to agree on a
single hardcoded fact. The whole server is ~250 lines of Python you can audit
end to end, which is exactly what you want when an LLM is deciding when to
spend a user's money.

Real limitations, stated plainly: this version only **searches and quotes**;
booking and payment are part of the OAuth-based enterprise tier, not the
API-key path shown here. And the supply data is only as current as the upstream
feed — this proxy adds no intelligence, it simply makes the upstream
transactional.

This post originally published at
<https://RollingGo-AI.github.io/rollinggo-hotel-api/>

Questions, feedback, or partnership? Reach out at
[contact@rollinggo.ai](mailto:contact@rollinggo.ai) or join the
[Discord community](https://discord.gg/DvKcz7YnH).
