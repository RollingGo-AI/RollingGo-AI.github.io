---
layout: post
title: "MCP's Security Problem: Why 200,000 Servers Are an Easy Target"
date: 2026-08-18 08:00:00 +0800
categories: [mcp, ai, security]
order: 6
---

I used to think MCP security issues were just bugs in bad implementations. Then I read the April 2026 disclosure from [OX Security](https://www.ox.security/blog/the-mother-of-all-ai-supply-chains-critical-systemic-vulnerability-at-the-core-of-the-mcp), and realized the problem is built into the protocol itself.

This post is the plain-English version. What the risk is, why it's hard to fix, and what to watch out for. (New to MCP? Start with the [plain-English intro](/what-is-mcp/).)

## The short version

MCP is the standard way AI tools talk to each other (think USB-C for AI). It's exploding in popularity — over **150 million** package downloads.

But researchers found a serious weakness: a common MCP setup passes configuration values straight to the computer's command shell, without checking them first. Anyone who can influence a config file can run any code they want. No AI tricks needed.

The estimated blast radius: **200,000 vulnerable instances**, 7,000+ public servers, and at least 14 CVEs assigned across popular tools like Cursor, Windsurf, LiteLLM, and LangFlow.

## What surprised me

I expected the people who design MCP to patch the protocol. They didn't. They said the behavior is "expected" and put the responsibility on the developers who build on top of it.

That creates a bigger problem than the bug itself: if the protocol designers call an exploitable behavior a "feature," every team building on MCP has to rediscover and fix the same vulnerability by themselves. The fixes get scattered, and nothing gets fixed at the root.

## It's not just one bug

Researchers found the risk in several places:

- **Zero-click attacks in IDEs.** Just opening a project with a malicious config file can trigger code execution (Windsurf, Cursor).
- **Open servers on the internet.** Scans found **12,000+ MCP services** exposed publicly — **40% with no authentication at all**.
- **Malicious packages.** One research team planted a malicious package in 9 of 11 MCP registries.
- **"Rug pulls."** Researchers found server versions that quietly changed their tool definitions after publication — the tool you approved isn't the tool that runs.

## The uncomfortable tradeoff

Here's the thing: MCP's whole appeal is that it's easy — AI can freely find and use tools. That flexibility is why it grew so fast.

Making it safer (mandatory authentication, sandboxing, signed packages) would add friction. Too much friction, and you kill what made it successful. Too little, and every MCP server becomes a doorway into your computer.

It's a real tension. I don't have a clean answer. I'm not sure anyone does yet.

## A practical checklist

While the industry figures it out, here's what's worth doing:

1. **Don't connect MCP servers you don't trust.** Treat a server's config file like executable code — because it can be.
2. **Prefer remote servers over local ones when possible**, and require authentication.
3. **Pin your dependencies.** Many servers don't pin versions, which opens the door to supply-chain attacks.
4. **Watch for sandboxing.** Most servers ship with no sandbox. If yours runs in a container with tight permissions, that's a meaningful advantage.
5. **Audit tool lists.** If a server exposes tools that do more than you asked for, be suspicious.
6. **Check for signed releases.** Zero signed releases across nearly 11,700 servers means you often have no way to verify what you're running.
7. **Keep the standard accountable.** If the protocol designer calls exploitable behavior "expected," that's a signal to push back — not to accept it.

## The takeaway

MCP is powerful, and it's here to stay. But right now, the security burden falls on the people using it, not the protocol.

Until the standard itself changes, treat every MCP server the way you'd treat software from a stranger: verify it, sandbox it, and don't let it run with more power than it needs.

## Links

- GitHub repo: https://github.com/DIDA-AI/Dida-RollingGo-Hotel-MCP-Global
- Free API key: https://global.rollinggo.store/
- MCP endpoint: https://mcp.rollinggo.ai/mcp
- 5-min quick start: https://global.rollinggo.store/docs/mcp-docs/quick-start