---
layout: post
title: "What Is MCP?"
date: 2026-08-02 08:00:00 +0800
categories: [mcp, ai, beginner]
order: 1
---

You've heard "MCP" a lot lately. Every AI article mentions it. But almost nobody explains it simply. So here's the plain version: what MCP is, why it exists, and how it's different from an API.

There's no code in this post, just the mental model.

## The problem MCP solves

Chatbots are good at talking. They are bad at *doing*.

Ask an AI "what's the cheapest hotel near the airport?" and it will answer confidently. But it doesn't actually know. It's guessing from memory, and hotel prices change every few minutes.

The fix is to let the AI call a real tool. Instead of guessing, it asks a real service: "hey, what's the price right now?" That's the whole idea.

The question is — how do you let an AI call tools? There are two ways. An API, or MCP.

## What an API is (quick refresher)

An API is a set of fixed, named functions a service offers. For example, a hotel API might have:

- `searchHotels` — find hotels
- `getHotelDetail` — get details on one hotel

A programmer writes code to call these functions. The programmer decides *when* to call them and *how to use the results*. The API doesn't make decisions. It just waits for orders.

That's fine for normal software. But AI is different.

## What MCP is

MCP stands for **Model Context Protocol**. It's a standard way for AI to use tools.

The mental picture that works: **MCP is like USB-C for AI tools.** One plug, and any AI can connect to any tool that supports the standard.

With an API, *you* (the programmer) decide when to call a function. With MCP, *the AI* decides. The AI sees a list of available tools, picks the right one, and calls it — all by itself, in the middle of answering your question.

So the difference is really about who's in control:

| | API | MCP |
|---|---|---|
| Who decides when to call? | A human programmer | The AI itself |
| How do you connect? | Write custom code | Configure once, plug in |
| Standardized? | Each service has its own | One shared standard |
| AI can discover tools? | No, hard-coded | Yes, it browses them |

## Why MCP is useful

Three practical benefits:

**1. One way to connect to everything.**
Today, every service has its own API with its own rules. MCP gives them all the same shape. Connect once, reuse everywhere.

**2. The AI does the work.**
You don't have to plan out every call in advance. You say "book me a hotel near the Bund," and the AI figures out which tools to call and in what order.

**3. Real data instead of guessing.**
This is the big one. When an AI uses MCP, it can pull live prices and real availability instead of making things up. A hotel recommendation becomes something you can actually trust.

## A real example: RollingGo Hotel MCP

[RollingGo Hotel MCP](https://github.com/DIDA-AI/Dida-RollingGo-Hotel-MCP-Global) is a good example. It exposes **2M+ hotels** to any AI as three simple tools:

- `searchHotels` — find matching hotels
- `getHotelDetail` — get the real-time price for your exact stay
- `getHotelSearchTags` — understand filter words like "gym" or "free WiFi"

Connect an AI assistant to it via the [RollingGo partner center](https://global.rollinggo.store/), and the AI can search real hotels at real prices. No guessing.

## Honest limitations

MCP is young. The standard is still evolving, and security is an open problem (connecting an AI to tools means the AI can take actions — that deserves caution).

Also, MCP is a *transport* standard. It doesn't make data trustworthy by itself. The tools behind it still need to be real, live, and honest.

## The takeaway

The simplest way to tell API and MCP apart: with an API you write code to order from a fixed menu; with MCP the AI browses the menu and orders for itself. The value is that it stops guessing and starts checking real data.

## Links

- GitHub repo: https://github.com/DIDA-AI/Dida-RollingGo-Hotel-MCP-Global
- Free API key: https://global.rollinggo.store/
- MCP endpoint: https://mcp.rollinggo.ai/mcp
- 5-min quick start: https://global.rollinggo.store/docs/mcp-docs/quick-start
