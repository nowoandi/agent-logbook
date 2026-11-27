---
description: "Show the state of the project journal: which folders are connected, how many chats and commits they hold, when the journal was last refreshed"
allowed-tools:
  - mcp__plugin_project-logbook_journal__journal_status
---

# Состояние журнала

1. Вызови инструмент `journal_status` (аргументов у него нет).
2. Сведи ответ в короткую таблицу по папкам: папка (`project`), чатов (`chats`, по агентам —
   `byAgent`), коммитов git (`gitCommits`), первый чат (`firstChat`), последнее обновление
   (`synced`).
3. Скажи, подключена ли папка этого проекта `${CLAUDE_PROJECT_DIR}` (если выражение не заменилось
   путём — твоя основная рабочая папка). Пути сравнивай без учёта регистра и вида слешей; папка
   внутри подключённой тоже считается подключённой. Папка в списке, а `chats` = 0 — либо чатов в
   этой папке ещё не было, либо журнал ещё собирается после подключения.
4. `lastSync` — что было при последнем обновлении. `failed` или `unreadable` больше нуля — назови
   число и `firstError`; есть `refreshFailed` — скажи, что фоновое обновление упало, когда и с какой
   ошибкой: журнал отвечает по состоянию на `synced`.
5. «The journal has not been built yet», пустой список или папки проекта нет в списке — предложи
   `/project-logbook:connect`. Давнее `synced` действий не требует: журнал сам обновляется в фоне,
   когда стартует чат и когда к нему обращаются.
6. Инструмента `journal_status` нет — сервер журнала не запущен: скажи это и предложи открыть `/mcp`
   и проверить Node.js (нужна версия 22.13 или новее).
