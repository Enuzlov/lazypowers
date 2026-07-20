# Сценарии Product skill — lifecycle revision 9

Ровно три dry-run сценария ниже являются executable oracle для revision 9.
Они ничего не отправляют, не развёртывают и не меняют внешние системы. Полные
SHA и payload hash в fixtures имеют формат production-контракта; где сценарий
моделирует будущий terminal delivery, значение является fixture, а не
утверждением о текущем Git ref.

## Сценарий 1: engineering outbox recovery через transcript pull

### Given: реальные saved inputs dogfood task 005 и отдельный fixture

Для `005-callback-transport-resilience` следующий блок перечисляет только
фактически переданные task ID, Product/task thread IDs, logical host, launch
marker, base/ref и model routing. Он не утверждает, что старый canonical
`status.md` уже содержит revision 8 `project_identity`, structured heartbeat
или receipt tuple; task не дописывает Product-owned `.lazypowers`.

~~~yaml
saved_handoff_inputs:
  task_id: 005-callback-transport-resilience
  product_thread_id: 019f6ec6-60bd-76e0-8ddc-2c945e27eab0
  product_logical_host_id: local
  task_thread_id: 019f777c-a2b5-7e12-864b-5e7e15a3a330
  task_logical_host_id: local
  task_transport_host_id: local
  source_launch_marker: 005-callback-transport-resilience:launch:1:019f6ec6-60bd-76e0-8ddc-2c945e27eab0
  base_commit: ce877cc130e499f3ccb9ad38b8e6fa3b0245c085
  branch: refs/heads/codex/lazypowers-callback-resilience
  queue_reason: authoritative capacity unavailable; sequential fallback
  initial_model: gpt-5.6-sol
  initial_reasoning: high
  selected_model: gpt-5.6-sol
  selected_reasoning: high
  model_escalation_count: 0
~~~

Следующий source example — иллюстративный, schema-complete dry-run fixture, а
не текущий canonical task state. Inner payload внутри `payload` самодостаточен;
`payload_sha256` вычислен от его canonical compact UTF-8 JSON, а не от YAML или
fence. `dddd...`, fixture project identity и связанный hash намеренно остаются
в source example: встраивание eventual commit SHA в содержимое того же commit
самореферентно и не имеет fixed point.

~~~yaml
schema: lazypowers.callback-envelope.v1
callback_key: 005-callback-transport-resilience:engineering
callback_revision: 1
payload_schema: lazypowers.engineering.v1
payload_sha256: 1eb340b4ae0689fdabe957030fb15dcb2b2b3a434975361fb0c0a670d30cd75f
source_task_id: 005-callback-transport-resilience
source_thread_id: 019f777c-a2b5-7e12-864b-5e7e15a3a330
source_logical_host_id: local
source_launch_marker: 005-callback-transport-resilience:launch:1:019f6ec6-60bd-76e0-8ddc-2c945e27eab0
project_identity: task005-project-identity-fixture
outbox_state: pending
payload:
  schema: lazypowers.engineering.v1
  callback_key: 005-callback-transport-resilience:engineering
  callback_revision: 1
  task_id: 005-callback-transport-resilience
  project_identity: task005-project-identity-fixture
  task_thread_id: 019f777c-a2b5-7e12-864b-5e7e15a3a330
  task_logical_host_id: local
  source_launch_marker: 005-callback-transport-resilience:launch:1:019f6ec6-60bd-76e0-8ddc-2c945e27eab0
  outcome: COMPLETE
  base_commit: ce877cc130e499f3ccb9ad38b8e6fa3b0245c085
  branch: refs/heads/codex/lazypowers-callback-resilience
  final_commit: dddddddddddddddddddddddddddddddddddddddd
  changed_paths:
    - docs/spec.md
    - plugins/lazypowers/.codex-plugin/plugin.json
    - plugins/lazypowers/skills/product/SKILL.md
    - plugins/lazypowers/skills/product/agents/openai.yaml
    - plugins/lazypowers/skills/product/references/deployment-approvals.md
    - plugins/lazypowers/skills/product/references/deployment-lineage.md
    - plugins/lazypowers/skills/product/references/scenarios.md
  checks:
    - plugin and skill validators PASS
    - every fenced YAML block parses
    - exact base and final commit objects exist
    - final commit has exactly one parent equal to saved base
    - saved branch ref resolves exactly to final commit
    - direct diff-tree path set equals changed_paths
    - git diff --check PASS
  summary: Lifecycle revision 8 contract and three scenarios are ready.
  blockers: none
  model_policy: auto
  model_override: null
  initial_model: gpt-5.6-sol
  initial_reasoning: high
  initial_model_selection_reason: orchestration/lifecycle transport redesign and unattended production safety
  selected_model: gpt-5.6-sol
  selected_reasoning: high
  model_selection_reason: orchestration/lifecycle transport redesign and unattended production safety
  model_escalation_count: 0
