# i10s — LLM instruction docs, safely distributed

**i10s** (numeronym for *instructions*) is a small corpus of methodology docs for working with LLMs, plus a **code-free, safe way to distribute them** into any repo.

- **Human-facing methodology:** `docs/corpus-driven-design.md`, `docs/corpus-driven-development.md`, `docs/writing-llm-instructions.md`
- **LLM-context payloads:** `docs/collaboration.md`, `docs/design-rigour.md`

## Use it

Hand your coding agent the URL of [`getting-started.md`](getting-started.md) and say *"get this and follow it."* It scaffolds an `llm/` dir + a pinned `llm.conf` in your repo; you set the pin, run the vendored sync, and reference `@llm/index.md` from your agent config.

**Safe by design:** you pin a version (never auto-`main`); every update is a **reviewable git diff** you commit (git is the gate); the only remote fetch is the pinned doc sync — no installer, no code, no dependencies.

## License

MIT — see [`LICENSE`](LICENSE).
