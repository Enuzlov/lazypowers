---
name: product
description: Use when a Product chat needs to discuss, draft, save, revise, queue, list, or manually dispatch product specifications to separate visible Codex task chats that use official Superpowers. The skill owns only the disk-backed specification queue and initial task-chat handoff; it never monitors task execution after dispatch.
---

# Lazypowers Product

## Keep the boundary thin

Own only:

1. product discussion and a saved specification;
2. the manual disk-backed queue;
3. one visible task-chat create call;
4. the task title and initial Superpowers handoff.

The task chat owns every later plan, worktree, branch, implementation, test,
debugging step, permission, final action, and report. Product does not own or
verify any of that work.

Do not create a runtime, state service, database, daemon, scheduler, poller,
deployment adapter, or Git orchestration layer.

## Read the project

Before drafting or dispatching:

1. resolve the exact absolute project root;
2. read the applicable project instructions;
3. read existing numeric folders under `.lazypowers/tasks/`;
4. preserve every historical file byte-for-byte.

Historical folders without a valid `dispatch.md` schema
`lazypowers.dispatch.v1` are not queue entries. Use their numeric prefixes only
when choosing the next free task number.

Use `superpowers:brainstorming` to clarify material questions about the user
result, acceptance criteria, constraints, and final action. Save the validated
design directly as the task's `spec.md`; do not create a second Product design
artifact for the same task.

## Draft one specification

Allocate the lowest unused positive three-digit prefix and a concise
lowercase-hyphen slug:

```text
.lazypowers/tasks/NNN-short-slug/
  spec.md
  dispatch.md
```

Write `spec.md` in this minimal shape:

```markdown
---
task_id: NNN-short-slug
title: Human-readable title
project_root: /exact/absolute/project/path
attempt_limit: 3
---

# Результат

Что должно получиться для пользователя.

# Критерии готовности

- Наблюдаемый критерий.

# Ограничения

Что нельзя менять или делать.

# Конечное действие

Нужны ли установка, deployment или публикация и на какой target.
```

Keep implementation planning out of the specification. Official Superpowers
will plan inside the task chat.

Write `dispatch.md` with exactly:

```yaml
schema: lazypowers.dispatch.v1
task_id: NNN-short-slug
state: draft
task_thread_id: null
title: NNN — Human-readable title
launch_marker: null
client_thread_id: null
```

Treat `launch_marker` and `client_thread_id` as optional when reading an
existing v1 record. Write both fields for every new draft.

Allow only `draft|queued|launching|launched`.

Product may revise a draft. If the user changes a queued specification, first
return it to `draft`, apply the change, and show the revised summary. Never
change a launched specification; a material follow-up is a new task. Keep
`dispatch.title` equal to `NNN — <spec title>` while the task is a draft.

Before ordinary approval, combined approval, or manual dispatch, fresh-read
both files and require `dispatch.title == "NNN — <spec title>"` before any state change or create.
Derive `NNN` and the spec title from the fresh-read task ID and spec. Only after
this validation may title assignment use `dispatch.title`.

## Queue only after direct approval

Show a short summary:

```text
NNN — <title>
Результат: <one sentence>
Готово, когда: <short criteria>
Конечное действие: <action and target, or none>
Поставить в очередь?
```

When the user directly agrees, fresh-read both files, require matching task IDs
and `state: draft`, then change only the state to `queued`. A direct
affirmative answer changes only `draft → queued`.

When the same user message explicitly approves the current draft and says
«подтверждаю и запускай», fresh-read the draft, write `queued`, and immediately
begin local dispatch in the same turn. When it says «подтверждаю и запускай
через $mini», make the same transition and hand the create transaction to
`$mini` for remote routing. Do not ask for a second approval.

## Show the manual queue

For “покажи очередь”:

1. read `.lazypowers/tasks/*/dispatch.md`;
2. accept only schema `lazypowers.dispatch.v1` with matching folder/task ID;
3. select only `state: queued`;
4. sort by numeric prefix;
5. show `NNN — <title>`.

Do not inspect Git, platform capacity, task chats, or historical lifecycle
files while listing the queue.

