---
name: obsidian-templater
description: Write Templater templates for Obsidian with dynamic commands, date manipulation, file operations, and user scripts. Use when the user mentions Templater, tp.date, tp.file, dynamic templates, or automated note creation in Obsidian.
---

# Obsidian Templater Skill

This skill enables skills-compatible agents to create and edit Templater templates for Obsidian, including dynamic commands, date/file manipulation, user interaction, and automation.

## Overview

[Templater](https://github.com/SilentVoid13/Templater) is a community plugin for Obsidian that provides a powerful templating language. It goes beyond Obsidian's core templates by offering:

- JavaScript execution within templates
- Dynamic date calculations and formatting
- File system operations (create, move, rename)
- User prompts and suggesters
- Automatic template triggers on file creation
- Custom user scripts
- Access to the Obsidian API

Templater uses a command syntax with `<% %>` delimiters and provides multiple built-in modules accessible through the `tp` object.

## Command Syntax

### Output Command

Evaluates the expression and inserts the result into the note:

```markdown
<% tp.date.now("YYYY-MM-DD") %>
```

### Execution Command

Runs JavaScript code without inserting output. Use `<%* %>` for statements:

```markdown
<%*
const title = tp.file.title;
const created = tp.file.creation_date("YYYY-MM-DD");
tR += `# ${title}\nCreated: ${created}`;
%>
```

### Whitespace Control

Use `-%>` to trim trailing newlines or `<%-` to trim leading whitespace:

```markdown
<%- tp.file.title -%>
```

This prevents blank lines from appearing where Templater commands are placed.

### The `tR` Variable

`tR` (template result) is a special variable that holds the accumulated output. Use `tR +=` inside `<%* %>` blocks to append content:

```markdown
<%*
if (tp.file.tags.includes("#project")) {
    tR += "## Project Tasks\n- [ ] Define scope\n- [ ] Set milestones";
}
%>
```

## tp.date Module

Date manipulation using [Moment.js](https://momentjs.com/docs/#/displaying/format/) format strings.

### tp.date.now

Returns the current date/time.

```markdown
<% tp.date.now() %>
<% tp.date.now("YYYY-MM-DD") %>
<% tp.date.now("YYYY-MM-DD", 7) %>
<% tp.date.now("YYYY-MM-DD", -30) %>
<% tp.date.now("YYYY-MM-DD", 0, "2025-01-15", "YYYY-MM-DD") %>
```

**Signature:**

```
tp.date.now(format?: string, offset?: number | string, reference?: string, reference_format?: string)
```

| Parameter        | Description                                          | Default      |
| ---------------- | ---------------------------------------------------- | ------------ |
| `format`         | Moment.js format string                              | `YYYY-MM-DD` |
| `offset`         | Days offset (number) or duration string (e.g. "1y")  | `0`          |
| `reference`      | Reference date string instead of now                 | now          |
| `reference_format`| Format of the reference date string                 | `format`     |

### tp.date.tomorrow

```markdown
<% tp.date.tomorrow("YYYY-MM-DD") %>
```

### tp.date.yesterday

```markdown
<% tp.date.yesterday("YYYY-MM-DD") %>
```

### tp.date.weekday

Returns a specific weekday relative to the current or reference date.

```markdown
<% tp.date.weekday("YYYY-MM-DD", 0) %>
<% tp.date.weekday("YYYY-MM-DD", 1) %>
<% tp.date.weekday("YYYY-MM-DD", -1) %>
```

**Signature:**

```
tp.date.weekday(format?: string, weekday?: number, reference?: string, reference_format?: string)
```

| Parameter | Description                                      |
| --------- | ------------------------------------------------ |
| `weekday`  | ISO weekday number: 1=Monday ... 7=Sunday. 0=this week's Monday. Negative for previous weeks. |

### Common Moment.js Formats

| Token  | Output              | Example        |
| ------ | ------------------- | -------------- |
| `YYYY` | 4-digit year        | `2025`         |
| `YY`   | 2-digit year        | `25`           |
| `MM`   | Month (zero-padded) | `01`-`12`      |
| `M`    | Month               | `1`-`12`       |
| `DD`   | Day (zero-padded)   | `01`-`31`      |
| `D`    | Day                 | `1`-`31`       |
| `dddd` | Full day name       | `Monday`       |
| `ddd`  | Short day name      | `Mon`          |
| `dd`   | Min day name        | `Mo`           |
| `HH`   | Hour (24h)          | `00`-`23`      |
| `hh`   | Hour (12h)          | `01`-`12`      |
| `mm`   | Minute              | `00`-`59`      |
| `ss`   | Second              | `00`-`59`      |
| `A`    | AM/PM               | `AM` or `PM`   |
| `MMMM` | Full month name     | `January`      |
| `MMM`  | Short month name    | `Jan`          |
| `X`    | Unix timestamp      | `1700000000`   |
| `Wo`   | Week of year        | `1st`-`53rd`   |

## tp.file Module

File operations and metadata for the current note.

### Properties (Read-Only)

```markdown
<% tp.file.content %>
<% tp.file.title %>
<% tp.file.tags %>
```

| Property   | Type       | Description                                    |
| ---------- | ---------- | ---------------------------------------------- |
| `content`  | `string`   | Full content of the file                       |
| `title`    | `string`   | File name without extension                    |
| `tags`     | `string[]` | Array of all tags in the file (including `#`)  |

### tp.file.creation_date

```markdown
<% tp.file.creation_date("YYYY-MM-DD") %>
<% tp.file.creation_date("YYYY-MM-DD HH:mm") %>
```

### tp.file.last_modified_date

```markdown
<% tp.file.last_modified_date("YYYY-MM-DD HH:mm") %>
```

### tp.file.cursor

Places the cursor at this position after template insertion. Use `order` for multiple cursors (Tab to navigate):

```markdown
<% tp.file.cursor(1) %>
<% tp.file.cursor(2) %>
```

### tp.file.cursor_append

Appends content at the active cursor position:

```markdown
<%* tp.file.cursor_append("text to insert") %>
```

### tp.file.exists

Checks if a file exists in the vault:

```markdown
<%* if (await tp.file.exists("Notes/My Note.md")) { %>
File exists!
<%* } %>
```

### tp.file.find_tfile

Returns the `TFile` object for a given filename:

```markdown
<%*
const tfile = tp.file.find_tfile("Some Note");
if (tfile) {
    tR += `Found: ${tfile.path}`;
}
%>
```

### tp.file.folder

Returns the folder path of the current file:

```markdown
<% tp.file.folder() %>
<% tp.file.folder(true) %>
```

| Parameter  | Description                              |
| ---------- | ---------------------------------------- |
| `relative` | `true` for vault-relative, `false` for full name only |

### tp.file.include

Includes the content of another file or section:

```markdown
<% tp.file.include("[[Template Part]]") %>
<% tp.file.include("[[Note#Section]]") %>
```

### tp.file.move

Moves the current file to a new path:

```markdown
<%* await tp.file.move("Archive/" + tp.file.title) %>
<%* await tp.file.move("Projects/" + tp.file.title, tp.file.find_tfile("Other Note")) %>
```

### tp.file.path

Returns the file path:

```markdown
<% tp.file.path() %>
<% tp.file.path(true) %>
```

### tp.file.rename

Renames the current file:

```markdown
<%* await tp.file.rename("New Title") %>
<%* await tp.file.rename(tp.date.now("YYYY-MM-DD") + " Meeting Notes") %>
```

### tp.file.selection

Returns the currently selected text in the editor:

```markdown
<% tp.file.selection() %>
```

## tp.system Module

System-level operations: clipboard, prompts, and suggesters.

### tp.system.clipboard

Returns the current clipboard content:

```markdown
<% tp.system.clipboard() %>
```

### tp.system.prompt

Displays a prompt dialog for user input:

```markdown
<% await tp.system.prompt("Enter project name") %>
<% await tp.system.prompt("Enter value", "default") %>
<% await tp.system.prompt("Enter notes", "", false, true) %>
```

**Signature:**

```
tp.system.prompt(prompt_text?: string, default_value?: string, throw_on_cancel?: boolean, multiline?: boolean, options?: { limit?: number, placeholder?: string })
```

| Parameter         | Type      | Description                                  |
| ----------------- | --------- | -------------------------------------------- |
| `prompt_text`     | `string`  | Text displayed in the prompt dialog          |
| `default_value`   | `string`  | Pre-filled value                             |
| `throw_on_cancel` | `boolean` | Throw error if user cancels (default: false) |
| `multiline`       | `boolean` | Allow multiline input (default: false)       |
| `options`         | `object`  | `limit`: max chars, `placeholder`: hint text |

### tp.system.suggester

Displays a suggestion popup for the user to select from:

```markdown
<%*
const choice = await tp.system.suggester(
    ["Option A", "Option B", "Option C"],
    ["value_a", "value_b", "value_c"]
);
tR += choice;
%>
```

**Signature:**

```
tp.system.suggester(items: string[] | Function, values: any[], throw_on_cancel?: boolean, placeholder?: string, limit?: number)
```

| Parameter         | Type                 | Description                                 |
| ----------------- | -------------------- | ------------------------------------------- |
| `items`           | `string[]` or `fn`   | Display text for each option (or render function) |
| `values`          | `any[]`              | Corresponding values returned on selection  |
| `throw_on_cancel` | `boolean`            | Throw error if user cancels                 |
| `placeholder`     | `string`             | Placeholder text in the search field        |
| `limit`           | `number`             | Max number of suggestions shown             |

Using a function for display:

```markdown
<%*
const files = app.vault.getMarkdownFiles();
const selected = await tp.system.suggester(
    (file) => file.basename,
    files,
    false,
    "Select a note"
);
if (selected) {
    tR += `[[${selected.basename}]]`;
}
%>
```

## tp.frontmatter Module

Access frontmatter (YAML properties) of the current note.

```markdown
<% tp.frontmatter.title %>
<% tp.frontmatter.tags %>
<% tp.frontmatter.status %>
<% tp.frontmatter["multi-word-key"] %>
```

Note: `tp.frontmatter` reads the properties at the time the template is executed. It returns the raw value (string, array, number, boolean) as defined in the YAML.

## tp.hooks Module

Execute callbacks after all templates in the current file have been processed.

```markdown
<%*
tp.hooks.on_all_templates_executed(async () => {
    const file = tp.file.find_tfile(tp.file.title);
    await app.fileManager.processFrontMatter(file, (fm) => {
        fm.processed = true;
        fm.processed_date = tp.date.now("YYYY-MM-DD");
    });
});
%>
```

This is useful for modifying the file after Templater has finished inserting all content, avoiding conflicts with template processing.

## tp.obsidian Module

Provides access to the Obsidian API modules.

```markdown
<%*
// Normalize a file path
const path = tp.obsidian.normalizePath("My Folder/My Note.md");

// Make an HTTP request
const response = await tp.obsidian.requestUrl("https://api.example.com/data");
const data = response.json;
tR += data.title;
%>
```

Common Obsidian API utilities available:

| Function          | Description                          |
| ----------------- | ------------------------------------ |
| `normalizePath()`  | Normalize file path for the vault   |
| `requestUrl()`     | HTTP request (bypasses CORS)        |
| `moment()`         | Moment.js instance                  |
| `Notice`           | Display a notification              |
| `MarkdownRenderer` | Render markdown programmatically    |

### requestUrl Example

````markdown
<%*
const res = await tp.obsidian.requestUrl({
    url: "https://api.example.com/data",
    method: "GET",
    headers: { "Authorization": "Bearer TOKEN" }
});
tR += res.json.result;
%>
````

## tp.web Module

Web content retrieval utilities.

### tp.web.daily_quote

```markdown
<% tp.web.daily_quote() %>
```

Returns a random daily quote as formatted text.

### tp.web.random_picture

```markdown
<% tp.web.random_picture("200x200") %>
<% tp.web.random_picture("600x400", "nature") %>
<% tp.web.random_picture("800x600", "landscape", true) %>
```

**Signature:**

```
tp.web.random_picture(size?: string, query?: string, include_size?: boolean)
```

## User Scripts

Templater supports custom JavaScript functions defined in a designated folder.

### Setup

1. Create a scripts folder in your vault (e.g., `Templates/scripts/`)
2. In Templater settings, set **User Scripts Folder** to the folder path
3. Create `.js` files in that folder

### Writing a User Script

Each script file should export a single function:

```javascript
// Templates/scripts/greeting.js
function greeting(tp) {
    const hour = new Date().getHours();
    if (hour < 12) return "Good morning";
    if (hour < 18) return "Good afternoon";
    return "Good evening";
}
module.exports = greeting;
```

Usage in template:

```markdown
<% tp.user.greeting(tp) %>
```

### Async User Script

```javascript
// Templates/scripts/fetch_weather.js
async function fetch_weather(tp) {
    const res = await tp.obsidian.requestUrl(
        "https://api.weatherapi.com/v1/current.json?key=KEY&q=Seoul"
    );
    const data = res.json;
    return `${data.current.temp_c}°C, ${data.current.condition.text}`;
}
module.exports = fetch_weather;
```

Usage:

```markdown
<% await tp.user.fetch_weather(tp) %>
```

### User Script with Parameters

```javascript
// Templates/scripts/format_list.js
function format_list(tp, items, ordered) {
    if (ordered) {
        return items.map((item, i) => `${i + 1}. ${item}`).join("\n");
    }
    return items.map((item) => `- ${item}`).join("\n");
}
module.exports = format_list;
```

Usage:

```markdown
<% tp.user.format_list(tp, ["Alpha", "Beta", "Gamma"], true) %>
```

## Dynamic Commands and Automation

### Folder Templates

Automatically apply a template when creating a new note in a specific folder:

1. Go to **Templater Settings > Folder Templates**
2. Add a mapping: Folder path -> Template path
3. New notes created in that folder will auto-apply the template

Example configuration:

| Folder            | Template                          |
| ----------------- | --------------------------------- |
| `Daily Notes`     | `Templates/Daily Note Template`   |
| `Projects`        | `Templates/Project Template`      |
| `Meeting Notes`   | `Templates/Meeting Template`      |

### Startup Templates

Templates that run automatically when Obsidian starts:

1. Go to **Templater Settings > Startup Templates**
2. Add template files to run on startup

Use cases:
- Update a dashboard or index note
- Sync data from external sources
- Run daily maintenance tasks

### Trigger on File Creation

Enable **Trigger Templater on new file creation** in settings to automatically apply templates via folder mappings when files are created by any method (not just the new note button).

## Control Flow

### Conditional (if/else)

```markdown
<%* if (tp.file.title.startsWith("Meeting")) { %>
## Attendees
-
## Agenda
-
<%* } else if (tp.file.title.startsWith("Project")) { %>
## Goals
-
## Timeline
-
<%* } else { %>
## Notes
-
<%* } %>
```

### Ternary Operator

```markdown
Status: <% tp.frontmatter.completed ? "Done" : "In Progress" %>
```

### Loops (for)

```markdown
<%*
const days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"];
for (const day of days) {
    tR += `### ${day}\n- [ ] \n\n`;
}
%>
```

### Iterating Over Files

```markdown
<%*
const folder = app.vault.getAbstractFileByPath("Projects");
if (folder) {
    const files = folder.children.filter(f => f.extension === "md");
    for (const file of files) {
        tR += `- [[${file.basename}]]\n`;
    }
}
%>
```

### Try/Catch

```markdown
<%*
try {
    const result = await tp.system.prompt("Enter a number");
    const num = parseInt(result);
    if (isNaN(num)) throw new Error("Not a number");
    tR += `Result: ${num * 2}`;
} catch (e) {
    tR += "Invalid input, using default: 0";
}
%>
```

## Advanced Patterns

### Dynamic File Naming

```markdown
<%*
const category = await tp.system.suggester(
    ["Work", "Personal", "Study"],
    ["work", "personal", "study"]
);
const title = await tp.system.prompt("Note title");
await tp.file.rename(`${category}/${tp.date.now("YYYY-MM-DD")} ${title}`);
%>
```

### Creating Notes from Template

````markdown
<%*
const projects = ["Alpha", "Beta", "Gamma"];
for (const project of projects) {
    const exists = await tp.file.exists(`Projects/${project}.md`);
    if (!exists) {
        const template = tp.file.find_tfile("Project Template");
        await tp.file.create_new(template, project, false, "Projects");
    }
}
%>
````

### Frontmatter Manipulation with tp.hooks

````markdown
<%*
tp.hooks.on_all_templates_executed(async () => {
    const file = tp.file.find_tfile(tp.file.title);
    await app.fileManager.processFrontMatter(file, (fm) => {
        fm.title = tp.file.title;
        fm.created = tp.date.now("YYYY-MM-DD");
        fm.tags = fm.tags || [];
        if (!fm.tags.includes("auto-generated")) {
            fm.tags.push("auto-generated");
        }
    });
});
%>
````

### Weekly Note with Date Calculations

```markdown
<%*
const weekStart = tp.date.weekday("YYYY-MM-DD", 1);
const weekEnd = tp.date.weekday("YYYY-MM-DD", 7);
%>
# Week of <% weekStart %> to <% weekEnd %>

