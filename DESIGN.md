# i10s — design

**`i10s`** — numeronym for *instructions* (i·nstruction·s). LLM instruction docs plus a **safe, code-free distribution mechanism** that vendors them into consuming repos. The docs are the easy part; the design work is the distribution mechanism and its trust model.

## Status

Built and in use: `llm.conf`, the vendored sync instruction, `synced/` + `local/` + a generated `index.md`, and three consuming repos on a pinned tag. Designed but **not** built: the drift-guard CI job, signed and protection-enabled tags, branch protection. Those are marked below where they appear — the trust argument leans on them, so it is currently weaker than the section describing it.

## What it is
Two kinds of content, plus a way to distribute them:
- **Human-facing methodology** — read by people (`corpus-driven-design`, `corpus-driven-development`).
- **llm-context payloads** — consumed by the model (`collaboration`, `design-rigour`, `pair-coding`).
- **Distribution** — syncs the source docs into a consumer's repo, safely, so everyone downstream stays current from one source.

## North star
1. **Safe by construction** — instructions flow into a consumer's LLM context, a supply-chain surface; the mechanism must make a compromised source *un-silent* (reviewable), never auto-applied.
2. **Code-free** — no compiled tool; the "sync" is an **instruction** the agent runs, driving `git`. The project distributes itself via the very thing it is about.
3. **Consumer-extensible** — a consumer's own local instructions sit alongside the synced ones, under one reference.
4. **Vendor-agnostic** — produces one neutral dir; each agent references it however it does (Claude `@import`, a root `AGENTS.md`, a Cursor rule). The mechanism never touches vendor dirs.

## The distribution mechanism
No binary. The moving parts:

- **`llm.conf`** (consumer repo root) — **source repo + a pinned ref (tag/commit) + an ordered list of glob patterns** selecting which source docs to pull. Pinned, never "latest". **Glob-style, ordered**: patterns match source files (a `*` auto-includes new docs at the pinned ref — surfaced in the sync diff); **order sets precedence** in the generated index (later = more authoritative), and globs expand in a **stable sort** so the index is deterministic. `local/` is appended after `synced/`, so local docs override the shared corpus. *(Precedence is **soft** — the model reads all of it; ordering is a convention, not a hard override.)*
- **A vendored sync *instruction*** (markdown, not code) that the consumer's agent runs. It:
  1. reads `llm.conf`;
  2. fetches the listed docs at the pinned ref **verbatim via `git`** — never retyped; fidelity comes from git, the model only orchestrates;
  3. mirrors them into **`llm/synced/`**;
  4. **regenerates `llm/index.md`** over `synced/` + `local/`;
  5. **leaves a git diff for review** — it does not commit unless asked.
- **Layout** (in the consumer repo):
  ```
  llm.conf            ← repo root: source repo + pinned ref + doc list
  llm/
  ├── synced/         pinned, vendored from source; a mirror — "edit the SOURCE, not here"
  ├── local/          the consumer's own instructions (files + subdirs); freely edited
  ├── index.md        GENERATED — @imports synced (conf order) then local; the single reference
  ├── sync.md         the vendored sync instruction
  └── README.md       what the dir is, and where to edit a rule
  ```
- **Agents reference `@llm/index.md`** — one line pulls shared + local. The consumer wires their agent to it once, by hand.

**Why `synced/` carries no "do not edit" banner.** It is byte-identical to the source at the pinned ref, and that fidelity is the guarantee the whole mechanism rests on — a banner would break it. So the warning lives in `llm/README.md`, beside the mirror rather than inside it.

## Safety / trust model
The mechanism moves **instructions into consumers' LLM context**, and instructions are *executable* — they steer the model. So it is a genuine supply-chain / trust boundary: **compromise the source → mass prompt-injection of every downstream LLM.** The safety rests on two cheap properties:

1. **Pinned ref, never "latest".** The conf names a tag or commit. Auto-following `main` is the *entire* danger; pinning removes it — you choose which version you run.
2. **The sync writes a reviewable git diff a human commits.** Content lands in committed `llm/synced/` as ordinary files; the update is a **diff you review**. **Git is the gate** — nothing enters an LLM's context unseen.

Reinforcements:
- **Bootstrapping trust closed** — the sync *instruction* is vendored locally (reviewed once), so you run *your own reviewed* instruction to fetch *pinned* content — not live remote code to fetch remote code.
- **Drift-guard (designed, not built)** — regenerate `synced/` at the pinned ref and `git diff --exit-code` in CI; catches hand-edits of the vendored copies. Deterministic *because git does the fetch*, not the model.
- **Provenance in the index** — the synced section records source repo + pinned ref + sync date, so the dir is self-describing.

Same **machine-generates → human-gate → git-review** shape as `f8n`'s data-refresh PR and `l10n`'s translation gate: the generator is allowed to be wrong, because a human sees the diff before it lands.

## Repo lockdown (root of trust)
This repo is the **single source** consumers pull instructions from — one compromise reaches every downstream LLM. The design shrinks the surface to near-nothing, then hardens what's left:

- **No code, no deps, no build.** Markdown docs and a markdown sync instruction — no binary, no post-install scripts, no dependency tree. The only attack is "maliciously edit a doc", which the controls below are meant to catch.
- **Branch protection on `main`** — required review, no force pushes, linear history. *(Not yet enabled.)*
- **Signed commits, and signed *protected* tags.** Tag protection is the load-bearing one: it means a pinned tag cannot be re-pointed at new content, so **pinned = immutable**. Without it, pinning is a weaker promise than it reads. *(Not yet enabled.)*
- **2FA required, minimal maintainer set, least-privilege Actions** — CI is a classic supply-chain vector even for a docs repo. *(Partially: no Actions exist yet.)*
- **Consumers pin to tags** and can check provenance from the sync index.

