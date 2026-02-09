---
name: obsidian-tasks
description: Write and query tasks using the Obsidian Tasks plugin with due dates, priorities, recurrence, dependencies, and powerful filters. Use when the user mentions Tasks plugin, task queries, due dates, priorities, recurring tasks, or task management in Obsidian.
---

# Obsidian Tasks Plugin Skill

This skill enables skills-compatible agents to create, manage, and query tasks using the [Obsidian Tasks](https://publish.obsidian.md/tasks/) plugin, a powerful task management system built on top of Obsidian's native checkbox syntax.

## Overview

The Tasks plugin extends Obsidian's basic checkbox tasks with:

- Due dates, scheduled dates, start dates, and other date types
- Priority levels from highest to lowest
- Recurring/repeating tasks
- Task dependencies with blocking relationships
- A powerful query language for filtering, sorting, and grouping tasks
- Custom statuses beyond simple done/not done

Tasks are written as standard Markdown checkboxes with emoji-based annotations for metadata. Queries are written in dedicated `tasks` code blocks.

## Basic Task Syntax

### Task Statuses

```markdown
- [ ] Todo (open task)
- [x] Completed task
- [/] In progress task
- [-] Cancelled task
```

| Marker | Name        | Type          | Next Status |
| ------ | ----------- | ------------- | ----------- |
| `[ ]`  | Todo        | `TODO`        | `[x]` Done  |
| `[x]`  | Done        | `DONE`        | `[ ]` Todo  |
| `[/]`  | In Progress | `IN_PROGRESS` | `[x]` Done  |
| `[-]`  | Cancelled   | `CANCELLED`   | `[ ]` Todo  |

Additional custom statuses can be configured in plugin settings (e.g., `[>]` Forwarded, `[?]` Question).

### Task Anatomy

A complete task with all metadata fields:

```markdown
- [ ] Task description 🆔 abc123 ⛔ xyz789 🔺 🔁 every week ➕ 2024-01-01 🛫 2024-01-05 ⏳ 2024-01-10 📅 2024-01-15
```

The order of emoji fields does not matter. The description always comes first, after the checkbox marker.

## Dates

Six date types, each indicated by a specific emoji. All dates use `YYYY-MM-DD` format.

| Emoji | Name      | Meaning                                         |
| ----- | --------- | ------------------------------------------------ |
| `➕`  | Created   | When the task was created                        |
| `🛫`  | Start     | When you can start working on the task           |
| `⏳`  | Scheduled | When you plan to work on the task                |
| `📅`  | Due       | The deadline for the task                        |
| `✅`  | Done      | When the task was completed (set automatically)  |
| `❌`  | Cancelled | When the task was cancelled (set automatically)  |

### Date Examples

```markdown
- [ ] Write project proposal 📅 2024-03-15
- [ ] Start research phase 🛫 2024-02-01 📅 2024-02-28
- [ ] Review meeting notes ⏳ 2024-01-20
- [ ] Plan team offsite ➕ 2024-01-10 🛫 2024-02-01 ⏳ 2024-02-15 📅 2024-03-01
- [x] Submit final report 📅 2024-01-15 ✅ 2024-01-14
- [-] Deprecated feature work 📅 2024-01-20 ❌ 2024-01-18
```

### Date Behavior

- **Start date**: Tasks before their start date can be hidden using `starts before today`.
- **Scheduled date**: When you intend to work on the task. Useful for planning.
- **Due date**: The deadline. Overdue tasks (past due, not done) are highlighted.
- **Done/Cancelled date**: Automatically set when task status changes (if enabled).
- **Created date**: Can be automatically set when a task is created (if enabled).

## Priorities

Six priority levels indicated by emoji, placed after the description:

| Emoji  | Name    |
| ------ | ------- |
| `🔺`  | Highest |
| `⏫`  | High    |
| `🔼`  | Medium  |
| (none) | Normal  |
| `🔽`  | Low     |
| `⏬`  | Lowest  |

```markdown
- [ ] Critical security patch 🔺 📅 2024-01-15
- [ ] Fix login bug ⏫ 📅 2024-01-20
- [ ] Update documentation 🔼 📅 2024-02-01
- [ ] Routine maintenance 📅 2024-02-15
- [ ] Nice-to-have feature 🔽 📅 2024-03-01
- [ ] Someday/maybe idea ⏬
```

Sort order: Highest > High > Medium > Normal (none) > Low > Lowest

## Recurrence

Recurring tasks automatically create a new copy when completed. Specified with `🔁` emoji.

### Recurrence Patterns

```markdown
- [ ] Daily standup 🔁 every day 📅 2024-01-15
- [ ] Weekly review 🔁 every week 📅 2024-01-19
- [ ] Monthly report 🔁 every month 📅 2024-01-31
- [ ] Annual review 🔁 every year 📅 2024-12-31
- [ ] Biweekly sprint 🔁 every 2 weeks 📅 2024-01-15
- [ ] Take medication 🔁 every 3 days 📅 2024-01-15
- [ ] Team sync 🔁 every Monday 📅 2024-01-15
- [ ] Gym workout 🔁 every Monday, Wednesday, Friday 📅 2024-01-15
- [ ] Weekday tasks 🔁 every weekday 📅 2024-01-15
- [ ] Pay rent 🔁 every month on the 1st 📅 2024-02-01
- [ ] Last day summary 🔁 every month on the last day 📅 2024-01-31
```

### "when done" Modifier

By default, the next date is calculated from the **original** due date. Adding `when done` calculates from the **completion date**:

```markdown
- [ ] Water plants 🔁 every week when done 📅 2024-01-15
```

- `🔁 every week 📅 2024-01-15` — Completed Jan 20 → next due Jan 22 (original + 1 week).
- `🔁 every week when done 📅 2024-01-15` — Completed Jan 20 → next due Jan 27 (completion + 1 week).

### Recurrence Behavior

When a recurring task is completed:
1. Current task is marked done with `✅` date.
2. New task created with next occurrence date, status `[ ]`, all metadata preserved.
3. New task placed above or below the completed task (configurable).

## Dependencies

Define relationships between tasks indicating which must complete before others.

### ID and Blocking

```markdown
- [ ] Design database schema 🆔 db-schema 📅 2024-01-20
- [ ] Implement API endpoints ⛔ db-schema 📅 2024-02-01
- [ ] Write integration tests ⛔ db-schema 📅 2024-02-05
```

- `🆔 id` — Assigns a unique ID to a task.
- `⛔ id` — This task depends on (is blocked by) the task with that ID.

### Multiple Dependencies

Separate multiple IDs with commas:

```markdown
- [ ] Design frontend 🆔 frontend 📅 2024-01-25
- [ ] Design backend 🆔 backend 📅 2024-01-25
- [ ] Integration testing ⛔ frontend,backend 📅 2024-02-10
```

### Dependency Chain

```markdown
- [ ] Research requirements 🆔 req 📅 2024-01-15
- [ ] Write specification ⛔ req 🆔 spec 📅 2024-01-22
- [ ] Implement feature ⛔ spec 🆔 impl 📅 2024-02-05
- [ ] Write tests ⛔ impl 🆔 test 📅 2024-02-12
- [ ] Deploy to production ⛔ test 📅 2024-02-19
```

Dependencies work across different files in the vault. Tasks can be both blocking and blocked simultaneously.

## Task Query Block

Queries are written in `tasks` code blocks with filters, sorting, grouping, and display options.

````markdown
```tasks
not done
due before tomorrow
path includes Projects
sort by due
limit 10
```
````

Each line is one instruction. Lines starting with `#` are comments.

## Filters

Multiple filters combine with AND logic by default.

### Status Filters

```
done
not done
status.name includes In Progress
status.type is TODO
status.type is DONE
status.type is IN_PROGRESS
status.type is CANCELLED
```

### Date Filters

All date types support the same pattern. Examples for due date:

```
has due date
no due date
due before YYYY-MM-DD
due before today
due before tomorrow
due after YYYY-MM-DD
due after yesterday
due on YYYY-MM-DD
due today
due this week
due next week
due last week
due in YYYY-MM
due before in 2 weeks
due after in 3 days
```

Replace `due` with `starts`, `scheduled`, `created`, `done`, or `cancelled` for other date types:

```
has start date
starts before today
has scheduled date
scheduled this week
has created date
created before 2024-01-01
has done date
done this week
```

### Text Filters

```
description includes meeting
description does not include draft
description regex matches /^(Bug|Fix):/i
heading includes Sprint 23
heading does not include Archive
```

### Path and File Filters

```
path includes Projects
path does not include Archive
path regex matches /Projects\/(Active|Pending)/
filename includes 2024
root includes Projects
folder includes Meetings
```

### Tag Filters

```
tags include #project
tags do not include #archive
tags include #work/urgent
tag regex matches /#(bug|fix)/i
has tags
no tags
```

### Priority Filters

```
priority is highest
priority is high
priority is medium
priority is normal
priority is low
priority is lowest
priority above normal
priority below high
priority is not none
```

### Recurrence Filters

```
is recurring
is not recurring
recurrence includes weekly
```

### Dependency Filters

```
is blocked
is not blocked
is blocking
is not blocking
has id
no id
id includes abc
```

### Boolean Combinations

Combine filters with AND, OR, NOT, and parentheses:

```
(due before tomorrow) AND (priority is high)
(path includes Work) OR (path includes Projects)
NOT (tags include #someday)
(priority above normal) AND NOT (path includes Archive)
((due today) OR (due tomorrow)) AND (not done)
```

### Explain

Add `explain` to any query to see how filters are interpreted:

````markdown
```tasks
not done
due this week
explain
```
````

## Sorting

Multiple sort lines are applied in order (first is primary, second is tiebreaker).

### Sort Options

```
sort by due
sort by priority
sort by description
sort by path
sort by filename
sort by start
sort by scheduled
sort by created
sort by done
sort by status
sort by status.name
sort by status.type
sort by urgency
sort by tag
sort by heading
sort by recurring
```

Add `reverse` for descending order:

```
sort by due reverse
sort by priority reverse
```

## Grouping

Organize tasks into sections with headers.

### Group Options

```
group by due
group by priority
group by status
group by status.name
group by status.type
group by path
group by folder
group by root
group by filename
group by heading
group by tags
group by recurring
group by recurrence
group by start
group by scheduled
group by created
group by done
group by happens
```

### Custom Groups with Functions

```
group by function task.due.format("YYYY-MM")
group by function task.due.format("YYYY [Week] WW")
group by function task.tags.join(", ")
group by function task.due.category.groupText
group by function task.priorityName
```

`task.due.category.groupText` produces: Overdue, Today, Tomorrow, This week, Next week, Future, Undated.

## Display Options

### Limits

```
limit 10
limit groups 3
```

### Hide/Show Fields

```
hide due date
hide done date
hide start date
hide scheduled date
hide created date
hide cancelled date
hide priority
hide recurrence rule
hide task count
hide backlink
hide urgency
hide edit button
hide postpone button
show urgency
show tree
short mode
```

## Complete Examples

### Today's Focus

````markdown
```tasks
not done
(due today) OR (scheduled today) OR (starts today)
sort by priority
sort by due
hide backlink
```
````

### Overdue Tasks

````markdown
```tasks
not done
due before today
sort by due
sort by priority
```
````

### High Priority Upcoming

````markdown
```tasks
not done
priority above normal
due before in 2 weeks
sort by priority
sort by due
limit 20
```
````

### Weekly Review (Completed)

````markdown
```tasks
done
done this week
group by done
sort by path
```
````

### Unblocked Actionable Tasks

````markdown
```tasks
not done
is not blocked
has due date
starts before tomorrow
sort by priority
sort by due
limit 15
```
````

### Tasks by Tag with Grouping

````markdown
```tasks
not done
tags include #work
group by tags
sort by due
hide backlink
limit 30
```
````

### Complete Project Management Example

`````markdown
---
title: Q1 Product Launch
tags:
  - project
  - active
status: in-progress
---

# Q1 Product Launch

## Phase 1: Planning

- [x] Define product requirements 🆔 req ⏫ ➕ 2024-01-02 📅 2024-01-10 ✅ 2024-01-09
- [x] Create project timeline ⛔ req 🆔 timeline 🔼 📅 2024-01-12 ✅ 2024-01-11
- [ ] Allocate team resources ⛔ timeline 🆔 resources 🔼 📅 2024-01-15

## Phase 2: Development

- [ ] Backend API development ⛔ resources 🆔 api ⏫ 🛫 2024-01-16 📅 2024-02-15
- [ ] Frontend implementation ⛔ resources 🆔 ui ⏫ 🛫 2024-01-16 📅 2024-02-20
- [ ] Database migration ⛔ api 🔺 📅 2024-02-10
- [ ] Write unit tests ⛔ api,ui 🔼 📅 2024-02-25

## Phase 3: Launch

- [ ] QA testing ⛔ api,ui 🆔 qa ⏫ ⏳ 2024-02-26 📅 2024-03-05
- [ ] Performance optimization ⛔ qa 🔼 📅 2024-03-08
- [ ] Write documentation ⛔ qa 🔽 📅 2024-03-10
- [ ] Deploy to production ⛔ qa 🔺 📅 2024-03-15

## Recurring Tasks

- [ ] Weekly standup notes 🔁 every week 📅 2024-01-19
- [ ] Sprint retrospective 🔁 every 2 weeks 📅 2024-01-26
- [ ] Monthly progress report 🔁 every month on the last day 📅 2024-01-31

## Active Tasks

```tasks
not done
path includes Q1 Product Launch
is not blocked
sort by priority
sort by due
```

## Blocked Tasks

```tasks
not done
path includes Q1 Product Launch
is blocked
sort by due
```

## Completed This Week

```tasks
done
path includes Q1 Product Launch
done this week
sort by done reverse
```
`````

## Urgency

The Tasks plugin calculates an urgency score automatically:

| Factor              | Score Impact |
| ------------------- | ------------ |
| Overdue / Due today | +12.0        |
| Due in next 7 days  | +8.8 to +2.0 |
| Priority: Highest   | +9.0         |
| Priority: High      | +6.0         |
| Priority: Medium    | +3.9         |
| Priority: Normal    | +1.95        |
| Priority: Low       | -1.8         |
| Priority: Lowest    | -3.6         |
| Has start date      | +4.0         |
| Scheduled date past | +5.0         |

````markdown
```tasks
not done
sort by urgency reverse
limit 10
```
````

## Tips and Best Practices

- Tasks can be placed anywhere in any Markdown file, including nested bullet points.
- Tasks under headings inherit the heading for `heading includes` filters.
- Define a **Global Query** in plugin settings to apply default filters to all queries.
- Use **Tasks plugin** for task-specific queries; use **Dataview** for note-level queries. They coexist without conflicts.
- Use `short mode` in daily note templates for compact task views.

## References

- [Tasks Plugin Documentation](https://publish.obsidian.md/tasks/)
- [Filters Reference](https://publish.obsidian.md/tasks/Queries/Filters)
- [Sorting Reference](https://publish.obsidian.md/tasks/Queries/Sorting)
- [Grouping Reference](https://publish.obsidian.md/tasks/Queries/Grouping)
- [Date Formats](https://publish.obsidian.md/tasks/Getting+Started/Dates)
- [Priority Reference](https://publish.obsidian.md/tasks/Getting+Started/Priority)
- [Recurring Tasks](https://publish.obsidian.md/tasks/Getting+Started/Recurring+Tasks)
- [Task Dependencies](https://publish.obsidian.md/tasks/Getting+Started/Task+Dependencies)
