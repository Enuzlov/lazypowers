# Сценарии Product skill — lifecycle revision 10

Ровно три dry-run сценария ниже являются executable oracle для lifecycle
revision 10. Они не отправляют сообщения, не запускают task-чаты, не выполняют
deployment и не меняют внешние системы. `e428f1...` — сохранённый approved base
task 015. Повторяющиеся `b...`, `d...`, `e...`, fixture identity и fixture
thread IDs — только иллюстративные значения правильного формата; это не
утверждение о реальном final delivery, Git ref, callback или production target.
Реальный delivery обязан подставить фактические значения и пересчитать hashes.

## Сценарий 1: singular engineering result и identical replay

### Given: schema-complete terminal engineering envelope

Product сохранил exact task/thread/logical-host/launch identity, approved base
`e428f1a63b9a9ecbdf080de688abe5a67b6ced08` и literal branch. Task сначала
публикует в собственном transcript отдельный полный envelope. Следующий блок —
schema-complete fixture; `payload_sha256` вычисляется только от canonical
compact UTF-8 JSON inner `payload`, не от YAML presentation или fence.

~~~yaml
schema: lazypowers.callback-envelope.v1
callback_key: 015-internal-pilot-stabilization:engineering
callback_revision: 1
payload_schema: lazypowers.engineering.v1
payload_sha256: 6dd2074d45aa33ef1ce0d8c01fb8363c0fc4fcddbdcd23b3bb78905c9151a563
source_task_id: 015-internal-pilot-stabilization
source_thread_id: task-015-thread-fixture
source_logical_host_id: local
source_launch_marker: 015-internal-pilot-stabilization:launch:1:product-thread-fixture
project_identity: task015-project-identity-fixture
outbox_state: pending
payload:
  schema: lazypowers.engineering.v1
  callback_key: 015-internal-pilot-stabilization:engineering
  callback_revision: 1
  task_id: 015-internal-pilot-stabilization
  project_identity: task015-project-identity-fixture
  task_thread_id: task-015-thread-fixture
  task_logical_host_id: local
  source_launch_marker: 015-internal-pilot-stabilization:launch:1:product-thread-fixture
  outcome: COMPLETE
  base_commit: e428f1a63b9a9ecbdf080de688abe5a67b6ced08
  branch: refs/heads/codex/lazypowers-internal-pilot-stabilization
  final_commit: dddddddddddddddddddddddddddddddddddddddd
  changed_paths:
    - README.md
    - docs/spec.md
    - docs/superpowers/plans/2026-07-21-internal-pilot-stabilization.md
    - plugins/lazypowers/.codex-plugin/plugin.json
    - plugins/lazypowers/skills/product/SKILL.md
    - plugins/lazypowers/skills/product/agents/openai.yaml
    - plugins/lazypowers/skills/product/references/scenarios.md
  checks:
    - plugin and skill validators PASS
    - every fenced YAML block parses
    - exact base and final commit objects exist
    - final commit has exactly one parent equal to saved base
    - saved literal branch ref resolves exactly to final commit
    - direct diff-tree path set equals changed_paths
    - git diff --check PASS
  summary: Lifecycle revision 10 local engineering snapshot is ready.
  blockers: none
  model_policy: auto
  model_override: null
  initial_model: gpt-5.6-sol
  initial_reasoning: high
  initial_model_selection_reason: lifecycle and persistent-state release-safety work
  selected_model: gpt-5.6-sol
  selected_reasoning: high
  model_selection_reason: lifecycle and persistent-state release-safety work
  model_escalation_count: 0
~~~

Product читает envelope только из exact saved source transcript и полностью
проверяет outer/inner schema и triple, source identity, project, launch marker,
canonical payload hash, business/model fields и Git. Для changed `COMPLETE`
оба commit objects существуют; final имеет ровно одного parent, равного saved
base; literal branch ref равен final; direct `diff-tree` paths как множество
равны `changed_paths`. Поскольку `d...` — fixture, эти Git-команды являются
oracle формы, а не заявлением, что fixture commit существует:

