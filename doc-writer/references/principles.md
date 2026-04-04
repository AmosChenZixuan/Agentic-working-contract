# Shared Documentation Principles

## First Principles Thinking

Question every assumption about what belongs in a document:

- **Why does this exist?** A doc exists to transfer knowledge that can't be derived from code. If the "knowledge" is just describing what the code already shows, the doc is noise.
- **Who is this actually for?** Write to that reader — not to be comprehensive, not to capture everything, but to serve a specific person with a specific gap.
- **What is the minimum needed?** Resist the urge to document "just in case." More docs = more maintenance burden and more stale content. If something can be learned from the code, don't duplicate it.
- **Would this survive a "so what?" test?** For each section: if the reader finishes it and doesn't know something that changes their behavior, cut it.

When in doubt, lean toward less. A focused document that gets read beats an exhaustive one that gets ignored.

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
