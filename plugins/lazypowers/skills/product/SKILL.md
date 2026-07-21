---
name: product
description: Use when a Product chat discusses, approves, queues, launches, escalates, reconciles, verifies, deploys, or reports product work through visible Codex task threads and official Superpowers, especially when task-level model routing, durable Markdown state, cross-host callbacks, exact Git delivery, or deployment approval are involved.
---

# Lazypowers Product

## Own the lifecycle

Own Product conversation, canonical state, queue, reconciliation, and approval.
Assign each approved task to one visible isolated-worktree Codex task chat; keep
official Superpowers and all internal agents inside that chat.
Require two distinct user gates:

1. Obtain explicit approval of the saved specification revision before launch.
2. For deployment, obtain a new direct Product-thread approval of the exact
   generated approval payload after engineering and preflight verification.

Let the task decide approved-scope details; request Product authority only for
changed scope/result/authority, security, access, or irrecoverable error. Do not build a runtime, state service, poller, deployment
adapter, database/daemon/scheduler, Docker/SSH layer, universal connector,
duplicate pipeline, dispatcher, or long-lived wait.
## Keep Product state canonical

Treat `.lazypowers/tasks/` and `.lazypowers/targets/` in the canonical Product
checkout as the only current state. Product is their only writer. Resolve and
save its logical host and absolute root; never let a task worktree read its
snapshot `.lazypowers` as current or write canonical Product state. Pass exact
canonical paths read-only only on the same host. Product freshly reads
canonical target state before generating/publishing authorization. A same-host
task rereads only the explicitly passed path; a cross-host task relies on the
exact immutable approved snapshot plus the mandatory target-state probe, never
snapshot alone, and creates no connector, resolver, or continuation roundtrip.
When receiving the result Product freshly reads canonical state and alone owns
CAS/promotion.
Use one numeric task folder:

~~~text
.lazypowers/tasks/NNN-short-slug/
  spec.md
  status.md
  result.md  # only the first accepted terminal engineering result
~~~
Keep an approved `spec.md` immutable. Write at least this revision 10 shape to
`status.md`; use `null` until a value exists:

~~~yaml
{task_id: NNN-short-slug, project_identity: null, status: draft, engineering_phase: pending, deployment_phase: none,
 deployment_authority_consumed: false, deployment_command_started: false, target_mutation_state: not_started, pending_callback_effect: null,
 queue_order: NNN, depends_on: [], queue_reason: null, superseded_by: null, supersede_reason: null, product_thread_id: null, product_logical_host_id: null, product_transport_host_id: null,
 task_thread_id: null, task_logical_host_id: null, task_transport_host_id: null, source_launch_marker: null, pending_client_thread_id: null, thread_cursor: null,
 pending_task_title_effect: null,
 product_state_host_id: null, product_state_root: null, approved_state_payload_sha256: null, governance_source: null, legacy_governance_policy: null, governance_choice_requested: false,
 base_commit: null, branch: null, final_commit: null, callback_correction_attempts: 0, callback_receipts: [], callback_outbox: [], product_heartbeat: null, task_heartbeat: null,
 model_policy: auto, model_override: null, initial_model: null, initial_reasoning: null, initial_model_selection_reason: null, selected_model: null, selected_reasoning: null,
 model_selection_reason: null, model_escalation_count: 0, pending_model_continuation_key: null, pending_model_continuation_outbox: null,
 deployment_attempt: 0, deploy_runner_id: null, deployment_attempt_history: [], deployment_approval_key: null, deployment_approved_action: null, deployment_approval_state: none,
 approval_payload: null, approval_payload_sha256: null, deployment_preflight_retry_count: 0, last_deployment_outcome: null, pending_deployment_effect: null,
 target_key: null, target: null, deployment_mode: null, expected_target_baseline: null, expected_target_provenance: null, incident_reconciliation_binding: null,
 lineage_authority_mode: null, lineage_authority_reason: null, deployed_sha_probe: null}
