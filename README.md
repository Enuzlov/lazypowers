# Lazypowers

Lazypowers is an instruction-only product-to-delivery workflow for Codex. It
lets you approve specifications, run isolated engineering tasks through the
official Superpowers workflow, verify exact Git results, and approve
deployments from one Product chat.

## How it works

The Product chat stays available while engineering is in progress. After you
approve a saved specification, Lazypowers either places the task in a durable
numeric queue or launches it in parallel when the platform provides an
authoritative, current capacity signal. Without that signal, tasks run
sequentially.

Each task runs in its own visible Codex task chat and isolated worktree. Inside
that task chat, official Superpowers owns planning, TDD, debugging, review,
verification, and branch finishing. The task makes intermediate technical
decisions autonomously when they remain inside the approved scope; changes to
scope, authority, security, or irreversible actions return to Product.

Engineering callbacks are published transcript-first. Product reads the full
callback from the source task transcript, verifies the exact base commit,
delivery commit, sole parent, branch ref, and direct changed path set, then
records the result exactly once. Product also owns the separate deployment
approval gate, so specification approval never silently becomes deployment
authority.

Model selection happens per task. Standard work starts with Terra at high
reasoning when available, frontier-risk work uses Sol, and one bounded
Terra-to-Sol escalation is allowed after systematic debugging. Target lineage,
fresh project-owned probes that can prove either factual repository absence
for initial publication or a factual deployed SHA, and one-use approval protect
deployment from stale state and duplicate execution.

Lazypowers deliberately has no runtime, database, daemon, scheduler, metrics
service, or universal deployment adapter. Its durable state is ordinary
Markdown under `.lazypowers/`, while official Superpowers remains responsible
for the engineering discipline inside each task.

## Install

Add the Git-backed marketplace and install the plugin:

```bash
codex plugin marketplace add Enuzlov/lazypowers
codex plugin add lazypowers@lazypowers
```

## Quick start

Open a Product chat and invoke the Product skill explicitly:

```text
$lazypowers:product Help me turn this product idea into an approved specification and verified delivery.
```

Lazypowers will keep discussion and approval in Product, create a visible task
chat only after the saved specification is approved, and ask again before any
deployment or other external action.

## License

Lazypowers is available under the MIT License.