## Goals
- [ ] <% tp.file.cursor(1) %>

## Daily Log
<%*
const days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"];
for (let i = 1; i <= 5; i++) {
    const date = tp.date.weekday("YYYY-MM-DD", i);
    tR += `### ${days[i-1]} (${date})\n- \n\n`;
}
%>

## Weekly Review
- What went well:
- What to improve:
- Next week focus:
```

## Complete Example

A comprehensive daily note template demonstrating multiple Templater features:

````markdown
---
title: <% tp.date.now("YYYY-MM-DD dddd") %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags:
  - daily-note
day: <% tp.date.now("dddd") %>
week: <% tp.date.now("Wo") %>
---

# <% tp.date.now("YYYY-MM-DD dddd") %>

## Navigation

<< [[<% tp.date.yesterday("YYYY-MM-DD dddd") %>|Yesterday]] | [[<% tp.date.tomorrow("YYYY-MM-DD dddd") %>|Tomorrow]] >>

## Morning Planning

### Today's Focus
- [ ] <% tp.file.cursor(1) %>

### Schedule
<%*
const day = tp.date.now("dddd");
if (day === "Monday") {
    tR += "- 09:00 Weekly team standup\n";
    tR += "- 10:00 Sprint planning\n";
} else if (day === "Friday") {
    tR += "- 15:00 Weekly review\n";
    tR += "- 16:00 Retrospective\n";
}
%>

## Notes

<% tp.file.cursor(2) %>

## Tasks Completed
-

## End of Day Review

**Energy Level:** <% tp.file.cursor(3) %>/10
**Mood:**
**Key Learning:**

---
*Created at <% tp.date.now("HH:mm") %>*
````

## Templater Settings Reference

| Setting                          | Description                                        |
| -------------------------------- | -------------------------------------------------- |
| Template Folder Location         | Path to your templates folder                      |
| Timeout                          | Max execution time for templates (default: 5s)     |
| Trigger on new file creation     | Auto-apply folder templates on file creation       |
| Folder Templates                 | Map folders to auto-applied templates              |
| Startup Templates                | Templates executed on Obsidian startup             |
| User Scripts Folder              | Path to custom `.js` scripts folder                |
| Enable System Commands           | Allow `tp.system.command()` (disabled by default)  |

## Common Pitfalls

1. **Async functions**: Always use `await` with async methods (`tp.system.prompt`, `tp.file.move`, `tp.file.rename`, `tp.file.exists`).
2. **Whitespace**: Templater commands leave blank lines. Use `-%>` trimming or `<%* %>` execution blocks.
3. **Frontmatter conflicts**: Modifying frontmatter during template execution can cause issues. Use `tp.hooks.on_all_templates_executed()` instead.
4. **File references**: `tp.file.include()` requires wikilink syntax: `"[[Note Name]]"`.
5. **tR scope**: `tR` is only available inside `<%* %>` blocks, not in `<% %>` output commands.
6. **Template folder**: Templates in the designated template folder are excluded from search and graph by default.

## References

- [Templater Documentation](https://silentvoid13.github.io/Templater/)
- [Templater GitHub Repository](https://github.com/SilentVoid13/Templater)
- [Moment.js Format Reference](https://momentjs.com/docs/#/displaying/format/)
- [Obsidian API Documentation](https://docs.obsidian.md/Reference/TypeScript+API)
- [Templater FAQ](https://silentvoid13.github.io/Templater/faq.html)