~~~
Use `pending|running|completed|failed|blocked` for `engineering_phase`;
`none|preflight_pending|preflight_completed|awaiting_approval|authorized|command_started|target_mutating|completed|failed|unknown|blocked`
for `deployment_phase`; and `not_started|started|unknown` for mutation. Set
overall `done` only after accepted terminal engineering for non-deploy work, or
accepted terminal deployment plus confirmed promotion/no-promotion transition.

Use only
`draft|approved|queued|running|awaiting_deploy_approval|deploy_queued|deploying|done|blocked`
for overall `status`, `none|awaiting_product_approval|approved|consumed` for
`deployment_approval_state`, and `null|DEPLOYED|FAILED|BLOCKED|UNKNOWN` for
`last_deployment_outcome`. Use deployment attempt `0` before one exists and a
positive integer afterward; keep read-only preflight retries between `0` and
`2`.
Report the exact phase (`engineering completed`, `preflight completed`,
`deployment authorized`, `deployment command started`, `target mutation started`,
or `deployment completed`); say complete only when overall `status: done`.

Store receipts as exact `{callback_key, callback_revision, payload_sha256}` and
outbox entries with that triple, source, and state. Store heartbeats as `{id,
intended_source_thread_id, intended_destination_thread_id, same_thread_marker,
creation_state}` where state is only `pending|active|ambiguous`.
## Resolve governance before launch

Allow only `null|lazypowers|legacy` for `governance_source` and
`null|read_only_constraints_only|authoritative_lifecycle` for
`legacy_governance_policy`. Accept only paired `null/null`,
`lazypowers/read_only_constraints_only`, or
`legacy/authoritative_lifecycle`; fail closed on a mixed pair.

When both legacy `.codex/orchestration` and `.lazypowers` exist with no saved
choice, atomically keep `engineering_phase: pending`, set `status: queued` and
exact `queue_reason: governance precedence not saved`, create no heartbeat or
task thread, and set `governance_choice_requested: true` when asking. If it is
already true, do not ask again. A direct Lazypowers choice atomically saves
exactly:

~~~yaml
governance_source: lazypowers
legacy_governance_policy: read_only_constraints_only
~~~

and saves/resumes one deterministic full queue-rescan effect. With Lazypowers
selected, ignore legacy lifecycle, queue, callback, and deployment state
machines, while still obeying technical, security, and project constraints
through the normal Codex instruction hierarchy.
A direct legacy-authority choice atomically saves:

~~~yaml
governance_source: legacy
legacy_governance_policy: authoritative_lifecycle
engineering_phase: blocked
status: blocked
queue_reason: governance delegated to legacy orchestration
~~~

Perform no Lazypowers lifecycle action afterward. Never delete or rewrite
legacy rules implicitly.
## Publish every callback transcript-first

Publish every engineering, model, correction, authorization, deployment, and
platform callback as a separate complete message in its source transcript
before any `send_message_to_thread` call. A summary or raw tool result is not a
callback. Wrap the complete typed inner payload as:

~~~yaml
schema: lazypowers.callback-envelope.v1
callback_key: exact inner callback key
callback_revision: exact inner callback revision
payload_schema: exact inner schema
payload_sha256: 64 lowercase hex
source_task_id: exact task ID
source_thread_id: exact source thread ID
source_logical_host_id: exact stable logical host
source_launch_marker: exact immutable launch marker
project_identity: exact saved immutable project identity
outbox_state: pending
payload:
  schema: exact typed callback schema
  callback_key: same key
  callback_revision: same revision
  # complete typed payload; no elision
~~~

For every task-sourced envelope, require `source_logical_host_id` to equal the
exact saved `task_logical_host_id`.

Compute `payload_sha256` from SHA-256 of the inner payload's single canonical
compact UTF-8 JSON representation:

- Allow only null, booleans, strings of valid Unicode scalar values, arrays,
  objects with unique string keys, and integers; reject floats, non-finite
  numbers, duplicate keys, unpaired surrogates, leading-zero integers, and
  `-0` before hashing.
