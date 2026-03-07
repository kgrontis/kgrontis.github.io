---
layout: post
title: "Hello World, Meet Pebbles"
date: 2026-03-05 12:00 +0000
categories: [ai, dotnet]
tags: [ai, dotnet, terminal, spectre-console]
---

AI is everywhere, and I'm overwhelmed by how little I understand and how fast things are moving. So I thought it's time to build something, not to catch up, but to finally get it.

## The Idea

I took inspiration from projects like [pi-mono](https://github.com/badlogic/pi-mono) and [qwen-code](https://github.com/QwenLM/qwen-code/). Since these tools are open source, I can read the code and actually understand how they work. I've always loved terminal apps, so it made sense that I'd fall in love with coding agent CLIs.

I called it **Pebbles**. You can check it out on [GitHub](https://github.com/kgrontis/Pebbles).

## Starting Small

I built it with .NET and Spectre.Console. The first version had:

- A mock AI provider that echoed responses
- Basic command handling
- A colorful terminal UI
- Input handling with history

The mock let me focus on the _experience_ first. What does it feel like to chat with an AI in the terminal?

## The First Conversation

```
❯ You: Hello, Pebbles!

⬡ Pebbles:
Hello! I'm a mock AI provider.
```

Simple first "prototype", but it works. I can type, see responses, and navigate with arrow keys.

## What I Learned

1. **Terminal UI is hard** - Layout, resizing, making it look good.
2. **Input handling is complex** - History, cursor movement, special keys.
3. **Architecture matters** - Clean separation between UI, commands, and AI paid off later.
4. **AI as a tutor** - I'm not writing everything by hand, and that's okay. I feel like the old ways of building software are long gone. I think using AI to help me understand how the tools I use every day actually work is the way to go, and maybe remove some of that magic.

## Why This Matters

AI is moving fast. Building Pebbles is my way of catching up by getting my hands dirty.

Every line of code teaches me something.

## What's Next?

Connect to a real AI. For now, I'll be exploring the [Coding Plan](https://www.alibabacloud.com/help/en/model-studio/coding-plan) offered by Alibaba Cloud. It offers a bunch of open-source models: Qwen3.5-Plus (vision), Kimi-K2.5 (vision), GLM-5, MiniMax-M2.5, Qwen3-Max-2026-01-23, Qwen3-Coder-Next, Qwen3-Coder-Plus, and GLM-4.7. See what happens when responses aren't predictable. Will it be slow? How is thinking working? Hard to integrate? Burn through free credits?

Not knowing, and building anyway. That's the fun part.
