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
```

Allow only `draft|queued|launching|launched`.

Product may revise a draft. If the user changes a queued specification, first
return it to `draft`, apply the change, and show the revised summary. Never
change a launched specification; a material follow-up is a new task. Keep
`dispatch.title` equal to `NNN — <spec title>` while the task is a draft.

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
and `state: draft`, then change only the state to `queued`.

Approval never launches a task. Do not create a task chat until the user gives
a separate manual dispatch command.

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

1. fresh-read its `spec.md` and `dispatch.md`;
2. require the exact schema, matching task IDs, `attempt_limit: 3`, an absolute
   project root, `state: queued`, and require the dispatch title to equal
   `NNN — <spec title>`;
3. write only `state: launching` before create;
4. call the platform's visible thread-creation tool exactly once in the
   specification's project root;
5. require one exact returned thread ID;
6. write that ID and `state: launched`;
7. set the visible title from `dispatch.title`;
8. report the created task chat to the user;
9. stop all Product work for that task.

Never launch the next queued task because another task appears finished.

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

When the create surface exposes model selection, request
`gpt-5.6-terra` with `high` reasoning unless the user explicitly overrides it.
If that pair is unavailable, use the platform default and mention the
substitution once in the created task chat; do not block Product dispatch.

An attempt is consumed only when the specification's final action actually
starts. Diagnosis, read-only inspection, preflight, development tests, code
changes, permission prompts, and a command authoritatively known not to have
started do not consume an attempt. Ambiguous command execution consumes the
attempt; continue only after read-only inspection establishes one safe current
state. Otherwise ask the user in the task chat.

## Handle only dispatch errors

If create authoritatively reports that no chat was created, return
`launching → queued`, report the failure, and wait for another manual dispatch
command.

If create does not yield one reliable thread ID, keep `state: launching`,
create no replacement, and ask the user to inspect visible chats and decide
whether to bind one exact chat or return the task to the queue.

If the chat exists but title assignment fails, keep `state: launched` and its
exact thread ID. Report the title failure once. Do not retry the title.

If capacity blocks create, return the task to `queued`. Do not schedule a
background retry.

## Stop after dispatch

Do not read, wait for, or message the task chat after dispatch.

Do not monitor progress, receive completion messages, verify Git, integrate
branches, clean worktrees, control final actions, or record
`running|blocked|failed|done`.

The user supervises the task directly in its own chat.

## Check the focused scenarios

Read [references/scenarios.md](references/scenarios.md) when changing or
checking draft, queue, create, title, or dispatch-stop behavior. Keep exactly
its three focused scenarios and no parallel runtime test framework.
