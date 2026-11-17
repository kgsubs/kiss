# Build a "kiss" question-session skill for this second brain

Your job: study how this second brain stores tasks, projects and decisions, then write a custom skill called `kiss` that walks the user through every open question waiting on them, one clickable question at a time, and writes each answer back into their own system. Work in four phases, in order. Do not skip a phase.

Rules for the whole job:
- Phase 1 is read-only. Change nothing until Phase 3.
- When you need the user, ask one question per turn, in plain words, and wait for the answer.
- Never invent a convention. Everything the skill relies on (file paths, task formats, done markers, tools, commands) must come from something you read or ran in this system.
- Never delete or move any of the user's tasks or notes. The skill only ticks, annotates and adds.

## Phase 1: Discover how tasks are stored (read-only)

1. Read the agent's instruction files first (for example CLAUDE.md, AGENTS.md, .cursorrules, GEMINI.md, a README at the root of the notes folder), and any index or map note they point to.
2. Find every place tasks and decisions live. Check all of these that exist:
   - Markdown or plain-text files with checkboxes (`- [ ]`, `- [x]`, `TODO`, `DONE`), org-mode, todo.txt, Taskpaper.
   - Project notes, decision logs, meeting notes, daily notes, inbox files.
   - Task apps reachable from here through a connector, MCP server, CLI or API (for example Todoist, Things, Notion, Linear, Jira, GitHub Issues, Asana, Trello, Apple Reminders, Google Tasks). List the connectors and CLIs actually available and test each with a read-only call.
3. For each source, record from real examples you read:
   - How an open item is marked and how a done item is marked.
   - How items are grouped into projects (folders, headings, tags, fields, labels).
   - How (if at all) the user marks an item as a question or decision that waits on them: headings such as "Decisions" or "Questions", tags, assignees, phrases such as "waiting on me", "my call", "decide", "?" at the end of a line. If there is no marker, note the phrasing patterns that open decision items actually use in this system.
   - How the user records a decision or a note on an item (an appended line, a comment, a dated sub-bullet, a field), and the date format they use.
   - Which projects are active and how that is shown (a status field, a folder, an index, recent edits).
   - Any standing rules or reference documents the user has said settle certain questions (so the skill applies them instead of asking).
4. Find the tools the agent has for:
   - Clickable multiple-choice questions (in Claude Code: AskUserQuestion, which shows 2 to 4 options plus a free-text "Other" box). If none exists, the fallback is lettered options printed as text, answered with one letter.
   - Running a background or sub-agent. If none exists, the scan runs inline.
   - Loading a skill or custom command, and where those files live.
5. Write your findings as a short source map: for each source, its location, the open marker, the done marker, the project grouping, the decision marker or phrasing, the write-back method, and one real example line. Note anything you could not read and why.

## Phase 2: Confirm with the user

Show the source map in a few plain lines. Then ask, one question per turn, only what the files could not answer. Typical gaps:
- Which sources should the scan include or skip.
- How the user wants a decision written back if the system has no clear convention.
- What word starts the session (default: the whole message is `kiss`).
Use the clickable question tool, recommended option first with "(Recommended)" at the end of its label.

## Phase 3: Write the skill

