# IntentOS

**Say it once. Watch it happen, safely.**

IntentOS is a voice-first AI that uses your computer the way a person would. You say what you want; it shows a plan, asks before anything sensitive, does the work live on screen in your real apps, checks that it worked, fixes itself when something breaks, and gets faster every time you repeat a task.

> wHACKiest 2026 · Track 1: Make MCP Matter (Less Magic, More Utility)\*\*
> Every capability is an MCP tool, it works in apps people already use (Spotify, WhatsApp, Chrome, VS Code, Office), and it is built around useful, reliable and controlled actions.

## The problem

People think in goals, but computers only take step-by-step commands, so every small goal turns into a dozen clicks across different apps.

"Send this to Dad" means: copy → open WhatsApp → find the chat → paste → send.

|Existing tool|Gap|
|-|-|
|Chatbots (ChatGPT-style)|Explain how to do a task, but don't do it|
|Built-in assistants|Act, but only on a narrow, fixed list of commands|
|Agents that act|Run blind: no plan up front, no confirmation before sending or deleting, miss their own failures, can't undo|
|All of them|Nothing learns; you repeat the same routines and the computer never gets faster|

## How it works

1. Press **Ctrl+Space** and speak, or type.
2. **Context** reads the active window, selection and open tabs.
3. The **planner** turns the goal into ordered tool calls, streamed so the first step can start early.
4. The **HUD** shows the plan; steps that send, delete, pay or post wait for your approval.
5. The **executor** runs each step with the fastest reliable method: learned skill → app tool → general computer use.
6. The **verifier** checks the result. On failure the executor tries the next method; if all fail, IntentOS asks you.
7. Every action is logged with its undo, successful runs are saved as skills, and IntentOS speaks a short confirmation.

### Example: "Play some 90s Bollywood music"

1. The plan appears: open Spotify, find a 90s Bollywood playlist, play it.
2. Spotify's search API picks the best-matching playlist.
3. The Spotify desktop app opens on that playlist and IntentOS clicks Play, visibly.
4. It checks the window title now shows a song, then says "Playing 90s Bollywood Hits."
5. "Pause", "next" and "louder" respond instantly.

## Architecture

```
            You (voice / text)                          HUD on screen
                   │                                 ▲ (plan, approvals, live feed)
                   ▼                                 │
 ┌──────────────── IntentOS core · MCP client (Python, FastAPI) ────────────────┐
 │  Context ──► Planner ──► Permissions ──► Executor ──► Verifier              │
 │                                             ▲            │                   │
 │                                             └── retry ───┘ (check failed)    │
 └───────────────┬──────────────────────────────────┬───────────────────────────┘
                 │ read / write                     │ tool calls
                 ▼                                  ▼
        Memory (SQLite)              MCP · one protocol to every tool
  skills, undo journal,      ┌──────────────┬───────────┬────────────────┬──────────┐
  workspaces, preferences    │intentos-     │ Browser   │ Gmail +        │ n8n      │
                             │desktop       │(Playwright│ Calendar       │(MCP      │
                             │(FastMCP)     │ Chrome)   │ MCP servers    │ Server   │
                             └──────────────┴───────────┴────────────────┴ Trigger) ┘
```

### Three ways IntentOS acts

|Order|Method|Used for|Typical speed|
|-|-|-|-|
|1|Learned skill: replay of a task done before, no model calls|Repeated tasks|1–3 s|
|2|App tool: deep links plus visible UI automation|Core apps (Spotify, WhatsApp, Chrome, VS Code, Office, settings)|2–5 s|
|3|General computer use: a vision model reads the screen and drives mouse and keyboard|Any other app or website|Several seconds per step|

## Core capabilities

|#|Capability|What it does|
|-|-|-|
|1|Voice in and out|Push Ctrl+Space and speak; IntentOS answers aloud|
|2|Context awareness|Understands "this": the open PDF, selected text, current tab, clipboard|
|3|Visible plan|Shows a step checklist before running anything; sensitive steps marked with a lock|
|4|Permissions|Spoken or tapped approval for sending, deleting, paying, posting, activating automations|
|5|Live execution|Apps open, the cursor moves, text is typed character by character|
|6|Verification|Confirms each step's result before moving on|
|7|Self-healing|Notices failures, retries another way, updates the saved skill|
|8|Learning|Repeated tasks get faster; "watch me" teaching; remembers preferences|
|9|Workspaces|"Presentation mode" and "back to what I was doing"|
|10|Undo and kill switch|"Undo that"; "stop" or Esc halts everything|
|11|Background automations|n8n routines on a schedule or on events|

## Tech stack

|Layer|Technology|
|-|-|
|Backend|Python 3.11, FastAPI, Uvicorn, asyncio|
|MCP|Official Python MCP SDK (FastMCP)|
|Planner LLM|Groq · Llama 3.3 70B (native tool calling, streaming)|
|Vision / computer use|Vision model with computer-use support|
|Speech|Silero VAD, Groq Whisper, Deepgram Aura · faster-whisper and pyttsx3 offline fallbacks|
|Desktop control|pywinauto (UI Automation), pyautogui, pycaw, screen-brightness-control, Win32|
|Browser|Playwright (visible Chrome)|
|App APIs|Spotify Web API (search), `spotify:` and `whatsapp://` deep links, `code` CLI|
|Cloud apps|Gmail and Google Calendar MCP servers|
|Automations|n8n with MCP Server Trigger|
|Storage|SQLite (action journal, skills, preferences, workspaces)|
|HUD|Transparent always-on-top window, HTML/CSS/JS|

## Getting started

**Requirements:** Windows 10/11, Python 3.11, Spotify and WhatsApp desktop apps.

```bash
git clone <this-repo-url>
cd intentos
python -m venv .venv
.venv\\Scripts\\activate
pip install -r requirements.txt
playwright install chromium
```

Create a `.env` file in the project root:

```env
GROQ\_API\_KEY=
DEEPGRAM\_API\_KEY=
SPOTIFY\_CLIENT\_ID=
SPOTIFY\_CLIENT\_SECRET=
```

Then start IntentOS and press **Ctrl+Space** to talk.

## Safety

* Every sensitive step (send, delete, pay, post, activate automation) needs an explicit yes.
* No arbitrary shell: terminal commands come from an allow-list.
* Deleted files go to an undo folder; every reversible action is logged with its inverse.
* "Stop" or Esc halts everything immediately.

## Roadmap

|Priority|Scope|
|-|-|
|**P0 · Demo works**|intentos-desktop MCP server, Spotify and WhatsApp tools, streaming planner, HUD with plan checklist and live feed, voice|
|**P1 · Trust**|Verification after every step, self-healing fallback chain, approvals, kill switch, context awareness|
|**P2 · Learning and reach**|General computer use, skills compiled from successful runs, Chrome / VS Code / Office tools, n8n automations|
|**P3 · Polish**|Workspaces, undo journal, "watch me" teaching, preference memory|