- Sort object keys recursively by Unicode scalar value; preserve array order.
- Use minimal base-10 integers, recursively sorted keys, and separators exactly
  `(',', ':')` with no whitespace.
- Do not normalize strings. Escape quote, backslash, backspace, tab, LF, form
  feed, and CR as `\"`, `\\`, `\b`, `\t`, `\n`, `\f`, and `\r`; encode other
  U+0000-U+001F as lowercase `\u00xx`. Do not optionally escape slash,
  printable ASCII, or non-ASCII; encode them directly as UTF-8.
- Never hash YAML, Markdown fences, or human-readable prose.

Replay of the same key/revision/hash is a no-op after its first receipt; the
same key/revision with another hash conflicts without effect. Accept a higher
revision only when it equals the exact next revision durably requested in the
Product correction outbox for that phase-stable key; resend keeps revision/hash.
Engineering and model-escalation always start at revision exactly `1`.

Use push only as an optional wake hint, never a receipt. Timeout, ambiguity,
absence, or missing message ID cannot change correctness. Do not invent a
transport timeout or wait for push before considering the envelope published.
## Reconcile through source transcript pull

At the start of every Product turn and on each verified Product heartbeat:

1. Read canonical Product state and reconcile pending target effects first as
   required by the lineage reference.
2. Reconcile `pending_task_title_effect` from its saved exact thread/task/title
   tuple. Retry only the identical saved effect until authoritative readback
   confirms that exact thread/title, then clear it. Failure or ambiguity keeps
   the same effect and never creates a replacement chat. Because title is only
   UX metadata, an unresolved title effect does not block callback acceptance,
   queue order, Git verification, or any other correctness transition.
3. Reconcile every other pending callback/outbox/receipt-marker/rescan effect
   and every `pending_model_continuation_key` in durable/transcript order.
   Authoritative non-delivery permits only identical retry; do not read/apply a
   newer envelope until the earlier effect/retry is fully reconciled.
4. Read the saved transcripts of every unfinished task, including commentary
   produced before a source turn becomes terminal. Use platform wait snapshots
   only to locate new transcript state; read the source transcript for the
   complete envelope.
5. Accept only an envelope read from that transcript after schema, canonical
   source, payload hash, business, model, and applicable Git verification.
   Complete all fail-closed result-shape validation before receipt, status,
   cursor, result creation, effect, or queue advancement.
6. For the first valid terminal engineering envelope, build only this result
   object with exactly the three top-level keys shown:

   ~~~yaml
   schema: lazypowers.result.v1
   accepted_callback_envelope: exact complete terminal engineering envelope object
   receiver_verification: exact deterministic verification object
   ~~~

   `accepted_callback_envelope` is one object, never an array. The deterministic
   verification object has exact `source_identity_verified`,
   `payload_sha256_verified`, `business_fields_verified`, and
   `model_fields_verified` all `true`, plus `git` with exact `base_commit`,
   `base_commit_exists`, `final_commit`, `final_commit_exists`, `sole_parent`,
   `branch_ref`, `branch_ref_commit`, and `direct_changed_paths` sorted by raw
   UTF-8. Changed COMPLETE uses verified values; other outcomes use null for
   final/existence/parent/ref commit. Serialize by the envelope canonical JSON
   rules as UTF-8, append exactly one LF, and use no BOM. Include no transcript
   cursor, timestamp, time, tool/run ID, or prose. The first valid terminal
   engineering `result.md` is immutable; hash these exact bytes as
   `result_sha256`.
7. Apply the full pre-effect boundary separately to every negative result
   form. Reject `accepted_callback_envelopes` before receipt, status, cursor,
   result, effect, or queue advancement. If `accepted_callback_envelope` is an
   array, reject it before receipt, status, cursor, result, effect, or queue
   advancement. Reject a second accepted envelope before receipt, status,
   cursor, result, effect, or queue advancement. Reject any extra top-level key
   before receipt, status, cursor, result, effect, or queue advancement. Reject
   every rewrite or append attempt before receipt, status, cursor, result,
   effect, or queue advancement. Reject any deployment or later-lifecycle
   mutation of the engineering result before receipt, status, cursor, result,
   effect, or queue advancement. A same key/revision with a different hash
   fails closed before receipt, status, cursor, result, effect, or queue
   advancement. None may partially create or advance any Product state/effect.