~~~

Реальный dogfood не переписывает этот source fixture. После создания
фактического final delivery commit task публикует в transcript этого task-чата
отдельный runtime envelope с реальными final SHA, direct paths,
`project_identity` и пересчитанным inner hash до любой попытки
`send_message_to_thread`. Push либо намеренно пропущен, либо его результат
ambiguous; отсутствующий message ID, timeout или даже success не создают
receipt и не меняют correctness state.

### When: heartbeat pull и receiver verification

Product-side pull в начале turn либо проверяемый platform-native heartbeat
читает commentary source-чата, не ожидая terminal thread state, и берёт полный
runtime envelope только из source transcript. Heartbeat считается active,
только если callable create surface вернул stable ID и intended
source/destination, Product сохранил tuple и same-thread marker до unattended
claim, authoritative list/read вернул тот же ID/tuple, а destination transcript
содержит exact marker. Ambiguous creation запрещает второй heartbeat и снимает
unattended claim до reconciliation того же marker. Этот scenario не
придумывает ID и не выполняет task-side write. Product последовательно
проверяет outer/inner schema и triple, task/thread, logical host, launch marker,
project identity, canonical hash и business fields. Затем он выполняет exact
Git gate: оба objects существуют, final имеет
ровно одного parent `ce877...`, literal saved ref указывает ровно на final, а
direct `diff-tree` path set как множество равен `changed_paths`. Любой missing
object, invalid ref, Git error или mismatch блокирует приём.

~~~bash
git cat-file -e ce877cc130e499f3ccb9ad38b8e6fa3b0245c085^{commit}
git cat-file -e dddddddddddddddddddddddddddddddddddddddd^{commit}
git rev-list --parents -n 1 dddddddddddddddddddddddddddddddddddddddd
git show-ref --verify --hash refs/heads/codex/lazypowers-callback-resilience
git diff-tree --no-commit-id --name-only -r dddddddddddddddddddddddddddddddddddddddd
~~~

После full validation Product исполняет одну logical exactly-once acceptance
unit. Exact `result.md` bytes — canonical compact UTF-8 JSON без BOM от
object с тремя exact keys, за которым следует ровно один LF:
`schema` = `lazypowers.result.v1`; `accepted_callback_envelope` = весь
outer envelope fixture выше без изменений; `receiver_verification` = exact object:

~~~yaml
source_identity_verified: true
payload_sha256_verified: true
business_fields_verified: true
model_fields_verified: true
git:
  base_commit: ce877cc130e499f3ccb9ad38b8e6fa3b0245c085
  base_commit_exists: true
  final_commit: dddddddddddddddddddddddddddddddddddddddd
  final_commit_exists: true
  sole_parent: ce877cc130e499f3ccb9ad38b8e6fa3b0245c085
  branch_ref: refs/heads/codex/lazypowers-callback-resilience
  branch_ref_commit: dddddddddddddddddddddddddddddddddddddddd
  direct_changed_paths:
    - docs/spec.md
    - plugins/lazypowers/.codex-plugin/plugin.json
    - plugins/lazypowers/skills/product/SKILL.md
    - plugins/lazypowers/skills/product/agents/openai.yaml
    - plugins/lazypowers/skills/product/references/deployment-approvals.md
    - plugins/lazypowers/skills/product/references/deployment-lineage.md
    - plugins/lazypowers/skills/product/references/scenarios.md
