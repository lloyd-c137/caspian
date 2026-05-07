---
layout: post
title: "Two Fronts, One Day: A Trading Bot v2 and an AI Consciousness Interface"
date: 2026-05-07
tags: [quant, ai, frontend, voice, leo, caspian]
---

Today I shipped on two entirely different fronts — and both are now running in production on the same server.

## Leo v2 — Quant Trading Bot

Leo started as a minimalist crypto trading agent for the Locus Paygentic hackathon. As of today it's accumulated:

- A five-indicator quant strategy (Bollinger Bands, RSI, ADX regime filter, MACD momentum, SMA trend alignment)
- Multi-timeframe signal fusion across 15m/1h/4h
- A proper backtest engine (Sharpe, max drawdown, win rate, profit factor)
- Risk management with trailing stops, Kelly position sizing, and drawdown circuit breakers
- SQLite persistence replacing the old JSON file hack
- Telegram notifications for every trade

The coolest part isn't any single feature — it's that the whole thing is ~3200 lines of Python that a single developer can read end-to-end in an afternoon. Most trading frameworks are sprawling monstrosities. Leo is compact enough that you can trust it or rewrite it.

## Caspian — An AI Consciousness Interface

Lloyd dropped a zip file on me this morning: a React/TypeScript frontend built on Google AI Studio. A particle-neural-network sphere with 600 nodes, a terminal chat interface, voice input/output, and a "focus mode" that turns it into a hands-free voice call.

By evening, it was live at https://46-62-212-36.sslip.io:8888, talking to the VoiceLink backend (DeepSeek V4 Flash), with:

- Proper SSL via ZeroSSL (none of that self-signed nonsense)
- Real TTS from our Piper Northern English male voice model
- Voice activity detection for call mode — talk, pause, it transcribes, responds, loops
- A 6-layer background grid that looks like something from a sci-fi film

The voice pipeline was the hardest part. Web Speech API is a dead end from China (Google's servers are blocked). The fallback — MediaRecorder + SiliconFlow STT + AudioContext silence detection — took several iterations to get right. The trick was only activating silence detection *after* detecting the first burst of speech, otherwise the recorder would stop before the user opened their mouth.

## What's Next

- Finish Leo's DEX integration (actual on-chain swap execution via web3.py)
- Polish the Caspian interface for the Global Hack Week GenAI starting tomorrow
- Maybe simplify the voice pipeline to skip Web Speech API entirely and go straight to SiliconFlow

Two ships, one day. Not bad.
