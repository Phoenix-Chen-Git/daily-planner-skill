---
name: daily-planner
description: AI-powered daily planning, task tracking, evening review, habit monitoring, goals management, and contribution calendar — fully conversational via OpenClaw. Replaces the standalone "Plan_and_log" desktop app. Use when the user wants to: (1) create a morning plan ("plan my day", "morning plan"), (2) check/mark tasks done/quit ("check tasks", "mark X as done"), (3) add a future task ("add task for tomorrow"), (4) do an evening summary/review ("summarize my day", "evening review"), (5) manage habits ("add habit", "check in habit", "show habits"), (6) manage goals ("add goal", "update goal progress", "review goals"), (7) view contribution calendar ("show calendar", "activity calendar"), (8) view/manage feedback ("show feedback", "add feedback"). Also triggered by generic entry phrases like "open planner", "daily planner", "open daily", "show planner".
---

# Daily Planner Skill

Full conversational replacement for the `Plan_and_log` desktop app.

---

## Entry Point: Dashboard + Main Menu

**Always show this first** whenever the skill is triggered without a specific sub-command (e.g. "plan my day" → go straight to Workflow 1 instead). When triggered by phrases like "open planner", "daily planner", or just invoked as a skill with no specific intent, render the dashboard then present the main menu via **inline buttons**.

### Step 1: Render the Dashboard

Collect the following data before sending the message:

1. **Today's date/time** — from `session_status`
2. **Today's plan stats** — load `~/.daily_planner/data/YYYY-MM-DD-plan.json`:
   - Count tasks: total, completed (done), quit, pending
   - Compute `done_pct = completed * 100 // total` (0 if no plan)
3. **Streak** — count consecutive days backwards from today where `completion_status` has at least one `"done"` entry
4. **Pending habits** — load `habits.json`, find habits with no check-in today
5. **Pending goals review** — load `goals.json`, check if any review is overdue (>7 days for weekly)
6. **Upcoming tasks** — scan plans for tomorrow and next 2 days for pending tasks

### Step 2: Format the Dashboard Message

Send a single message in this format (adapt based on what data is available):

```
📅 Daily Planner — [Weekday, Month DD]
─────────────────────────────────────

🗓️ Today's Progress
  [progress bar] X/Y done (Z%)  ✓N ✗N ○N

🔥 Streak: N days   (or ⭐ or no streak line if 0)

📌 Pending habits: [name1], [name2]   (skip if none)

📅 Coming up tomorrow: [task1], [task2]   (skip if none)

⚠️ Goals review due   (skip if no review overdue)
```

**Progress bar format:**
- Width 12 chars: `[green done_bar][red quit_bar][dim░ pending_bar]`
- Color: 🎉 ≥80%, 💪 ≥50%, 🚀 ≥20%, 📋 no plan/0%

Example:
```
📅 Daily Planner — Wednesday, Feb 25
─────────────────────────────────────

🗓️ Today's Progress
  ████████░░░░ 4/6 done (66%) ✓4 ✗0 ○2  💪

🔥 Streak: 3 days

📌 Pending habits: Morning Exercise, Reading
```

### Step 3: Send Inline Button Menu

After the dashboard message, send the menu as **inline buttons** using the `message` tool with `buttons`:

```json
[
  [
    {"text": "🌅 Plan my day",     "callback_data": "planner:plan"},
    {"text": "✅ Check tasks",      "callback_data": "planner:check"}
  ],
  [
    {"text": "📝 Add future task",  "callback_data": "planner:future"},
    {"text": "🌙 Evening summary",  "callback_data": "planner:summary"}
  ],
  [
    {"text": "💪 Habits",           "callback_data": "planner:habits"},
    {"text": "🎯 Goals",            "callback_data": "planner:goals"}
  ],
  [
    {"text": "📅 Calendar",         "callback_data": "planner:calendar"},
    {"text": "💬 Feedback",         "callback_data": "planner:feedback"}
  ]
]
```

When the user taps a button (or types the equivalent), route to the matching workflow:
- `planner:plan` → Workflow 1: Morning Plan
- `planner:check` → Workflow 2: Check Tasks
- `planner:future` → Workflow 3: Add Future Task
- `planner:summary` → Workflow 4: Evening Summary
- `planner:habits` → Workflow 5: Habits
- `planner:goals` → Workflow 6: Goals
- `planner:calendar` → Workflow 7: Calendar
- `planner:feedback` → Workflow 8: Feedback

