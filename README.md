# i10s — LLM instruction docs, safely distributed

**i10s** (numeronym for *instructions*) is a small corpus of methodology docs for working with LLMs, plus a **code-free, safe way to distribute them** into any repo.

## The docs

Two kinds: **human-facing methodology** (read by people) and **llm-context payloads** (loaded into the model). The table is also the router — when a new rule needs a home, find the row it belongs to rather than guessing.

| doc | kind | the rule that belongs in it |
|---|---|---|
| `collaboration.md` | payload | how a working session is conducted — correctness over speed, flagging uncertainty, absorbing corrections, prose style |
| `design-rigour.md` | payload | what to surface before shipping — trust boundaries, threat model, failure modes, irreversibility, dependencies |
| `pair-coding.md` | payload | code-level conventions — naming, explicitness over terseness, versioning, tool choice |
| `corpus-driven-design.md` | methodology | the design half — turning discussion into a durable corpus |
| `corpus-driven-development.md` | methodology | the build half — phased plans, the feedback loop, tests alongside code |
| `collaborative-agents.md` | methodology | shared context between agents and across repos — where it lives, how it flows |
| `collaborative-claude.md` | methodology | Claude Code specifics — how context loads, spanning a private corpus and repos |
| `writing-llm-instructions.md` | methodology | how to write and maintain these docs |

A rule that fits two rows belongs in the more specific one; a rule that fits none is a sign the corpus needs a new doc rather than a stretched existing one.

## Use it

Hand your coding agent the URL of [`getting-started.md`](getting-started.md) and say *"get this and follow it."* It scaffolds an `llm/` dir + a pinned `llm.conf` in your repo; you set the pin, run the vendored sync, and reference `@llm/index.md` from your agent config.

**Safe by design:** you pin a version (never auto-`main`); every update is a **reviewable git diff** you commit (git is the gate); the only remote fetch is the pinned doc sync — no installer, no code, no dependencies.

## License

MIT — see [`LICENSE`](LICENSE).