~~~bash
git cat-file -e e428f1a63b9a9ecbdf080de688abe5a67b6ced08^{commit}
git cat-file -e dddddddddddddddddddddddddddddddddddddddd^{commit}
git rev-list --parents -n 1 dddddddddddddddddddddddddddddddddddddddd
git show-ref --verify --hash refs/heads/codex/lazypowers-internal-pilot-stabilization
git diff-tree --no-commit-id --name-only -r dddddddddddddddddddddddddddddddddddddddd
~~~

### When: exact singular result bytes and recoverable ordering

До любого Product effect receiver проверяет будущую форму result. Exact
top-level key set равен только `schema`, `accepted_callback_envelope`,
`receiver_verification`; singular value — ровно один object, never array. Ниже
schema-complete result object fixture. Внутренний envelope повторён полностью,
чтобы fixture не зависел от YAML anchors, elision или внешнего текста.

~~~yaml
schema: lazypowers.result.v1
accepted_callback_envelope:
  schema: lazypowers.callback-envelope.v1
  callback_key: 015-internal-pilot-stabilization:engineering
  callback_revision: 1
  payload_schema: lazypowers.engineering.v1
  payload_sha256: 6dd2074d45aa33ef1ce0d8c01fb8363c0fc4fcddbdcd23b3bb78905c9151a563
  source_task_id: 015-internal-pilot-stabilization
  source_thread_id: task-015-thread-fixture
  source_logical_host_id: local
  source_launch_marker: 015-internal-pilot-stabilization:launch:1:product-thread-fixture
  project_identity: task015-project-identity-fixture
  outbox_state: pending
  payload:
    schema: lazypowers.engineering.v1
    callback_key: 015-internal-pilot-stabilization:engineering
    callback_revision: 1
    task_id: 015-internal-pilot-stabilization
    project_identity: task015-project-identity-fixture
    task_thread_id: task-015-thread-fixture
    task_logical_host_id: local
    source_launch_marker: 015-internal-pilot-stabilization:launch:1:product-thread-fixture
    outcome: COMPLETE
    base_commit: e428f1a63b9a9ecbdf080de688abe5a67b6ced08
    branch: refs/heads/codex/lazypowers-internal-pilot-stabilization
    final_commit: dddddddddddddddddddddddddddddddddddddddd
    changed_paths:
      - README.md
      - docs/spec.md
      - docs/superpowers/plans/2026-07-21-internal-pilot-stabilization.md
      - plugins/lazypowers/.codex-plugin/plugin.json
      - plugins/lazypowers/skills/product/SKILL.md
      - plugins/lazypowers/skills/product/agents/openai.yaml
      - plugins/lazypowers/skills/product/references/scenarios.md
    checks:
      - plugin and skill validators PASS
      - every fenced YAML block parses
      - exact base and final commit objects exist
      - final commit has exactly one parent equal to saved base
      - saved literal branch ref resolves exactly to final commit
      - direct diff-tree path set equals changed_paths
      - git diff --check PASS
    summary: Lifecycle revision 10 local engineering snapshot is ready.
    blockers: none
    model_policy: auto
    model_override: null
    initial_model: gpt-5.6-sol
    initial_reasoning: high
    initial_model_selection_reason: lifecycle and persistent-state release-safety work
    selected_model: gpt-5.6-sol
    selected_reasoning: high
    model_selection_reason: lifecycle and persistent-state release-safety work
    model_escalation_count: 0
receiver_verification:
  source_identity_verified: true
  payload_sha256_verified: true
  business_fields_verified: true
  model_fields_verified: true
  git:
    base_commit: e428f1a63b9a9ecbdf080de688abe5a67b6ced08
    base_commit_exists: true
    final_commit: dddddddddddddddddddddddddddddddddddddddd
    final_commit_exists: true
    sole_parent: e428f1a63b9a9ecbdf080de688abe5a67b6ced08
    branch_ref: refs/heads/codex/lazypowers-internal-pilot-stabilization
    branch_ref_commit: dddddddddddddddddddddddddddddddddddddddd
    direct_changed_paths:
      - README.md
      - docs/spec.md
      - docs/superpowers/plans/2026-07-21-internal-pilot-stabilization.md
      - plugins/lazypowers/.codex-plugin/plugin.json
      - plugins/lazypowers/skills/product/SKILL.md
      - plugins/lazypowers/skills/product/agents/openai.yaml
      - plugins/lazypowers/skills/product/references/scenarios.md