### Implementation

```
1. exec: mkdir -p ~/.daily_planner/data
2. exec: cat ~/.daily_planner/data/$(date +%Y-%m-%d)-plan.json
3. exec: cat ~/.daily_planner/data/habits.json
4. exec: cat ~/.daily_planner/data/goals.json
5. Build dashboard text
6. message(action=send, message=<dashboard>, buttons=[[...]])
7. Reply NO_REPLY (message tool handled the delivery)
```

After the user picks an option, proceed with that workflow directly — no need to re-show the dashboard unless they return to the main menu.

---

## Data Storage

All data lives in `~/.daily_planner/data/`. Always run:
```bash
mkdir -p ~/.daily_planner/data
```

### File Naming
| File | Purpose |
|---|---|
| `YYYY-MM-DD-plan.json` | Daily plan |
| `YYYY-MM-DD-log.json` | Daily summary/log |
| `habits.json` | Habit definitions + check-in history |
| `goals.json` | Goals with stages and reviews |
| `tool_feedback.json` | Feedback entries (pending) |
| `improves/archived_feedback.json` | Archived feedback |
| `~/.daily_planner/config.yaml` | User configuration |

### Read a file:
```bash
cat ~/.daily_planner/data/YYYY-MM-DD-plan.json
```

### Write JSON safely (use Python, never echo/string concat):
```bash
python3 -c "
import json
data = {...}
with open('/Users/USERNAME/.daily_planner/data/YYYY-MM-DD-plan.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
"
```

### List available plans:
```bash
ls ~/.daily_planner/data/*-plan.json 2>/dev/null | sort -r
```

---

## Config

Read from `~/.daily_planner/config.yaml` (fallback to defaults if not found):

```yaml
daily_jobs:
  - name: "Morning Exercise"
    description: "Physical activity to start the day"
  - name: "Work Tasks"
    description: "Professional responsibilities and projects"
  - name: "Learning"
    description: "Study or skill development"
  - name: "Personal Projects"
    description: "Side projects or hobbies"
```

Always read the config first:
```bash
cat ~/.daily_planner/config.yaml 2>/dev/null || echo "NOT_FOUND"
```

---

## Plan JSON Schema

```json
{
  "date": "2026-02-25T09:00:00",
  "jobs": [
    {
      "name": "Work Tasks",
      "description": "Professional responsibilities",
      "user_input": "Finish the protein design paper section 3",
      "sub_jobs": [
        {
          "name": "Write Methods",
          "description": "What to do",
          "sub_jobs": []
        }
      ],
      "carried_over_from": "2026-02-24",
      "chat_notes": []
    }
  ],
  "plan_content": "## Today's Plan\n\n...",
  "completion_status": {
    "Work Tasks": "done",
    "Write Methods": null
  },
  "last_checked": "2026-02-25T20:00:00",
  "refinement_history": []
}
```

**completion_status values:**
- `null` or missing = **pending** `[ ]`
- `"done"` = **completed** `[✓]`
- `"quit"` = **abandoned** `[✗]`

---

## Log JSON Schema

```json
{
  "date": "2026-02-25T21:00:00",
  "plan": { "...full plan data..." },
  "review": [
    {
      "job_name": "Work Tasks",
      "status": "Yes",
      "quality": "Good",
      "problem": null,
      "sub_reviews": [
        {
          "task_name": "Write Methods",
          "status": "Yes",
          "quality": "Excellent",
          "sub_reviews": []
        }
      ],
      "chat_notes": []
    }
  ],
  "summary": "## Daily Summary\n\n...",
  "refinement_history": [],
  "chat": [
    { "user": "...", "assistant": "..." }
  ],
  "tool_reflection": null
}
```

---

## Habits JSON Schema (`habits.json`)

```json
{
  "habits": [
    {
      "id": "abc12345",
      "name": "Morning Exercise",
      "target_days": 30,
      "created_at": "2026-02-01T00:00:00",
      "check_ins": ["2026-02-01", "2026-02-02"],
      "current_streak": 2,
      "best_streak": 5,
      "status": "active"
    }
  ]
}
```