8. Atomically create the exact result bytes. Existing bytes and
   `result_sha256` must match; otherwise fail closed without a Product effect.
   Then atomically write status with the receipt triple, phase/overall state,
   `thread_cursor`, and one `pending_callback_effect` binding result hash,
   exact receipt marker, and deterministic terminal-rescan marker.
9. Do not consider it accepted until receipt/state/result and every applicable
   marker/rescan effect are durably committed or recoverably pending. Recover
   a crash between physical writes by revalidating the envelope and
   identical existing result, then completing status without rewrite. Publish
   the Product receipt marker only from the saved effect, confirm that exact
   marker by reading the Product transcript, idempotently run/resume rescan
   from its saved marker, and clear the effect only after both confirmations.
10. On replay, prove byte-identical result bytes, one receipt, one terminal
   rescan, and no repeated queue advancement. Do not rewrite `result.md`,
   repeat an effect, or change counters. Missing text, unavailable state, and
   temporary errors remain inconclusive and never permit replacement launch.

Keep every later lifecycle event in canonical status/outbox/effect/target
records. Deployment evidence may reference `result.md` and `result_sha256`, but
deployment must never mutate, change, rewrite, or append `result.md`. Reject
such an attempt before receipt, status, cursor, result, effect, or queue
advancement. Historical revision 9
state, including task 014 incident evidence, is immutable: never migrate,
normalize, or rewrite it into the revision 10 result shape.

For every Product transcript effect, atomically save state plus the complete
pending outbox/effect as its committed publication state, then emit it to the
transcript. Reconcile a crash by replaying the identical saved effect and
confirm transcript evidence before clearing; physical file writes and
transcript emission are not one transaction.

Base acceptance on saved task/thread/logical host, project, immutable marker,
and envelope hash. Transport host is mutable routing, never authority:
authoritatively re-resolve it before routing/after handoff without changing
logical identity. Use thread-only routing only when uniquely supported and the
saved thread/marker verify.

While unfinished tasks or unaccepted outbox envelopes exist, create at most one
temporary platform-native Product heartbeat. Treat it as active only when the
callable create surface returns a stable heartbeat ID plus exact intended
source/destination, Product saves that tuple and a deterministic same-thread
marker before any unattended claim, authoritative list/read returns the same
ID/tuple, and destination-transcript read confirms that marker. Ambiguous
creation sets `creation_state: ambiguous`, forbids a second heartbeat, and
removes the unattended claim until the saved marker is authoritatively
reconciled. Stop only after terminal reconciliation and no pending envelope.
For Product-to-task correction or authorization, first save the complete
canonical envelope as a pending Product outbox, then publish it from that state;
let the task pull and verify it via an optional push hint or its own verified
platform-native heartbeat.

If the callable platform lacks a verifiable same-thread heartbeat, state that
unattended continuation is unavailable: leave production deployment blocked
and recover ordinary callbacks at the next Product turn. Never replace this
fallback with a daemon, polling process, or claim of unattended operation.

## Retire only a never-launched superseded task

Retirement is a Product-only canonical-state transition; a Runner never
applies it. Retirement identical replay is a no-op only when every saved
relation, reason, phase, status, and queue-reason value matches exactly;
conflicting evidence fails closed without a partial effect. Treat the relation
and reason as validated Product-owned retirement
input, not as preexisting task state or a pre-retirement migration. Require
`superseded_by` to identify an exact terminal superseding task whose overall
status is `done`, and validate the exact reason before one atomic write:

~~~yaml
superseded_by: exact terminal task ID
supersede_reason: exact validated Product-owned reason
~~~

Also require every never-launched/clean-state predicate:

