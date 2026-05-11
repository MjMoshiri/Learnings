# Learnings

Single home for everything I study outside of work. Notes, sprints, and project write-ups across AI/ML, systems, languages, web, math, startup.

Replaces a scattered set of small repos that existed before. Those are archived; their distilled content lives here.

## Layout

```
topics/    knowledge by subject. one folder per topic, one file per concept
log/       chronological learning events. YYYY-MM-DD-slug.md
builds/    one-pager per notable project (what it was, what I learned)
inbox/     raw notes not yet filed into a topic
bin/       scripts (new-note, index)
```

That's it. No category layer. No journal vs Timeline split. Filenames sort to give chronology; folder structure gives subject.

## How I use it

- New concept worth keeping: `bin/new-note <topic> <concept-slug>`. Scaffolds the topic file with frontmatter and drops a log entry.
- Topic doesn't exist yet: `mkdir topics/<topic>` then run `new-note`. No category to pick.
- Learning sprint without a clear topic: drop in `inbox/`, promote later.
- Every note carries minimal frontmatter: `topic`, `status` (`done` / `wip` / `ongoing`).
- `bin/index` regenerates `topics/_.md` (the master index) from the filesystem.
- Each topic folder may have its own `_.md` landing page covering what's in there, what's next, and where to start.

## Recall

- "What did I write about X?" → `ls topics/<topic>/`. Filenames are concepts.
- "What was unfinished?" → `grep -lr "status: wip" topics/`.
- "When did I touch this?" → `git log -- topics/<topic>/`.
- "What did I learn last month?" → `ls log/` and read the dates.

## Writing rules

Plain language. No filler. No corporate words. If a paragraph could come from a chatbot, it gets rewritten. The `mj-rewrite` agent under `.claude/agents/` enforces this for anything Claude generates here.