~~~

Product сериализует этот object по canonical payload rules как compact UTF-8
JSON, добавляет ровно один LF byte `0x0a`, не добавляет BOM и атомарно создаёт
`result.md`. JSON object keys рекурсивно сортируются; array order сохраняется;
whitespace отсутствует; domain/escaping/integer rules совпадают с callback
hashing. Result содержит один complete accepted engineering envelope и не
содержит cursor, timestamp, wall clock, tool/run ID или human prose. Для exact
fixture выше `result_sha256` равен
`e0b22783b09c00df93e38e6c0d18f4b4f29d881ecb6452f6ec5166656f4c731f` и
пересчитывается из этих exact JSON+LF bytes.

Logical acceptance unit соблюдает recoverable порядок:

1. Сначала атомарно создать exact `result.md` или подтвердить byte-for-byte
   identity и saved `result_sha256`; mismatch блокирует всё без rewrite.
2. Затем отдельной atomic `status.md` write сохранить один receipt triple,
   phases/status, source `thread_cursor` и один `pending_callback_effect`,
   связывающий receipt с result hash, exact Product receipt marker и
   deterministic terminal-rescan marker.
3. Только из saved effect опубликовать marker, подтвердить exact transcript
   read, idempotently завершить/resume rescan по saved marker и после обоих
   подтверждений очистить effect.

Crash после шага 1 заново формирует те же engineering bytes, подтверждает
identity и завершает status write без rewrite. Acceptance не объявляется до
durable commit или recoverably pending состояния всех частей.

### Then: exact replay counters and full pre-effect negative matrix

После первого acceptance и после identical replay exact
`callback_key + callback_revision + payload_sha256` oracle остаётся:

~~~yaml
result_write_count: 1
result_byte_identity: true
result_sha256_unchanged: true
receipt_count: 1
receipt_marker_publish_count: 1
terminal_rescan_count: 1
queue_advancement_count: 1
replayed_effect_count: 0
engineering_phase: completed
deployment_phase: none
status: done
~~~

Replay не переписывает result, не меняет bytes/hash/counters, не повторяет
receipt/effect/rescan и не продвигает очередь второй раз.

Каждая строка следующей таблицы — самостоятельный full pre-effect assertion.
Она блокируется **до receipt, status, cursor, result, effect или queue
advancement** и не создаёт частичного Product state:

| Invalid candidate | Exact fail-closed boundary |
|---|---|
| plural key `accepted_callback_envelopes` | reject before receipt, status, cursor, result, effect or queue advancement |
| singular `accepted_callback_envelope` whose value is an array | reject before receipt, status, cursor, result, effect or queue advancement |
| a second accepted envelope in the same result | reject before receipt, status, cursor, result, effect or queue advancement |
| any extra/fourth top-level key | reject before receipt, status, cursor, result, effect or queue advancement |
| rewrite or append after the first accepted result | reject before receipt, status, cursor, result, effect or queue advancement |
| deployment callback or later lifecycle event attempting to mutate, change, rewrite or append engineering `result.md` | reject before receipt, status, cursor, result, effect or queue advancement |
| same callback key/revision with a different payload hash | conflict and reject before receipt, status, cursor, result, effect or queue advancement |

## Сценарий 2: transcript pull recovery, title effect и superseded retirement

### Given: one exact resolved task thread and one pending title effect

Product calls the authoritative resolver only for the saved pending client
identity. Exactly one match resolves `task_thread_id: task-015-thread-fixture`;
zero remains pending, multiple matches fail closed, and any saved client/final
identity forbids replacement launch. Only **after authoritative resolution of
that one exact `task_thread_id`** Product atomically saves exactly one effect:

~~~yaml
pending_task_title_effect:
  task_thread_id: task-015-thread-fixture
  task_id: 015-internal-pilot-stabilization
  title: 015 — Стабилизация перед внутренним пилотом
  application_state: pending
task_chat_create_count: 1
replacement_chat_create_count: 0
~~~