~~~yaml
task_thread_id: null
pending_client_thread_id: null
source_launch_marker: null
callback_receipts: []
deployment_authority_consumed: false
deployment_command_started: false
target_mutation_state: not_started
pending_task_title_effect: null
~~~

Also require no Product or task heartbeat, no `result.md`, empty callback
outbox, no pending callback or model-continuation effect, and no pending
deployment effect. Retirement creates no heartbeat, result, callback, receipt,
outbox, deployment attempt/effect, target effect, task chat, or queue
advancement. Any such lifecycle artifact or any mismatched supersede
relation/reason fails closed.

When every input and predicate matches, one atomic status write persists only:

~~~yaml
superseded_by: exact validated terminal task ID
supersede_reason: exact validated Product-owned reason
engineering_phase: blocked
deployment_phase: none
status: blocked
queue_reason: exact same validated supersede_reason
~~~

Retirement identical replay checks all six saved values and is a no-op only
when every value matches exactly. Any conflicting saved relation, reason,
phase, status, or queue reason fails closed without a partial effect.

After valid acceptance of `015-internal-pilot-stabilization`, Product may apply
this transition only to `012-lazypowers-public-release-v2`, with
`superseded_by: 014-lazypowers-public-release-v3` and this exact saved reason:

~~~text
superseded by completed release path 014-lazypowers-public-release-v3; retired after stabilization 015-internal-pilot-stabilization
~~~

Do not migrate or normalize any other historical revision 9 task, including
006, 008, 010, 011, 013, or 014. Keep task 014 `result.md` unchanged as
immutable incident evidence.
## Decide readiness, lineage, and model exactly

Treat a task as ready only when dependencies are `done`, paths do not conflict,
its worktree is isolatable, and authoritative current capacity covers the chat
plus internal agents. Process ready folders numerically; without unambiguous
capacity use sequential mode and save exactly
`authoritative capacity unavailable; sequential fallback`.

**REQUIRED REFERENCE:** Read
[references/deployment-lineage.md](references/deployment-lineage.md) before a
deployable launch, Approval 2, the final pre-command gate, target promotion,
migration, reconciliation, or incident handling. Apply its exact target
record, mode, ancestry/equality, CAS, provenance, evidence, and retirement
rules. Require the saved base to equal or descend from the current target
baseline before launch; fail closed on missing, stale, ambiguous, invalid, or
Git-error evidence.

Use an exact user `model_override` only if the callable create-thread surface
offers that model/reasoning pair and reasoning is at least `high`; otherwise
leave the task queued and all initial/selected fields null. With
`model_policy: auto`, select exact `gpt-5.6-terra + high` for bounded standard
work, allowing only `gpt-5.6-sol + high` as a quality-increasing fallback.
Select exact `gpt-5.6-sol + high` for orchestration/lifecycle,
architecture/system-boundary, auth/security/secrets, persistent-state,
migration/rollback, production/external action, data-loss risk, and other
frontier work. Use another frontier candidate only when the callable surface
uniquely identifies it as current and it supports `high`.

Before launch, save the same exact pair and concrete selection reason to both
immutable `initial_*` and current `selected_*`. Never silently substitute,
lower reasoning, or recompute selection on a later turn.

## Launch exactly one task chat

Before `create_thread`, authoritatively resolve and save immutable
`project_identity` and the intended stable `task_logical_host_id`. Also save a
verified full lowercase 40-hex base commit, one literal valid
`refs/heads/...` branch, Product thread/logical host, canonical state
location/hash, governance decision, model history, and applicable target
tuple. Leave `task_transport_host_id: null` until thread creation resolves its
current transport binding. Reject unsafe refs and treat identifiers as data.

Create only when pending and final task identities are all null. Include an
immutable unique launch marker binding task ID, exact `project_identity`, exact
`task_logical_host_id`, Product thread/logical host, and
`approved_state_payload_sha256`. Create on that intended logical task host.
Save returned client ID immediately; authoritatively resolve one thread ID and
save its current transport binding separately as `task_transport_host_id`.
Zero matches remain pending; multiple matches block resolution. Any saved
temporary/final identity forbids a replacement chat.

