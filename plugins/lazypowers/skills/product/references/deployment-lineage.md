# Snapshot target deployment lineage — lifecycle revision 9

Use this contract for every Git-backed snapshot deployment target. The target
record is the only historical baseline authority; Product/main, local HEAD,
commit time, task order, `depends_on`, task status, and task-local
`base_commit` are not substitutes. For revision 9 execution, a matching
project-owned target-state probe is also mandatory factual evidence. Neither
the record nor the probe may be guessed from the other.

Only Product writes the target record in canonical Product state. A task
worktree never treats its snapshot `.lazypowers` as current. Same-host tasks
may use an explicitly passed canonical path read-only. Product freshly reads
the record before generating/publishing authorization and binds the immutable
approved snapshot. Cross-host tasks use that snapshot plus the mandatory
target-state probe, never snapshot alone, and create no connector, resolver, or
continuation. Product freshly rereads canonical state on result receipt and
alone owns CAS/promotion. Mutable transport bindings affect routing only.

## Contents

- [Validate the target record](#validate-the-target-record)
- [State-shape invariants](#state-shape-invariants)
- [Serialize launch and check the engineering base](#serialize-launch-and-check-the-engineering-base)
- [Require exact authority before Approval 2](#require-exact-authority-before-approval-2)
- [Require the target-state probe at each gate](#require-the-target-state-probe-at-each-gate)
- [Recheck immediately before the command](#recheck-immediately-before-the-command)
- [Promote the baseline crash-safely](#promote-the-baseline-crash-safely)
- [Reconcile existing histories](#reconcile-existing-histories)
- [Interpret ancestry commands fail-closed](#interpret-ancestry-commands-fail-closed)

## Validate the target record

Store one Markdown record at `.lazypowers/targets/<target_key>.md` with this
exact YAML contract:

~~~yaml
schema: lazypowers.target-lineage.v1
target_key: example-production
target: Example production
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
    - project-owned-probe
  working_directory: exact project-relative directory
  output_contract: one_full_lowercase_git_sha
~~~

Require `target_key` to match `^[a-z0-9][a-z0-9._-]*$` before using it as a
filename. Treat the display `target` only as data, never as a path or shell
expression. Require every non-null SHA in target, task, approval, and callback
state to match `^[0-9a-f]{40}$`. Use only
`uninitialized|current|incident` for `lineage_state` and only `snapshot` for
`deployment_mode`.

For a revision 9 production target, require `deployed_sha_probe` to be a
non-null mapping with a non-empty array of exact non-empty string argv tokens,
an exact project-relative working directory, and literal output contract
`one_full_lowercase_git_sha` or `repository_state_v1`. Reject an absolute directory, `..` traversal,
shell-composed argv, missing executable, unknown fields that make execution
ambiguous, or a probe descriptor that differs from the approved tuple. Treat
argv tokens as data and invoke them directly without a shell. The probe belongs
to the user project; Lazypowers supplies no SSH/API adapter or probe runtime.
The one-token argv above is illustrative: accept any non-empty ordered token
array, with the executable at index zero and zero or more exact argument tokens
after it.

### Parse the two output contracts

Keep `one_full_lowercase_git_sha` unchanged and default for existing configured
projects: successful unambiguous stdout is exactly one full lowercase 40-hex
Git SHA. Normalize that successful observation to `state: present` and that
full SHA. Do not migrate or rewrite existing target records or callback
histories.

For `repository_state_v1`, require exit `0` and stdout bytes equal to exactly
one of these two parametric byte forms followed by exactly one LF byte:

~~~text
{"state":"absent","sha":null}\n
{"state":"present","sha":"<sha>"}\n
~~~

Here `\n` denotes the single final byte `0x0a`; the two characters backslash
and `n` are not stdout. In the present form, `<sha>` is a metavariable replaced
by any value matching exactly `^[0-9a-f]{40}$`; the angle brackets are not
stdout, and no other bytes are permitted.

The object has exactly the keys `state` then `sha`; `absent` pairs only with
JSON null, and `present` pairs only with a full lowercase 40-hex SHA. This
probe-output serialization is a distinct exact contract from the recursively
sorted canonical JSON used for callback payload hashing: do not sort these
probe keys. The approved order is `state` then `sha`; reordered `sha,state`
remains invalid. Reject as
unknown every other byte sequence or semantic pair, including a BOM, invalid
UTF-8, CRLF, missing/final-extra LF, leading/trailing whitespace, reordered,
missing, duplicate, or unknown keys, multiple JSON values, malformed JSON,
short/uppercase/non-hex SHA, `absent` with a SHA, and `present` with null or no
SHA. Non-zero exit, interruption, descriptor drift, or unavailable/ambiguous
output is also unknown. An unknown observation never supplies a state or SHA
and always fails closed.

### State-shape invariants

Always validate cross-fields before launch and reconciliation, and again at Approval
2, each pre-action gate, migration, and CAS. Invalid shapes are unknown and
fail closed; no exceptional mode bypasses this predicate.

- `uninitialized` requires null current SHA, all three source fields null,
  `updated_at: null`, and empty `incident_evidence`.
- `current` requires a full SHA, non-empty source task/callback, positive source
  attempt, and non-null RFC3339 timestamp. Historical incident evidence may
  remain.
- `incident` requires non-empty `incident_evidence` and non-null RFC3339
  timestamp. Null current SHA requires all source fields null; full current SHA
  requires the complete source triplet and positive attempt.

Thus uninitialized requires updated_at: null; current requires a full SHA,
complete source provenance, and a non-null timestamp; incident requires
non-empty incident_evidence and a non-null timestamp. Partial provenance,
evidence/state contradiction, or timestamp violation always fails closed.
The lineage-state predicate above is unchanged by the probe addition: probe
validity is an independent revision 9 execution gate and never repairs an
invalid lineage shape.

For a non-deploy task, keep `target_key`, `target`, `deployment_mode`,
`expected_target_baseline`, `lineage_authority_mode`, and
`lineage_authority_reason`, `expected_target_provenance`, and
`incident_reconciliation_binding` null. Before launching deployable work, require an
explicit safe key, exact display target, `deployment_mode: snapshot`, expected
baseline, authority mode/reason, and for revision 9 production the exact probe
descriptor. Require the record key, target, mode, and probe descriptor to equal
the task tuple. Missing record, malformed schema or SHA, unsafe key, and
key/target/mode/probe mismatch always fail closed and cannot receive an
exceptional bypass. Only a valid matching `uninitialized` record may use
`initial`. A valid matching `incident` record may enter only an explicitly
approved `reconciliation`: bind the complete canonical evidence identity list, factual full SHA, and
complete source triplet in immutable `incident_reconciliation_binding`, use
`expected_target_provenance: null`, and require an explicit non-null reason. A
null-SHA incident cannot launch deployment until separate
Product authority establishes factual state without guessing.

Use this exact conditional mapping in status, approval/callback tuple, and
baseline effect:

~~~yaml
incident_reconciliation_binding:
  factual_current_deployed_sha: exact full lowercase incident SHA
  factual_source_task_id: exact incident source task ID
  factual_source_deployment_attempt: exact positive incident source attempt
  factual_source_callback_key: exact incident source callback key
  incident_evidence_identities:
    - every canonical incident evidence identity in deterministic order
~~~

Every `incident_evidence` entry is a structured mapping of all identity fields
plus `recorded_at`; never store or trust an `identity` field. Independently
derive identity by excluding `recorded_at` and serializing the remaining mapping
as compact UTF-8 JSON with lexicographically sorted keys and separators
`(',', ':')`. Reject duplicate derived identities. Sort the complete list by raw
UTF-8 bytes; this is stable bytewise serialization. At launch, Approval 2, the
final pre-command gate, every callback, and the
baseline effect, require exact equality with the current record's newly derived
complete list. Addition, removal, field change, duplicate, or reorder of the
saved canonical binding list fails closed; source record entry reorder does not.
Historical evidence entry order is irrelevant:
recovery CAS retains every structured entry and timestamp while permitting
record order to change.

## Serialize launch and check the engineering base

Before `create_thread`, reread the exact target record and rescan unfinished
tasks. Only one task for a snapshot `target_key` may have an unresolved launch
or an active delivery lifecycle (`running`, `awaiting_deploy_approval`,
`deploy_queued`, or `deploying`). Keep another task for that key queued with
this exact reason, substituting the active task ID:

~~~text
snapshot target <target_key> already has active delivery task <task_id>; sequential lineage guard
~~~

Do not block non-deploy tasks or tasks for another key.

For `lineage_state: current`, require the saved `base_commit` to exist. Permit
it only when it equals `current_deployed_sha` or this command exits `0`:

~~~bash
git merge-base --is-ancestor <current_deployed_sha> <base_commit>
~~~

Save that exact `current_deployed_sha` as `expected_target_baseline`. Exit `1`,
Git error, missing commit, invalid data, changed record, or ambiguous result
blocks launch with the exact observed evidence. `uninitialized` may proceed
only as separately authorized `initial`; `incident` proceeds only as separately
approved bound `reconciliation`.

## Require exact authority before Approval 2

After accepting and Git-verifying the engineering callback, immediately before
showing Approval 2, reread the same canonical record and exact tuple. Validate
the approved project-owned probe descriptor, execute it read-only, and parse it
only by its selected output contract. Preserve the exact successful
observation as `approval_probe_observed_state` plus
`approval_probe_observed_sha` in the canonical approval payload. For `initial`
with a valid matching `uninitialized` record require `absent/null`; every other
mode requires `present/<full SHA>` equal to its factual current baseline. A
missing/malformed/unknown probe or mode/pair mismatch shows no prompt and grants
no authority. For `normal`, additionally
require all of:

1. `lineage_state: current` and record key/target/mode equal the saved tuple.
2. `current_deployed_sha` equals `expected_target_baseline`.
3. `current_deployed_sha` and `approved_sha` are distinct full lowercase SHAs.
4. `git merge-base --is-ancestor <current_deployed_sha> <approved_sha>` exits
   `0`.

If any condition fails or is unknown, do not show ordinary approval. Block the
task with exactly:

~~~text
stale delivery; integration from current target baseline required
~~~

An integrated result is a new Product-approved `reconciliation` task rooted in
the current deployed SHA, passes official Superpowers again, and receives a
new Approval 2.

Production `initial` is available only with a valid `uninitialized` record and
an explicitly approved `repository_state_v1` observation of exact
`absent/null`. `one_full_lowercase_git_sha` cannot represent absence and never
authorizes `initial`. Never invent a sentinel, infer emptiness, or guess a
baseline.

For incident reconciliation, Approval 2 instead requires the valid incident
record to match every saved binding component and complete canonical evidence
identity list exactly and the approved SHA to
be a strict descendant of its factual current SHA. Any record/evidence/source
change is stale and blocks approval.

Use exactly one `lineage_authority_mode`:

- `normal`: require the current SHA to be a strict ancestor of the approved
  SHA; reason is null.
- `initial`: require an absent baseline and explicit first-deploy authority.
- `same_sha_redeploy`: require approved SHA equal expected current SHA and a
  precise non-null reason. At Approval 2, capture the record's exact
  `source_task_id`, `source_deployment_attempt`, and `source_callback_key` as
  immutable `expected_target_provenance`; require it unchanged at the final
  pre-command gate and in the baseline effect.
- `rollback`: require separate authority for the exact expected current SHA,
  older target SHA, key, target, and reason.
- `supersede`: require separate authority for the exact expected current SHA,
  non-descendant target SHA, key, target, and reason.
- `reconciliation`: require a separately engineered descendant of the current
  baseline that represents the approved integrated result. For incident,
  require explicit non-null reason and exact immutable incident
  evidence/baseline/provenance binding at launch, Approval 2, the final
  pre-command gate, and CAS. For ordinary `current`, the binding is null.

Never infer or transfer exceptional authority from `normal`, another mode,
attempt, SHA, baseline, target key, target, task ordering, `depends_on`, or a
platform permission.
Normal, initial, same-SHA, rollback, and supersede may not recover incident.

## Require the target-state probe at each gate

For every revision 9 production attempt, use the exact approved descriptor at
all three gates:

1. before Product renders Approval 2, parse and save exact
   `approval_probe_observed_state`/`approval_probe_observed_sha`; require `absent/null` only for valid
   `initial`, otherwise `present/<expected factual baseline>`;
2. at the final pre-command gate, parse exact `final_probe_observed_state`/`final_probe_observed_sha`
   and require the same mode-specific pair approved at gate 1;
3. after a successful command, parse exact
   `post_deployment_probe_observed_state`/`post_deployment_probe_observed_sha` and require
   `present/<approved SHA>` before the task may publish outcome `DEPLOYED`.

Record argv descriptor, validated working directory, exit status, exact stdout
bytes, parsed state/SHA pair, timestamp, and relevant stderr as checks. Probe execution is read-only.
Non-zero exit, malformed or ambiguous output, descriptor drift, baseline
mismatch, interruption, or unavailable result is unknown and fails closed.
Historical target state remains necessary for lineage/CAS but never replaces
actual production observation.

## Recheck immediately before the command

In the internal deploy-Runner, first verify the saved branch ref and local HEAD
resolve exactly to the approved SHA. After accepting and deduplicating the
exact Product authorization, complete every read-only preflight and any
turn-bound platform permission before running this full gate immediately before
the one approved process invocation:

1. On the same host, freshly read the exact passed canonical Product target
   path and compare it with the immutable authorization snapshot. Cross-host,
   validate that exact approved snapshot and require the mandatory production
   probe below; never use snapshot alone or create a connector, resolver, or
   Product command continuation. Require matching schema, safe key, display
   target, `snapshot` mode, valid state shape, and approved authority mode.
2. Require `current_deployed_sha` to equal the approved
   `expected_target_baseline`, including exact null only for `initial`.
3. For `same_sha_redeploy`, require record provenance equal approved
   `expected_target_provenance`. For incident `reconciliation`, require exact
   complete canonical evidence identity list, baseline, and provenance equal
   the approved `incident_reconciliation_binding`; expected provenance remains null.
4. Repeat the mode-specific ancestry/equality rule. For `normal`, require
   distinct SHAs and exit `0` from
   `git merge-base --is-ancestor <current_deployed_sha> <approved_sha>`.
5. Run the exact approved project-owned probe and parse
   `final_probe_observed_state`/`final_probe_observed_sha`. For valid `initial` require `absent/null`;
   every other mode requires `present/<expected factual baseline>`. Do not
   build a generic deployment adapter.

After the final probe there is no target-changing step before the command. The
task publishes the append-only command-start marker as its final transcript
action and immediately invokes the exact approved command once. The marker is
not a callback and requires no Product continuation. Any mismatch, unavailable
same-host canonical state, cross-host snapshot without matching probe, exit
`1`, Git error, unavailable commit, ambiguous record, or unknown deployed state
runs no command and remains command-started false plus mutation `not_started`;
it may use only the bounded local read-only preflight retry rule.
For `initial`, a factual `present/<SHA>` at this gate means the target appeared
before the marker: block invocation. Never reinterpret it as normal or adopt
the SHA.
Once the marker is published, command-started is true and old authority is
never retried. Use mutation `unknown` for an opaque command or `started` only
with verifiable project-owned evidence. Old authority cannot authorize an
integrated, edited, replacement SHA, or second invocation.

## Promote the baseline crash-safely

Task status and the target record are separate files, not one transaction.
Only an otherwise-valid transcript-first `lazypowers.deployment-result.v1`
with outcome `DEPLOYED`, command invocation count `1`, and a successful
post-deployment observation `present/<approved SHA>` may begin
promotion. On result receipt Product freshly reads and state-shape validates
the named canonical target record; only Product owns CAS/promotion or incident.
A missing, malformed, ambiguous, non-zero, absent, or mismatching post-probe
after possible invocation yields factual `UNKNOWN`, cannot reuse consumed old
authority, and does not create an effect. In the same atomic `status.md` write that accepts
the exact `{callback_key, callback_revision, payload_sha256}` receipt and
outcome, keep lifecycle nonterminal and save:

~~~yaml
pending_deployment_effect:
  marker: NNN-short-slug:deployment-effect:target-baseline:<attempt>:<callback-revision>
  effect_type: target_baseline_update
  target_key: exact safe target key
  expected_current_deployed_sha: exact approved expected baseline or null for initial
  approved_sha: exact approved full lowercase SHA
  lineage_authority_mode: exact approved mode
  expected_target_provenance: null or exact approved same-SHA source provenance
  incident_reconciliation_binding: null or exact approved complete evidence list/factual-state binding
  deployed_sha_probe: exact approved project-owned probe descriptor
  post_deployment_probe_observed_state: present
  post_deployment_probe_observed_sha: exact approved full lowercase SHA
  result_payload_sha256: exact accepted result inner-payload hash
  provenance:
    task_id: exact task ID
    deployment_attempt: exact positive attempt
    callback_key: exact deployment-result callback key
    callback_revision: exact accepted callback revision
~~~

Before other Product work, reconcile every saved `target_baseline_update` in
this fail-closed order. First validate that the immutable effect contains the
exact approved probe descriptor, accepted result hash, and post-deployment
observed pair `present/<approved SHA>`. If not, perform no CAS or target mutation,
block with the malformed-effect evidence, and require Product reconciliation.
Then compare provenance as the exact
`source_task_id + source_deployment_attempt + source_callback_key` triplet:

1. Reread and state-shape validate only the named canonical target record and
   saved effect; require its probe descriptor to equal the effect descriptor.
2. If the record contains the approved SHA and its provenance equals the new
   saved provenance, confirm an already-applied replay without rewriting.
3. For `same_sha_redeploy` only, require expected SHA equal approved SHA and
   record provenance equal the non-null approved `expected_target_provenance`;
   then atomically update only source provenance and `updated_at`. Preserve the
   probe descriptor byte-for-byte. This is the
   sole provenance-only CAS and prevents same-SHA ABA overwrite.
4. For `reconciliation` from incident, require exact unchanged valid incident
   complete canonical evidence identity list, factual SHA, and source provenance from
   `incident_reconciliation_binding`; require approved SHA be its strict
   descendant. Atomically write approved SHA, `current`, new provenance, and
   timestamp. Preserve the probe descriptor and the complete multiset of
   structured `incident_evidence` mappings and every `recorded_at` value; entry order is not part of equality and may change.
   Canonical binding equality is independently derived from
   duplicate-free compact JSON identities, and the binding list itself must be
   sorted by raw UTF-8 bytes.
5. If the record contains the approved SHA with provenance matching neither
   step 2 nor the explicitly bound step 3 state, record an incident. This rule
   applies to `same_sha_redeploy`; never overwrite different provenance merely
   because the SHA is equal.
6. For other SHA-changing modes, require approved SHA differ from expected SHA
   and require the complete expected state: matching schema/key/target/mode,
   applicable `lineage_state`, and exact expected current SHA. Then atomically
   write approved SHA, `lineage_state: current`, new provenance, and
   `updated_at`, preserving the probe descriptor. For `initial`, the valid matching record must be
   `uninitialized` with null current SHA and null source provenance; other
   modes require their approved current-state predicate.
7. Every other observed state is an incident. Preserve the observed SHA and
   provenance, set `lineage_state: incident`, and derive exact evidence from
   the effect marker, target key, expected/observed/approved SHA, authority
   mode, expected/new provenance, conditional incident binding, exact probe
   descriptor, post-probe observed state/SHA pair, accepted result payload hash, task,
   attempt, callback key/revision, mismatch, and a recorded timestamp. Define identity by every
   listed field except timestamp. If `incident_evidence` already contains that
   identity, confirm incident recording without appending; otherwise append it
   once with the timestamp.

For confirmed steps 2, 3, 4, or 6, atomically set `done` and replace the
baseline effect with the normal terminal full numeric queue-rescan marker.
For conflict step 5 or 7, recording evidence or finding its identical identity also
completes this effect as an incident outcome: atomically keep the original task
blocked with factual `DEPLOYED`, replace the baseline effect with a
`terminal_queue_rescan` marker, run/resume it, and clear only after completion.
This is `incident_effect_retired: true`; never recreate the old effect from
receipt replay.

Only a separate Product-approved reconciliation task may recover the valid
incident, with explicit non-null reason and exact incident
evidence/baseline/provenance binding. Its valid `DEPLOYED` performs step 4 and
retains historical incident_evidence. The retired old-task effect cannot
re-incident the recovered target. Never guess or overwrite production state.

Receipt replay never reapplies the CAS; it resumes whichever baseline/rescan
effect is currently saved and is otherwise a no-op.
`FAILED`, `BLOCKED`, `UNKNOWN`, hash conflict, probe failure/mismatch, and
proven-false preflight never create a baseline effect or mutate a target
record. Never read or mutate another `target_key` as this deployment's
baseline.

## Reconcile existing histories

Run migration only after explicit Product authorization for the user project.
Read durable accepted deployment history and valid `DEPLOYED` callbacks in
actual deployment order; do not infer history from main/HEAD, commit time, task
number, `depends_on`, latest card, or `done`.
Validate the resulting record against State-shape invariants before writing;
partial provenance, missing incident evidence/timestamp, or any invalid shape
blocks migration.

Migration also requires separate Product authority for any production probe
descriptor and factual execution of that probe. It never fills
`current_deployed_sha` from Git history when the production observation is
absent or ambiguous.

- For one unambiguous descendant history, create `lineage_state: current` with
  the last confirmed SHA and its exact provenance.
- For actually sequential non-descendant deployed SHAs, preserve the last
  actually deployed SHA, set `lineage_state: incident`, and record the exact
  graph/history evidence. Do not assume capabilities from the former line
  survived.
- For absent history, keep `uninitialized`. For ambiguous history, use
  `incident`. Require Product-approved reconciliation before normal delivery.

Plugin installation never migrates user projects automatically.

## Interpret ancestry commands fail-closed

Validate every SHA as full lowercase hex and verify each commit exists before
calling Git. Interpret `git merge-base --is-ancestor A B` only as:

- exit `0`: `A` is an ancestor of or equal to `B`; separately require
  inequality wherever strict ancestry is required;
- exit `1`: valid negative ancestry result; block the guarded action;
- any other exit, missing object, stderr indicating Git failure, interrupted
  command, or unavailable/ambiguous result: unknown; block the guarded action.

Record the command, validated inputs, exit status, and relevant stderr as
evidence. Never convert a Git error into permission to launch, approve, or
start an external action.
