---
name: obsidian-dataview
description: Write Dataview queries (DQL and DataviewJS) for Obsidian. Use when the user mentions Dataview, DQL, DataviewJS, dynamic queries, TABLE, LIST, TASK queries, or data-driven views in Obsidian notes.
---

# Obsidian Dataview Skill

This skill enables skills-compatible agents to write valid Dataview queries for the Obsidian Dataview plugin, including DQL (Dataview Query Language), DataviewJS, and inline queries.

## Overview

[Dataview](https://blacksmithgu.github.io/obsidian-dataview/) is a community plugin for Obsidian that treats your vault as a database. It allows you to query, filter, sort, and display data from your notes using:

- **DQL (Dataview Query Language)** - SQL-like declarative queries
- **DataviewJS** - Full JavaScript API for advanced logic
- **Inline Queries** - Embed dynamic values within text

Dataview reads metadata from YAML frontmatter (properties), inline fields (`Key:: Value`), and implicit file metadata (`file.name`, `file.ctime`, etc.). All queries render dynamically and update automatically when notes change.

## DQL (Dataview Query Language)

DQL queries are written inside `dataview` code blocks. Every query starts with a **query type**, optionally followed by **FROM**, **WHERE**, **SORT**, **GROUP BY**, **LIMIT**, and **FLATTEN** clauses.

### LIST Queries

````markdown
```dataview
LIST
FROM #project
```

```dataview
LIST status
FROM "20. Projects"
WHERE status != "done"
```

```dataview
LIST WITHOUT ID file.name + " (" + status + ")"
FROM #project
```
````

### TABLE Queries

````markdown
```dataview
TABLE status, priority, due
FROM #task
WHERE !completed
SORT due ASC
```

```dataview
TABLE
  status AS "Status",
  dateformat(due, "yyyy-MM-dd") AS "Due Date",
  file.tags AS "Tags"
FROM "20. Projects"
SORT due ASC
```

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  file.size AS "Size (bytes)",
  file.mday AS "Last Modified"
FROM "30. AI & Tech"
SORT file.mday DESC
```
````

### TASK Queries

````markdown
```dataview
TASK
FROM "20. Projects"
WHERE !completed
SORT file.name ASC
```

```dataview
TASK
FROM #work
WHERE !completed AND contains(text, "urgent")
GROUP BY file.link
```
````

### CALENDAR Queries

````markdown
```dataview
CALENDAR date
FROM "10. Daily Notes"
```

```dataview
CALENDAR file.cday
FROM #meeting
```
````

## Data Sources (FROM)

The `FROM` clause specifies which notes to include. Omit to query all notes.

````markdown
```dataview
LIST FROM #project
LIST FROM #project/active
LIST FROM "20. Projects"
LIST FROM "20. Projects/Work"
LIST FROM [[Project Alpha]]
LIST FROM outgoing([[Project Alpha]])
```
````

Combining sources:

````markdown
```dataview
LIST FROM #project AND "20. Projects"
LIST FROM #project OR #task
LIST FROM -#archive
LIST FROM #project AND -"20. Projects/Archive"
```
````

## WHERE (Filtering)

````markdown
```dataview
TABLE status, priority
FROM #project
WHERE status = "active" AND priority >= 3
```

```dataview
LIST FROM "10. Daily Notes"
WHERE file.mday >= date(today) - dur(7 days)
```

```dataview
LIST FROM ""
WHERE contains(file.name, "Meeting")
```

```dataview
LIST FROM ""
WHERE regexmatch("^2024-\d{2}", file.name)
```

```dataview
LIST FROM #project
WHERE due
```

```dataview
LIST FROM ""
WHERE contains(file.tags, "#important")
```
````

## SORT, GROUP BY, LIMIT, FLATTEN

### SORT

````markdown
```dataview
TABLE priority, status, due
FROM #task
SORT priority DESC, due ASC
```
````

### GROUP BY

After grouping, use `rows` to access items within each group.

````markdown
```dataview
TABLE rows.file.link AS "Notes", length(rows) AS "Count"
FROM #project
GROUP BY status
```

```dataview
TABLE rows.file.link AS "Notes"
FROM "10. Daily Notes"
GROUP BY dateformat(file.cday, "yyyy-MM") AS "Month"
```
````

### LIMIT

````markdown
```dataview
TABLE file.mday AS "Modified"
FROM ""
SORT file.mday DESC
LIMIT 10
```
````

### FLATTEN

FLATTEN expands an array field so each element becomes its own row.

````markdown
```dataview
TABLE tag, file.link
FROM ""
FLATTEN file.tags AS tag
WHERE tag != "#daily"
GROUP BY tag
```
````

## Implicit Fields (File Metadata)

| Field              | Type     | Description                          |
| ------------------ | -------- | ------------------------------------ |
| `file.name`        | Text     | File name without extension          |
| `file.path`        | Text     | Full file path                       |
| `file.folder`      | Text     | Folder containing the file           |
| `file.link`        | Link     | Link to the file                     |
| `file.size`        | Number   | File size in bytes                   |
| `file.ctime`       | DateTime | Creation time                        |
| `file.cday`        | Date     | Creation date                        |
| `file.mtime`       | DateTime | Last modification time               |
| `file.mday`        | Date     | Last modification date               |
| `file.tags`        | List     | All tags (including subtags)         |
| `file.etags`       | List     | Explicit tags only                   |
| `file.aliases`     | List     | Aliases from frontmatter             |
| `file.tasks`       | List     | All tasks in the note                |
| `file.lists`       | List     | All list items in the note           |
| `file.frontmatter` | Object   | Raw frontmatter as object            |
| `file.day`         | Date     | Date from filename (if parsable)     |
| `file.inlinks`     | List     | Incoming links to this note          |
| `file.outlinks`    | List     | Outgoing links from this note        |

### Task Fields

| Field       | Type    | Description                     |
| ----------- | ------- | ------------------------------- |
| `text`      | Text    | Task text content               |
| `completed` | Boolean | Whether the task is checked     |
| `status`    | Text    | Status character (space, x)     |
| `line`      | Number  | Line number in the source file  |
| `section`   | Link    | Heading the task is under       |
| `tags`      | List    | Tags within the task text       |
| `subtasks`  | List    | Nested subtasks                 |

## Inline Fields

Inline fields add metadata directly within note content, outside of frontmatter.

```markdown
Status:: Active
Priority:: High
Due Date:: 2024-03-15

This task is [status:: in-progress] and assigned to [assignee:: Alice].

This project started on (start-date:: 2024-01-01) and is going well.
```

- `Key:: Value` on its own line -- full-line field
- `[Key:: Value]` inline -- visible in reading view
- `(Key:: Value)` inline -- hidden in reading view (only value shown)

All inline fields are accessed the same way as frontmatter fields in queries.

## Functions

### String Functions

| Function                           | Description                        |
| ---------------------------------- | ---------------------------------- |
| `contains(str, substr)`            | Check if contains substring        |
| `startswith(str, prefix)`          | Check prefix                       |
| `endswith(str, suffix)`            | Check suffix                       |
| `replace(str, old, new)`          | Replace substring                  |
| `regexmatch(pattern, str)`         | Regex match test                   |
| `regexreplace(str, pattern, repl)` | Regex replace                      |
| `length(str)`                      | String/list length                 |
| `lower(str)` / `upper(str)`       | Case conversion                    |
| `split(str, delim)`               | Split into list                    |
| `trim(str)`                        | Remove whitespace                  |
| `substring(str, start, end)`       | Extract substring                  |

### Date & Duration Functions

| Function                   | Description                              |
| -------------------------- | ---------------------------------------- |
| `date(text)`               | Parse date (`date("2024-01-15")`)        |
| `date(today)`              | Today's date                             |
| `date(now)`                | Current date and time                    |
| `date(tomorrow)`           | Tomorrow's date                          |
| `dur(text)`                | Parse duration (`dur("3 days")`)         |
| `dateformat(date, fmt)`    | Format date (`"yyyy-MM-dd"`, `"HH:mm"`) |
| `striptime(date)`          | Remove time component                    |

Common `dateformat` patterns: `yyyy` (2024), `MM` (01), `dd` (15), `EEE` (Mon), `EEEE` (Monday), `HH:mm` (14:30).

### List/Array Functions

| Function                     | Description                  |
| ---------------------------- | ---------------------------- |
| `length(list)`               | Number of elements           |
| `contains(list, item)`      | Check membership             |
| `filter(list, (x) => ...)`  | Filter by predicate          |
| `map(list, (x) => ...)`     | Transform elements           |
| `flat(list)`                 | Flatten nested lists         |
| `sort(list)`                 | Sort ascending               |
| `reverse(list)`              | Reverse order                |
| `sum(list)` / `average(list)` | Numeric aggregation        |
| `min(list)` / `max(list)`   | Min/max value                |
| `join(list, sep)`            | Join into string             |
| `any(list, (x) => ...)`     | True if any match            |
| `all(list, (x) => ...)`     | True if all match            |
| `nonnull(list)`              | Remove null values           |

### Utility Functions

| Function                          | Description                     |
| --------------------------------- | ------------------------------- |
| `default(value, fallback)`        | Use fallback if null            |
| `choice(cond, trueVal, falseVal)` | Ternary/conditional value       |
| `link(path, display)`             | Create a link                   |
| `elink(url, display)`             | Create an external link         |
| `embed(link)`                     | Embed a link                    |
| `string(value)` / `number(str)`   | Type conversion                |
| `round(num, dec)`                 | Round to decimal places         |
| `typeof(value)`                   | Get type name                   |

### Function Examples

````markdown
```dataview
TABLE
  default(status, "N/A") AS "Status",
  choice(priority > 3, "High", "Normal") AS "Level",
  dateformat(file.cday, "yyyy-MM-dd EEE") AS "Created",
  length(file.outlinks) AS "Links"
FROM #project
SORT priority DESC
```
````

## DataviewJS

DataviewJS provides JavaScript for complex queries. Written inside `dataviewjs` code blocks with the `dv` API object.

### Querying and Rendering

````markdown
```dataviewjs
// Query pages
const pages = dv.pages("#project")
  .where(p => p.status === "active")
  .sort(p => p.priority, "desc");

// Render table
dv.table(
  ["Project", "Status", "Priority", "Due"],
  pages.map(p => [p.file.link, p.status, p.priority, p.due])
);
```

```dataviewjs
// Render list
const recent = dv.pages("")
  .sort(p => p.file.mtime, "desc")
  .limit(10);
dv.list(recent.map(p => p.file.link));
```

```dataviewjs
// Render task list
const tasks = dv.pages("#work").file.tasks
  .where(t => !t.completed);
dv.taskList(tasks, false); // false = don't group by file
```

```dataviewjs
// Text output
dv.header(2, "Summary");
dv.paragraph(`Active: **${dv.pages("#project").where(p => p.status === "active").length}**`);
```

```dataviewjs
// Current page
const cur = dv.current();
dv.paragraph(`Tags: ${cur.file.tags.join(", ")}`);
```
````

### Advanced Patterns

````markdown
```dataviewjs
// Conditional rendering
const overdue = dv.pages("#task")
  .where(p => p.due && p.due < dv.date("today") && !p.completed);

if (overdue.length > 0) {
  dv.header(3, "Overdue Tasks");
  dv.table(["Task", "Due"], overdue.map(p => [p.file.link, p.due]));
} else {
  dv.paragraph("No overdue tasks.");
}
```

```dataviewjs
// Aggregation
const projects = dv.pages("#project");
const statuses = projects.groupBy(p => p.status);

dv.table(
  ["Status", "Count", "Projects"],
  statuses.map(g => [
    g.key || "No Status",
    g.rows.length,
    g.rows.map(p => p.file.link).join(", ")
  ])
);
```
````

### DataviewJS API Reference

| Method                          | Description                    |
| ------------------------------- | ------------------------------ |
| `dv.pages(source)`              | Query pages from source        |
| `dv.page(path)`                 | Get a single page by path      |
| `dv.current()`                  | Get the current page           |
| `dv.table(headers, rows)`       | Render a table                 |
| `dv.list(items)`                | Render a bullet list           |
| `dv.taskList(tasks, grouped)`   | Render interactive tasks       |
| `dv.paragraph(text)`            | Render a paragraph             |
| `dv.header(level, text)`        | Render a heading (1-6)         |
| `dv.el(tag, text, attrs)`       | Render custom HTML element     |
| `dv.date(str)`                  | Parse a date                   |
| `dv.io.load(path)`              | Load file content as string    |
| `dv.io.csv(path)`               | Load and parse a CSV file      |

### Page Array Methods

| Method                | Description               |
| --------------------- | ------------------------- |
| `.where(predicate)`   | Filter pages              |
| `.sort(field, order)` | Sort (`"asc"` / `"desc"`) |
| `.limit(n)`           | Take first N results      |
| `.map(fn)`            | Transform each page       |
| `.flatMap(fn)`        | Map and flatten           |
| `.groupBy(fn)`        | Group by field/function   |
| `.distinct()`         | Remove duplicates         |
| `.length`             | Number of results         |
| `.array()`            | Convert to plain array    |

## Inline Queries

### DQL Inline

Use single backticks with `=` prefix. `this` refers to the current note.

```markdown
Today is `= date(today)`.
This note: `= this.file.name`
Status: `= default(this.status, "Not set")`
Due: `= dateformat(this.due, "yyyy-MM-dd")`
Tags: `= length(this.file.tags)` tags on this note.
```

### DataviewJS Inline

Use single backticks with `$=` prefix:

```markdown
Total projects: `$= dv.pages("#project").length`
Active: `$= dv.pages("#project").where(p => p.status === "active").length`
```

## Complete Example

`````markdown
---
title: Project Dashboard
tags:
  - dashboard
cssclasses:
  - dashboard
---

# Project Dashboard

> [!summary] Overview
> Active: `$= dv.pages("#project").where(p => p.status === "active").length`
> Completed: `$= dv.pages("#project").where(p => p.status === "done").length`

## Active Projects

```dataview
TABLE WITHOUT ID
  file.link AS "Project",
  status AS "Status",
  priority AS "Priority",
  dateformat(due, "yyyy-MM-dd") AS "Due Date",
  length(file.outlinks) AS "Links"
FROM #project
WHERE status = "active"
SORT priority DESC, due ASC
```

## Recent Activity

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  dateformat(file.mday, "yyyy-MM-dd HH:mm") AS "Modified",
  file.folder AS "Folder"
FROM "20. Projects"
SORT file.mday DESC
LIMIT 10
```

## Tasks by Status

```dataviewjs
const projects = dv.pages("#project");
const statuses = projects.groupBy(p => p.status);

dv.table(
  ["Status", "Count", "Projects"],
  statuses.map(g => [
    g.key || "No Status",
    g.rows.length,
    g.rows.map(p => p.file.link).join(", ")
  ])
);
```

## Monthly Archive

```dataview
TABLE rows.file.link AS "Notes", length(rows) AS "Count"
FROM "10. Daily Notes"
GROUP BY dateformat(file.cday, "yyyy-MM") AS "Month"
SORT rows.file.cday DESC
```
`````

## Common Patterns

````markdown
```dataview
LIST FROM ""
WHERE file.mday = date(today)
SORT file.mtime DESC
```

```dataview
LIST FROM ""
WHERE length(file.inlinks) = 0
SORT file.name ASC
```

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  (date(today) - file.cday).days + " days ago" AS "Age"
FROM ""
SORT file.cday ASC
LIMIT 10
```
````

## References

- [Dataview Documentation](https://blacksmithgu.github.io/obsidian-dataview/)
- [DQL Reference](https://blacksmithgu.github.io/obsidian-dataview/queries/dql/)
- [DataviewJS Reference](https://blacksmithgu.github.io/obsidian-dataview/api/code-reference/)
- [Functions Reference](https://blacksmithgu.github.io/obsidian-dataview/reference/functions/)
- [Metadata / Fields Reference](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-pages/)
- [Examples and Resources](https://blacksmithgu.github.io/obsidian-dataview/resources/examples/)