After authoritative resolution of the one exact `task_thread_id`, atomically
save one title effect as `pending_task_title_effect`, bound to that thread ID,
exact task ID, and this computed title; then apply it only to that thread:

~~~text
NNN — <title from approved spec frontmatter>
~~~

Take `NNN` from the exact task ID and the title text verbatim from immutable
approved `spec.md`. Exact task 015 title is
`015 — Стабилизация перед внутренним пилотом`. Identical replay is a no-op.
Failure or ambiguity keeps the same pending effect for identical retry and
never creates a replacement chat. Clear the effect only after authoritative
read confirms the exact thread/title. The visible title is UX metadata only: it
is never callback/source identity, never enters a canonical payload/hash, and
is not a callback acceptance, queue-order, Git-verification, or correctness
gate.

Handoff the approved spec, exact base/ref, `project_identity`, exact saved
`task_logical_host_id`, Product source identity, immutable launch marker/state
hash, model history, capacity mode, target tuple, and read-only canonical paths
when applicable. Require the Runner to begin with `superpowers:writing-plans`,
use official Superpowers throughout, and create the expected branch from the
saved base. For changed delivery, require exactly one final delivery commit
whose sole parent is the saved base and whose saved branch ref resolves to it.
For a valid no-change delivery, require no delivery commit,
`final_commit: null`, and `changed_paths: []`. Forbid push, PR, merge,
deployment, migration,
rollback, or other external action without the later exact authority.

Require the Runner to publish the full `lazypowers.engineering.v1` payload in a
transcript-first envelope under phase-stable key `[task_id]:engineering`. It
has this exact inner shape; push afterward is optional:
~~~yaml
schema: lazypowers.engineering.v1
callback_key: NNN-short-slug:engineering
callback_revision: 1 first; otherwise exact saved next correction revision
task_id: exact task ID
project_identity: exact saved immutable project identity
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
outcome: COMPLETE|BLOCKED|FAILED|NEEDS_PRODUCT_DECISION
base_commit: exact saved full SHA
branch: exact saved refs/heads/... ref
final_commit: full SHA or null
changed_paths: []
checks: []
summary: concise text
blockers: none or exact facts
model_policy: exact saved policy
model_override: null or exact saved override
initial_model: exact saved initial model
initial_reasoning: exact saved initial reasoning
initial_model_selection_reason: exact saved initial reason
selected_model: exact saved current model
selected_reasoning: exact saved current reasoning
model_selection_reason: exact saved current reason
model_escalation_count: 0 or 1
~~~

`COMPLETE` with changes requires full final SHA and exact direct paths; no-change `COMPLETE` requires null final and `[]`. Other outcomes require null
final, `[]`, exact blockers, and no Git effect. On valid terminal acceptance
map `engineering_phase/deployment_phase/status` exactly: non-deploy COMPLETE
`completed/none/done`; deployable COMPLETE atomically creates initial attempt `1`
from `0` with empty attempt history and `completed/preflight_pending/running`;
FAILED `failed/none/blocked`; BLOCKED `blocked/none/blocked`; NEEDS_PRODUCT_DECISION
`blocked/none/blocked` with exact blockers/decision requirement. Every mapping
creates the same deterministic result/pending receipt-marker/rescan effect.

## Verify Git and bounded corrections

For every non-null SHA require `^[0-9a-f]{40}$`; for changed delivery run
option-safe read-only checks:

~~~bash
git cat-file -e <base_commit>^{commit}
git cat-file -e <final_commit>^{commit}
git rev-list --parents -n 1 <final_commit>
git show-ref --verify --hash <saved-branch-ref>
git diff-tree --no-commit-id --name-only -r <final_commit>
~~~

Require both objects, exactly one final parent equal to the sole saved base,
branch stdout exactly equal to final commit, and callback `changed_paths` as a
set exactly equal to the direct `diff-tree` paths. For no-change delivery,
require `final_commit: null` and `changed_paths: []`. Fail closed on every
missing object, invalid ref, mismatch, ambiguous output, or Git error. Do not
rerun the engineering pipeline.

