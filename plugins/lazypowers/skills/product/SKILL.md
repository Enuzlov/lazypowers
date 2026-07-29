---
name: product
description: Use when a Product chat needs to discuss, draft, save, revise, queue, list, or manually dispatch product specifications to separate visible Codex task chats that use official Superpowers, or create a successor Product chat. The skill owns only the disk-backed specification queue, initial task-chat handoff, and explicit Product-role handoff; it never monitors task execution after dispatch.
---

# Lazypowers Product

## Keep the boundary thin

Own only:

1. product discussion and a saved specification;
2. the manual disk-backed queue;
3. one visible task-chat create call;
4. the task title and initial Superpowers handoff.
5. one explicit Product-role handoff to a successor chat.

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

## Run the optional external-task hook

Before each direct or combined `draft → queued` approval, use this exact order:

1. Fresh-read and validate `spec.md` and `dispatch.md`: matching task ID,
   `state: draft`, and `dispatch.title == "NNN — <spec title>"`.
2. Inspect only `<project_root>/.lazypowers/external-tasks.yaml`.
3. Skip the hook when that file is absent or the exact current approval message
   contains «без задачи»; continue the ordinary approval flow without a call.
4. Otherwise require schema `lazypowers.external-tasks.v1` and a skill name
   matching `^[a-z0-9][a-z0-9-]{0,63}$`. A malformed config is one warning.
5. If that named skill is missing or unavailable, show exactly one warning; use
   no fallback or retry, then continue at step 8. Otherwise invoke that one
   installed personal skill exactly once with the exact project root, fresh
   `spec.md` and `dispatch.md` paths, `dispatch.title`, and opaque `options`.
6. Capture and validate the helper stdout internally; helper stdout is not the
   enclosing Product response. Accept exactly one JSON object and no other
   output. A `confirmed` result is exactly one object with
   `status: "confirmed"`, a non-empty opaque `external_id`, and `url` as an
   HTTPS URL with no credentials; a `warning` result is exactly one object with
   `status: "warning"` and `code` as a safe short code. Unknown or extra keys,
   unsafe URLs, multiple outputs, malformed, or ambiguous output become one
   warning.
7. Show at most one warning.
8. Fresh-read `spec.md` and `dispatch.md`, then revalidate the original task
   ID, title, and `state: draft` invariants.
9. Perform the existing `draft → queued` transition.
10. For combined approval, continue the existing dispatch in the same turn.

Provider failure never blocks queue or dispatch and never causes automatic
retry. This optional hook adds no state, callback, monitoring, or authority for
external actions.

A current direct or combined approval authorizes only the single configured
hook invocation in the current turn. It grants no broader installation, retry,
monitoring, or unrelated authority.

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

Each selected task owns an independent transaction. `launched` tasks and other
active Runner chats never block another queued task. A `launching` record blocks
only a second create for that same task and, while unresolved, Product handoff;
it does not impose a global active-task limit.

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

Если full spec содержит confirmed external-task binding и прямо просит
$<provider> complete, после успешной основной работы выполни одну best-effort
попытку через этот skill. Ошибка внешнего complete не отменяет основную работу
и не создаёт callback в Product.
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

## Hand off the Product role

Trigger this flow only when the user explicitly says «создай новый продуктовый
чат». Do not infer it from a request to launch a task, show the queue, shorten
context, or continue in another chat.

Use this exact sequence:

1. Resolve the exact absolute project root and fresh-read all applicable project
   instructions.
2. Fresh-read every canonical `.lazypowers/tasks/*/dispatch.md`. Accept only
   exact schema `lazypowers.dispatch.v1` with a matching folder/task ID.
3. For every accepted record in `launching`, run the existing task recovery:
   resume exact launch-marker lookup without a new create, deduplicate exact
   thread IDs, and finish binding/title when one final ID is known.
4. If any accepted task remains in unresolved `launching`, stop only this
   Product handoff and report which task transaction must be recovered or
   resolved. Do not call `create_thread` for a Product successor.
5. Treat `draft`, `queued`, and `launched` as nonblocking. Do not copy, rewrite,
   normalize, queue, launch, or otherwise mutate those records during handoff.
   Existing `launched` tasks and active Runner chats are irrelevant to handoff
   capacity.
6. Generate one UUID and form one chat-local marker
   `lazypowers.product-handoff.v1:<uuid>`. Do not save it in a Product session
   file or add it to a task record.
7. Put the exact marker on line 1 of the complete prompt below and call
   `create_thread` exactly once in the same exact project root. Omit model,
   reasoning, Git ref, worktree, and other starting bindings unless the user
   explicitly requested them.
8. If create returns `threadId`, use that exact ID. If it returns only
   `clientThreadId`, retain that opaque value in chat context and perform no
   more than four fresh `list_threads` reads within 60 seconds. Match the exact
   Product marker in the initial prompt and deduplicate candidates by exact
   thread ID.
9. Bind one distinct ID automatically. For two or more distinct IDs, create no
   replacement and ask the user to choose one exact ID. For zero, keep the
   handoff ambiguous, create no replacement, and retain the current Product
   role. Later recovery resumes lookup for the same marker and never calls
   create again.
10. After one final ID exists, call `set_thread_title` with exact title
    `Lazypowers Product`, read back the visible title, and allow one idempotent
    rename retry only when it mismatches. If the second readback still
    mismatches, report the title failure and retain the current Product role;
    do not create a replacement.
11. Only after unique binding and exact naming, report the successor, stop all
    Product mutations in the old chat, and end. Do not read, wait for, message,
    or monitor the successor after handoff.

Use this complete create prompt, substituting only the exact marker and project
root:

```text
lazypowers.product-handoff.v1:<uuid>

Ты — новый Product-чат проекта <exact absolute project root>.
Используй $lazypowers:product и прими Product-роль только для этого проекта.

До любой Product-мутации:
1. fresh-read прочитай все применимые инструкции проекта;
2. прочитай утверждённый source of truth командой
   git show refs/heads/main:docs/spec.md;
3. fresh-read прочитай канонические .lazypowers/tasks/ и принимай только
   валидные lazypowers.dispatch.v1.

Не считай снимок очереди из этого prompt авторитетным: очередь не копируется и
не передаётся, её единственный источник — канонические файлы проекта.
Сохраняй независимость task-create транзакций и не вводи глобальное ограничение
«одна активная задача».

Не читай и не мониторь старый Product-чат, существующие task-чаты или Runner.
Не создавай callback, heartbeat, Product session state или фоновую работу.
Обсуждай следующие спецификации и выполняй Product-команды только по прямым
сообщениям пользователя в этом чате.
```

This handoff does not change task state except when completing recovery of a
task-create transaction that was already `launching` before the handoff
command. It never creates `.lazypowers/product-session.md`, a queue snapshot,
callback, heartbeat, monitoring loop, or global task lock.

## Check the focused scenarios

Read [references/scenarios.md](references/scenarios.md) when changing or
checking draft, queue, create, title, or dispatch-stop behavior. Keep exactly
its three focused scenarios and no parallel runtime test framework.