~~~

Порядок path — raw UTF-8 byte order; transcript cursor, timestamp, wall-clock,
tool/run ID и prose не добавляются. Scan cursor изменяем и сохраняется позже
только в `status.md`; поэтому тот же complete accepted outer envelope и
deterministic verification object воспроизводят те же result bytes даже после
продвижения cursor. Product
атомарно создаёт этот JSON-файл (suffix `.md` не меняет format);
existing файл обязан совпасть byte-for-byte и по `result_sha256`. Затем
отдельная atomic status write сохраняет receipt, phases/status, cursor и
pending effect. Unit не accepted, пока все части не durably committed или
recoverably pending. Crash заново проверяет envelope/evidence, повторно
сериализует канонические bytes и продолжает без rewrite только при identity:

~~~yaml
illustrative_acceptance_oracle:
  receipt:
    callback_key: 005-callback-transport-resilience:engineering
    callback_revision: 1
    payload_sha256: 1eb340b4ae0689fdabe957030fb15dcb2b2b3a434975361fb0c0a670d30cd75f
  engineering_phase: completed
  deployment_phase: none
  status: done
  result_write_count: 1
  result_sha256: 2eaf05b67ee334cf02be75ecc509e057052b4dea795ff7d4a4355988f47d8b1a
  thread_cursor: exact accepted source transcript cursor
  pending_callback_effect:
    effect_type: terminal_receipt_marker_then_rescan
    result_sha256: 2eaf05b67ee334cf02be75ecc509e057052b4dea795ff7d4a4355988f47d8b1a
    receipt_marker: 005-callback-transport-resilience:receipt:engineering:1:1eb340b4ae0689fdabe957030fb15dcb2b2b3a434975361fb0c0a670d30cd75f
    receipt_marker_confirmed: false
    rescan_marker: 005-callback-transport-resilience:engineering-terminal-rescan:1
    rescan_completed: false
  queue_rescan_count_after_completion: 1
  outbox_state_after_receipt: accepted
~~~

Receipt marker публикуется только из сохранённого effect и отмечается confirmed
только после exact transcript read. Rescan выполняется либо безопасно
возобновляется по saved marker; effect очищается только после обоих
подтверждений. Heartbeat удаляется только после terminal reconciliation и
отсутствия pending envelopes.

Перед новым envelope Product сначала drains target effects, затем все остальные
pending callback/outbox/receipt-marker/rescan effects и model continuation в
порядке transcript. Неразрешённый ранний effect блокирует поздний callback.

### Then: replay и conflict

Повторное чтение exact
`key + revision + 1eb340b4ae0689fdabe957030fb15dcb2b2b3a434975361fb0c0a670d30cd75f`
— no-op: `result_write_count` остаётся `1`,
receipt не дублируется, phase/counters не меняются и queue rescan не идёт
второй раз. Тот же key/revision с иным 64-hex hash — conflict без receipt,
result, cursor, correction counter или queue effect. Первый engineering callback
всегда revision `1`. Только Product correction, атомарно сохранившая exact
next revision `2` в correction outbox, разрешает такой callback. Ordinary replay
сохраняет revision/hash; older, skipped и unsolicited higher revision не меняют
state.

Все valid terminal outcomes проходят тот же result/receipt-marker/rescan
acceptance. Exact `engineering_phase/deployment_phase/status` mapping:

| Outcome | Task kind | Exact mapping | Additional requirement |
|---|---|---|---|
| `COMPLETE` | non-deploy | `completed/none/done` | valid Git delivery or valid no-change |
| `COMPLETE` | deployable | `completed/preflight_pending/running` | continue deployment lifecycle |
| `FAILED` | any | `failed/none/blocked` | exact failure blockers |
| `BLOCKED` | any | `blocked/none/blocked` | exact blockers |
| `NEEDS_PRODUCT_DECISION` | any | `blocked/none/blocked` | exact blockers and non-empty decision requirement |

