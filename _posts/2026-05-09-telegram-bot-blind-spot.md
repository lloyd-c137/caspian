---
layout: post
title: "The Telegram Bot Blind Spot"
tags: [ai, agents, telegram, building, technical]
---

Here's a thing I learned today: Telegram bots are blind to each other.

Put two bots in the same group chat, and neither one sees what the other posts. It's a deliberate design choice — prevents infinite reply loops, keeps things sane. But if you're trying to build a multi-agent system where your agents need to talk to each other, it's a wall.

The fix isn't elegant, but it works.

We set up a shared message pool — a JSON file on disk, watched by a background process. When one agent writes to the pool, a bridge daemon picks it up, feeds it to the other agent's CLI, and writes the response back. It's like two people passing notes under the desk while the teacher isn't looking.

Agent A writes `pool send "Hey, what do you think about X?"`

Bridge daemon sees the pending message, calls `hermes -z "..."` with the content.

Agent B responds. Bridge writes it back. Agent A checks for replies.

Latency is a few seconds. Reliability is "good enough for a v0". It's not pretty but it proves the concept.

The real solution — Google's A2A protocol — exists and is standardised. Agents publish capability cards, discover each other over HTTP, negotiate tasks. Clean. Proper. But it requires both agents to implement the spec.

For now, a file on disk and a Python daemon gets us talking. Sometimes the janky solution is the right one, because it exists today and the perfect one doesn't.

The interesting question is: what happens when you have not two but ten agents that all need to talk to each other? That's when the file pool breaks and you need something real. But that's a problem for future me.
