# Learnings Repo — Claude Context

## What this is

Mohammadjavad Moshiri's personal learning archive. Single repo replacing a dozen small ones that grew over years of self-study. Topics span AI/ML, Android, iOS, web frameworks, systems languages, math, cloud, startup. Active and ongoing.

## Who MJ is

- BSc Computing & Electrical Engineering, Tampere University (2024, with distinction, GPA 4.4)
- Head of AI at Oway (YC S24, logistics AI)
- Strong ML, signal processing, probability, matrix analysis background
- Builds production systems daily; uses this repo for learning outside work

## How to help

- New notes go under `notes/<topic>/` as plain markdown
- Daily learning sprints go under `journal/` as `YYYY-MM-DD.md`
- Project write-ups go under `projects/<name>.md`, one page each
- Update `Timeline.md` whenever meaningful work lands. Each entry links to the commit SHA on GitHub
- Never invent a topic folder if a close one already exists. Read the tree first
- All generated prose (READMEs, notes, summaries, project pages, Timeline entries) must pass through the `mj-rewrite` agent before being written to disk

## Repo structure

```
notes/        topic notes, one folder per subject
journal/      daily entries from learning sprints, kept verbatim
projects/     one-pager per build worth remembering
Timeline.md   tamper-evident chronological log
README.md     short overview
```

## Timeline rules

Every Timeline.md entry must:
1. Have an exact date (YYYY-MM-DD)
2. Link to the commit SHA on GitHub where the work lives
3. Say what was done in plain words
4. Be honest. No padding. One short line is usually enough
5. Not list folders created or restate mechanics. Say what was actually learned or built

## What this repo is NOT

- Not a mirror of old repos. Don't recreate the original structure of archived projects
- Not a code dump. Notes and write-ups, not raw source from past projects
- Not a portfolio. Production work lives in its own repos