Treat schema/source/hash/business/Git/model mismatches as mechanical
corrections in the same task chat. Request at most two corrections. The first
callback revision is exactly `1`; each request atomically saves the exact next
revision in a Product transcript-first correction outbox for the same phase key
and increments `callback_correction_attempts`. Accept only that saved next value;
keep the task running until valid or exhausted, then block with evidence. Reset
the counter to `0` in the atomic acceptance of any valid engineering or model
escalation callback. Do not ask the user to repair it or create a replacement.

Allow at most one same-chat Terra-to-Sol continuation, only after official
Superpowers systematic debugging and only for
`repeated_debugging_failure|unexpected_architectural_complexity` with concrete
evidence. Accept only this exact transcript-first inner payload:

~~~yaml
schema: lazypowers.model-escalation.v1
callback_key: NNN-short-slug:model-escalation
callback_revision: 1 first; otherwise exact saved next correction revision
task_id: exact task ID
project_identity: exact saved immutable project identity
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
current_model: gpt-5.6-terra
current_reasoning: high
requested_tier: frontier
reason: repeated_debugging_failure|unexpected_architectural_complexity
evidence:
  - exact attempted check and result
~~~

Require exact source/project/marker, saved Terra/high, count `0`, permitted
reason, and non-empty evidence after systematic debugging; malformed fields use
bounded corrections. Before routing atomically save
`pending_model_continuation_key: [task_id]:model-escalation:1` and a pending
outbox containing exactly that marker, `selected_model: gpt-5.6-sol`,
`selected_reasoning: high`, exact `model_selection_reason`, and
`model_escalation_count: 1`. Only confirmation of that exact marker in the same
task transcript atomically copies intended values to current `selected_*`/count
and clears both pending fields; `initial_*` never change. Ambiguity never sends
a duplicate; retry the identical continuation only after authoritative
non-delivery. Do not apply terminal engineering while pending is unresolved.
Scope/authority/branch/security changes need Product authority. Never open a
new chat or allow a second escalation.

## Continue deployment without a started roundtrip

**REQUIRED REFERENCE:** Read
[references/deployment-approvals.md](references/deployment-approvals.md) before
creating or accepting `lazypowers.deployment-ready.v1`, generating the Product
prompt, publishing or accepting `lazypowers.deployment-authorized.v1`, handling
its task-side receipt, retrying preflight, publishing a command-start marker,
or accepting `lazypowers.deployment-result.v1`. Apply its exact payload,
approval, attempt, probe, command, outcome, and permission rules. Also read the
lineage reference at every target gate.

Treat specification approval, forwarded text, a push hint, and platform
permission as no deployment authority. Generate human approval prose from the
canonical `approval_payload`; bind the direct Product-thread user reply to its
`approval_payload_sha256`. Show task/attempt, full SHA/ref, target/key, action,
expected baseline and observed probe state/SHA pair, authority mode, changed-path count,
result link, and short payload fingerprint. Any machine payload/hash or
required meaning change requires new approval.

Atomically consume authority with publication of the exact authorization:
one canonical status write saves the complete envelope in pending outbox as
committed publication state and sets `deployment_authority_consumed: true`.
Transcript delivery is a recoverable emission only from that saved outbox; a
crash resumes the identical emission while authority stays consumed. The task
must pull and validate the envelope, then publish an append-only task-side
receipt for its exact key/revision/hash before acting. A duplicate authorization
is a no-op and never invokes the command twice.

