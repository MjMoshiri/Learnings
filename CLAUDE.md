# Learnings Repo — Claude Context

## What this is

Mohammadjavad Moshiri's personal learning archive. Single repo replacing a dozen small ones that grew over years of self-study. Topics span AI/ML, systems, languages, web, math, cloud, startup. Active and ongoing.

## Who MJ is

- BSc Computing & Electrical Engineering, Tampere University (2024, with distinction, GPA 4.4)
- Head of AI at Oway (YC S24, logistics AI)
- Strong ML, signal processing, probability, matrix analysis background
- Builds production systems daily; uses this repo for learning outside work

## Repo structure

```
topics/    knowledge by subject. one folder per topic, one file per concept
log/       chronological learning events. YYYY-MM-DD-slug.md
builds/    one-pager per notable project
inbox/     raw notes not yet filed
bin/       new-note, index
```

Three top-level content dirs: `topics/`, `log/`, `builds/`. No `journal/`, no `notes/`, no `Timeline.md`. The log itself IS the timeline; the filename is the date.

## Conventions

- **Folders and files are kebab-case lowercase.** Always.
- **Topic files carry frontmatter:**
  ```
  ---
  topic: <topic>
  status: done | wip | ongoing
  tags: [optional, list]
  ---
  ```
- **`_.md` is the topic landing page** when a folder needs one. Sorts first alphabetically. Hand-written. Covers what's in the folder, what's missing, where to start.
- **Single-file topics:** put the content in `_.md`. No need for a separate file.
- **Log entries are verbatim.** MJ's own daily writing isn't rewritten.
- **Scratch goes in `inbox/`.** Promote to a topic once the shape is clear (3+ related notes is the rule of thumb).

## Workflow — adding a new note

1. Pick a topic folder under `topics/`. If none fits, create it: `mkdir topics/<new-topic>`.
2. Run `bin/new-note <topic> <slug>` to scaffold the file with frontmatter and drop a log entry.
3. Write the note in mj-rewrite voice (see below).
4. If the topic folder is new and worth introducing, write a short `_.md` landing page.
5. Run `bin/index` to refresh `topics/_.md`.
6. Commit. The log entry already records the event; no separate Timeline update needed.

## Workflow — coming back to a topic after months

- `cat topics/<topic>/_.md` — landing page (if one exists)
- `ls topics/<topic>/` — every concept covered
- `grep -lr "status: wip" topics/<topic>/` — anything left half-done
- `git log --oneline -- topics/<topic>/` — chronology of changes

## Voice

All generated prose (READMEs, notes, summaries, build pages, log entries) must pass through the `mj-rewrite` agent at `.claude/agents/mj-rewrite.md` before being written to disk. Hard bans: em dashes, "delve", "robust", "leverage", "comprehensive", "ensure" as filler, generic AI sound. MJ's existing journal entries stay verbatim.

## What this repo is NOT

- Not a mirror of old repos. Don't recreate the original structure of archived projects
- Not a code dump. Notes and write-ups, not raw source from past projects
- Not a portfolio. Production work lives in its own repos
