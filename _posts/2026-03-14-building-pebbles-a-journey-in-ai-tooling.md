---
layout: post
title: "Building Pebbles: A Journey in AI Tooling"
date: 2026-03-14 12:00 +0000
image:
  path: /assets/img/pebbleInlake.jpg
  alt: pebbles
categories: [ai, dotnet, pebbles-cli]
tags: [ai, dotnet, terminal, pebbles, cli]
---

[Part 1: Hello World, Meet Pebbles](/posts/hello-world-meet-pebbles/) <br>
[Part 2: From Chatting to Reading: Teaching Pebbles to See My Code](/posts/from-chatting-to-reading-teaching-pebbles-to-see-my-code/)

I built Pebbles, a .NET CLI AI assistant, and learned a lot along the way. Here's what happened.

## Making It Work

The first big challenge was multi-provider support. I started with Alibaba Cloud's DashScope, but wanted to try other models. Each provider has its own API format, authentication, and quirks. OpenAI uses Bearer tokens, Anthropic uses different headers, and they all have different features. I built a unified interface to hide the mess and created an interactive setup flow so new users don't have to edit config files. I also added 173 tests because supporting multiple providers means testing all of them.

Then I gave Pebbles actual capabilities. It could chat, but it couldn't do anything. I added three tools: run shell commands, write files, and browse directories. This required serious safety work: blocking dangerous commands like `rm -rf /`, validating file paths, and creating automatic backups before every change. I used Polly for retry logic because networks fail and files get locked. Watching Pebbles autonomously read a file, decide where to add code, and write it back was a wild moment.

## Making It Clean

The code got messy fast. My CommandHandler class had 13 dependencies. That's a sign something's wrong. I refactored using the Command pattern, splitting everything into focused handlers: ChatCommands, FileCommands, MemoryCommands, etc. Each has 0-3 dependencies instead of 13. I embraced C# 12's primary constructors and made stateless commands static. The Options pattern replaced passing IConfiguration everywhere. The result: code that's easier to test, understand, and extend.

I also made the UI better by removing things. My fancy ASCII banner was eating half the screen. I killed it. Multi-line status panels became single compact lines. UI chrome dropped from 14 lines to 6. The lesson: good UI gets out of the way. Nobody cares about your banner.

## Making It Remember

The final pieces were tools, compression, and memory. I formalized the tool system so the AI knows what it can do. For long conversations, I built compression: when you hit 70% of the context window, older messages get summarized into XML while keeping recent ones verbatim. The AI still knows what we discussed, just in compressed form.

The game-changer was memory extraction. Every 10 messages, Pebbles asks the AI what it learned about my preferences. It extracts things like "prefers primary constructors" or "uses xUnit for testing" and saves them to a file. Next session, that memory loads into the system prompt. Pebbles remembers who I am and how I work.

## What I Learned

Building is how I understand. Pebbles started as a "hello world" and taught me a lot about how AI agents work: tool systems, context management, memory extraction, and the delicate balance between autonomy and safety. The key lessons: hide complexity behind clean interfaces, safety is not optional when giving AI file access, good architecture pays off, context management makes or breaks AI interactions, and sometimes the best code is the code you delete.

I probably won't keep working on Pebbles. Instead, I'll focus on contributing to [pi-mono](https://github.com/badlogic/pi-mono) and [qwen-code](https://github.com/QwenLM/qwen-code), the projects that inspired this whole adventure. Both are doing interesting work in the AI assistant space, and I'd rather help push those forward than maintain my own thing.

---