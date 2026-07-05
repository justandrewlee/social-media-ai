# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Is

**Social Media AI** surfaces the real outlier Reels from tracked competitors. It scrapes competitors' recent videos, flags the outliers (views vs. the creator's own baseline), and analyzes each with Gemini to capture a factual title, an on-screen description, the full verbatim transcript, and why it popped. Those feed Andrew's Home Base Idea Review inbox as raw source material.

> **Changed 2026-07-04:** the pipeline no longer generates adapted concepts, angles, or hooks. Andrew wants the raw reel and brings his own take. The Gemini analysis now emits Title / Description / Transcript / Why it popped, and the separate Claude concept-generation step was removed (see Pipeline Overview). The concept code (`claude.ts`, the "New Concepts Instruction" config field) still exists but is no longer called.

---

## How to Run

```bash
cd app
npm install
npm run dev
# Open http://localhost:3000
```

**Required environment variables** (in `.env` at project root):
- `APIFY_API_TOKEN` — Apify Instagram scraper
- `GEMINI_API_KEY` — Google Gemini video analysis
- `ANTHROPIC_API_KEY` — legacy, for the retired Claude concept step; not used by a pipeline run as of 2026-07-04

---

## Tech Stack

- **Next.js 16** (App Router) + **TypeScript**
- **Tailwind CSS** + **shadcn/ui** components
- **CSV files** for data storage (in `data/` directory)
- **Apify** — Instagram scraping
- **Google Gemini 2.0 Flash** — Video analysis (title, description, verbatim transcript, why it popped)
- **Claude Sonnet** — New concept generation (retired 2026-07-04; code kept but not called)

---

## How The System Works

### Pipeline Overview

1. **Input** — Select a config and parameters (max videos, top-K, days lookback) via the Run page
2. **Load Config** — Retrieve the analysis prompt and creator list from CSV (the new-concepts prompt is still read but no longer used)
3. **Scrape** — For each competitor creator, scrape recent Instagram Reels via Apify
4. **Filter & Rank** — Filter by date, sort by views, take top-K most viral
5. **Analyze** — Download video, upload to Gemini, analyze (extracts Title, Description, verbatim Transcript, Why it popped)
6. **Save** — Append results to `data/videos.csv`, viewable in the Videos page with thumbnails

> The old step 6 (Claude concept generation) was removed 2026-07-04. `newConcepts` is now always empty; the raw Gemini analysis is what the competitor-research skill reads.

### Customizable Prompts Per Config

- **Analysis Instruction** — How Gemini should break down the video (currently: Title / Description / Transcript / Why it popped)
- **New Concepts Instruction** — legacy field, retained in config but no longer used by the pipeline

---

## Workspace Structure

```
.
├── CLAUDE.md                              # This file
├── .env                                   # API keys (not committed)
├── app/                                   # Next.js application
│   ├── src/
│   │   ├── app/                           # Pages and API routes
│   │   │   ├── page.tsx                   # Dashboard
│   │   │   ├── videos/page.tsx            # Videos browser with thumbnails
│   │   │   ├── run/page.tsx               # Pipeline runner with live progress
│   │   │   ├── configs/page.tsx           # Config management
│   │   │   ├── creators/page.tsx          # Creator management
│   │   │   └── api/                       # API routes (configs, creators, videos, pipeline)
│   │   ├── lib/                           # Core logic
│   │   │   ├── pipeline.ts               # Pipeline orchestration
│   │   │   ├── apify.ts                  # Apify scraper client
│   │   │   ├── gemini.ts                 # Gemini video analysis client
│   │   │   ├── claude.ts                 # Claude concept generation client (legacy; not called since 2026-07-04)
│   │   │   ├── csv.ts                    # CSV read/write utilities
│   │   │   └── types.ts                  # TypeScript interfaces
│   │   └── components/                    # UI components (shadcn + custom)
│   └── package.json
├── data/                                  # CSV data storage
│   ├── configs.csv                        # Pipeline configurations
│   ├── creators.csv                       # Instagram creator accounts
│   └── videos.csv                         # Analyzed video results
├── context/                               # Background context for Claude
├── plans/                                 # Implementation plans
└── .claude/commands/                      # Slash commands (prime, create-plan, implement)
```

---

## App Pages

| Page | Path | Description |
|------|------|-------------|
| Dashboard | `/` | Summary stats, recent videos |
| Videos | `/videos` | Browse results with thumbnails, expandable analysis (title, description, transcript, why it popped) |
| Run Pipeline | `/run` | Select config, set params, run with live progress streaming |
| Configs | `/configs` | CRUD for pipeline configs (prompts, categories) |
| Creators | `/creators` | CRUD for competitor Instagram accounts |

---

## Commands

### /prime
Initialize a new session with full context awareness.

### /create-plan [request]
Create a detailed implementation plan in `plans/`.

### /implement [plan-path]
Execute a plan step by step.

---

## Critical Instruction: Maintain This File

After any change to the workspace, ask:
1. Does this change add new functionality?
2. Does it modify the workspace structure documented above?
3. Should a new command be listed?
4. Does context/ need updates?

If yes, update the relevant sections.

---

## Session Workflow

1. **Start**: Run `/prime` to load context
2. **Work**: Use commands or direct Claude with tasks
3. **Plan changes**: Use `/create-plan` before significant additions
4. **Execute**: Use `/implement` to execute plans
5. **Maintain**: Claude updates CLAUDE.md and context/ as the workspace evolves
