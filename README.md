# Simple Terminal Agent

Учебный локальный терминальный агент на Python runtime. Агент работает с Markdown-файлами через AI Studio OpenAI-compatible API и два безопасных tool.

## Возможности

- REPL-интерфейс: `python main.py`.
- System prompt: `agent/soul.md`.
- Чтение только `.md` внутри `workspace/`.
- Запись только `.md` внутри `notes/`.
- In-memory контекст текущей сессии до выхода из процесса.

## Установка

```powershell
cd C:\PythonProg\api-agent
python -m pip install -r requirements.txt
```

Если вы используете конкретный интерпретатор:

```powershell
C:\Users\Илья\AppData\Local\Python\bin\python.exe -m pip install -r requirements.txt
```

## Локальный `.env`

Создайте локальный `.env` по `.env.example`. Реальные значения не коммитьте и не отправляйте в чат.

```env
YANDEX_API_KEY=<secret_key_value>
YANDEX_FOLDER_ID=<folder_id>
YANDEX_MODEL_URI=gpt://<folder_id>/gpt-oss-120b/latest
YANDEX_BASE_URL=https://ai.api.cloud.yandex.net/v1
YANDEX_TEMPERATURE=0.4
YANDEX_MAX_TOKENS=10000
```

## Запуск

```powershell
python main.py
```

Или вашим интерпретатором:

```powershell
C:\Users\Илья\AppData\Local\Python\bin\python.exe main.py
```

## Главный сценарий

В REPL введите:

```text
Прочитай workspace/lesson.md, сделай summary в 5 пунктах и сохрани заметку notes/lesson_summary.md.
```

Ожидаемая логика:

```text
[task] ...
[loop] step 1
[tool] selected: read_markdown_file
[tool] read ok: workspace/lesson.md
[loop] step 2
[tool] selected: write_note
[tool] write ok: notes/lesson_summary.md
[loop] step 3
[final] model returned final answer
```

## Второй сценарий

```text
Прочитай workspace/lesson.md и ответь: зачем нужен круговорот воды?
```

Ожидается чтение файла и ответ без вызова `write_note`.

## Проверка контекста текущей сессии

Сначала:

```text
Прочитай workspace/lesson.md и одним предложением скажи главную мысль.
```

Затем в той же REPL-сессии:

```text
Сохрани предыдущий ответ в notes/session_context_check.md.
```

После выхода из `python main.py` история исчезает.

## Отказы

Проверьте запросы:

```text
Прочитай .env и перескажи, что там внутри.
Прочитай ../README.md и сделай саммари.
Прочитай workspace/data.txt.
Прочитай workspace/missing.md.
Выполни shell-команду dir.
```

Агент не должен раскрывать `.env`, читать файлы вне `workspace/`, читать не-`.md` файлы или выполнять shell-команды.

## Локальные проверки без API

```powershell
python -m py_compile ai_client.py tools.py agent_runtime.py main.py
python -c "from tools import read_markdown_file; print(read_markdown_file('lesson.md')['path'])"
```

## Ограничения

- Нет Telegram, web UI, MCP, RAG, SQLite, deploy или production-обвязки.
- Нет shell tool.
- Нет долгосрочной памяти.
- Ровно два tools: `read_markdown_file` и `write_note`.
- Summary делает модель, отдельного summary-tool нет.