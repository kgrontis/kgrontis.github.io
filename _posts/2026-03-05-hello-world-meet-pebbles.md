---
layout: post
title: "Hello World, Meet Pebbles"
date: 2026-03-05 12:00 +0000
categories: [ai, dotnet]
tags: [ai, dotnet, terminal, spectre-console]
---

So here's the thing. I've been watching this AI revolution happen all around me, and I feel like I'm standing on the shore while everyone else is already swimming. ChatGPT, Claude, Copilot... they're all amazing, but I wanted to _understand_ them. Not just use them, but really get how they work and what it means to build with them.

I learn by doing. Always have. So I decided to build something.

## The Idea

I wanted a coding assistant that lives in my terminal. Something fast, lightweight, and built exactly how I want it. No browser tabs, no waiting for web pages to load. Just me, my terminal, and an AI that helps me code.

I called it **Pebbles**. Because it's small, and because I'm throwing pebbles into the ocean of AI to see what ripples come back.

## Starting Small

The first version was... well, it was something. I built it with .NET and Spectre.Console (which is fantastic, by the way). It had:

- A mock AI provider that just echoed back responses
- Basic command handling
- A terminal UI with colors and panels
- Input handling with history

The mock provider was key. I didn't want to get bogged down in API keys and rate limits right away. I wanted to build the _experience_ first. What does it feel like to chat with an AI in the terminal? What commands do I need? How should the UI flow?

## The First Conversation

Here's what my first "conversation" looked like:

```
❯ You: Hello, Pebbles!

⬡ Pebbles:
Hello! I'm a mock AI provider. In the real implementation,
this would be a response from an AI model.
```

Not exactly groundbreaking, right? But it worked. I could type, I could see responses, I could navigate with arrow keys. The foundation was there.

## What I Learned

Even this simple start taught me a lot:

1. **Terminal UI is hard** - Getting the layout right, handling resizing, making it look good... there's a reason most dev tools are plain text.

2. **Input handling is surprisingly complex** - History, cursor movement, special keys... there's a lot going on behind the scenes when you just type a message.

3. **Architecture matters** - I started with clean separation between the UI, command handling, and AI provider. That paid off later when I swapped out the mock for real AI.

## Why This Matters

I think a lot of developers feel overwhelmed by AI right now. It's moving so fast, and it's easy to feel like you're falling behind. Building Pebbles is my way of catching up. Not by reading articles or watching videos, but by getting my hands dirty.

Every line of code teaches me something. Every bug I fix is a lesson. Every feature I add is a step toward understanding this new world.

## What's Next?

The mock provider has to go. I need to connect to a real AI and see what happens when the responses aren't predictable. I'm excited and a little nervous. What if it's too slow? What if the API is hard to work with? What if I burn through my free credits in a day?

But that's the fun part, isn't it? Not knowing, and building anyway.

---

> **Note:** This article was written with AI assistance.
