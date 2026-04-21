# System Design Interview Prep — Coding Agent Skill

A level-aware, whiteboard-driven system design interviewer, packaged as a
coding-agent skill (Anthropic-style `SKILL.md` with YAML frontmatter). Works
with any agent that honors the `SKILL.md` convention — **Cursor**, **Claude
Code**, **Codex CLI**, and similar tools. A Windsurf workaround is included
below since Windsurf does not natively support `SKILL.md`-style skills yet.

The agent plays the interviewer. You pick a level (junior / mid / senior / staff),
a problem, and a whiteboard tool (Excalidraw, tldraw, Miro, Whimsical, FigJam,
Google Drawings, Mural). On every candidate turn, the agent takes a screenshot of
your live board via the Chrome DevTools MCP, analyzes it alongside what you said,
and probes with exactly one interviewer move. At the end you get a calibrated,
lenient rubric score in chat.

> **Requires the Chrome DevTools MCP** (`user-chrome-devtools`) in whichever
> agent you wire this into. The skill will refuse to run without it.

## How It Works

1. **Pre-flight** — before anything, the agent verifies the `user-chrome-devtools`
  MCP is connected. If it's missing, the skill refuses to start rather than
   silently degrading into a text-only interview.
2. **Intake** — asks for level, problem-sourcing path (web search / you-provide /
  discuss together), time budget, and whiteboard tool.
3. **Problem statement** — locks scope before the board opens. Web-search path
  proposes 2-3 representative problems for your JD/domain (and falls back to
   the you-provide / discuss paths if the agent runtime has no `WebSearch`).
4. **Board setup** — opens your chosen whiteboard in a new tab, waits for you to
  log in and create a blank canvas, confirms via screenshot, then opens a
   Google countdown-timer tab in the background so elapsed time is a real,
   screenshot-readable signal rather than a hallucinated wall clock.
5. **Interview loop** — every time you send a message, the agent first
  screenshots the board, then responds with one scoped probe, clarifying
   question, scenario injection, or trade-off challenge. At phase transitions
   it peeks at the timer tab to decide when to push toward deep-dives.
6. **Evaluation** — 8-dimension rubric scored 0-5 in 0.5 increments with
  partial credit, N/A allowed, level-weighted must-haves. Report is chat-only.

## Installation

Clone the repo once, anywhere on disk:

```bash
git clone https://github.com/afzalmukhtar/system-design-interview-prep-skill.git \
  ~/skills/system-design-interview-prep
```

Then symlink it into each agent's skills directory. Symlinks let `git pull` in
one place propagate updates to every agent. If you prefer a detached copy, use
`cp -R` instead of `ln -s`.

### Cursor

Personal (available in all projects):

```bash
mkdir -p ~/.cursor/skills
ln -s ~/skills/system-design-interview-prep \
  ~/.cursor/skills/system-design-interview-prep
```

Project (single repo only):

```bash
mkdir -p <your-repo>/.cursor/skills
ln -s ~/skills/system-design-interview-prep \
  <your-repo>/.cursor/skills/system-design-interview-prep
```

### Claude Code

Personal:

```bash
mkdir -p ~/.claude/skills
ln -s ~/skills/system-design-interview-prep \
  ~/.claude/skills/system-design-interview-prep
```

Project:

```bash
mkdir -p <your-repo>/.claude/skills
ln -s ~/skills/system-design-interview-prep \
  <your-repo>/.claude/skills/system-design-interview-prep
```

### Codex CLI (OpenAI)

Personal:

```bash
mkdir -p ~/.codex/skills
ln -s ~/skills/system-design-interview-prep \
  ~/.codex/skills/system-design-interview-prep
```

### Windsurf

Windsurf does not natively support the `SKILL.md` skill format as a
first-class concept. Two reasonable workarounds:

**Option A — expose it as a rule** (auto-loaded into every session):

```bash
mkdir -p ~/.codeium/windsurf/memories
cp ~/skills/system-design-interview-prep/SKILL.md \
   ~/.codeium/windsurf/memories/system-design-interview-prep.md
```

Or put it in a workspace-local rule:

```bash
mkdir -p <your-repo>/.windsurf/rules
ln -s ~/skills/system-design-interview-prep/SKILL.md \
  <your-repo>/.windsurf/rules/system-design-interview-prep.md
```

**Option B — paste on demand**: open `SKILL.md` and paste its body into the
Cascade chat at the start of a session. Less automatic, but zero setup.

Windsurf's rule system changes frequently; consult current Windsurf docs for
the active path if the ones above have moved.

### Any other agent with `SKILL.md` support

The skill is just a directory with `SKILL.md` + `README.md`. Symlink or copy
the directory into whatever path your agent scans for skills.

## Usage

Once installed, the skill activates automatically when you ask the agent to
run, mock, or practice a system design interview. Example prompts:

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
- `WebSearch` (agent built-in) — only in Phase 1 when you pick the web-search
problem path

**Does not use:**

- `take_snapshot`, `evaluate_script`, any console/network/dialog tool
- `click`, `type_text`, `fill`, `press_key`, `emulate`, `resize_page`, `upload_file`
- Terminal commands, file reads or writes, git
- Any other MCP server

`take_snapshot` is intentionally excluded even though it ships in the same MCP.
Whiteboards render diagrams in canvas/SVG layers where DOM text says almost
nothing useful; a raster screenshot is cheaper, richer signal, and avoids DOM
noise. Future maintainers: please don't "optimize" this by adding snapshots.

The agent will never draw on your board, never reveal the model answer during
the interview body, never share rubric scores mid-session, and never persist
transcripts or reports to disk.

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

