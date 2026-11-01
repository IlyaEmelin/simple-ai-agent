# spec.md: simple-terminal-agent 5.9.1

## Цель

Собрать учебный локальный терминальный агент на Python runtime. Агент запускается командой `python main.py`, принимает задачи в REPL, вызывает модель AI Studio через OpenAI-compatible API и использует ровно два безопасных tool.

## Структура проекта

```text
simple-terminal-agent/
├─ main.py
├─ ai_client.py
├─ agent_runtime.py
├─ tools.py
├─ requirements.txt
├─ .env
├─ .env.example
├─ AGENTS.md
├─ Plan.md
├─ README.md
├─ spec.md
├─ agent/
│  └─ soul.md
├─ workspace/
│  └─ lesson.md
└─ notes/
   └─ lesson_summary.md
```

## Runtime contract

- `main.py` запускает терминальный REPL и хранит `session_messages` только в памяти процесса.
- `agent/soul.md` хранит system prompt.
- `ai_client.py` вызывает AI Studio API и предоставляет функцию `call_model(messages, tools=None)`.
- `agent_runtime.py` содержит agent loop, передаёт модели system prompt, историю и список tools.
- `tools.py` содержит единственные разрешённые Python tools.
- Модель не читает файлы и не выполняет функции сама. Она возвращает structured tool-call: имя функции и JSON-аргументы.
- Python runtime проверяет tool-call, выполняет разрешённую функцию и возвращает результат модели через `role: tool`.
- Если модель может ответить без tool, loop завершается на первом шаге.
- Если нужен файл или запись заметки, loop должен пройти: model call -> tool-call -> tool result -> следующий model call -> final answer.

## Разрешённые tools

| Tool | Что делает | Жёсткая граница |
| --- | --- | --- |
| `read_markdown_file(path)` | Читает Markdown-файл из `workspace/` | Только относительный путь, только `.md`, только внутри `workspace/` |
| `write_note(filename, content)` | Создаёт или перезаписывает Markdown-заметку в `notes/` | Только относительный путь, только `.md`, только внутри `notes/` |

Summary делает модель после чтения файла. Отдельного summary-tool нет.

## Основные сценарии

### Главный сценарий

1. Пользователь запускает `python main.py`.
2. Терминал показывает prompt `agent>`.
3. Пользователь просит прочитать `workspace/lesson.md`, сделать summary и сохранить `notes/lesson_summary.md`.
4. Runtime отправляет модели system prompt, задачу и список двух tools.
5. Модель выбирает `read_markdown_file`.
6. Runtime проверяет путь, расширение и границы `workspace/`.
7. Python читает разрешённый Markdown-файл.
8. Модель делает summary сама.
9. Модель выбирает `write_note`.
10. Runtime проверяет путь, расширение и границы `notes/`.
11. Python сохраняет `notes/lesson_summary.md`.
12. Модель возвращает финальный ответ.

### Второй сценарий

Пользователь просит прочитать `workspace/lesson.md` и ответить на вопрос без записи заметки. Runtime читает файл через `read_markdown_file`, модель отвечает по содержанию, `write_note` не вызывается.

### Контекст текущей сессии

В рамках одного запуска `python main.py` runtime хранит user messages, model responses, tool calls и tool results в `session_messages`. Следующий ввод может сослаться на предыдущий ответ. После выхода история исчезает.

## Отказы

Агент должен отказывать или возвращать ошибку tool при запросах:

- прочитать `.env`;
- прочитать путь вне `workspace/`;
- прочитать не-`.md` файл;
- прочитать несуществующий файл;
- выполнить shell-команду;
- использовать неизвестный tool.

## Запреты scope

- Не добавлять Telegram-интерфейс.
- Не добавлять MCP, RAG, web UI, SQLite, долгосрочную память или внешние сервисы.
- Не выполнять shell-команды через агента.
- Не читать `.env`.
- Не ходить в интернет.
- Не работать с произвольными файлами проекта.
- Не добавлять больше двух tools.
- Не превращать агента в универсального помощника.

## Критерии готовности

- `python main.py` запускает REPL.
- Агент читает `workspace/lesson.md` через tool.
- Агент создаёт `notes/lesson_summary.md` через tool.
- Агент отвечает на вопрос по Markdown без записи заметки.
- Агент в рамках одной REPL-сессии может сослаться на предыдущий ответ.
- `.env`, `../`, absolute path, не-`.md`, missing file и shell-команды не выполняются.
- В коде, заметках и README нет секретов.