The number comes from exact task ID and the title text verbatim from immutable
approved spec frontmatter. Publication applies only to that exact thread.
Identical retry is a no-op. Failure or ambiguous title application retains the
same pending effect with the same thread/task/title tuple for a later identical
retry and creates no replacement or second chat. Product clears the effect
only after authoritative readback confirms exact thread/title.

Visible title is UX-only metadata. It never participates in callback/source
identity, canonical payload/hash, queue order, Git verification or callback
acceptance, and unresolved title is not a correctness gate.

### When: push absent or ambiguous, transcript pull accepts once

Task publishes the complete scenario-1 engineering envelope in its saved
source transcript before any optional push. Push is deliberately absent or its
delivery is ambiguous: no message ID, timeout, ambiguity and even success do
not create a receipt. On the next Product turn or verified platform-native
heartbeat, Product reads the exact saved source transcript without waiting for
terminal thread state. It verifies the complete envelope and Git evidence,
creates the singular result once, saves one receipt/effect, confirms one marker
and performs one terminal rescan. The title effect may still be unresolved and
does not block this acceptance.

~~~yaml
transcript_recovery_oracle:
  source_thread_id: task-015-thread-fixture
  push_state: absent_or_ambiguous
  source_transcript_reads: 1_or_more
  accepted_result_count: 1
  engineering_receipt_count: 1
  terminal_rescan_count: 1
  queue_advancement_count: 1
  replacement_chat_create_count: 0
~~~

Identical later pulls use the same receipt triple and preserve the exact
scenario-1 bytes/counters. Source identity is saved thread/logical host,
immutable launch marker, task/project and payload hash—not transport host,
push delivery or visible title.

### Then: one Product-only retirement of an exact never-launched task

Only after valid acceptance of task 015, Product—not a Runner—evaluates task
`012-lazypowers-public-release-v2`. Validated Product-owned inputs are exact:

~~~yaml
superseded_by: 014-lazypowers-public-release-v3
superseder_overall_status: done
supersede_reason: superseded by completed release path 014-lazypowers-public-release-v3; retired after stabilization 015-internal-pilot-stabilization
~~~

The superseder resolves to exactly one terminal task with overall `status:
done`; relation and reason match the Product decision. Before the atomic write,
the never-launched task satisfies **every** negative predicate:

~~~yaml
task_thread_id: null
pending_client_thread_id: null
source_launch_marker: null
callback_receipts: []
deployment_authority_consumed: false
deployment_command_started: false
target_mutation_state: not_started
product_heartbeat: null
task_heartbeat: null
callback_outbox: []
pending_callback_effect: null
pending_model_continuation_key: null
pending_model_continuation_outbox: null
pending_deployment_effect: null
pending_task_title_effect: null
result_md_exists: false
~~~

Any non-matching predicate, heartbeat, existing result, receipt/outbox,
pending callback/model/deployment/title effect, authority, command marker or
mutation blocks retirement without partial state. No pre-retirement migration
copies the Product-owned input into task fields.

With all predicates true, one atomic status write persists exactly these six
values:

~~~yaml
superseded_by: 014-lazypowers-public-release-v3
supersede_reason: superseded by completed release path 014-lazypowers-public-release-v3; retired after stabilization 015-internal-pilot-stabilization
engineering_phase: blocked
deployment_phase: none
status: blocked
queue_reason: superseded by completed release path 014-lazypowers-public-release-v3; retired after stabilization 015-internal-pilot-stabilization
~~~

The retirement effect creates no task chat, replacement chat, Product/task
heartbeat, `result.md`, callback, receipt, outbox, model continuation,
deployment attempt/effect, target effect or queue advancement. Exact replay
compares all six saved values and is one no-op; mismatched relation, reason,
phase, status or queue reason fails closed. Therefore `retirement_write_count`,
retired-task queue-effect count and every forbidden-artifact count remain `1`,
`0` and `0` respectively. Historical tasks 006, 008, 010, 011, 013 and 014,
including task 014 incident result, are never migrated or rewritten.

## Сценарий 3: deployment isolation и revision 9 production gates

### Given: immutable engineering result and valid initial target

