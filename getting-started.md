# Getting started with i10s

i10s distributes a small corpus of LLM instruction docs into your repo, safely: pinned to a version, vendored as ordinary files, updated only through reviewable git diffs. No installer, no code, no dependencies — just this doc and git.

**To set up**, hand your coding agent this file's URL and say *"get this and follow it."* It scaffolds the following in your repo — and nothing else. It fetches no real docs and wires nothing silently; everything it creates is a reviewable diff you commit.

## 1. Create the structure

```
llm/
├── synced/     # pinned docs vendored from i10s (managed — edit the SOURCE, not here)
├── local/      # your own instruction docs (yours — freely edited)
└── index.md    # generated — lists synced + local; the single reference
```

## 2. Create `llm.conf` at the repo root

```yaml
# i10s source + a PINNED ref (tag/commit — never a branch) + which docs to pull.
source: github.com/cjr--/i10s
ref:    v0.1.0          # pin me to a real tag
docs:                  # ordered globs; later = higher (soft) precedence; local/ overrides synced/
  - docs/*.md
```

## 3. Vendor the sync instruction

Copy `sync.md` from the i10s repo (at your pinned ref) into `llm/sync.md`, so future updates run *your own reviewed* instruction — never live remote code.

## 4. Run the first sync

Follow `llm/sync.md`. It fetches the docs at your pinned ref into `llm/synced/`, regenerates `llm/index.md`, and leaves it as a diff for you to review and commit.

## 5. Wire your agent

Add `@llm/index.md` to your `CLAUDE.md` / `AGENTS.md` (or your tool's equivalent). One line pulls the shared corpus plus your local additions.

## Updating later

Bump `ref` in `llm.conf` and run `llm/sync.md` again — you'll see exactly what changed as a diff before committing. That review is the safety gate.
