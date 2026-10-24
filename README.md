# pz3-http

Учебный HTTP-сервер на стандартной библиотеке `net/http` (ПЗ №3).
Без сторонних фреймворков: только `http.ServeMux`, `http.Handler`,
`encoding/json`.

## Структура

```
pz3-http/
├── cmd/server/main.go          # маршруты + запуск сервера
├── internal/
│   ├── api/
│   │   ├── handlers.go         # ListTasks, CreateTask, GetTask
│   │   ├── middleware.go       # Logging: метод, путь, статус, время
│   │   └── responses.go        # JSON(), BadRequest(), ErrorResponse
│   └── storage/
│       └── memory.go           # MemoryStore (sync.RWMutex)
├── go.mod                      # module example.com/pz3-http
├── requests.md                 # тестовые curl-запросы
└── README.md
```

## API

| Метод | Путь          | Тело / query           | Ответ                       |
|-------|---------------|------------------------|-----------------------------|
| GET   | `/health`     | —                      | `200 {"status":"ok"}`       |
| GET   | `/tasks`      | `?q=<text>` (опц.)     | `200 [Task, ...]`           |
| POST  | `/tasks`      | `{"title":"..."}`      | `201 Task` / `400` / `422`  |
| GET   | `/tasks/{id}` | —                      | `200 Task` / `404`          |

**Task:**
```json
{ "id": 1, "title": "Купить молоко", "done": false }
```

**Ошибки — единый формат:**
```json
{ "error": "task not found" }
```

## Запуск

```bash
go run ./cmd/server
# слушает :8080
```

Проверка:

```bash
curl -i http://localhost:8080/health
curl -i http://localhost:8080/tasks
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Купить молоко"}'
curl -i http://localhost:8080/tasks/1
```

Полный набор запросов — в [`requests.md`](./requests.md).

### Windows PowerShell

Кириллица в теле запроса ломается в PS 5.1 — используйте `chcp 65001`
и `curl.exe` (а не алиас `curl` → `Invoke-WebRequest`).

## Сборка

```bash
# Windows
go build -o .\bin\server.exe .\cmd\server
.\bin\server.exe

# macOS / Linux
go build -o ./bin/server ./cmd/server
./bin/server
```

## Логирование

Каждый запрос пишется в stdout middleware `api.Logging`:

```
2026/10/16 18:17:44 GET /health 200 565.4μs
```
