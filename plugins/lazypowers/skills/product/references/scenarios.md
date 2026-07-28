# Сценарии тонкого Product-диспетчера

Ровно три сценария проверяют всю активную поверхность Lazypowers 0.4.1.

## Сценарий 1: Combined local approval и launch

### Given

- Product сохранил `025-short-slug` со `state: draft`.

### When / Then

- Product проверяет `dispatch.title == "025 — <spec title>"` до любого state change или create.
- На обычное прямое «да» Product fresh-read проверяет task ID и меняет только
  `draft → queued`; create не вызывается.
- На одно сообщение «подтверждаю и запускай» Product fresh-read проверяет тот
  же draft, записывает `queued` и в этом же turn начинает local create
  transaction без второго подтверждения.
- Transaction сохраняет один launch marker до единственного `create_thread`.

## Сценарий 2: Async Mac mini launch

### Given

- Пользователь говорит «подтверждаю и запускай через $mini».
- Model, reasoning и starting-state bindings отсутствуют.

### When

1. Product передаёт `$mini` transaction без отсутствующих execution bindings.
2. Remote create возвращает один `clientThreadId`.
3. Fresh lookup возвращает две host-alias rows с одним и тем же final thread ID.
4. Product дедуплицирует rows по exact thread ID.

### Then

- Сохраняется один final `task_thread_id` и `state: launched`.
- `set_thread_title` получает точное numbered title `025 — <spec title>`.
- Product проверяет title readback и прекращает работу с задачей.

## Сценарий 3: Recovery без duplicate create

### Given

- Existing dispatch уже имеет `state: launching`, launch marker и, возможно,
  `client_thread_id`.

### When / Then

- Product всегда выполняет recovery-before-create и не генерирует новый marker.
- Zero candidates оставляет `launching`; следующая Product interaction
  возобновляет lookup без duplicate create и без просьбы найти thread ID.
- Two distinct IDs оставляют `launching`; replacement не создаётся, а Product
  просит пользователя выбрать один exact ID.
- Authoritative no-create возвращает задачу в `queued`.
- После final binding title mismatch вызывает один idempotent retry; повторная
  title failure сохраняет `launched`, сообщает ошибку один раз и завершает
  dispatch.
