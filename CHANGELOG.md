# Изменения

## 1.0.0

Первая версия.

- MCP-сервер журнала, восемь инструментов: journal_brief, journal_search, journal_recent,
  journal_session, journal_tasks, journal_status, journal_connect, journal_open. Сервер сам
  подсказывает модели, когда звать журнал, — и там, где хуки не срабатывают.
- Сводка в начале чата (хук SessionStart) в подключённых папках; в остальных журнал молчит.
- Навык `agent-logbook` и команды `/agent-logbook:connect`, `view`, `status`.
- Задачи из файлов проекта (`tasks/<ключ>.md`) с отметкой, какие взяты в других ветках, и профиль
  проекта `.agent-logbook.json` для своих названий папки, полей и значений.
- Страница журнала: центр управления, карта чатов и областей кода, лента по дням, задачи.
- Папка данных `%LOCALAPPDATA%\AgentLogbook` (Windows) и `~/.agent-logbook` (macOS, Linux), другое
  место — переменная `AGENT_LOGBOOK_HOME`.
- Codex: навык и MCP-сервер; хука нет.