Require the approved project-owned read-only `deployed_sha_probe` before the
Product prompt, immediately before the command with no target-changing step in
between, and after a successful command. Apply the exact parser and mode/stage gates from the lineage reference.
Preserve `approval_probe_observed_state`/`approval_probe_observed_sha`, `final_probe_observed_state`/`final_probe_observed_sha`, and `post_deployment_probe_observed_state`/`post_deployment_probe_observed_sha` in approval, ready, marker, result, and canonical hashes as required by the approval reference.
Accept `deployed_sha_probe.argv` as any non-empty ordered array of exact
non-empty string tokens: index `0` is the executable and zero or more later
tokens are arguments. Treat every shown argv as illustrative, not
prescriptive, and preserve token order.
Keep `one_full_lowercase_git_sha` unchanged and default, normalized to
`present/<full SHA>`; do not migrate existing records or histories. Allow
`repository_state_v1` only by its exact canonical JSON+LF parser. Before
approval and command, valid `initial` requires `absent/null`; every other mode
requires `present/<factual baseline>`. After a successful command require
`present/<approved SHA>` for `DEPLOYED`. Block target appearance before the
initial marker; after possible invocation, missing/mismatching state is
`UNKNOWN` and consumed old authority is never reused.

Adding or implementing a probe and updating canonical target configuration
requires a separate Product-approved project task and fresh lifecycle/approval.
This task defines no migration/update adapter and grants no inherited authority
for either change. Keep the missing-probe task blocked until that separate task
and fresh approval complete.

Preserve the three-envelope protocol when initial read-only preflight or the first mandatory probe fails before valid ready/approval. Publish a typed `lazypowers.deployment-result.v1` `BLOCKED` branch with exact `failed_stage: initial_preflight|approval_probe`, non-empty failing check and blocker;
null Runner/key/authorization triple and `approval_payload_sha256` because those stages cannot yield a complete valid payload, authority/command false, mutation `not_started`, null marker,
and invocation count `0`. Product accepts it transcript-first and atomically records receipt, `deployment_phase: blocked`, overall `status: blocked`,
`last_deployment_outcome: BLOCKED`, and one terminal receipt-marker/rescan effect, with no ready/prompt, baseline, authority, or command effect.
Every non-BLOCKED post-authorization result requires the exact non-null authorization triple and consumed authority true.

After authorization receipt, allow at most two local read-only preflight
retries, run the final probe/lineage gate, publish an append-only command-start
marker, set `deployment_command_started: true` immediately before process
invocation, invoke at most once, probe afterward, and publish the result
envelope. Do not require a `deployment-started -> Product continuation`
roundtrip. Set `target_mutation_state: started` only from verifiable
project-owned evidence; for an opaque command after invocation use `unknown`.
That `unknown` fact is monotonic and never refines to `not_started`.

Retry under the same authority only when transcript and checks prove
`deployment_command_started: false` and `target_mutation_state: not_started`.
Any command-started `true` or mutation `started|unknown` requires a new positive
attempt, payload/hash, and direct Product approval. Preserve factual `UNKNOWN`
and fail closed. A terminal `FAILED` may combine command-started `true` with
mutation `not_started` only when separately approved project-owned
instrumentation proves mutation remained not-started throughout invocation;
such a command is not opaque with respect to its mutation boundary. In both
instrumented failure and opaque `unknown`, authority remains consumed and the
old authority cannot be retried. Promote baseline only as the Product-owned
exactly-once effect of an accepted `DEPLOYED` result with matching post-deploy
probe where required.

Create a new attempt in one atomic reset after appending immutable prior attempt evidence (Runner/key, approval/authorization receipts, command marker,
consumed/command/mutation facts, outcome, result receipt/effect). Set attempt to prior+1 and exactly: `deploy_runner_id`, approval key/action/payload/hash,
outcome/effect null; approval state `none`; authority/command false; mutation `not_started`; retries `0`; `deployment_phase: preflight_pending`; overall
`status: running`. Do this before the new preflight so the attempt is active and reconcilable. Bind new Runner/key only on first valid ready and await
approval only after saved prompt. Never erase prior mutation evidence; require a new payload/hash and direct approval.
## Use the behavioral scenarios

**REQUIRED REFERENCE:** Read
[references/scenarios.md](references/scenarios.md) before changing or dry-running
launch, transcript outbox, reconciliation, heartbeat, identity rebinding,
queue, governance, deployment, or fail-closed behavior. Keep exactly its three
focused lifecycle scenarios; do not add a parallel runtime test pipeline.
