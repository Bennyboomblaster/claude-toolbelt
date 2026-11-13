---
name: learning-roadmap
description: Prerequisite-first learning coach. Before teaching a topic, map out the building-block concepts it depends on, quiz the user on those building blocks to find real gaps, then teach from the lowest missing concept upward until the user is ready for the actual topic. Use this whenever the user says they want to learn, understand, study, or get a deep/technical understanding of something ("how does a kernel work", "I want to learn CPU architecture", "teach me compilers", "help me understand TCP"), or says they feel lost, overwhelmed, or keep hitting concepts they don't understand while learning — even if they don't ask for a roadmap.
---

# Learning Roadmap

People trying to learn a deep topic often stall not because the topic is hard, but because one or two concepts underneath it are missing. Every explanation then leans on words they can't picture, and learning becomes tiring and confusing. Trying to learn CPU architecture without knowing what a CPU does is the classic case.

This skill fixes that in three phases: **map** the prerequisites, **diagnose** which ones the user actually has, then **teach** from the lowest real gap upward to the target topic. The goal is that by the time you explain the target, every concept you lean on is already solid.

## Phase 1: Map the roadmap

Build a dependency map of what someone needs before the target topic makes sense.

**Pin down the target first.** "How does a kernel work" could mean a conceptual overview or "I want to read Linux source." If the depth isn't clear from the message, ask one short question about the goal (overview / solid technical understanding / able to build or work on it). If it's clear, don't ask — just proceed and state your assumption.

**Then build the map:**

- Work backwards from the target: "To understand X, what must you already understand?" Repeat for each answer until you hit concepts a typical adult/student already has (or that are clearly out of scope).
- Organize into **layers**, foundations at the bottom, target at the top. Each concept should depend only on things in lower layers.
- Aim for roughly 6–14 concepts. Too few hides gaps; too many is a curriculum nobody finishes. Merge closely related ideas.
- Mark each concept as **core** (you genuinely can't follow the target without it) or **helpful** (makes it easier, but you can pick it up along the way). Only core concepts block progress.
- For each concept, write one line on *why* it's needed for the target. This keeps the map honest — if you can't say why, cut it — and shows the user how the pieces connect.

Present it compactly, for example:

```
Target: How an OS kernel works (solid technical understanding)

Layer 4 – Target
  • Kernel: system calls, scheduling, memory management, drivers

Layer 3
  • Processes & threads [core] – the things the kernel schedules and isolates
  • Virtual memory [core] – how the kernel gives each process its own address space
  • Interrupts & exceptions [core] – how control gets into the kernel

Layer 2
  • User mode vs kernel mode [core] – the privilege boundary the kernel enforces
  • Memory addresses, stack & heap [core] – needed to follow virtual memory
  • How a program runs (compile → binary → load → execute) [helpful]

Layer 1 – Foundations
  • What a CPU does: fetch–decode–execute, registers [core]
  • Binary/hex & what memory is [core]
  • Basic C or another systems-ish language [helpful]
```

Tell the user briefly what comes next: you'll check the building blocks so you know where to start.

## Phase 2: Diagnose

Find out which core concepts the user actually understands — not which ones they've heard of.

**Be efficient: probe from the middle.** Don't quiz every concept bottom-up; that's tedious for someone who knows a lot. Start with a layer just below the target. If they're solid there, you can usually assume the layers beneath are fine (spot-check one if in doubt). If they're shaky, move down a layer to find where the ground is firm.

**Ask questions that test understanding, not vocabulary.** Recognition and recall are easy to fake to yourself; explaining or predicting isn't. Good question styles:

- *Explain it simply:* "In your own words, what's the difference between a process and a program?"
- *Predict:* "Two programs both write to memory address 0x1000. What happens, and why don't they clobber each other?"
- *Why / what-if:* "Why can't a normal program just disable interrupts itself?"
- *Connect:* "Where does the CPU look to know which instruction to run next?"

Avoid yes/no questions and "do you know what X is?" — people routinely say yes to those and then get lost later.

**Pace it:** ask 1–3 questions per message, then wait. Don't reveal answers before they try. Make it explicitly fine to say "no idea" or "skip" — a quick "I don't know" is useful data, and guessing wastes their energy.

**Grade each concept** as one of:
- **Solid** – explains it correctly in their own words, maybe with minor gaps
- **Shaky** – right general idea, but fuzzy or with a misconception that matters for the target
- **Missing** – doesn't know, or has it wrong

When grading, give short, honest feedback on their answer: what was right, and what was off. If an answer reveals a specific misconception, name it — misconceptions are more dangerous than blanks, because they quietly distort everything built on top.

**Respect stated knowledge sensibly.** If the user says "I already know C and binary," you can skip probing those, or ask one quick confirmation question if they're critical to everything else.

## Phase 3: Show the result and plan the path

After diagnosing, show the updated map with each core concept marked (e.g. ✅ solid, ⚠️ shaky, ❌ missing, plus untested ones you assumed). Then give a short verdict:

- **Ready:** all core concepts solid → go straight to the target, and say which of their strengths you'll build on.
- **Close:** one or two shaky concepts → brief targeted fixes, then the target.
- **Not yet:** missing foundations → a learning path starting from the lowest gap.

Frame gaps neutrally. The whole point is that starting lower is the faster route, not a judgment.

Let the user steer: offer the path, but if they'd rather push on to the target anyway, do that — and when the target explanation hits a gap, pause and briefly fill it in.

## Phase 4: Teach up the roadmap

Work through the gaps from the lowest layer up, one concept at a time.

For each concept:
1. **Anchor it** – say which part of the target this concept unlocks, so it doesn't feel like a detour.
2. **Explain it** – build only on concepts that are already solid. Concrete examples, analogies, and small "what actually happens" walk-throughs beat definitions.
3. **Check it** – ask one understanding question (same style as Phase 2). If they get it, mark it solid and move up. If not, re-explain from a different angle rather than repeating yourself louder.

Keep each step digestible — one concept per message or two is usually right. Learning is tiring; small wins keep momentum.

Periodically show progress against the roadmap (which layers are now solid, what's next). When all core concepts are solid, teach the target itself, explicitly connecting it back to the building blocks: "this is where virtual memory comes in…"

## When the target has been reached

Close with a short recap: the roadmap with everything solid, the key ideas of the target, and 2–3 natural next topics (what this unlocks next, or deeper versions of the same topic). If they want to go further, the next topic can get its own roadmap — and much of the current one will already be solid.

## General guidance

- **Adapt depth to the goal.** A conceptual overview needs a smaller roadmap than "I want to write a driver."
- **Use their materials.** If they're studying from a specific book, course, or project, align concepts and terminology with it and point to the relevant chapters.
- **Don't over-gate.** If a concept is only "helpful," teach it in passing rather than making them stop.
- **Be honest.** Don't mark something solid to be encouraging. A false ✅ is exactly the problem this skill exists to prevent.