Write the skill where this agent loads skills or custom commands (Claude Code: `~/.claude/skills/kiss/SKILL.md`, with a `description` in the frontmatter saying it runs only when the user's whole message is the trigger word). If the agent has no skill mechanism, write `KISS.md` at the root of the notes folder and add one line to the main instruction file: "When my whole message is `kiss`, follow KISS.md."

The skill holds two parts: an adapter section built from the confirmed source map, and the behavior below, copied in full with the placeholders filled.

### Adapter section (fill from Phase 1 and 2)

- Sources to scan, each with its path or tool, open marker, decision marker or phrasing, and project grouping.
- How to tick an item done in each source, with one real before-and-after example.
- How to record a decision in each source, with one real example, including the date format.
- Standing rules or documents that settle questions.
- The clickable question tool's name and its option limit, or the lettered-text fallback.
- The path of the saved scan file: `<skill folder>/last-scan.md`. If the folder is under version control, add that file to the ignore list.

### Behavior (copy in full)

**Never delete or move** any of the user's tasks or notes. The skill only ticks, annotates and adds.

**Trigger.** Runs only when the user's whole message is the trigger word, never on any other wording and never on the agent's own initiative. First reply exactly `kiss (y/n)?` and end the turn. If the answer is no, nothing starts. Otherwise reply exactly `fresh (y/n)?` and end the turn. Yes runs a new scan. No skips the scan and resumes from `last-scan.md` at the project picker with the questions still marked open; if that file does not exist, say so in one line and run a new scan.

**Scan.** Say in one line that the scan is running. Run it in a sub-agent if one is available, and wait for it, since nothing can be asked until it returns. Read every source in the adapter fresh and in full, never from memory or an earlier scan. Collect every open item that waits on the user to decide, plus any question left open in the current conversation. For each question return: project, the question in one line, source location and line, the facts needed to ask it (each with its source), the options with what each does and costs, the recommended option and why, whether a standing rule already decides it (then it is applied and noted, not asked), and whether a later record already answered it (then tick or fix the item instead of asking). Drop duplicates and already-answered items and list what was dropped and why. Leave out items that are the user's own actions rather than decisions, and items the user parked until a condition that has not been met.

**Freeze.** Check every question has a source that exists and no two ask the same thing. Overwrite `last-scan.md` with the scan date and each question with id, project, source, facts, options and status (open, answered, skipped). Only the last scan is kept. This file is the session's memory if the conversation is cut short. Questions raised later by work started during the session go to their owning record for the next session, never into this one.

**Project picker.** One clickable question: options are project names with their open-question count, for example `Home reno (4)`, plus `Stop`. If the tool caps the number of options, show as many projects as fit with `More projects` and `Stop`, and page. After a project's questions are done, show the picker again with only the projects that still have open questions. When none are left, the session ends.

**Asking a question.**
- One question per clickable question, single choice.
- Before asking, re-read the source lines the question rests on. Numbers come from the data, with what they measure.
- Write it simple the first time: the question is one or two short sentences, each option one short sentence, everyday words, things named by everyday examples, nothing lost that the user needs to decide.
- The question text stands alone (the dialog may be all the user sees) and never states a recommendation.
- At most three real options plus `Simplify`. The recommended option is first, its label ending `(Recommended)`, its description saying what it does, its real impact and why it is recommended. Each other option says what it does and its real impact. `Simplify` says "Ask this again in plainer, more direct words."
- `Simplify` re-asks the same question in still plainer words, keeping every option, its impact and the recommendation, and keeps a `Simplify` option.
- An answer typed in the free-text box is the user's decision; if it needs clarifying, ask one follow-up in the same format.

**After each answer.**
- Record the decision at once in the source that owns the item, using the adapter's write-back method, with the user's own words and the date, and tick the item if it is now done. Update `last-scan.md`.
- If the answer starts work, start it in the background (a sub-agent or background task) so the session keeps moving. If it can only be done in the main conversation, queue it in `last-scan.md` under "After kiss" and do nothing on it yet. Never stop the session to do work. Results that arrive mid-session wait until the end.
- Anything that spends money or sends, posts or deletes outside this machine needs the user's explicit go in their own words, which the decision itself can be.
- Then ask the project's next question.

**Stopping.** If the user picks `Stop`, declines a question or interrupts, ask one clickable question: "Do you want to exit kiss?" with `Yes, exit` and `No, continue`. No returns to the question they were on. Anything typed in the free-text box is recorded first, affected questions are adjusted or dropped, then the exit question is asked again. If a turn was interrupted and the next message is not an answer, read `last-scan.md` and ask the exit question.

**Ending.** The session ends when every frozen question is answered or the user confirms exit. Record the end state in `last-scan.md`. The reply is exactly `kiss exit`, with no summary. Then start the queued main-conversation tasks.

## Phase 4: Test

1. Run the scan once in read-only mode (no write-back) and show the user the project picker counts and the first three questions as they would appear.
2. Pick one real open item the user agrees to use as the test, answer it through the skill, and confirm the write-back landed in the right place, in the user's own format, by re-reading the source.
3. Fix anything that did not match, then tell the user in one line that `kiss` is ready and where the skill file is.

Done when: the skill file exists where the agent loads it, its adapter section names every confirmed source with a real tick and decision example, the read-only scan ran, one real write-back was verified by re-reading the source, and `last-scan.md` is ignored by version control if the folder uses it.