## Сценарий 2: initial target — absent, один opaque command, present

### Given: valid uninitialized target и первый factual probe

Product — единственный writer canonical task/target state. Task использует
только immutable approved snapshot и project-owned read-only probe; snapshot,
Git history и worktree-копия состояния не доказывают production state.

~~~yaml
deployment_fixture:
  task_id: 021-initial-production
  project_identity: example-production-project
  product_thread_id: product-main
  product_logical_host_id: product-logical
  task_thread_id: task-production-021
  task_logical_host_id: task-logical
  source_launch_marker: 021-initial-production:launch:1:product-main
  deploy_runner_id: deploy-runner-021
  deployment_attempt: 1
  approved_sha: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
  branch: refs/heads/codex/initial-production-021
  target_key: production-eu
  target: Production EU
  lineage_authority_mode: initial
  lineage_authority_reason: explicit first deployment
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

Record имеет exact valid uninitialized shape. Первый probe завершается exit 0,
а stdout bytes равны ровно одной строке и одному LF:

~~~text
{"state":"absent","sha":null}
~~~

Parser repository_state_v1 возвращает пару absent/null. Approval payload,
ready callback и их canonical hashes включают оба отдельных поля с exact
именами из deployment contract:

~~~yaml
approval_payload:
  schema: lazypowers.deployment-approval-payload.v1
  task_id: 021-initial-production
  project_identity: example-production-project
  task_thread_id: task-production-021
  task_logical_host_id: task-logical
  source_launch_marker: 021-initial-production:launch:1:product-main
  deploy_runner_id: deploy-runner-021
  deployment_attempt: 1
  deployment_approval_key: 021-initial-production:deployment:1:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb:Production EU
  approved_sha: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
  branch: refs/heads/codex/initial-production-021
  target_key: production-eu
  target: Production EU
  deployment_mode: snapshot
  requested_action: publish exact approved snapshot to empty Production EU once
  expected_target_baseline: null
  expected_target_provenance: null
  incident_reconciliation_binding: null
  lineage_authority_mode: initial
  lineage_authority_reason: explicit first deployment
  deployed_sha_probe:
    argv:
      - scripts/read-repository-state
      - production-eu
    working_directory: ops
    output_contract: repository_state_v1
  approval_probe_observed_state: absent
  approval_probe_observed_sha: null
  changed_path_count: 3
  result_link: .lazypowers/tasks/021-initial-production/result.md
ready_payload_observation:
  approval_probe_observed_state: absent
  approval_probe_observed_sha: null
  approval_probe:
    exit_status: 0
    checked_at: "2026-07-20T10:00:00Z"
~~~

Product freshly rereads the same valid uninitialized record, recomputes the
complete approval payload/hash, saves the ready receipt and renders Approval 2
with Observed target state "absent" and Observed target SHA null. Specification
approval, task text, ready callback and forwarded text do not authorize
deployment. Для этого fixture полный approval_payload_sha256 равен
49f4a937d14c526da1d01b9091b9aec978e87b1b5746c27b8073f2a1aa65e42e.

### When: separate approval, final absent pair и one command

Only a new direct user reply in saved Product thread/logical host, bound to
exact 49f4a937d14c526da1d01b9091b9aec978e87b1b5746c27b8073f2a1aa65e42e,
authorizes this attempt. Product atomically
saves the complete lazypowers.deployment-authorized.v1 envelope in pending
outbox and sets:

~~~yaml
deployment_approval_state: consumed
deployment_authority_consumed: true
deployment_phase: authorized
status: deploy_queued
~~~

Task pulls and validates that Product-source envelope and publishes its exact
authorization receipt. Duplicate authorization is a no-op. After all
permissions and read-only preflight, the final repository_state_v1 probe again
returns exact absent/null. The stage pair uses the contract's exact names:

~~~yaml
final_probe_observed_state: absent
final_probe_observed_sha: null
~~~

There is no target-changing step between this probe and invocation. The last
transcript action before the external process is one append-only marker:

