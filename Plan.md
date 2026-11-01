# Plan.md: 5.9.1 Простой терминальный агент на Python runtime

## 1. Текущий статус

- Приоритет: `P0`
- Текущее состояние: проект обновлён из smoke-check AI Studio client до каркаса учебного терминального агента.
- Основная цель цикла: собрать маленький терминальный agent runtime с двумя безопасными tools, логом действий и локальной проверкой.
- Обязательное ограничение: не строить Telegram-бота, production-сервис, RAG, MCP, shell tool, web UI, SQLite, внешние интеграции или универсального ассистента.
- Основной контекст проекта: `spec.md`, agent runtime contract, `Plan.md`, `AGENTS.md`, затем фактические команды из `README.md` / runbook.
- Сейчас в работе: локальные проверки выполнены; реальные LLM-сценарии нужно пройти пользователю с локальным `.env`.

## 1.1 Что нужно от пользователя для AI Studio API

Пользователь должен локально заполнить `.env`. Реальные значения нельзя отправлять в чат, переносить в README, `Plan.md`, заметки или код.

Нужны данные:

- `YANDEX_API_KEY`: secret value созданного API key.
- `YANDEX_FOLDER_ID`: ID папки AI Studio / Yandex Cloud.
- `YANDEX_MODEL_URI`: полный URI выбранной текстовой модели, например `gpt://<YANDEX_FOLDER_ID>/gpt-oss-120b/latest`.
- `YANDEX_BASE_URL`: OpenAI-compatible endpoint `https://ai.api.cloud.yandex.net/v1`.
- `YANDEX_TEMPERATURE`: например `0.4`.
- `YANDEX_MAX_TOKENS`: например `10000`.

Минимальный `.env` локально:

```env
YANDEX_API_KEY=<secret_key_value>
YANDEX_FOLDER_ID=<folder_id>
YANDEX_MODEL_URI=gpt://<folder_id>/gpt-oss-120b/latest
YANDEX_BASE_URL=https://ai.api.cloud.yandex.net/v1
YANDEX_TEMPERATURE=0.4
YANDEX_MAX_TOKENS=10000
```

## 1.2 Целевой сценарий

1. Пользователь запускает `python main.py`.
2. Терминал показывает prompt `agent>`.
3. Пользователь просит прочитать `workspace/lesson.md`, сделать summary и сохранить `notes/lesson_summary.md`.
4. Runtime отправляет модели system prompt из `agent/soul.md`, задачу и список двух tools.
5. Модель выбирает `read_markdown_file`.
6. Runtime проверяет путь, расширение и границы `workspace/`.
7. Python читает разрешённый Markdown-файл.
8. Модель делает summary сама, без отдельного summary-tool.
9. Модель выбирает `write_note`.
10. Runtime проверяет путь, расширение и границы `notes/`.
11. Python сохраняет `notes/lesson_summary.md`.
12. Модель возвращает финальный ответ пользователю.
13. В терминале виден лог задачи, tools, чтения, записи и финального ответа.

## 2. Требования к результату

- `spec.md` описывает agent runtime contract 5.9.1.
- `Plan.md` ведёт работу по циклу `Spec -> Plan -> AGENTS.md -> Work -> Check -> README`.
- `AGENTS.md` ограничивает ИИ-помощника учебным агентом и активным пунктом плана.
- `requirements.txt` содержит только минимальные зависимости.
- `.env.example` содержит имена переменных без реальных секретов.
- `ai_client.py` возвращает сообщение модели целиком, чтобы runtime видел `tool_calls`.
- `tools.py` проверяет относительные пути, расширение `.md` и выход за разрешённые папки.
- `agent_runtime.py` содержит ограниченный loop с `MAX_STEPS` и принимает `conversation_history`.
- `main.py` запускает простой терминальный REPL.
- `workspace/lesson.md` даёт локальный Markdown для проверки.
- `README.md` / runbook содержит фактические команды запуска и проверки.

## 3. Что делаем по шагам

- [x] 0. Обновить `spec.md` и agent runtime contract
- [x] 1. Обновить `Plan.md`
- [x] 2. Обновить `AGENTS.md`
- [x] 3. Собрать структуру проекта и окружение
- [x] 4. Принять `agent/soul.md`
- [x] 5. Реализовать `tools.py`
- [x] 6. Создать `ai_client.py` для AI Studio API
- [x] 7. Собрать `agent_runtime.py`
- [x] 8. Собрать `main.py`
- [x] 9. Проверить сценарии и отказы
  - Действие: пройти локальные safety-checks tools и синтаксис без чтения `.env`; реальные LLM-сценарии проверить через `python main.py`.
  - Результат: локально без API подтверждено, что `lesson.md` читается, временная заметка пишется, запрещённые пути и неизвестный tool отклоняются.
  - Проверка: синтаксис Python прошёл; реальные LLM-сценарии `summary + note`, `question without note`, `session context` требуют запуска `python main.py` с локальным `.env`.
- [x] 10. Финально обновить `README.md` / runbook
  - Действие: записаны команды установки, запуска, главного сценария, проверки контекста и отказов.
  - Результат: следующий урок получает воспроизводимую базу agent runtime.

## 4. Прямые зависимости и проверки

- `.env` не читать, не печатать и не копировать.
- Проверка синтаксиса: `python -m py_compile ai_client.py tools.py agent_runtime.py main.py`.
- Проверка безопасного чтения: `read_markdown_file("lesson.md")`.
- Проверка отказов: `../README.md`, `.env`, `workspace/data.txt`, `workspace/missing.md`, неизвестный tool.

## 5. Проверки и ошибки

Выполнено без чтения `.env`:

```powershell
C:\Users\Илья\AppData\Local\Python\bin\python.exe -m py_compile C:\PythonProg\api-agent\ai_client.py C:\PythonProg\api-agent\tools.py C:\PythonProg\api-agent\agent_runtime.py C:\PythonProg\api-agent\main.py
C:\Users\Илья\AppData\Local\Python\bin\python.exe -c "import openai, dotenv; print('deps ok')"
```

Локальные tool-checks без API:

- `read_markdown_file('lesson.md')` вернул `workspace/lesson.md`.
- `write_note('safety_tmp.md', 'ok')` создал временный файл `notes/safety_tmp.md`; файл удалён после проверки.
- `../README.md` отклонён: выход через `..` запрещён.
- `.env` отклонён как не Markdown-файл.
- `workspace/data.txt` отклонён как не Markdown-файл.
- `workspace/missing.md` отклонён как несуществующий файл.
- неизвестный tool отклонён.

Ограничение проверки:

- Реальный AI Studio вызов требует локального `.env` и может расходовать квоты.
- `.env` не читался и не выводился.