This is an independent deployable fixture; it does not change task 015's
`deploy_allowed: false` local-engineering scope. It begins after its own valid
singular engineering acceptance. Product pins that fixture's canonical
JSON+LF `result.md` bytes and hash. The repeated `c...` hash below is
illustrative and is not scenario 1's computed hash or a real delivery hash:

~~~yaml
engineering_result_guard:
  result_sha256: cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
  result_write_count: 1
  top_level_keys:
    - schema
    - accepted_callback_envelope
    - receiver_verification
  accepted_callback_envelope_type: object
~~~

The deployment fixture uses illustrative `b...` approved SHA, `d...`
approval-payload hash and `e...` authorization-payload hash. None is a real
delivery, Product authority, authorization or production observation.
Product is the only writer of canonical target state. Initial mode starts from
this exact valid `uninitialized` record and approved project-owned read-only
probe:

~~~yaml
deployment_fixture:
  task_id: 021-initial-production
  project_identity: example-production-project-fixture
  product_thread_id: product-thread-fixture
  product_logical_host_id: local
  task_thread_id: task-production-021-fixture
  task_logical_host_id: local
  source_launch_marker: 021-initial-production:launch:1:product-thread-fixture
  deploy_runner_id: deploy-runner-021-fixture
  deployment_attempt: 1
  approved_sha: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
  branch: refs/heads/codex/initial-production-021
  target_key: production-eu
  target: Production EU
  lineage_authority_mode: initial
  lineage_authority_reason: explicit first deployment to an uninitialized target
  deployed_sha_probe:
    argv:
      - scripts/read-repository-state
      - production-eu
    working_directory: ops
    output_contract: repository_state_v1
canonical_target_before_approval:
  schema: lazypowers.target-lineage.v1
  target_key: production-eu
  target: Production EU
  deployment_mode: snapshot
  lineage_state: uninitialized
  current_deployed_sha: null
  source_task_id: null
  source_deployment_attempt: null
  source_callback_key: null
  updated_at: null
  incident_evidence: []
  deployed_sha_probe:
    argv:
      - scripts/read-repository-state
      - production-eu
    working_directory: ops
    output_contract: repository_state_v1
~~~

Before the Product prompt, exit `0` probe stdout is exactly canonical JSON+LF
`{"state":"absent","sha":null}\n`; parser yields `absent/null`. Product freshly
reads the same canonical target record. The generated canonical approval
payload contains `approval_probe_observed_state: absent`,
`approval_probe_observed_sha: null`, exact task/attempt/Runner, approved
SHA/ref, target/key, action, expected null baseline/provenance, mode/reason,
probe descriptor, changed-path count and exact `result_link`; it does not add
`result_sha256`. The `lazypowers.deployment-ready.v1` envelope separately
carries `approval_payload_sha256` for that canonical approval payload.

Ready acceptance writes its receipt only to `status.md`, stores its pending
receipt-marker effect/outbox there, and leaves engineering result bytes/hash
unchanged. Prompt rendering itself is not authority.

### When: separate Product approval, lineage gates and one command

Specification approval, task text, forwarded text, push hint, ready callback
and platform permission are not deployment authority. Only a **new direct
exact Product approval** in the saved Product thread/logical host, bound to the
fresh canonical `approval_payload_sha256`, exact positive attempt, full SHA/ref,
target/key, action, baseline/probe pair, authority mode and provenance/binding,
authorizes this fixture.

Product freshly rereads canonical target state and atomically publishes
authorization: one canonical status write both stores the complete
`lazypowers.deployment-authorized.v1` envelope in pending outbox and sets
`deployment_authority_consumed: true`. Transcript emission is recoverable only
from that outbox. Task pulls/validates it and appends its exact authorization
receipt; duplicate authorization is a no-op and never invokes twice.

All three lineage gates remain mandatory:

1. Before launch, saved base equals current baseline or verified Git ancestry
   proves baseline is its ancestor; missing/Git-error evidence blocks.
2. Before Product approval, fresh canonical record, expected baseline,
   provenance/binding, approved-SHA mode relation and first factual probe all
   match.
