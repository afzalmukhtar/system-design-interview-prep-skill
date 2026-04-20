# System Design Interview Prep — Cursor Skill

A level-aware, whiteboard-driven system design interviewer, packaged as a
[Cursor Agent Skill](https://docs.cursor.com/context/skills).

The agent plays the interviewer. You pick a level (junior / mid / senior / staff),
a problem, and a whiteboard tool (Excalidraw, tldraw, Miro, Whimsical, FigJam,
Google Drawings, Mural). On every candidate turn, the agent takes a screenshot of
your live board via the Chrome DevTools MCP, analyzes it alongside what you said,
and probes with exactly one interviewer move. At the end you get a calibrated,
lenient rubric score in chat.

## How It Works

1. **Intake** — asks for level, problem-sourcing path (web search / you-provide /
   discuss together), time budget, and whiteboard tool.
2. **Problem statement** — locks scope before the board opens. Web-search path
   proposes 2-3 representative problems for your JD/domain.
3. **Board setup** — opens your chosen whiteboard in a new tab, waits for you to
   log in and create a blank canvas, confirms via screenshot, then starts.
4. **Interview loop** — every time you send a message, the agent first
   screenshots the board, then responds with one scoped probe, clarifying
   question, scenario injection, or trade-off challenge.
5. **Evaluation** — 8-dimension rubric scored 0-5 in 0.5 increments with
   partial credit, N/A allowed, level-weighted must-haves. Report is chat-only.

## Installation

### As a Personal Skill (all projects)

```bash
ln -s /Users/afzal985/opt/personal-projects/system-design-interview-prep \
  ~/.cursor/skills/system-design-interview-prep
```

Or copy if you prefer a detached install:

```bash
cp -R /Users/afzal985/opt/personal-projects/system-design-interview-prep \
  ~/.cursor/skills/system-design-interview-prep
```

### As a Project Skill (single repo)

```bash
cp -R /Users/afzal985/opt/personal-projects/system-design-interview-prep \
  <your-repo>/.cursor/skills/system-design-interview-prep
```

## Usage

The skill activates automatically when the user asks to run, mock, or practice a
system design interview. Example prompts:

- "Interview me for a senior SDE system design round on a ride-sharing backend
  using Excalidraw."
- "Mock a staff-level system design interview. Use the attached JD to pick a
  realistic problem. I'll use Miro."
- "Run a mid-level design interview. Problem: design a URL shortener. Board:
  tldraw."
- "Quiz me on system design. Junior level. Let's pick a problem together.
  Whiteboard: FigJam."

## Tool Footprint

Strictly whitelisted — the skill body enforces this:

**Uses:**

- `user-chrome-devtools.new_page` — open the whiteboard once
- `user-chrome-devtools.navigate_page` — only if you switch boards mid-session
- `user-chrome-devtools.list_pages` — resolve the whiteboard tab id
- `user-chrome-devtools.select_page` — refocus the tab before a screenshot
- `user-chrome-devtools.take_screenshot` — core interview loop
- `WebSearch` — only in Phase 1 when you pick the web-search problem path

**Does not use:**

- `take_snapshot`, `evaluate_script`, any console/network/dialog tool
- `click`, `type_text`, `fill`, `press_key`, `emulate`, `resize_page`, `upload_file`
- Terminal commands, file reads or writes, git
- Any other MCP server

The agent will never draw on your board, never reveal the model answer during
the interview body, and never persist transcripts or reports to disk.

## Design Notes

- **Lenient scoring.** Partial credit in 0.5 increments. Missing nice-to-haves
  doesn't penalize; missing the 2-3 must-haves for your level does.
- **N/A is valid.** Dimensions the problem genuinely doesn't need (e.g.
  multi-region for an internal admin tool) are excluded from the average.
- **Screenshot-first invariant.** Every candidate turn begins with
  `select_page` + `take_screenshot`. Candidates who narrate without drawing
  still get screenshotted — that silence is itself signal.
- **Level-weighted probes.** Junior gets happy-path CRUD; staff gets CAP,
  blast radius, migration strategy, and cost.
