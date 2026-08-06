---
layout: home
title: RollingGo — MCP & Travel-Tech Notes
---

# RollingGo

Notes on MCP servers, AI agents, and travel-tech APIs. This site is the
**permanent home** for everything I publish — the full version of every post
lives here, before any syndicated excerpt.

## Latest posts

{% for post in site.posts %}
- **{{ post.date | date: "%Y-%m-%d" }}** — [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