3. Immediately before command, same-host task rereads only the explicitly
   passed canonical path (cross-host uses exact immutable approved snapshot),
   always repeats the factual probe and mode-specific ancestry/equality, and
   permits no target-changing step between gate and command. Snapshot alone is
   never evidence; Product freshly rereads canonical state again on result.

Exactly one lineage mode is allowed. `normal` requires a strict descendant of
the current baseline; `initial` requires exact valid `uninitialized` state;
`same_sha_redeploy` requires SHA equality plus exact bound source provenance;
`rollback` and `supersede` require their explicit exceptional reason and exact
expected current SHA; an incident accepts only `reconciliation` with complete
immutable evidence/provenance binding. `uninitialized`, `current` and
`incident` target records must each have their full exact state shape; partial
or unknown records receive no exceptional bypass.

Platform permission is a separate non-delegable system boundary in the task
chat. Its informational callback validates source/attempt/key/revision,
sanitized command scope, reason and fact of Product authority, but cannot
supply or change approved SHA, approval hash/key, target/action or lineage
tuple. Permission neither replaces nor repeats Product approval.

After authorization receipt, task may make at most two local read-only
preflight retries. Final probe again returns exact `absent/null`. With no
target-changing step afterward, task publishes one append-only command-start
marker immediately before invoking the process:

~~~yaml
schema: lazypowers.deployment-command-start.v1
marker_key: 021-initial-production:deployment-command-start:1
task_id: 021-initial-production
task_thread_id: task-production-021-fixture
task_logical_host_id: local
source_launch_marker: 021-initial-production:launch:1:product-thread-fixture
deploy_runner_id: deploy-runner-021-fixture
deployment_attempt: 1
deployment_approval_key: 021-initial-production:deployment:1:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb:Production EU
authorization_callback_key: 021-initial-production:deployment-authorized:1
authorization_callback_revision: 1
authorization_payload_sha256: eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee
approval_payload_sha256: dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd
approved_sha: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
final_probe_observed_state: absent
final_probe_observed_sha: null
deployment_command_started: true
target_mutation_state: unknown
command_invocation_count: 1
~~~

One opaque command runs exactly once. Historical mutation state remains
`unknown`; it never refines backward to `not_started`. Successful command is
followed by the same approved probe returning exact canonical JSON+LF
`{"state":"present","sha":"bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}\n`.
The transcript-first deployment result contains `DEPLOYED`, attempt and full
authorization triple with the same exact illustrative
`authorization_payload_sha256: eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee`,
the same distinct exact illustrative
`approval_payload_sha256: dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd`,
initial mode/null baseline, approval and final `absent/null`, post
`present/<approved SHA>`, consumed authority true, command started true,
mutation unknown, exact marker and invocation count 1. Neither hash supplies
authority: the direct Product approval remains bound to the approval-payload
hash, while the authorization callback triple independently binds its payload.

### Then: isolated receipts, one promotion and unchanged result

Ready, authorization and deployment-result evidence never enters
`result.md`. It remains only in `status.md` receipts, callback outbox/effects,
attempt history and Product-owned target lifecycle. Every record may reference
the immutable engineering `result_sha256`, but cannot mutate its bytes.

~~~yaml
deployment_isolation_oracle:
  engineering_result_write_count: 1
  engineering_result_byte_identity: true
  engineering_result_sha256_unchanged_after_ready: true
  engineering_result_sha256_unchanged_after_authorization: true
  engineering_result_sha256_unchanged_after_result: true
  ready_receipt_count_in_status: 1
  authorization_publication_count_in_outbox: 1
  authorization_receipt_count_in_status: 1
  deployment_result_receipt_count_in_status: 1
  command_invocation_count: 1
  target_baseline_effect_count: 1
  baseline_promotion_count: 1
  terminal_deployment_rescan_count: 1
~~~

After full result/source/hash/business/permission/lineage validation, Product
freshly rereads the same uninitialized target. One accepted `DEPLOYED` result
with matching post-probe creates one Product-owned `target_baseline_update`.
Expected-state CAS promotes it once to `current`, approved SHA and exact new
source task/attempt/callback provenance. Confirmation replaces the baseline
effect with one terminal numeric rescan. Replay repeats neither receipt,
command, CAS, promotion, incident nor rescan, and result hash stays unchanged.

