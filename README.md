# Lazypowers

Lazypowers is a thin, instruction-only Product dispatcher for Codex. It saves
product specifications to a small manual queue and creates one separate visible
task chat for each task the user chooses to launch. Official Superpowers owns
all implementation work inside that task chat.

## Prerequisites

- Codex and a project workspace.
- Official Superpowers.
- Permission to create visible Codex task chats and isolated worktrees.

Lazypowers includes no runtime, database, daemon, scheduler, poller, deployment
adapter, callback transport, or Git automation.

## Install official Superpowers

In the Codex app, open **Plugins**, find **Superpowers** in the Coding category,
and install it from the official Codex plugin marketplace. In Codex CLI, open
`/plugins`, search for `superpowers`, and select **Install Plugin**. Start a new
task after installation so the official skills are available.

## Install Lazypowers

Use the Git-backed Lazypowers marketplace:

```bash
codex plugin marketplace add Enuzlov/lazypowers
codex plugin add lazypowers@lazypowers
```

## Verify

```bash
codex plugin list --marketplace lazypowers --json
```

The plugin must report version `0.4.0`. In a new Product chat, invoke:

```text
$lazypowers:product Помоги описать задачу, сохранить её в очередь и запустить в отдельном чате.
```

## Workflow

1. Discuss the desired result in the Product chat.
2. Product saves the draft under `.lazypowers/tasks/NNN-short-slug/spec.md`.
3. Approve the specification to place it in the manual queue.
4. Say `запусти следующую` or name one or more queued task numbers.
5. Continue every implementation, permission, Git, retry, deployment, and
   completion conversation directly in the created task chat.

Approval never launches a task automatically. Product creates a task chat only
after a manual launch command.

The task chat starts with `superpowers:writing-plans`, uses official
Superpowers throughout, and may make at most three actual final-action attempts
with systematic debugging between failures.

## What Product does after launch

Nothing.

After Product saves the exact task thread ID and sets the visible title
`NNN — <spec title>`, it stops working with that task. It does not:

- monitor or wait for the task chat;
- receive callbacks or heartbeat messages;
- reconcile status;
- verify commits or branches;
- integrate or clean up worktrees;
- approve deployment attempts;
- track target lineage;
- record completion.

The user supervises the work directly in the task chat.

## Queue files

New tasks use:

```text
.lazypowers/tasks/NNN-short-slug/
  spec.md
  dispatch.md
```

`dispatch.md` uses schema `lazypowers.dispatch.v1` and states `draft`,
`queued`, `launching`, or `launched`. Existing historical Lazypowers state is
left untouched and ignored by the new queue unless it contains that exact
dispatch schema.

## Errors

- Authoritative create failure returns the task to `queued`.
- Ambiguous create leaves it `launching`; Product does not create a replacement.
- Title failure keeps the task `launched` and is reported once.
- Missing capacity leaves the task queued with no background retry.
- Missing Superpowers is handled in the task chat, not by Product.

## Upgrade

```bash
codex plugin marketplace upgrade lazypowers
codex plugin add lazypowers@lazypowers
```

Start a new Product chat after upgrading so Codex loads the new skill.

## Uninstall

```bash
codex plugin remove lazypowers@lazypowers
```

Remove the marketplace only if no other plugin from it is needed:

```bash
codex plugin marketplace remove lazypowers
```

## License

Lazypowers is available under the MIT License.
