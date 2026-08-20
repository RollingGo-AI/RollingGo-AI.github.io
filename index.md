---
layout: home
title: RollingGo — MCP & Travel-Tech Notes
---

# RollingGo

Hi, I build AI tools for travel and write about what I learn along the way —
MCP servers, AI agents, and why booking a hotel is harder than it looks.

## Latest posts

{% for post in site.posts %}
- **{{ post.date | date: "%Y-%m-%d" }}** — [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

## Contact

- Email: [contact@rollinggo.ai](mailto:contact@rollinggo.ai)
- Discord: [discord.gg/DvKcz7YnH](https://discord.gg/DvKcz7YnH)