### Deployment fail-closed matrix

Each row is an independent revision 9 gate retained by revision 10. Before a
command marker, failure is `BLOCKED` with no invocation/effect. After possible
invocation, uncertainty is `UNKNOWN`, no baseline effect, consumed authority
is retained, and old authority is never reused.

| Input or state | Exact oracle |
|---|---|
| malformed/truncated JSON, JSON non-object, invalid UTF-8 or BOM | probe unknown; no ready/prompt/command before authorization |
| leading/trailing whitespace, extra stdout, multiple JSON values/lines, missing LF, CRLF or extra LF | probe unknown; exact canonical JSON plus one LF required |
| missing, duplicate, reordered or unknown probe key | probe unknown; no inferred state/SHA |
| `absent` with SHA; `present` with null/missing/short/uppercase/non-hex SHA | invalid state/SHA pair; fail closed |
| non-zero exit, interruption, unavailable/ambiguous result or invalid GitHub auth | unknown; auth failure never means absence |
| unsupported output contract | fail closed; only `one_full_lowercase_git_sha` or `repository_state_v1` |
| pre-ready initial preflight or approval probe failure | typed `BLOCKED` result with exact failed stage, null Runner/key/authorization triple and approval hash, authority/command false, mutation `not_started`, null marker, invocation `0`; no ready/prompt/baseline/authorization effect |
| initial + valid uninitialized + approval `absent/null` | may render Product prompt; still no authority |
| initial approval `present/<any SHA>` | `BLOCKED` before prompt; never adopt SHA |
| normal, same_sha_redeploy, rollback, supersede or reconciliation with `absent/null` | `BLOCKED`; each requires factual `present/full lowercase SHA` |
| invalid target shape, descriptor drift, stale provenance/binding or mode ancestry/equality mismatch | fail closed at applicable lineage gate |
| approved initial `absent/null`, final probe `present/<any SHA>` | target appeared before command; no marker, command false, mutation `not_started`, invocation `0`; never convert/adopt |
| target-changing action before final probe or between final gate and marker | ordering violation; no valid invocation authority |
| final-gate failure after consumed authority but before marker | terminal `BLOCKED`; no invocation, old authority only retryable when transcript proves command false and mutation `not_started` |
| post probe `present/<approved SHA>` plus successful command evidence | `DEPLOYED`; exactly one baseline effect eligible |
| post probe `absent/null`, `present/<different SHA>`, malformed/unavailable output or ambiguous command after possible invocation | `UNKNOWN`; no baseline effect or automatic retry |
| duplicate authorization or result replay | exactly-once no-op; never second command/receipt/promotion/rescan |
| command started true with opaque mutation boundary | mutation stays `unknown`; old consumed authority cannot retry |
| command started true and mutation `not_started` without separately approved project instrumentation proving the entire interval | invalid `FAILED`; opaque invocation cannot claim not-started |
| Product authority without current sandbox/security/SSH/turn permission | external action remains blocked at platform boundary |
| target CAS observes another SHA/provenance | append incident once, retain factual `DEPLOYED`, block task, terminal rescan; recovery requires separate Product-approved reconciliation task |

`one_full_lowercase_git_sha` remains unchanged/default for existing projects:
exit `0` with exactly one full lowercase 40-hex SHA normalizes to
`present/<same SHA>`; it never represents absence and cannot authorize initial.
Existing records/histories are not migrated. Missing/multiple/short/uppercase
SHA, extra output, non-zero or unavailable results remain unknown.

No new attempt may erase historical mutation evidence. After a terminal
attempt, Product first appends complete immutable evidence, then one atomic
reset creates positive attempt+1 with null Runner/key/action/payload/hash/effect,
approval state `none`, authority/command false, mutation `not_started`, retry
count `0`, phase `preflight_pending` and overall `running`. A new execution
needs a new Runner/key/payload/hash and another separate direct Product
approval. These Markdown scenarios create no runtime pipeline, parser,
adapter, connector, probe, wrapper, daemon, scheduler or external action.
