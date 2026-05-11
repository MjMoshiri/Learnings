# Learnings

Single home for everything I study outside of work. Notes, summaries, and project write-ups across topics: AI/ML, Android, iOS, web, systems, languages, startup, math.

Replaces a scattered set of small repos that existed before. Those are archived; their distilled content lives here.

## Layout

```
notes/      topic notes, organized by subject
journal/    dated daily entries from active learning sprints
projects/   one-pager per notable build (what it was, what I learned)
Timeline.md chronological log of meaningful work, linked to commits
```

## How I use it

- New concept I want to remember: drop a markdown file under `notes/<topic>/`.
- Active learning sprint: daily entries go in `journal/` as `YYYY-MM-DD.md`.
- New project worth keeping: short page in `projects/` covering what, why, and what I took from it.
- Anything meaningful gets a line in `Timeline.md` with the commit SHA.

## Writing rules

Plain language. No filler. No corporate words. If a paragraph could come from a chatbot, it gets rewritten. The `mj-rewrite` agent under `.claude/agents/` enforces this for anything Claude generates here.
