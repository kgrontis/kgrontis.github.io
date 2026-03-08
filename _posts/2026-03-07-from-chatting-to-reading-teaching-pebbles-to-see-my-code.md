---
layout: post
title: "From Chatting to Reading: Teaching Pebbles to See My Code"
date: 2026-03-07 12:00 +0000
categories: [ai, dotnet]
tags: [ai, dotnet, terminal, pebbles]
---

*March 5-7, 2026*

---

I connected Pebbles to a real AI. It felt nice.

## Choosing the Alibaba Coding Plan

I went with Coding Plan. The API docs could use some work, but I managed to get it working. I wanted to support multiple models from day one. Why limit myself?

Setting up the API key was the first hurdle. Environment variables, config files, trying not to commit secrets to git. The usual dance. But once that was sorted, I was ready.

I have eight models to choose from: qwen3.5-plus, qwen3-coder-plus, qwen3-coder-next, qwen3-max-2026-01-23, glm-4.7, glm-5, MiniMax-M2.5, and kimi-k2.5.

## The Moment of Truth

I typed my first real message:

```
❯ You: Write a hello world in C#
```

And then nothing. For like 3 seconds. Probably because the models are hosted in Singapore and I am in Europe.

But then:

```
⬡ Pebbles:
Here's a simple "Hello, World!" program in C#:

┌─ csharp ──────────────────────────────────
│ using System;
│
│ class Program
│ {
│     static void Main()
│     {
│         Console.WriteLine("Hello, World!");
│     }
│ }
└──────────────────────────────────────────
```

It worked. It actually worked.

## Streaming is Magic

I implemented streaming responses. The text appears character by character instead of all at once. It's a small thing, but it makes the experience feel alive. Like there's something on the other end thinking and typing.

Watching the code block slowly appear on my screen, that was the moment I understood why people get excited about AI. It's not just the output. It's the process of getting there.

## The Cost Reality Check

I added token tracking because GLM-5 told me to implement it. After a few conversations:

```
⎯ 156 input → 423 output • $0.0012 • 579 total tokens
```

I don't know if those numbers are accurate. I'll revisit this in the future since it's not very important to me at the moment.

## Configuration Adventures

I spent quite a while trying to understand how others are doing it and what feels right to me. At the moment I've set up the API key as an env var and added the models in the app settings.

```json
{
  "Pebbles": {
    "DefaultModel": "qwen3.5-plus",
    "AvailableModels": [
      "qwen3.5-plus",
      "qwen3-coder-plus",
      "qwen3-coder-next",
      "qwen3-max-2026-01-23",
      "glm-4.7",
      "glm-5",
      "MiniMax-M2.5",
      "kimi-k2.5"
    ]
  }
}
```

Eight models!

## But Wait, It Was Flying Blind

So I had this AI assistant in my terminal. It could chat, it could answer questions, but it couldn't actually *see* my code. I'd ask it about my code, and it would give me generic answers because it had no context.

That had to change.

## The @ Symbol Idea

I wanted a simple way to reference files. Everyone is using the `@` symbol so why reinvent the wheel. It's easy to type, it feels like "at" or "address", and it's not used much in regular chat.

So now I could type:

```
❯ You: What does the Process method do in @Services/ChatService.cs?
```

And Pebbles would load that file and include it in the context.

## Building the File Picker

But typing paths is annoying. So I built an interactive file picker. When you type `@`, you get:

```
❯ You: Check this file: @
                        Configuration/
                        Models/
                        Services/
                        UI/
                        Program.cs
                        Pebbles.csproj
                        README.md
```

You can navigate with arrow keys, press Tab to select, and even drill into directories. It's like having a tiny file explorer inside your chat.

## The Technical Bits

Building this required:

- A `FileService` to handle loading and parsing
- Autocomplete logic that triggers on `@`
- Path resolution (relative, absolute, nested directories)
- Binary file detection (because you don't want to accidentally feed a PNG to the AI)
- Size limits (1MB per file - learned that lesson the hard way)
- Filtering out hidden files and directories
- Sorted results (directories first, then files, both alphabetically)
- A scrollable popup with up to 10 items
- Navigation with arrow keys, Enter or Tab to select, Escape to dismiss
- Tracking the current directory path for incremental navigation
- Filtering by typing partial names after `@`
- Lazy loading file content when the message is submitted
- Caching of loaded files in the `FileService`

Of course, the file picker is still super buggy, but we have something 
`¯\_(ツ)_/¯`!!! Need to stay motivated and fix it in the future!

## The First Real Code Review

Once it was working, I tried my first real code review:

```
❯ You: Review this code for potential issues:
      @Program.cs
      @Services/ChatService.cs

✓ Program.cs (2.5 KB)
✓ Services/ChatService.cs (4.1 KB)
Loaded 2 file(s) into context

⬡ Pebbles: I've reviewed the files. Here are my suggestions...
```

And it found a potential null reference issue I'd missed. It's just pattern matching, but it found something useful.

## Context Management

I also added a `/context` command to see what's loaded, and `/clearfiles` to start fresh. Managing context is important because:

1. Token limits are real
2. Too much context confuses the AI
3. You want to be intentional about what you're sharing

## The Project Context File

It wasn't really my idea. All the cool kids are doing it. So I created a global `.pebbles` folder that needs to exist either at project level or globally in the user folder. Inside it, I added support for `.pebbles/agent/AGENTS.md`:

```markdown
# Project Guidelines

## Code Style
- Follow existing project conventions
- Use meaningful variable names
- Keep functions under 50 lines
- Add comments for complex logic only
```
> this one is taken from `pi-mono` I believe

Now every conversation starts with this context. It's like giving Pebbles a briefing before we start working.

## What Surprised Me

The speed. I expected it to feel sluggish compared to the mock provider, but with streaming, it actually feels faster. Your brain processes the text as it arrives, so by the time the response is done, you've already read most of it.

Also, the variety in responses between models is fascinating. Same prompt, completely different approaches. Some are concise, some are verbose, some are more "creative" than others.


## What I Learned

Context is everything. The same AI model gives wildly different answers depending on what you feed it. A vague question gets a vague answer. A specific question with relevant code gets a specific, useful answer.

Also, file handling is never as simple as it seems. Path separators, encoding issues, permission errors etc. There's always something.

## The Joy of It

There's something deeply satisfying about typing `@` and watching that file picker appear. It's like my terminal just got a little bit smarter. A little bit more *alive*.

And when the AI references something from my code in its answer? Chef's kiss. It actually *understands* (or appears to understand) what I'm working on.

## The Bigger Picture

I think a lot of developers feel overwhelmed by AI right now. It's moving so fast, and it's easy to feel like you're falling behind. Building Pebbles is my way of catching up. Not by reading articles or watching videos, but by getting my hands dirty.

Every line of code teaches me something. Every bug I fix is a lesson. Every feature I add is a step toward understanding this new world.