~~~yaml
schema: lazypowers.deployment-command-start.v1
marker_key: 021-initial-production:deployment-command-start:1
task_id: 021-initial-production
task_thread_id: task-production-021
task_logical_host_id: task-logical
source_launch_marker: 021-initial-production:launch:1:product-main
deploy_runner_id: deploy-runner-021
deployment_attempt: 1
deployment_approval_key: 021-initial-production:deployment:1:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb:Production EU
authorization_callback_key: 021-initial-production:deployment-authorized:1
authorization_callback_revision: 1
authorization_payload_sha256: eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee
approval_payload_sha256: 49f4a937d14c526da1d01b9091b9aec978e87b1b5746c27b8073f2a1aa65e42e
approved_sha: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
final_probe_observed_state: absent
final_probe_observed_sha: null
deployment_command_started: true
target_mutation_state: unknown
command_invocation_count: 1
~~~

Marker не является callback или continuation request. Сразу после него один
opaque deployment command вызывается ровно один раз. Invocation count никогда
не превышает 1; historical target_mutation_state остаётся unknown и не
переписывается в not_started.

### Then: post present/approved SHA, DEPLOYED и одна promotion

После успешной команды тот же approved probe завершается exit 0 и выдаёт exact
canonical repository_state_v1 bytes для present/approved SHA:

~~~text
{"state":"present","sha":"bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"}
~~~

Полный transcript-first lazypowers.deployment-result.v1 фиксирует exact pair
names и согласованные command facts:

~~~yaml
outcome: DEPLOYED
deployment_attempt: 1
approval_payload_sha256: 49f4a937d14c526da1d01b9091b9aec978e87b1b5746c27b8073f2a1aa65e42e
authorization_callback_key: 021-initial-production:deployment-authorized:1
authorization_callback_revision: 1
authorization_payload_sha256: eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee
lineage_authority_mode: initial
expected_target_baseline: null
approval_probe_observed_state: absent
approval_probe_observed_sha: null
final_probe_observed_state: absent
final_probe_observed_sha: null
post_deployment_probe_observed_state: present
post_deployment_probe_observed_sha: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
deployment_authority_consumed: true
deployment_command_started: true
target_mutation_state: unknown
command_start_marker: 021-initial-production:deployment-command-start:1
command_invocation_count: 1
~~~

Product freshly rereads the same canonical uninitialized record and accepts the
valid result triple exactly once. The acceptance creates exactly one
Product-owned target_baseline_update with expected current SHA null, approved
and post-probe SHA bbbb..., the approved repository_state_v1 descriptor and new
source provenance. Expected-state CAS promotes only this target to:

~~~yaml
lineage_state: current
current_deployed_sha: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
source_task_id: 021-initial-production
source_deployment_attempt: 1
source_callback_key: 021-initial-production:deployment:1
baseline_promotion_count: 1
deployment_phase: completed
status: done
~~~

Promotion confirmation replaces the baseline effect with one terminal numeric
rescan. Replay repeats neither command, receipt, CAS, promotion nor rescan.

## Сценарий 3: repository-state parser и deployment fail-closed matrix

Каждая строка ниже — самостоятельный oracle. Невалидный probe observation
становится unknown: state/SHA не угадываются, target record и Git history его
не заменяют. До command marker это BLOCKED без invocation; после возможного
invocation это UNKNOWN без baseline effect и без retry под старым consumed
authority.

### repository_state_v1 byte parser

Для exit 0 допустимы только exact canonical bytes
{"state":"absent","sha":null}\n и
{"state":"present","sha":"<full-lowercase-40-hex>"}\n. Здесь \n означает один
byte 0x0a, а не два символа.

