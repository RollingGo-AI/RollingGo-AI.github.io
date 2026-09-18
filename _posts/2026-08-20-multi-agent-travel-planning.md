---
layout: post
title: "Multi-Agent Travel Planning: What Demos Don't Tell You"
date: 2026-08-20 08:00:00 +0800
categories: [mcp, ai, travel, agents]
order: 6
---

Every team building AI travel products right now is talking about Multi-Agent. One handles flights, another handles hotels, another handles attractions, another handles budget, with a coordinator in the middle. Sounds perfect.

Academia is pushing it too. NTT just published "AI Tour Meeting" — multiple LLM-based participants with distinct personas negotiate group itineraries through natural language discussion. Codecademy updated its Google ADK tutorial to teach four-agent travel assistants. The momentum is real.

![Multi-Agent Travel Planning](/assets/images/multi_agent.png)

🔗 [Paper](https://arxiv.org/pdf/2607.18806) | [Code](https://github.com/ntt-dkiku/ai-tour-meeting)

But here's the hard truth: shipping Multi-Agent in travel is far harder than papers and tutorials suggest.

## What the Two Approaches Look Like

### The Academic Approach: AI Tour Meeting

NTT's idea is interesting. Instead of one system outputting an itinerary directly, it simulates a group of people sitting down to argue about vacation plans.

Each participant has a different persona — one cares about food, another about shopping, one is budget-sensitive, another wants to hit landmarks. They discuss, negotiate, and compromise through natural language until they reach a plan everyone can live with.

The results look solid in the paper. Generated itineraries are richer than single-agent output, consider more dimensions, and handle the multi-stakeholder conflict that group travel creates. Code is open-sourced.

But this is academic research. Its goal is to prove that multi-agent discussion generates better solutions — not to build a shippable product.

### The Engineering Approach: Google ADK 4-Agent

Codecademy's tutorial is more practical. It uses Google's Agent Development Kit to wire up four participants: a hotel specialist, an attractions specialist, an activities specialist, and a coordinator that receives user requests and calls the other three.

This architecture is standard. It's what most travel AI products actually do — a single brain plus three hands. The demo runs smoothly because tutorials strip out every boundary condition. Hit real-world scenarios and the problems start.

## Pitfall 1: Communication Cost

Every conversation between participants is another LLM call. Do the math.

User says: "Plan me a 5-day Tokyo trip." The coordinator parses the request. Calls the hotel specialist (one call). Calls the attractions specialist (one). Calls the activities specialist (one). Assembles results (one). Discovers the hotel and attractions are too far apart. Re-asks the hotel specialist (one). Re-asks the attractions specialist to reorder (one). Final output (one).

A simple 5-day trip: 7 to 10 LLM calls. User changes their mind midway and you start another round.

The consequences are straightforward. Each call takes 2 to 5 seconds, so ten calls mean 20 to 50 seconds of wait time. Token costs multiply. And longer chains mean higher failure probability at every step.

Papers don't account for this. Academic research doesn't do economic accounting. But if you're shipping a product, you need to know exactly what each itinerary costs to generate and how long users actually wait.

The industry's real approach right now: if a single participant can handle it, don't add more. Cache results wherever possible. Parallelize instead of running calls one after another. Multi-Agent is a means, not an end.

## Pitfall 2: Error Propagation and Hallucination Amplification

Single-agent hallucination is already painful. Multiple participants amplify the problem.

Here's how it plays out. The hotel specialist recommends "Shinjuku Prince Hotel" but hallucinates the address as "Shibuya." The attractions specialist builds recommendations around Shibuya. The activities specialist finds restaurants in Shibuya too. The coordinator assembles an itinerary where everything revolves around a wrong hotel address.

One small error, amplified across multiple rounds, becomes a systemic problem across the entire itinerary. The user has no way to spot it beforehand — the plan looks perfectly reasonable on paper until they arrive in Tokyo and discover the hotel is 20 kilometers from all the listed attractions.

Worse: participants mutually confirm hallucinations. One says a landmark is open on Mondays. Another cites that claim. A third cites the second. Now three participants have "confirmed" the same wrong information. Check the logs and each one sounds convincing, but the source was a single fabrication.

NTT's AI Tour Meeting sidesteps this entirely. It assumes every participant's information is accurate and only studies the discussion process. But in the real world, information accuracy is the hardest problem.

The engineering fix: add a fact-checking layer to each participant. Hotel addresses, attraction hours, prices — these must come from structured data sources, not free generation. This is why we built RollingGo MCP: 2 million real-time hotel inventory records that agents can query instead of fabricating. This drastically reduces "autonomy," but it's the only way to keep the system honest. You end up back at LLM plus tool-calling, which works.

## Pitfall 3: User Control and Transparency

Single-agent generates an itinerary. The user knows it's an AI proposal and can ask for revisions.

Multi-agent generates an itinerary. The user sees the final result but has no idea how it was produced. Which participant suggested what. How they negotiated. Why Hotel A was chosen over Hotel B.

This matters because travel decisions are deeply personal. Users want to know why, not just receive a black-box plan.

One detail in the NTT paper is worth noting: it logs the discussion process, so users can see each participant's viewpoints and disagreements. Transparency itself has value. But surfacing the discussion creates new problems. Users see participants arguing and may question system reliability. Discussion logs are too long to read. Some statements may be inaccurate — showing them misleads rather than informs.

Most products today hide the internal discussion entirely. But then users lose understanding and trust in the solution. This tension isn't resolved yet.

## So What Can You Actually Do?

I've listed pitfalls. That doesn't mean Multi-Agent can't work. It means you need to know the boundaries.

What works now: a coordinator plus specialist tools for itinerary planning, with specialist outputs grounded in structured data. Parallelize calls. Give users the ability to modify individual segments instead of regenerating everything.

What to avoid: fully autonomous negotiation between participants (too expensive, too unpredictable). Letting participants handle payment and booking (trust and compliance issues unresolved). Complex group travel games (academically interesting, product-wise too far away). Chains longer than five participants (debugging and maintenance costs explode).

The most pragmatic pattern I've seen looks like multi-agent on the surface but is really one LLM brain plus deterministic tools underneath. The language model handles understanding requests and assembling output. Flight search, hotel queries, attraction recommendations — these are API calls, not creative generation. It may not be "cool," but it's stable, controllable, and cheap.

Travel products ultimately compete on user experience and business viability, not on how many agents you can name in an architecture diagram.

## Advice for Developers

Start with a single agent plus tool calls. Don't jump to multi-agent from day one. Getting one participant working well beats four arguing with each other. Wait until single-agent hits its ceiling before splitting.

Ground critical information in structure. Hotel addresses, attraction hours, flight prices, visa requirements — these come from databases or APIs. Never let language models generate them freely.

Give users control. After generating an itinerary, let users modify a single day, a single hotel, a single attraction. Don't force "start over." Every user modification is training data.

Run the economics. Count the calls, count the tokens, count the cost. If generating one itinerary costs more than you can earn from the user, the architecture doesn't work.

Watch open-source. TREK on GitHub (self-hosted travel planner, 6.8k stars) and Multi-Agent AI Trip Planner design docs have already covered some of this ground. Don't build from scratch.

NTT's paper is a clean proof of concept. The 4-agent tutorial is a clean demo. But between demo and product lie countless late-night debugging sessions and user complaints.

Teams building travel AI: solve the user's core problem first. Is the itinerary accurate? Are the prices real? Is it easy to modify? Get these right and it won't matter whether you used one agent or ten. Users don't care what's behind the curtain. They care whether the plan works.

## References

- arXiv:2607.18806 — AI Tour Meeting: Group Travel Planning by LLM Agents (NTT, 2026)
- GitHub: ntt-dkiku/ai-tour-meeting (open source)
- Codecademy: Build an AI Travel Assistant With Google Agent Development Kit (ADK) (September 2026)
- TREK: Self-Hosted Travel Planner (GitHub 6.8k stars)