**Streak calculation:** Count consecutive days backwards from today. A check-in for yesterday counts if user hasn't checked in today yet. A gap breaks the streak.

---

## Goals JSON Schema (`goals.json`)

```json
{
  "goals": [
    {
      "id": 1,
      "name": "Get PADI AOW Certification",
      "type": "monthly",
      "priority": "high",
      "created_at": "2026-02-01T00:00:00",
      "deadline": "2026-10-31",
      "status": "active",
      "progress": 20,
      "sub_goals": [
        {
          "id": "1.1",
          "name": "Complete eLearning",
          "created_at": "...",
          "deadline": "2026-03-31",
          "status": "active",
          "sub_goals": []
        }
      ],
      "stages": {
        "positive": "Starting eLearning, motivated",
        "negative": "Pool training feels hard",
        "current": "Doing open water dives",
        "improve": "Completing AOW dives"
      }
    }
  ],
  "archived": [],
  "reviews": []
}
```

**Goal stages by progress %:**
- 🌱 Positive (0-10%): Starting with good intentions
- 🔥 Negative (10-30%): Struggle phase
- ⚡ Current (30-50%): Working through it
- 🚀 Improve (50-100%): Getting better

**Goal types:** `long_term`, `yearly`, `monthly`, `weekly`
**Priorities:** `high` 🔴, `medium` 🟡, `low` 🟢

---

## Feedback JSON Schema (`tool_feedback.json`)

```json
{
  "feedback_entries": [
    {
      "id": "abc12345",
      "date": "2026-02-25T21:00:00",
      "original_feedback": "I want the plan to include time blocks",
      "final_understanding": "User wants time-blocked schedule in the plan",
      "understanding_history": [
        { "user_input": "...", "ai_understanding": "..." }
      ],
      "status": "pending"
    }
  ]
}
```

---

## Workflow 1: Morning Plan (`/plan` or "plan my day")

**Goal:** Create a structured daily plan — this is the `plan.py` equivalent.

### Steps:

1. **Check for existing today's plan:**
   ```bash
   cat ~/.daily_planner/data/$(date +%Y-%m-%d)-plan.json 2>/dev/null || echo "NOT_FOUND"
   ```
   If exists → ask: "You already have a plan for today. Create a new one?" If No, exit.

2. **Check yesterday's plan for unfinished tasks:**
   ```bash
   cat ~/.daily_planner/data/$(date -v-1d +%Y-%m-%d)-plan.json 2>/dev/null || echo "NOT_FOUND"
   # On Linux: date -d "yesterday" +%Y-%m-%d
   ```
   Extract tasks where `completion_status[name]` is `null` or missing.
   If unfinished tasks found → show them and ask (checkbox-style, one message): "Which tasks to carry over? (list them, say 'all', 'none', or specific names)"
   Mark carried-over tasks with `"carried_over_from": "YYYY-MM-DD"`.

3. **Load daily job categories from config** (see Config section above).

4. **Collect tasks per category — one at a time:**
   For each job category:
   - Show category name + description
   - Ask: "What do you need to do for **[category]**? (or type 'skip' to skip)"
   - After user replies, ask: "Do you want to add sub-tasks for '[task]'? (Yes/No)"
   - If Yes → collect sub-tasks (recursively: ask name, then ask for sub-sub-tasks)
   - Continue to next category

5. **Generate AI plan** using this system prompt:
   > "You are a helpful assistant that creates organized daily plans. Based on the following job inputs, create a detailed daily plan with checkboxes. Break down each job into specific, actionable tasks. Use markdown format with checkbox syntax (- [ ]). For sub-tasks, use nested indentation to show hierarchy. Preserve the hierarchy of sub-tasks."

   Format the prompt with all jobs, their descriptions, user_input, and sub_jobs.

6. **Refinement loop:**
   - Display the generated plan
   - Ask: "Do you want to refine this plan? (Yes/No)"
   - If Yes → ask for feedback, regenerate with: "Here is the current daily plan:\n\n{plan}\n\nUser feedback: {feedback}\n\nPlease update the plan based on the feedback..."
   - Repeat until user says No

