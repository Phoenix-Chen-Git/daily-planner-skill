# daily-planner-skill

An [OpenClaw](https://openclaw.ai) agent skill — a fully conversational replacement for the [Plan_and_log](https://github.com/Phoenix-Chen-Git) desktop app.

## Features

| Feature | Trigger |
|---|---|
| 🌅 Morning plan with AI generation | "plan my day" |
| ✅ Check / mark tasks done, quit, pending | "check tasks" |
| 📝 Add task to a future date | "add task for tomorrow" |
| 🌙 Evening review + AI summary + chat | "summarize my day" |
| 💪 Habit tracking with streaks | "manage habits" |
| 🎯 Goals with 4-stage progress system | "manage goals" |
| 📅 GitHub-style contribution calendar | "show calendar" |
| 💬 Tool feedback viewer | "show feedback" |

## Entry Point

Say **"open planner"** or **"daily planner"** to get the dashboard + inline button menu:

```
📅 Daily Planner — Wednesday, Feb 25
─────────────────────────────────────

🗓️ Today's Progress
  ████████░░░░ 4/6 done (66%) ✓4 ✗0 ○2  💪

🔥 Streak: 3 days

📌 Pending habits: Morning Exercise, Reading

[ 🌅 Plan my day ] [ ✅ Check tasks  ]
[ 📝 Add future  ] [ 🌙 Eve. summary ]
[ 💪 Habits      ] [ 🎯 Goals        ]
[ 📅 Calendar    ] [ 💬 Feedback     ]
```

## Data Storage

All data lives in `~/.daily_planner/data/`:

| File | Purpose |
|---|---|
| `YYYY-MM-DD-plan.json` | Daily plan |
| `YYYY-MM-DD-log.json` | Daily summary/log |
| `habits.json` | Habits + streaks |
| `goals.json` | Goals with stages & reviews |
| `tool_feedback.json` | Improvement feedback |

Compatible with existing `Plan_and_log` data — reads the same JSON files.

## Config

Create `~/.daily_planner/config.yaml` to customize job categories:

```yaml
daily_jobs:
  - name: "Morning Exercise"
    description: "Physical activity to start the day"
  - name: "Work Tasks"
    description: "Professional responsibilities"
  - name: "Learning"
    description: "Study or skill development"
  - name: "Personal Projects"
    description: "Side projects or hobbies"
```

## Installation

```bash
# Via clawhub (once published)
clawhub install daily-planner

# Or manually — clone into your skills directory
git clone https://github.com/Phoenix-Chen-Git/daily-planner-skill ~/.agents/skills/daily-planner
```

## Goal Stages

Goals track progress through 4 stages:

| Stage | Progress | Description |
|---|---|---|
| 🌱 Positive | 0–10% | Starting with good intentions |
| 🔥 Negative | 10–30% | Struggle phase |
| ⚡ Current | 30–50% | Working through it |
| 🚀 Improve | 50–100% | Getting better |

## License

MIT
