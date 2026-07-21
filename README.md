# Lazypowers

Lazypowers is an instruction-only Product workflow for Codex. It keeps the
Product chat available while isolated Runner task chats use official
Superpowers to implement approved specifications and return Git-verifiable,
transcript-first engineering callbacks.

## Prerequisites

- Codex and a Git repository for the work.
- Official Superpowers, which owns planning, TDD, debugging, review,
  verification, and branch finishing inside each Runner task chat.
- Permission to create local task worktrees. Lazypowers does not grant
  permission for external actions.

## Install official Superpowers

In the Codex app, open **Plugins**, find **Superpowers** in the Coding category,
and install it from the official Codex plugin marketplace. In Codex CLI, open
`/plugins`, search for `superpowers`, and select **Install Plugin**. Start a
new Codex task after installation so the official skills are available.

## Install Lazypowers

Use the Git-backed Lazypowers marketplace. Do not create or install a personal
marketplace/local duplicate of Lazypowers.

```bash
codex plugin marketplace add Enuzlov/lazypowers
codex plugin add lazypowers@lazypowers
```

## Verify

Confirm that the GitHub marketplace provides the expected plugin version:

```bash
codex plugin list --marketplace lazypowers --json
```

The installed `lazypowers` plugin must report version `0.3.2`. In a new Product
chat, invoke the skill explicitly:

```text
$lazypowers:product Help me turn this product idea into an approved specification and verified Git delivery.
```

If the skill is available, Codex recognizes `$lazypowers:product`; the Product
chat can then save and discuss a specification before anything is launched.

## First workflow

1. Discuss the outcome in the Product chat and save the approved specification.
2. Approve that saved specification. Lazypowers queues it numerically, or runs
   it in parallel only when the platform supplies an authoritative current
   capacity signal; otherwise it uses sequential fallback.
3. Product resolves one visible, isolated Runner task chat. These numbered visible task chats use
   `NNN — <approved spec title>` so the queue is transparent. The title is
   UX-only metadata: it is not callback identity, Git evidence, or a
   correctness gate.
4. The Runner uses official Superpowers and publishes its full engineering
   callback in its own transcript. Product reconciles the first valid terminal
   callback as one immutable singular engineering result and verifies the exact
   Git base, commit, parent, branch, and changed paths.
5. Any deployment or other external action needs a separate, direct Product
   approval after engineering verification. Specification approval alone is
   never external-action authority.

## Upgrade

Refresh the GitHub marketplace, then reinstall the same marketplace plugin;
this updates Lazypowers without introducing a personal duplicate:

```bash
codex plugin marketplace upgrade lazypowers
codex plugin add lazypowers@lazypowers
```

## Uninstall

To uninstall Lazypowers, remove the installed GitHub-marketplace plugin:

```bash
codex plugin remove lazypowers@lazypowers
```

Remove the `lazypowers` marketplace only if no other plugin from it is needed:

```bash
codex plugin marketplace remove lazypowers
```

## Limitations

Lazypowers stores durable Product state as ordinary Markdown under
`.lazypowers/`; it has no runtime, database, daemon, scheduler, polling
process, metrics service, or built-in deployment adapter. It does not promise
unattended continuation or a guaranteed heartbeat. Parallel work depends on
authoritative capacity, and model choices depend on models the platform
currently exposes.

When a callable platform cannot provide a verifiable same-thread heartbeat,
unattended continuation is unavailable. A callback already published in the
Runner transcript is reconciled on the next Product turn instead. Product
still requires separate approval for every external action and cannot bypass
platform permissions.

## Troubleshooting

- **`$lazypowers:product` is missing:** confirm official Superpowers is
  installed, confirm `lazypowers@lazypowers` appears in the GitHub marketplace,
  then start a new Codex task.
- **An old plugin version remains visible:** run the upgrade commands above,
  verify `0.3.2` with `codex plugin list --marketplace lazypowers --json`, and
  start a new task so Codex does not use stale plugin context.
- **Superpowers is unavailable in a Runner:** install it from the official
  Codex marketplace and launch a new Runner only after it is available.
- **A callback seems delayed:** do not create a replacement task chat. If a
  verifiable heartbeat is unavailable, return to the Product chat; it pulls
  and reconciles the published callback on the next Product turn.

## License

Lazypowers is available under the MIT License.
