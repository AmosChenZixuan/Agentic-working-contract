# Shared Documentation Principles

## The core test

Before writing anything, ask: *could a reader learn this by reading the code, running git log, or checking the framework docs?* If yes, leave it out. Documentation exists to capture what isn't derivable from the source of truth it describes.

## What belongs in docs

- **Why** a decision was made, especially when the alternative was reasonable
- **Contracts between systems** — what one component expects from another
- **Non-obvious conventions** an agent or contributor needs to follow
- **Orientation** — where to start, what things are called, what the shape of the system is

## What doesn't belong

- Enumerations of things that change with every feature (component lists, column lists, route lists — unless the list itself is the contract)
- Implementation details that are obvious from reading the code
- Changelogs or activity summaries — that's what git is for
- Anything you'd have to update after every PR

## Style rules

- Write for the audience, not for completeness
- One sentence is better than three if it conveys the same thing
- Concrete over vague: name the file, the command, the field — not "the relevant configuration"
- Don't annotate the obvious: if a path says `notes/{video_id}.md`, don't add a comment saying "stores notes by video id"
- Decisions get a *why*; facts don't need one unless they're surprising