**Defence in depth:** lockdown *prevents* source compromise; the consumer-side **pinned ref + reviewed diff** *contains* it — even a compromised source only reaches a consumer who bumps their pin and merges a reviewed diff. Neither side is trusted alone. Note the asymmetry while the lockdown items are unbuilt: the consumer-side containment is real today, the source-side prevention is mostly intent.

## Consumer extension (`synced/` vs `local/`)
Consumers add their own instruction files under **`local/`**; the generated **`index.md` aggregates both**, so one `@llm/index.md` pulls the shared corpus *and* the consumer's own conventions. The split keeps the trust boundary crisp: **`synced/` is managed** (the sync owns it, never hand-edit); **`local/` is the consumer's** (the sync never touches it — no clobber, no false drift-guard positives).

## Getting started
The entry is user-initiated: the **README links a stable URL to `getting-started.md`**. The user hands that URL to their agent — *"get this file and follow it."* No installer, no `curl | bash`; the user starts it, from an inspectable URL.

`getting-started.md` is **deliberately minimal — it scaffolds and stops.** Followed, it creates the `llm/` structure, writes an **empty commented `llm.conf`**, vendors the **sync instruction**, and adds `llm/README.md`. Then the consumer fills in the conf with a pinned ref, runs the sync, reviews the diff, and wires their agent once.

**Why that's safe:** `getting-started.md` only scaffolds — it fetches no real docs and wires nothing silently, and everything it creates is a reviewable diff. The *only* trust-critical remote fetch in the whole lifecycle is the **sync**, at a **pinned ref**, verbatim. (`getting-started.md` itself loads from `main` for friendliness, which is acceptable because it produces only local scaffold.)

## Naming
- **`i10s`** — the project. Numeronym family fit; distinctive and searchable.
- **`llm/`** and root **`llm.conf`** — the consumer-facing names. A consumer seeing `i10s/` in their repo wouldn't know what it is; `llm/` they would. **Visible, not a dot-dir** — the content is human-readable instruction docs meant to be discovered, read and extended (especially `local/`), not machine config. Hiding it fights the ethos and dodges search and doc-site tooling. `i10s` appears only *inside* `llm.conf`.

## What ships
Eight methodology docs under `docs/`, split between human-facing methodology and llm-context payloads. The README carries a router table saying what each doc is for — a new rule is placed by finding its row, and a rule that fits no row is a signal the corpus needs a new doc rather than a stretched existing one.

**Licence — MIT.** Simple, permissive, universally understood. *(CC-BY is the doc-purist option — MIT is written for "the Software" — but this repo is docs **plus** a runnable sync instruction, and MIT covers both with less ceremony.)*

## Positioning
`i10s` pushes **down** into agent dirs — instructions for the machine. `d11n` pulls **up** into a site — documentation for humans. They look similar (both conf-driven, both cross-repo) and share nothing that matters: opposite direction, opposite consumer, different trust model. **No shared substrate** — `i10s` stays standalone and simple: no code, no dependencies, no CI. That idea is dropped, not deferred; a shared substrate would drag `i10s` toward exactly the complexity it was designed to avoid.

## Open questions
- **Signed and protected tags** — the pinning promise is only as strong as tag immutability, and tag protection is not yet enabled. This is the largest gap between what the trust model claims and what the repo enforces.
- **Drift-guard in CI** — designed, unbuilt. Currently nothing detects a hand-edited `synced/` file except a human noticing.
- **Index format stability** — `index.md` is regenerated by an instruction rather than a program, so its exact shape depends on the instruction being followed precisely. Consumers diff it on every sync, which surfaces a deviation but does not prevent one.

## Change log
- 2026-08-25: **design doc published.** Reconciled with what actually shipped: the sync **leaves a diff** rather than auto-committing a PR, and the consumer layout gained `sync.md` and `README.md`. Added a **Status** section and marked the unbuilt lockdown items in place — the trust model reads stronger than the repo currently enforces, and a design doc that hides that is worse than one that admits it. Recorded why `synced/` carries no inline "do not edit" banner (it would break byte-identical fidelity, so the warning lives in `llm/README.md`).
- 2026-07-06: **created.** Distribution designed **code-free** — a vendored sync *instruction* driving `git`, not a binary. Safety = **pinned ref + reviewable git-diff gate**; bootstrapping trust closed by vendoring the instruction; optional drift-guard. Local dir **`llm/`** with **`synced/`** (managed) + **`local/`** (the consumer's) + a generated **`index.md`** aggregating both; agents reference `@llm/index.md`. **Consumer-facing names are a visible `llm/` + root `llm.conf`**, not a dot-dir — the content is readable docs meant to be discovered and extended. **Getting started from nothing:** no installer; the README links a static `getting-started.md` the user hands to their agent, which **scaffolds only** — no doc fetch, no silent wiring, everything a reviewable diff. **No remote instruction execution** anywhere in the lifecycle; the only trust-critical fetch is the pinned sync. Added **repo lockdown** as the source-side half of defence in depth. Settled: **MIT**; **glob-style ordered `llm.conf`** (order = soft precedence, `local/` overrides `synced/`, stable sort → deterministic index); **standalone, no shared substrate**; **no scheduled CI** — updating is just running the sync when you want.
