# Daily Planner App — Full Feature Reference

## App Entry Points

| Script | Purpose | Conversational trigger |
|---|---|---|
| `daily.py` | Main menu + dashboard | `/plan` menu |
| `plan.py` | Morning planning | "plan my day" |
| `check.py` | Mark tasks done/quit/pending | "check tasks" |
| `summarize.py` | Evening review + AI summary + chat | "summarize my day" |
| `calendar_view.py` | Contribution calendar + goal management | "show calendar" / "manage goals" |
| `feedback.py` | View/manage tool improvement feedback | "show feedback" |

## Dashboard (from `daily.py`)

Displayed on main menu load:
- Date + time header
- Today's task stats: completed/total with progress bar + color coding
- Streak: 🔥 if ≥3 days, ⭐ if 1-2
- Mini calendar (16 weeks)
- Upcoming tasks (next N days, from config `preferences.upcoming_days`, default 7)
- Goals summary (top 5 active)

## Plan Flow (`plan.py`)

1. Load config → get daily_jobs
2. Check if today's plan exists → ask to overwrite
3. Check yesterday's plan → find unfinished (completion_status[name] is null/missing)
4. Show unfinished tasks → checkbox select to carry over
5. For each daily_job category → ask user_input → recursively collect sub_jobs
6. Add carried-over tasks to jobs_input
7. Generate plan via AI
8. Refinement loop (show plan → ask refine → regenerate with feedback)
9. Save plan JSON

## Check Flow (`check.py`)

1. Hierarchical date selection: Year → Month → Week → Day
2. Load plan for selected date
3. Display plan content (markdown)
4. Interactive task tree with 3-state toggle:
   - `[ ]` pending → `[✓]` done → `[✗]` quit → `[ ]` pending
5. Save updated completion_status
6. Display completion summary table + progress bar

## Summarize Flow (`summarize.py`)

1. Load today's plan
2. Display plan content
3. For each job → Did you finish? (Yes/No/Partial)
   - Yes → quality (Excellent/Good/Okay)
   - No/Partial → problem description
   - Recursively review sub_jobs
4. Generate AI summary
5. Summary refinement loop
6. Interactive chat with DeepSeek (has context of summary)
7. Tool reflection feedback (iterative AI understanding confirmation)
8. Save log JSON

## Goal Stages (from `calendar_view.py`)

Progress % → Stage:
- 0-10%: 🌱 Positive (starting well)
- 10-30%: 🔥 Negative (struggle phase)
- 30-50%: ⚡ Current (working through it)
- 50-100%: 🚀 Improve (getting better)

## Calendar Intensity (from `calendar_view.py`)

Completion % → Intensity:
- 0%: `□` dim (no tasks done)
- 1-24%: `▪` green4
- 25-49%: `▪` green3
- 50-74%: `▪` green1
- 75-100%: `▪` bold bright_green
- Today: `◉` bold cyan

## Feedback Flow (`feedback.py`)

1. Show pending entries table (max 10)
2. Options: view detail, add new, mark implemented/dismissed
3. Add new → iterative AI understanding:
   - AI: "I understand that you want..."
   - User: Yes / No / Refine
   - Loop until Yes or user gives up
4. Mark done → archive to `improves/archived_feedback.json`

## Habit Logic (from `lib/storage.py`)

Streak algorithm (consecutive days back from today):
```
expected = today
streak = 0
for date in sorted(check_ins, reverse=True):
    if date == expected:
        streak += 1; expected -= 1 day
    elif date == yesterday and expected == today:
        streak += 1; expected = date - 1 day
    else: break
```

Auto-complete habit when streak ≥ target_days.

## Goal Review Types

- Weekly: due every 7 days
- Monthly: due every 30 days
- Yearly: due every 365 days

Review asks per goal: done, not_done, closer (great/steady/same/behind/rethink), next_actions, optional progress update.

## Config YAML (`~/.daily_planner/config.yaml`)

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

deepseek:
  model: "deepseek-chat"
  temperature_planning: 0.0
  temperature_chat: 0.7
  max_tokens: 2000
  api_base: "https://api.deepseek.com"

preferences:
  timezone: "Asia/Shanghai"
  language: "en"
  upcoming_days: 7
  log_schedule:
    weekly_day: 0      # Monday
    monthly_day: 1     # 1st of month
    yearly_month: 1    # January
    yearly_day: 1      # 1st
```