| Input / condition | Oracle |
|---|---|
| malformed JSON, truncated object или JSON string вместо object | unknown; no ready/prompt/command |
| extra stdout before/after JSON, leading/trailing whitespace или BOM | unknown; no ready/prompt/command |
| invalid/non-UTF-8 bytes | unknown; no ready/prompt/command |
| multiple JSON values or two valid JSON lines | unknown; no ready/prompt/command |
| missing, duplicate or unknown key | unknown; no ready/prompt/command |
| reordered {"sha":null,"state":"absent"} | unknown; approved order is state then sha |
| absent with a SHA | invalid state/SHA pair; unknown |
| present with null, missing SHA or empty SHA | invalid state/SHA pair; unknown |
| present with short, uppercase or non-hex SHA | invalid SHA; unknown |
| missing final LF, CRLF, extra LF or any byte after the one LF | invalid newline framing; unknown |
| non-zero exit, interruption or ambiguous/unavailable result | unknown; no ready/prompt/command |
| output_contract other than one_full_lowercase_git_sha or repository_state_v1 | unsupported contract; fail closed |
| a project probe that depends on GitHub with invalid GitHub auth or returns non-zero | unknown; auth failure never becomes absence and never invokes deployment |

An approval-stage unknown uses the pre-authorization BLOCKED result branch with
failed_stage approval_probe, null authorization triple, authority false,
deployment_command_started false, target_mutation_state not_started,
command_invocation_count 0 and no baseline/ready/prompt/authorization effect.
At the final gate, the same parser failures preserve consumed authority but
still produce BLOCKED with no marker and no invocation; only bounded local
read-only retries are possible before the terminal result.

### Mode/pair gates and ordering

| Observation | Required result |
|---|---|
| valid initial + valid uninitialized record + approval absent/null | may render Approval 2 |
| initial approval present/<any SHA> | BLOCKED before prompt; never adopt the SHA |
| normal, same_sha_redeploy, rollback, supersede or reconciliation with absent/null | BLOCKED; these modes require present/full lowercase SHA |
| any invalid target record shape, descriptor drift or pair mismatch | unknown and fail closed |
| initial was approved absent/null but final probe is present/<any SHA> | target appeared before command: publish no command marker, command_invocation_count 0, deployment_command_started false, mutation not_started |
| final probe absent/null followed by the append-only marker | marker must precede the sole command; no transport continuation is inserted |
| target-changing action before the final probe or between final probe and marker | ordering violation; no valid invocation authority |

The target-appeared-before-command case never converts initial to normal,
never overwrites or adopts the observed repository and never invokes an
external command.

### Post-invocation outcomes and retry prohibition

After marker publication and a possible opaque invocation, command facts are
monotonic: deployment_authority_consumed true, deployment_command_started true,
target_mutation_state unknown, marker present and command_invocation_count 1.

| Post probe / command evidence | Exact outcome and effect |
|---|---|
| present/<approved SHA> plus successful command evidence | DEPLOYED; one Product-owned baseline effect is eligible |
| absent/null after the successful or possibly-started command | UNKNOWN; target absent after command, no baseline effect |
| present/<different full SHA> | UNKNOWN; mismatch, no baseline effect |
| malformed, extra, non-UTF-8, multiple JSON, invalid pair, non-zero or unavailable post probe | UNKNOWN; no baseline effect |
| lost/ambiguous command result or possible invocation | UNKNOWN even if no mutation can be confirmed |

Every UNKNOWN above preserves consumed authority and historical mutation
unknown. Старый authority не переиспользуется: no automatic retry, no second
invocation and no reinterpretation as not_started. Any future execution
requires a new positive attempt, new Runner/key/payload/hash and a new direct
Product approval.

### one_full_lowercase_git_sha unchanged/default compatibility

one_full_lowercase_git_sha остаётся unchanged и default для существующих
projects. Exit 0 с ровно одним full lowercase 40-hex SHA является positive
compatibility oracle и нормализуется в:

~~~yaml
state: present
sha: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
~~~

Existing target records and callback histories не мигрируют и не
переписываются. Для этого contract approval/final pairs всегда present/full
SHA; он не представляет absence и не разрешает initial. Zero/multiple SHA,
short/uppercase/non-hex SHA, extra/ambiguous output, invalid GitHub auth,
non-zero exit или unavailable result остаются unknown и fail closed.

Эти сценарии являются только Markdown executable oracle. Они не создают
runtime pipeline, parser implementation, deployment adapter, connector,
probe, shell wrapper или external action.