7. **Save plan:**
   ```json
   {
     "date": "<ISO timestamp>",
     "jobs": [...jobs_input...],
     "plan_content": "<generated markdown>",
     "refinement_history": [...],
     "completion_status": {}
   }
   ```
   Save to `~/.daily_planner/data/YYYY-MM-DD-plan.json`

8. **Confirm saved** and display the final plan.

---

## Workflow 2: Check / Mark Tasks (`/check` or "check tasks" / "mark X as done")

**Goal:** Update task completion status — this is the `check.py` equivalent.

### Steps:

1. **Select which date's plan:**
   - Default: today
   - If user specifies a date, use that
   - Can also show available dates: `ls ~/.daily_planner/data/*-plan.json | sort -r`
   - The original app has hierarchical Year → Month → Week → Day navigation; in conversational mode, ask which date

2. **Load the plan** for that date.

3. **Display all tasks** with current status:
   ```
   [ ] Work Tasks
     [ ] Write Methods (sub-task)
   [✓] Morning Exercise
   [✗] Learning
   ```

4. **Ask which tasks to update:**
   - User can say "mark Work Tasks as done", "mark Learning as quit", "mark all as done"
   - Or list multiple: "mark Work Tasks done, Morning Exercise done"
   - Cycle states: pending `[ ]` → done `[✓]` → quit `[✗]` → pending `[ ]`

5. **Handle sub-tasks:** If parent is marked done/quit, ask about sub-tasks too.

6. **Save** updated `completion_status` back to plan JSON.
   Update `"last_checked": "<ISO timestamp>"`.

7. **Display completion summary:**
   - Table of tasks with status icons
   - Progress bar: `[✓done_bar][✗quit_bar][░░pending]  ✓N ✗N ○N (X% done)`
   - Emoji based on completion: 🎉 ≥80%, 💪 ≥50%, 🚀 ≥20%, 📋 <20%

---

## Workflow 3: Add Future Task (`/future` or "add task for tomorrow" / "add task for [date]")

**Goal:** Add a task to a future date's plan — `add_future_task()` in `daily.py`.

### Steps:

1. Ask which date (tomorrow, specific date, etc.)
2. Load or create that date's plan JSON (empty structure if not existing)
3. Ask which job category (from config)
4. Ask task description
5. Append to `jobs[]` and update `plan_content`
6. Save

---

## Workflow 4: Evening Summary (`/summary` or "summarize my day" / "evening review")

**Goal:** Review the day and generate AI summary with optional chat — `summarize.py` equivalent.

### Steps:

1. **Load today's plan.** If none → tell user to create a plan first.

2. **Display today's plan content.**

3. **Review each job one at a time:**
   For each job in `plan_data.jobs`:
   - Show job name and what was planned (`user_input`)
   - Ask: "Did you finish **[job name]**?" → Yes / No / Partial
   - If Yes → ask: "How did it go?" → Excellent / Good / Okay
   - If No or Partial → ask: "What was the problem?"
   - If job has `sub_jobs` → review each sub-job the same way (recursively)

4. **Generate AI summary** using:
   > System: "You are a thoughtful reflection assistant that helps people learn from their daily experiences."
   
   Prompt includes:
   - Original Plan section (all jobs + user_input)
   - Review section (each job: status, quality/problem)
   - "Please create a thoughtful summary with sections for: 1. Accomplishments 2. Challenges 3. Reflection 4. Recommendations for Tomorrow"

5. **Summary refinement loop** (same as plan refinement):
   - Display summary
   - Ask if they want to refine
   - Regenerate with feedback if Yes

6. **Interactive chat:**
   - Offer: "Want to chat about your day?"
   - If Yes → have a free-form conversation about the day
   - Initialized with summary as context
   - Continue until user says "exit" or "done"
   - Save chat messages to log

7. **Tool Reflection:**
   - Ask: "What do you think this tool can be updated or improved? (Press Enter to skip)"
   - If feedback given → iterative understanding loop:
     a. Generate AI understanding: "I understand that you want..."
     b. Ask: "Is this understanding correct?" → Yes / No / Refine
     c. If Refine → ask how to refine description, regenerate
     d. If No → ask for new description, regenerate
     e. If Yes → save to `tool_feedback.json`
   - Max 10 pending feedback entries

