# Сценарии Product handoff

Ровно три сценария проверяют активную поверхность Lazypowers 0.4.3.

## Сценарий 1: Передача при пассивной очереди

### Given

- Пользователь явно говорит «создай новый продуктовый чат».
- Каноническая очередь содержит `draft`, `queued` и нескольких `launched`,
  включая одновременно работающие независимые Runner.
- Ни один валидный `lazypowers.dispatch.v1` не остаётся в неразрешённом
  `launching`.

### When / Then

- Product не считает пассивную очередь или активные Runner блокером и не
  изменяет ни один task-файл.
- Product создаёт ровно один successor в том же exact project root и задаёт
  точное название `Lazypowers Product`.
- Successor fresh-read читает применимые инструкции проекта,
  `git show refs/heads/main:docs/spec.md` и канонические
  `.lazypowers/tasks/`; снимок или копия очереди не передаётся.
- Только после успешного binding и exact naming старый Product прекращает
  Product-мутации и не мониторит новый чат.

## Сценарий 2: Независимые task-запуски

### Given

- Пользователь выбирает несколько независимых задач из `queued`.
- Другие задачи уже имеют `state: launched` и продолжают выполняться в
  отдельных task-чатах и изолированных worktree.
- У одной task-create транзакции может временно оставаться
  `state: launching`.

### When / Then

- Каждая выбранная задача сохраняет собственные marker, единственный create,
  binding и title; глобального ограничения «одна активная задача» нет.
- `launched` и работающие Runner не блокируют запуск другой выбранной
  `queued` задачи.
- `launching` запрещает второй create только для той же задачи;
  он блокирует только Product handoff до обычного recovery и не блокирует независимые
  task-dispatch транзакции.

## Сценарий 3: Неоднозначный Product create

### Given

- Product создал один marker
  `lazypowers.product-handoff.v1:<uuid>` и вызвал `create_thread` ровно один
  раз.
- Create вернул только асинхронный идентификатор либо исход операции
  неоднозначен.

### When / Then

- Product выполняет ограниченный exact-marker lookup и дедуплицирует
  кандидатов по exact thread ID.
- Один distinct ID связывается автоматически; два или более требуют выбора
  пользователя; ноль оставляет Product handoff неоднозначным.
- `automatic replacement` запрещён: второй marker и второй `create_thread` не
  создаются.
- Пока один successor не связан и не назван точно `Lazypowers Product`, старый
  Product сохраняет единственную Product-роль и не прекращает мутации.