## Dispatch only on command

Support:

- “запусти следующую” — select the lowest numbered queued task;
- “запусти NNN” — select that exact queued task;
- “запусти NNN и MMM” — dispatch only those named queued tasks independently.

For each selected task:

1. Fresh-read spec and dispatch. Require the exact schema, matching task IDs,
   `attempt_limit: 3`, an absolute project root, and the pre-dispatch title
   equality above. If state is `launching`, run recovery and do not call create.
   Otherwise require `state: queued`.
2. Generate one UUID and persist
   `launch_marker: lazypowers.launch.v1:<task_id>:<uuid>`,
   `client_thread_id: null`, and `state: launching`.
3. Put the exact launch marker on the first line of the create prompt.
4. Call `create_thread` exactly once.
5. If it returns `threadId`, persist it and `state: launched`.
6. If it returns only `clientThreadId`, persist that value and run automatic
   resolution.
7. Never generate another marker or call create again while the same record is
   `launching`.

Never launch the next queued task because another task appears finished.

Absent model, reasoning, base/ref, or other starting-state bindings are omitted
from the tool call so platform defaults apply. Pass only bindings explicitly
requested by the user.

## Give the Runner one complete handoff

Pass the full exact `spec.md` plus:

```text
Ты — самостоятельный Runner задачи NNN.

Выполни приложенную утверждённую спецификацию.
Используй официальный Superpowers.
Начни с superpowers:writing-plans.
Для изоляции используй platform worktree или
superpowers:using-git-worktrees.

Самостоятельно планируй, реализуй, тестируй и диагностируй ошибки.
Допускается максимум три фактические попытки конечного действия.
После первой и второй неудачи сначала используй
superpowers:systematic-debugging, исправь причину в том же worktree и только
затем начинай следующую попытку.

Не отправляй сообщения о ходе работы или результате в Product-чат.
Все вопросы, platform permissions и итог показывай пользователю только в этом
task-чате.

Утверждённая спецификация уже разрешает прямо названное в ней конечное
действие; не запрашивай повторное подтверждение Product. Обязательное
разрешение платформы запрашивай у пользователя только в этом task-чате.
```

An attempt is consumed only when the specification's final action actually
starts. Diagnosis, read-only inspection, preflight, development tests, code
changes, permission prompts, and a command authoritatively known not to have
started do not consume an attempt. Ambiguous command execution consumes the
attempt; continue only after read-only inspection establishes one safe current
state. Otherwise ask the user in the task chat.

## Handle only dispatch errors

If create authoritatively reports that no chat was created, return
`launching → queued` by atomically persisting `state: queued`,
`launch_marker: null`, and `client_thread_id: null`; report the failure and
wait for another manual dispatch command.

Automatic resolution performs no more than four fresh `list_threads` reads
within 60 seconds. Match the exact launch marker from the initial prompt.
Deduplicate candidates by exact thread ID before counting them.

- Zero distinct IDs: keep `launching`; on the next Product interaction resume
  lookup before any create. Do not ask the user to find a thread ID.
- One distinct ID: persist `task_thread_id` and `state: launched`.
- Two or more distinct IDs: keep `launching`, create no replacement, and ask
  the user to choose one exact ID.

After one final ID exists, call `set_thread_title` with exact
`dispatch.title`, fresh-read the title, and perform one idempotent rename retry
only when readback mismatches. A second mismatch keeps `launched`, reports the
title error once, and ends dispatch.

If capacity blocks create, return the task to `queued`. Do not schedule a
background retry.

## Stop after dispatch

Immediately after binding and title completion, report the created task chat
and stop all Product work for that task.

Do not read, wait for, or message the task chat after dispatch.

Do not monitor progress, receive completion messages, verify Git, integrate
branches, clean worktrees, control final actions, or record
`running|blocked|failed|done`.

The user supervises the task directly in its own chat.

## Check the focused scenarios

Read [references/scenarios.md](references/scenarios.md) when changing or
checking draft, queue, create, title, or dispatch-stop behavior. Keep exactly
its three focused scenarios and no parallel runtime test framework.
