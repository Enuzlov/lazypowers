# Deployment approval contract — lifecycle revision 9

Use this contract for every initial or retry deployment. Product owns the
canonical task and target records; a task transcript supplies durable facts but
never writes `.lazypowers`. Preserve the Product skill's source identity,
bounded correction, model, Git, capacity, reconciliation, exactly-once, and
queue rules.

**REQUIRED REFERENCE:** Read
[deployment-lineage.md](deployment-lineage.md) before creating or accepting a
ready payload, showing Approval 2, publishing authorization, running the final
pre-command gate, accepting `DEPLOYED`, reconciling a baseline effect,
migrating history, or handling an incident. It is authoritative for the target
record, state shape, ancestry/equality modes, provenance, incident binding,
command exits, and compare-and-set order. This file is authoritative for the
approval payload, transcript-first deployment callbacks, command boundary,
retry rules, and platform-permission boundary.

## Contents

- [Keep independent durable deployment facts](#keep-independent-durable-deployment-facts)
- [Use transcript-first envelopes and stable routing](#use-transcript-first-envelopes-and-stable-routing)
- [Create the canonical approval payload](#create-the-canonical-approval-payload)
- [Accept deployment ready and request Product authority](#accept-deployment-ready-and-request-product-authority)
- [Publish and consume one authorization](#publish-and-consume-one-authorization)
- [Run bounded preflight and exactly one command](#run-bounded-preflight-and-exactly-one-command)
- [Accept the attempt result](#accept-the-attempt-result)
- [Reconcile Product-owned effects](#reconcile-product-owned-effects)
- [Keep platform permission informational](#keep-platform-permission-informational)

## Keep independent durable deployment facts

Persist at least these deployment fields in canonical `status.md`:

~~~yaml
engineering_phase: pending
deployment_phase: none
deployment_authority_consumed: false
deployment_command_started: false
target_mutation_state: not_started
deployment_attempt: 0
deployment_attempt_history: []
deploy_runner_id: null
deployment_approval_key: null
deployment_approved_action: null
deployment_approval_state: none
approval_payload: null
approval_payload_sha256: null
deployment_preflight_retry_count: 0
last_deployment_outcome: null
pending_deployment_effect: null
target_key: null
target: null
deployment_mode: null
expected_target_baseline: null
expected_target_provenance: null
incident_reconciliation_binding: null
lineage_authority_mode: null
lineage_authority_reason: null
deployed_sha_probe: null
callback_receipts: []
callback_outbox: []
~~~

Use only:

- `pending|running|completed|failed|blocked` for `engineering_phase`;
- `none|preflight_pending|preflight_completed|awaiting_approval|authorized|command_started|target_mutating|completed|failed|unknown|blocked` for `deployment_phase`;
- `not_started|started|unknown` for `target_mutation_state`;
- `none|awaiting_product_approval|approved|consumed` for
  `deployment_approval_state`;
- `null|DEPLOYED|FAILED|BLOCKED|UNKNOWN` for
  `last_deployment_outcome`.

Attempt is `0` before one exists and positive afterward. Keep
`deployment_preflight_retry_count` in `0..2`. Acceptance of the first deployable
engineering result atomically changes attempt `0` to `1`, requires empty
`deployment_attempt_history`, and sets the exact current-attempt state block
below before first preflight; no prior evidence is appended. A later
attempt is legal only after the prior terminal effect/rescan is reconciled. In
one atomic write, first
append (or require an identical already-appended entry) this exact prior-attempt
evidence; never edit or remove an older entry:

~~~yaml
deployment_attempt_history:
  - deployment_attempt: exact prior positive attempt
    deploy_runner_id: exact prior Runner
    deployment_approval_key: exact prior key
    deployment_approved_action: exact prior action or null
    approval_payload: exact prior payload or null
    approval_payload_sha256: exact prior hash or null
    authorization_receipt: exact accepted key/revision/hash triple or null
    deployment_authority_consumed: exact prior boolean
    deployment_command_started: exact prior boolean
    target_mutation_state: exact prior not_started|started|unknown
    command_start_marker: exact prior marker or null
    last_deployment_outcome: exact prior terminal outcome
    result_receipt: exact accepted key/revision/hash triple or null
    effect_retirement_marker: exact completed terminal effect/rescan marker
~~~

The same atomic write sets the current attempt state exactly:

~~~yaml
deployment_attempt: exact prior attempt plus 1
deployment_phase: preflight_pending
status: running
deploy_runner_id: null
deployment_approval_key: null
deployment_approval_state: none
deployment_approved_action: null
approval_payload: null
approval_payload_sha256: null
deployment_authority_consumed: false
deployment_command_started: false
target_mutation_state: not_started
last_deployment_outcome: null
deployment_preflight_retry_count: 0
pending_deployment_effect: null
~~~

Bind a new Runner and a new attempt-specific approval key only on the first
otherwise-valid ready callback; both must differ from prior-attempt values and
then remain immutable for the attempt. Change approval state from `none` to
`awaiting_product_approval` only after ready validation and durable generated
prompt. The reset never rewrites historical command/mutation evidence, receipts,
markers, outcomes, or payloads. Every new attempt requires a new payload/hash
and a new direct Product approval; old authority never carries forward. The
phase/status reset occurs before the new preflight so the attempt is
immediately active, visible to reconciliation, and not mistaken for the prior
terminal state.

The three execution facts are independent:

- Product atomically consumes authority with publication: one canonical write
  saves the complete authorization in pending outbox as committed publication
  state and sets `deployment_authority_consumed: true`. Transcript delivery is
  its recoverable emission. Consumed authority cannot authorize another run.
- The task publishes the command-start marker and crosses the invocation
  boundary at most once. Product projects that ordered transcript fact as
  `deployment_command_started: true`.
- `target_mutation_state` stays `not_started` only while checks prove no
  target-changing action began. Use `started` only with verifiable
  project-owned evidence. For an opaque command, set it to `unknown` as soon as
  the invocation boundary is crossed; `unknown` is monotonic and never refines
  to `not_started`.

Do not use the legacy combined external-start field for revision 9 tasks.
Retry under the same authority is possible only before the command marker,
when the transcript and checks prove both
`deployment_command_started: false` and
`target_mutation_state: not_started`.

Keep the complete immutable target authority tuple through ready, Product
prompt, authorization, task receipt, command marker, result, and baseline
effect: task, positive attempt, approval key, full approved SHA, branch,
target key/display target, snapshot mode, expected baseline, authority
mode/reason, exact approved action, probe descriptor, conditional same-SHA
provenance, conditional incident reconciliation binding, and every applicable
stage-specific state/SHA observation pair. Bind incident
evidence only as specified by the lineage reference. Never guess a null or
unknown production baseline.

## Use transcript-first envelopes and stable routing

Carry each `lazypowers.deployment-ready.v1`,
`lazypowers.deployment-authorized.v1`, `lazypowers.deployment-result.v1`, and
platform callback as the complete inner payload of a
`lazypowers.callback-envelope.v1`. Publish the envelope as a separate message
in its source transcript before any optional push hint. Validate the outer
source, inner schema/key/revision, and recomputed canonical payload SHA-256.
Store receipts as exact `{callback_key, callback_revision, payload_sha256}`.
The same triple is an exactly-once no-op after acceptance; the same
key/revision with another hash is a conflict with no state or side effect.

For task-sourced envelopes, accept source authority only from the saved task
thread, task logical host, immutable launch marker, task/project identity, and
payload hash. For Product authorization, the task applies the symmetric check
against the saved Product thread, Product logical host, launch marker,
task/project identity, and payload hash. A mutable `*_transport_host_id` is
only a current routing binding: authoritatively re-resolve it after handoff or
routing error, and never include it in business authority or substitute it for
logical identity. A unique thread-only route is permitted only when the
callable surface supports and verifies it.

Only Product writes canonical `.lazypowers/tasks/` and
`.lazypowers/targets/`. A same-host task may read only the exact canonical
paths passed in its handoff. Product freshly reads canonical target state
before generating/publishing authorization and binds it in the immutable
approved snapshot. At the final gate a same-host task rereads that passed path;
a cross-host task relies on the exact snapshot plus mandatory probe, never the
snapshot alone, and creates no connector, resolver, or Product continuation.
When receiving the result Product freshly rereads canonical target state and
alone owns CAS/promotion. Never treat a worktree `.lazypowers` as current.

Push success is not a receipt. Timeout, ambiguity, no push, or no message ID
does not block transcript pull or prove delivery. Product and task reconcile
their saved source transcripts in message order using a verified
platform-native heartbeat when available. It counts as verified only when the
callable create surface returns a stable heartbeat ID plus exact intended
source/destination, Product saves the tuple before treating it active,
authoritative list/read returns the same ID and tuple, and destination
transcript read confirms the saved same-thread marker. Ambiguous creation
forbids a second heartbeat and removes unattended claims until that saved
marker is reconciled. Without such a heartbeat, state that unattended
production continuation is unavailable and fail closed; do not add a daemon,
scheduler, polling process, or transport adapter.

## Create the canonical approval payload

After accepted engineering and successful read-only preflight, the internal
deploy-Runner creates exactly this machine payload:

~~~yaml
schema: lazypowers.deployment-approval-payload.v1
task_id: NNN-short-slug
project_identity: exact saved immutable project identity
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
deploy_runner_id: exact internal Runner ID
deployment_attempt: exact positive attempt
deployment_approval_key: NNN-short-slug:deployment:<attempt>:<full-sha>:<target>
approved_sha: exact full lowercase SHA
branch: exact saved refs/heads/... ref
target_key: exact safe target key
target: exact display target
deployment_mode: snapshot
requested_action: exact external action
expected_target_baseline: exact full lowercase SHA or null only where the lineage mode permits it
expected_target_provenance: null or exact approved same-SHA source provenance
incident_reconciliation_binding: null or exact approved complete incident binding
lineage_authority_mode: normal|initial|same_sha_redeploy|rollback|supersede|reconciliation
lineage_authority_reason: null for normal or exact exceptional reason
deployed_sha_probe:
  argv:
    - project-owned-probe
  working_directory: exact project-relative directory
  output_contract: one_full_lowercase_git_sha|repository_state_v1
approval_probe_observed_state: absent|present
approval_probe_observed_sha: null only with absent, otherwise exact full lowercase deployed SHA
changed_path_count: exact nonnegative integer
result_link: exact saved engineering result link
~~~

The probe argv shown above is illustrative, not a two-token shape. Accept any
non-empty ordered array of exact non-empty string tokens: element zero is the
executable and every later element, when present, is one exact argument. Keep
array order in the payload and hash.

Read [deployment-lineage.md](deployment-lineage.md#parse-the-two-output-contracts)
for the only normative parser definitions. For
`one_full_lowercase_git_sha`, the normalized pair is always
`present/<full SHA>`. For `repository_state_v1`, preserve its exact valid
`absent/null` or `present/<full SHA>` pair. Only valid `initial` with a matching
`uninitialized` target may use `absent/null`; all other modes require
`present/<expected factual baseline>`. The pair is part of the complete
canonical approval payload and therefore of `approval_payload_sha256`.

The key construction preserves the exact task, attempt, SHA, and display
target. Validate it as data, never as shell input. Bind
`deploy_runner_id` on the first otherwise-valid ready callback for the attempt;
all later authorization, markers, and results must match it exactly.

Compute `approval_payload_sha256` as lowercase SHA-256 of the one canonical
compact UTF-8 JSON representation of the complete payload. Apply exactly the
callback-envelope canonical JSON domain: unique string object keys recursively
sorted by Unicode scalar value, preserved array order, minimal base-10
integers, JSON null/booleans/strings/arrays/objects only, separators
`(',', ':')`, no whitespace or normalization, and the required deterministic
control-character escaping. Reject floats, non-finite numbers, duplicate keys,
leading-zero integers, `-0`, and unpaired surrogates before hashing. Never hash
YAML, Markdown fences, or prompt prose.

Generate, rather than hand-copy, the Approval 2 prompt from that payload in
this exact field order:

~~~text
Deployment Approval 2
Task: <task_id>
Attempt: <deployment_attempt>
Approved SHA: <approved_sha>
Branch: <branch>
Target: <target>
Target key: <target_key>
Action: <requested_action>
Expected baseline: <expected_target_baseline>
Observed target state: <approval_probe_observed_state>
Observed target SHA: <approval_probe_observed_sha>
Authority mode: <lineage_authority_mode>
Changed paths: <changed_path_count>
Engineering result: <result_link>
Payload SHA-256: <approval_payload_sha256>
Fingerprint: <first 12 lowercase hex characters of approval_payload_sha256>
Approve this exact payload?
~~~

Render JSON null as literal `null`, integers as minimal base-10, and strings as
their deterministic canonical JSON string literal on one line. This prevents a
newline or delimiter inside target/action/result data from changing prompt
structure. Never paraphrase a value. The user's direct reply is recorded
against the full `approval_payload_sha256`, not byte-for-byte prompt prose.
Regenerating the same prompt from the unchanged payload is harmless. Any
payload/hash change, any changed required prompt meaning, or an
absent/mismatched line requires a new prompt and direct approval.

## Accept deployment ready and request Product authority

The task publishes only this ready inner callback after branch/ref/HEAD,
read-only preflight, target lineage, and the first required probe succeed:

~~~yaml
schema: lazypowers.deployment-ready.v1
callback_key: NNN-short-slug:deployment-ready:<attempt>
callback_revision: 1
task_id: NNN-short-slug
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
deploy_runner_id: exact internal Runner ID
deployment_attempt: exact positive attempt
deployment_approval_key: exact saved key
approval_payload: exact lazypowers.deployment-approval-payload.v1 object
approval_payload_sha256: exact canonical payload hash
approval_probe_observed_state: exact state from approval_payload
approval_probe_observed_sha: exact SHA or null from approval_payload
preflight_checks: []
approval_probe:
  exit_status: 0
  checked_at: exact RFC3339 timestamp
~~~

Require the top-level observation pair to equal the pair in the approval
payload and the mode-specific factual state required by the lineage reference.
Both fields are present in the ready payload and therefore in its canonical
inner-payload hash; `approval_probe_observed_sha` is null only for valid
`initial` with state `absent`. Require the probe
descriptor to equal the current approved target record. Missing configuration,
non-zero exit, stderr or output that makes the result ambiguous, invalid
contract bytes/pair, or a mismatch fails closed before a
prompt. Never infer deployed SHA from Git history, Product main/HEAD, task
number/order, timestamps, `depends_on`, or the target record alone.

Adding or implementing a probe and updating canonical target configuration
requires a separate Product-approved project task and fresh lifecycle/approval.
This contract defines no migration/update adapter and grants no inherited
authority for either change. Keep the missing-probe task blocked until that
separate task and fresh approval complete.

On the first valid ready envelope Product atomically:

1. rereads and state-shape validates the canonical target record and runs the
   complete Approval 2 lineage/equality gate;
2. validates and recomputes the approval payload/hash and probe result;
3. stores its receipt, binds the Runner, copies `requested_action` exactly to
   `deployment_approved_action`, and stores the exact payload/hash;
4. sets phases to `preflight_completed` then `awaiting_approval`, overall
   status to `awaiting_deploy_approval`, and approval state to
   `awaiting_product_approval`;
5. saves the complete generated prompt as a pending Product effect. Product
   publishes only from that saved effect, confirms the exact transcript marker,
   and then clears it; a crash resumes identical publication.

For `normal`, stale/unknown record or ancestry blocks with exactly:

~~~text
stale delivery; integration from current target baseline required
~~~

For same-SHA, incident, and exceptional modes, enforce the exact conditional
provenance, binding, equality, strict-descendant, and reason rules from the
lineage reference. Specification approval, task-chat text, forwarded text,
silence, urgency, a push hint, an unknown source, and platform permission are
not deployment authority. The task never asks the user for Product approval.

## Publish and consume one authorization

Immediately before authorization, Product rereads canonical status and the
target record and requires the direct user reply to the generated prompt in
the saved Product thread/logical host. Require the unchanged task, positive
attempt, full SHA/ref, target tuple, action, expected baseline, first probe
state/SHA pair, authority mode/reason, conditional provenance/binding, and full
approval payload/hash.

Product first prepares this complete inner callback for a Product transcript
envelope:

~~~yaml
schema: lazypowers.deployment-authorized.v1
callback_key: NNN-short-slug:deployment-authorized:<attempt>
callback_revision: 1
task_id: NNN-short-slug
product_thread_id: exact saved Product thread
product_logical_host_id: exact saved Product logical host
source_launch_marker: exact immutable launch marker
deploy_runner_id: exact bound Runner ID
deployment_attempt: exact positive attempt
deployment_approval_key: exact saved key
approval_payload: exact unchanged approval payload
approval_payload_sha256: exact unchanged full hash
user_approval_payload_sha256: same exact full hash
authorized_action: exact requested_action from the payload
~~~

Publication and consumption are one atomic canonical status transaction: save
this complete envelope/triple as pending outbox committed publication state and
set `deployment_authority_consumed: true`, approval `consumed`, phase
`authorized`, and overall `deploy_queued`. Emit the transcript envelope only
from that state and confirm it before clearing; a crash resumes the identical
emission. There is no durable interval in which authorization is published but
reusable; authority stays consumed when transcript delivery or optional push
is absent, ambiguous, interrupted, or fails.

Before acting, the task pulls and validates the complete Product-source
envelope and its embedded payload/hash against the ready payload and saved
logical identity. It scans its transcript for the exact authorization receipt,
command marker, and result. On first acceptance it publishes an append-only
receipt containing the authorization key/revision/hash and Runner/attempt. A
duplicate of the same triple is a no-op. A key/revision hash conflict, changed
tuple, another Runner, or already-present command marker/result blocks and
never invokes a command.

The authorization receipt is task-side deduplication evidence; it is not a
new Product approval and does not require Product acknowledgment. No
deployment-phase callback or Product continuation is permitted between a
valid authorization receipt and the local preflight/command sequence.

## Run bounded preflight and exactly one command

After the authorization receipt and before any command marker, the task may
retry failed local read-only preflight at most twice after the initial try.
Each retry increments `deployment_preflight_retry_count` to `1` then `2` and
keeps attempt, authority, action, and payload/hash unchanged. Corrections use a
separate bounded counter. A retry is legal only while the transcript and
checks prove command-started false and target mutation `not_started`; it never
runs a target-changing command and needs no Product roundtrip. After the
second retry, failure produces `BLOCKED` with exact checks and no invocation.

Before the one invocation, obtain every turn-bound platform permission and
finish every other preflight step. Then run the complete final gate:

1. verify branch ref and local HEAD resolve exactly to approved SHA;
2. on the same host, reread the exact canonical Product target path passed
   read-only and compare it with the immutable authorization snapshot; on
   another host, use that exact approved snapshot plus the mandatory probe,
   never snapshot alone and without a connector, resolver, or continuation;
3. enforce the snapshot/current same-host record state shape, target tuple,
   expected baseline, conditional provenance/binding, and mode-specific
   ancestry/equality;
4. execute the exact approved project-owned probe, parse it by the lineage
   reference, and require `final_probe_observed_state`/`final_probe_observed_sha` to equal the approved
   mode-specific pair: `absent/null` for valid `initial`, otherwise
   `present/<expected factual baseline>`.

Unavailable/ambiguous same-host canonical state, record/snapshot mismatch, or
a cross-host snapshot without its matching mandatory probe fails closed before
marker/command. Report `BLOCKED` with command-started false and mutation
`not_started`.

After that final probe there is no target-changing step before the approved
command. Publish this append-only task transcript marker and immediately make
the single process invocation:

~~~yaml
schema: lazypowers.deployment-command-start.v1
marker_key: NNN-short-slug:deployment-command-start:<attempt>
task_id: NNN-short-slug
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
deploy_runner_id: exact bound Runner ID
deployment_attempt: exact attempt
deployment_approval_key: exact approval key
authorization_callback_key: exact accepted authorization key
authorization_callback_revision: exact accepted authorization revision
authorization_payload_sha256: exact accepted authorization envelope payload hash
approval_payload_sha256: exact approved payload hash
approved_sha: exact approved SHA
final_probe_observed_state: absent for valid initial, otherwise present
final_probe_observed_sha: null for valid initial, otherwise exact expected factual baseline
deployment_command_started: true
target_mutation_state: unknown for opaque; not_started only for separately approved instrumented evidence
command_invocation_count: 1
~~~

The marker is an invocation-boundary fact, not a callback and not a request
for continuation. It is append-only and must be the final transcript action
before invoking exactly the approved command once. Product later consumes it
in transcript order and never asks the task to continue it. Once it exists,
the old authority cannot be retried even if the invocation response is lost.
For an opaque command, record target mutation as `unknown` immediately after
crossing this boundary; the marker itself records `unknown`, which is monotonic
and never refines to `not_started`. For a separately approved project-owned
instrumented command, the marker may record `not_started` only when evidence
proves that fact through the boundary, then a later append-only evidence marker
may advance it to `started`. Such a command is not opaque with respect to the
mutation boundary. Never move mutation state backward and never leave an opaque
invocation as `not_started`.

For `initial`, `present/<any SHA>` at the final gate means the target appeared
before the marker: block invocation and never adopt or overwrite it. After a
successful command, immediately run the same approved probe again and require
exact `post_deployment_probe_observed_state`/`post_deployment_probe_observed_sha` of
`present/<approved SHA>`. A non-zero, malformed, unknown, absent, or mismatching
post-probe cannot report `DEPLOYED`. After a possible invocation, any missing
or mismatching pair reports `UNKNOWN`; consumed old authority is never reused.
Interruption, lost command output, possible invocation, or uncertain mutation
has the same result. There is no blocking transport roundtrip anywhere in this
execution path.

## Accept the attempt result

The three-envelope protocol also has one typed fail-closed result path before
ready or authorization. If the initial read-only preflight or mandatory first
probe fails, the task publishes this conditional
`lazypowers.deployment-result.v1` inner payload transcript-first; it does not
invent a fourth callback schema:

~~~yaml
schema: lazypowers.deployment-result.v1
callback_key: NNN-short-slug:deployment:<attempt>
callback_revision: 1
task_id: NNN-short-slug
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
deploy_runner_id: null
outcome: BLOCKED
deployment_attempt: exact positive attempt
deployment_approval_key: null
approval_payload_sha256: null # failing stage cannot yield a complete valid approval payload
authorization_callback_key: null
authorization_callback_revision: null
authorization_payload_sha256: null
approved_sha: exact intended full lowercase SHA
branch: exact intended branch
target_key: exact intended target key
target: exact intended target
deployment_mode: snapshot
expected_target_baseline: exact intended baseline or null only where lineage permits it
expected_target_provenance: null or exact intended same-SHA provenance
incident_reconciliation_binding: null or exact intended incident binding
lineage_authority_mode: exact intended mode
lineage_authority_reason: exact intended reason
approved_action: null
deployment_authority_consumed: false
deployment_command_started: false
target_mutation_state: not_started
deployment_preflight_retry_count: 0
command_start_marker: null
command_invocation_count: 0
approval_probe_observed_state: absent|present for a valid parsed approval-stage observation, otherwise null
approval_probe_observed_sha: null when state is absent or null, full SHA with present
final_probe_observed_state: null
final_probe_observed_sha: null
post_deployment_probe_observed_state: null
post_deployment_probe_observed_sha: null
failed_stage: initial_preflight|approval_probe
checks:
  - exact non-empty failing check including the stage
summary: concise blocked text
blockers: exact non-empty blocker facts
~~~

No ready receipt, prompt, approval, authorization, Runner/key binding, or
command marker may precede this variant. Product validates its exact outer
source/triple and conditional nullability, then atomically records its receipt,
`deployment_phase: blocked`, overall `status: blocked`,
`last_deployment_outcome: BLOCKED`, and one terminal receipt-marker/numeric
rescan effect. It leaves authority/command false and mutation `not_started` and
creates no baseline, authority, command, ready/prompt, or approval effect.

For every result after accepted authorization, including post-authorization
`BLOCKED`, the task publishes this complete inner callback transcript-first:

~~~yaml
schema: lazypowers.deployment-result.v1
callback_key: NNN-short-slug:deployment:<attempt>
callback_revision: 1
task_id: NNN-short-slug
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
deploy_runner_id: exact bound Runner ID
outcome: DEPLOYED|BLOCKED|FAILED|UNKNOWN
deployment_attempt: exact saved attempt
deployment_approval_key: exact approved key
approval_payload_sha256: exact approved payload hash
authorization_callback_key: exact accepted authorization key
authorization_callback_revision: exact accepted authorization revision
authorization_payload_sha256: exact accepted authorization envelope payload hash
approved_sha: exact approved SHA
branch: exact approved branch
target_key: exact approved target key
target: exact approved target
deployment_mode: snapshot
expected_target_baseline: exact approved baseline
expected_target_provenance: null or exact approved same-SHA provenance
incident_reconciliation_binding: null or exact approved incident binding
lineage_authority_mode: exact approved mode
lineage_authority_reason: exact approved reason
approved_action: exact durable approved action
deployment_authority_consumed: true
deployment_command_started: true|false
target_mutation_state: not_started|started|unknown
deployment_preflight_retry_count: 0|1|2
command_start_marker: exact marker key or null
command_invocation_count: 0|1
approval_probe_observed_state: exact approved absent|present
approval_probe_observed_sha: exact approved null|full SHA paired with state
final_probe_observed_state: absent|present|null
final_probe_observed_sha: null|full SHA paired with final state
post_deployment_probe_observed_state: absent|present|null
post_deployment_probe_observed_sha: null when state is absent or null, full SHA with present
checks: []
summary: concise text
blockers: none or exact facts
~~~

Require exact outer source/triple, bound Runner, authorization triple,
approval payload hash, target/lineage tuple, action, probe facts, command facts,
outcome consistency, and checks. Accept the attempt-specific result even
though its authority is consumed; it reports that already-authorized attempt.
Never let a later result erase an earlier command marker or mutation evidence.
Before accepting it, Product freshly reads and state-shape validates the named
canonical target record. Product alone uses that observation for the saved
baseline effect and later CAS/promotion or incident; the task never promotes.
Receipt, outcome/state, and effects form one logical exactly-once acceptance
unit. It is accepted only after every part is durably
committed or recoverably pending; separate file/transcript operations do not
claim physical cross-store atomicity.

Every post-authorization result requires the exact non-null authorization
key/revision/hash, exact non-null approval payload hash, and
`deployment_authority_consumed: true`. Therefore every non-`BLOCKED`
post-authorization outcome necessarily has that triple and consumed authority;
the pre-authorization conditional `BLOCKED` variant above is the only result
whose authorization triple is null and authority false.

Treat each stage pair as atomic and keep both named fields in every result, so
their values (including permitted nulls) are included in the canonical
inner-payload hash. The approval pair is non-null and exactly matches the
approved payload for every post-authorization result. A pre-authorization
`initial_preflight` failure has `null/null`; an `approval_probe` failure may
preserve a valid parsed `absent/null` or `present/<full SHA>` mismatch, but an
unknown/malformed observation is `null/null`. The final pair is `null/null`
only when that probe was not run or produced unknown output; a valid parsed
pair follows its state/SHA rule. The post pair is `null/null` when not run or
unknown, otherwise it preserves valid `absent/null` or `present/<full SHA>`.
Only `present/<approved SHA>` permits `DEPLOYED`. Never preserve one field without
its paired field semantics.

- `DEPLOYED` requires command-started true, marker present, invocation count
  `1`, successful command evidence, the exact valid final pair, and a
  post-deployment pair `present/<approved SHA>`. For `initial`, its final pair
  must have been `absent/null`. An opaque command may retain mutation `unknown`; matching
  post-probe proves deployed state but does not rewrite that historical fact.
  Atomically accept the receipt/outcome and create the exact Product-owned
  baseline effect from the lineage reference. Keep lifecycle nonterminal until
  promotion or incident retirement is confirmed.
- `BLOCKED` before the marker requires command-started false, mutation
  `not_started`, invocation count `0`, and exact failed preflight evidence. It
  creates no baseline effect. Exhausted local retries do not reuse authority.
- `FAILED` after invocation requires command-started true. Mutation may be
  `started|unknown`, or `not_started` only when separately approved project-owned instrumentation proves mutation remained
  `not_started` throughout invocation. That command is not opaque with respect
  to its mutation boundary. It creates no baseline effect. In both the
  instrumented `not_started` case and opaque `unknown` case, authority remains
  consumed and old-authority retry remains forbidden. A conclusively failed
  read-only preflight is `BLOCKED`, not a target failure.
- Any lost/ambiguous invocation result, possible start, opaque uncertainty,
  or missing/unknown/absent/mismatching post-probe pair is `UNKNOWN`. It creates no baseline
  effect and can never reuse old authority.

For `BLOCKED`, `FAILED`, or `UNKNOWN`, atomically persist receipt, exact
facts/outcome, terminal deployment phase, overall blocked state, and the exact
terminal numeric-rescan effect before running it. A later retry requires a new
positive attempt, key, ready payload/hash, and direct Product approval. Only
pre-command local read-only retries may remain in the same attempt.

## Reconcile Product-owned effects

At the start of every Product turn, reconcile `target_baseline_update` before
any transcript effect, callback, receipt replay, or other Product work. Then
reconcile every other pending callback/outbox/receipt-marker/rescan effect and
pending model continuation in durable/transcript order. Do not pull/apply a
newer envelope for a task while an earlier effect is unresolved. A
valid production `DEPLOYED` result may create it only when the verified
post-deployment pair is exact `present/<approved SHA>`. Apply the lineage reference's
exact expected-SHA/provenance/binding CAS and append-once incident order.

Confirmed promotion replaces the effect with one terminal numeric rescan and
sets the task done only after that transition. A CAS/provenance conflict keeps
factual `DEPLOYED`, records or confirms identical incident evidence, blocks the
original task, retires/replaces the old baseline effect with terminal rescan,
and never recreates it on replay. Recovery belongs to a separate exactly bound
Product-approved reconciliation task. `FAILED`, `BLOCKED`, `UNKNOWN`, hash
conflict, missing/mismatching probe, and preflight failure never promote or
mutate a target baseline.

For other Product transcript effects, publish from canonical Product state
first, then use optional routing only as a wake hint. Reconcile the exact
outbox marker/payload from destination transcripts, keep ambiguity pending,
and retry only after authoritative non-delivery. Receipt replay resumes the
already-saved effect but creates no second prompt, authorization, command, CAS,
incident, result write, or queue rescan.

## Keep platform permission informational

Use read-only preflight and the established approved command path/prefix. When
a turn-bound platform approval is required, publish this task-sourced inner
callback transcript-first:

~~~yaml
schema: lazypowers.platform-approval-required.v1
callback_key: NNN-short-slug:platform-approval:<attempt>
callback_revision: 1
task_id: NNN-short-slug
task_thread_id: exact saved task thread
task_logical_host_id: exact saved task logical host
source_launch_marker: exact immutable launch marker
deploy_runner_id: exact bound Runner ID
deployment_attempt: exact attempt
command_scope: sanitized command or prefix
reason: exact platform requirement
product_authority_already_granted: true|false
~~~

Validate source, Runner, attempt, stable key/revision/hash, sanitized scope,
exact reason, and truthful authority flag. Persist its receipt and optional
Product explanation while asserting that the durable deployment tuple is
unchanged. It supplies or changes no SHA, branch, approval key/payload/hash,
target, lineage tuple, action, probe, command fact, or mutation fact. A
platform permission never grants Product authority, widens command scope,
bypasses sandbox/security/SSH requirements, or authorizes Product to run the
action. Obtain it before the final pre-command probe so no permission
roundtrip separates that probe from the one command invocation.
