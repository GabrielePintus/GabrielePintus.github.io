---
layout: post
title: "Building a knowledge base with Obsidian"
date: 2026-03-21
categories: productivity
key: obsidian-knowledge
related_posts:
  - obsidian-sync
---


I started building a personal knowledge base in **Obsidian**. The idea is simple. Whenever I read something worth keeping, I write a short note in Markdown and save it in the vault. I am not trying to copy everything. I only keep the parts that matter, so later I can search for them, browse them, and find the original source again without starting from zero.

## Where I started

I started with research. Right now I am focused on **explainability in machine learning**, especially counterfactual explanations. These are methods that answer questions like *"what would need to change for this model to give a different output?"* I am also thinking about refining and expanding my master's thesis on [JEPA](https://github.com/GabrielePintus/JEPA-Thesis), so I want a place where reading notes, ideas, and work in progress can live together.

The vault is called `Academic`, and it is organized like this:

```
Academic/
├── Papers/       ← one note per paper
├── Research/     ← project diaries
└── Assets/       ← PDFs and images
```

For each paper I read, I create a note in `Academic/Papers/`. I usually include the authors, the venue, a link to the PDF, a short description of the problem, and a summary of the method with enough detail that I can understand it again later. If a key equation or figure is central to the paper, I keep that too.

The `Research/` folder serves a different purpose. `Papers/` contains clean reading notes. `Research/` contains working notes. For each project or paper I am actively working on, I keep a running document that works like a diary. Every time I sit down to work, I write what I am trying, what worked, what failed, and what I am thinking. It is not polished, and it is not meant to be. It is just an honest record of the process.

By the end of a project, everything is already there: failed experiments, reasons behind decisions, and the way the approach changed over time. Writing the actual paper becomes much easier when I do not have to reconstruct the whole process from memory. I can read back through the diary and see the story taking shape on its own.

It also makes switching between projects much easier. When I come back to something after days or weeks away, I do not have to rebuild the whole context from scratch. I can read the last few entries and pick up where I left off.