8. **Save log:**
   ```json
   {
     "date": "<ISO>",
     "plan": {<full plan>},
     "review": [...review_data...],
     "summary": "<markdown>",
     "refinement_history": [...],
     "chat": [{user, assistant}...],
     "tool_reflection": {...} or null
   }
   ```

---

## Workflow 5: Habits (`/habits` or "manage habits")

**Goal:** Track daily habits with streaks — Storage.habits methods.

### Load habits:
```bash
cat ~/.daily_planner/data/habits.json 2>/dev/null || echo '{"habits":[]}'
```

### Sub-commands:

**List habits:**
- Load `habits.json`
- Calculate current_streak for each (consecutive days back from today)
- Display: name, 🔥 streak, best streak, target days, status (active/completed)
- Show progress toward target: `██░░░ 5/30 days`

**Add habit** ("add habit [name]"):
- Ask name and target_days (default 30)
- Generate UUID id (use `python3 -c "import uuid; print(str(uuid.uuid4())[:8])"`)
- Append to habits.json
- Confirm added

**Check in** ("check in [habit name]"):
- Find habit by name (fuzzy match okay)
- Check if already checked in today (date in `check_ins` list)
- If not → add today's date to `check_ins`
- Recalculate streak
- If streak ≥ target_days → set `status: "completed"`
- Show updated streak

**Delete habit** ("delete habit [name]"):
- Find by name, confirm before deleting
- Remove from habits array, save

**Streak calculation algorithm:**
```python
today = date.today()
expected = today
streak = 0
for date_str in sorted(check_ins, reverse=True):
    check_date = parse(date_str).date()
    if check_date == expected:
        streak += 1
        expected = expected - timedelta(days=1)
    elif check_date == today - timedelta(days=1) and expected == today:
        streak += 1
        expected = check_date - timedelta(days=1)
    else:
        break  # gap
```

---

## Workflow 6: Goals (`/goals` or "manage goals")

**Goal:** Manage long-term goals with 4-stage progress — `calendar_view.py` goals functions.

### Load goals:
```bash
cat ~/.daily_planner/data/goals.json 2>/dev/null || echo '{"goals":[],"archived":[],"reviews":[]}'
```

### Sub-commands:

**List goals** ("show goals", "my goals"):
- Load goals.json, filter active goals
- Sort by priority (high → medium → low)
- For each goal, show: priority emoji, name, stage emoji + name, progress bar + %
- Show completed count at bottom
- Check if weekly/monthly/yearly review is due (7/30/365 days since last)

**Add goal** ("add goal"):
- Ask: name, type (long_term/yearly/monthly/weekly), priority (high/medium/low), deadline (YYYY-MM-DD, optional)
- Ask for 4 stage descriptions:
  - 🌱 Positive (0-10%): what does starting well look like?
  - 🔥 Negative (10-30%): what struggles might you face?
  - ⚡ Current (30-50%): what does working through it look like?
  - 🚀 Improve (50-100%): what does success look like?
- Create goal with id = max(existing_ids) + 1, progress=0, status=active
- Save to goals.json

**Add sub-goal** ("add sub-goal to [goal name]"):
- Select parent goal (max 3 levels deep)
- Ask name and deadline (optional)
- Generate ID: `"parent_id.N"` where N = next number
- Append to parent's `sub_goals[]`

**Update progress** ("update [goal name] progress to X%"):
- Find goal by name
- Show current stage descriptions
- Update progress (0-100)
- Show stage change if applicable
- If progress ≥ 100, ask: "Mark as completed?"

**Goal review** ("review goals", "weekly review", "monthly review"):
- Select review type: weekly / monthly / yearly
- For each active goal:
  - Show current stage
  - Ask: "What have you DONE towards this goal?"
  - Ask: "What have you NOT done?"
  - Ask: "Are you getting closer?" (great/steady/same/behind/rethink)
  - Ask: "What will you do next?"
  - Optionally update progress
- Ask for overall reflection
- Save review to `goals.json` under `"reviews"` array
- Check review due: weekly if >7 days, monthly if >30 days, yearly if >365 days

**View all goals** ("show all goals"):
- Group by type, show all including completed

**Archive goal** ("archive [goal name]"):
- Confirm, then move from `goals` to `archived` array
- Add `archived_at` timestamp

---

## Workflow 7: Contribution Calendar (`/calendar` or "show calendar")

**Goal:** Show GitHub-style activity calendar — `calendar_view.py` equivalent.

### Steps:

1. Load all plan files: `ls ~/.daily_planner/data/*-plan.json | sort`

2. For each date in the range, calculate:
   - total tasks, completed, quit, pending
   - intensity level (0-4): 0=no plan, 1=<25% done, 2=<50%, 3=<75%, 4=≥75%

3. Display text-based calendar:
   ```
   Jan  Feb  Mar  Apr ...
   M  □ □ ▪ □ ▪ ▪ ...
   W  □ ▪ ▪ □ □ □ ...
   F  ▪ ▪ □ ▪ ▪ □ ...
   
   Less □ ▪ ▪ ▪ More  ◉ Today
   📊 X tasks completed across Y active days
   🔥 Z-day streak
   ```

4. Options: year view (52 weeks), half-year (26), quarter (13), monthly detail

5. Monthly detail shows table: date, day, ✅ done, 🚫 quit, ⏳ pending, progress bar

6. Also show active goals summary alongside calendar

---

## Workflow 8: Feedback Viewer (`/feedback` or "show feedback")

**Goal:** View and manage tool improvement feedback — `feedback.py` equivalent.

### Steps:

1. Load `tool_feedback.json`
2. Show pending entries in a list with date and preview
3. Options:
   - View details of an entry (shows original feedback + AI understanding)
   - Add new feedback (triggers iterative understanding loop, max 10 pending)
   - Mark as implemented → archive to `improves/archived_feedback.json`
   - Mark as dismissed → archive
   - View archived feedback

---

## Dashboard Stats

When showing any overview, include:
- Today's date and time
- Task progress: `██████░░░░ 6/10 done (60%) 💪`
- Current streak: `🔥 5-day streak!` (bold green if ≥7, yellow if ≥3)
- Recent activity summary from calendar data

**Streak emoji:** 🔥 for ≥3 days, ⭐ for 1-2 days. Colors: bold magenta for ≥30, bold green for ≥7, yellow for ≥3.

---

## AI Prompt Templates

### Plan Generation:
```
System: "You are a helpful planning assistant that creates clear, actionable daily plans with proper hierarchy."

User: "Based on the following job inputs, create a detailed daily plan with checkboxes. Break down each job into specific, actionable tasks. Use markdown format with checkbox syntax (- [ ]). For sub-tasks, use nested indentation to show hierarchy.

## [Job Name]
Description: [description]
What to do: [user_input]
Sub-tasks:
  - Sub-task: [sub_name]
    What to do: [sub_description]

Please create a well-organized daily plan with clear, actionable tasks."
```

### Summary Generation:
```
System: "You are a thoughtful reflection assistant that helps people learn from their daily experiences."

User: "Based on the following plan and review, create a comprehensive daily summary with sections for:
1. Accomplishments
2. Challenges
3. Reflection
4. Recommendations for Tomorrow

## Original Plan:
[for each job: name + user_input]

## Review:
[for each job: name, Status: Yes/No/Partial, Quality: X (if yes), Issue: X (if no/partial)]"
```

### Summary Refinement:
```
System: "You are a thoughtful reflection assistant that refines daily summaries based on user feedback."

User: "Here is the current daily summary:\n\n{summary}\n\nUser feedback: {feedback}\n\nPlease update the summary based on the feedback. Keep the same structure and format."
```

### Feedback Understanding:
```
System: "You are a product manager confirming your understanding of user feedback."

User: "The user wants to improve a daily planning and logging tool. Their feedback:\n\n{feedback}\n\nPlease summarize your understanding of what they want. Start with 'I understand that you want...' and be specific."
```

---

## Notes

- Always `mkdir -p ~/.daily_planner/data` before writing
- Use Python for JSON read/write — never raw string concatenation
- Get today's date from `session_status` or `date +%Y-%m-%d` via exec
- Be conversational — one thing at a time, don't dump all questions at once
- Status icons: ✓ done, ✗ quit, ○ pending, 🔥 streak, 🌱🔥⚡🚀 goal stages
- Progress bar colors: green ≥80%, yellow ≥50%, orange ≥20%, cyan <20%
- Completion % = done/(done+quit+pending) × 100
