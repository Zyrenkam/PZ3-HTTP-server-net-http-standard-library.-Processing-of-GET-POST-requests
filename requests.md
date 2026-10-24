# Тестовые запросы `pz3-http`

Коллекция `curl`-запросов для проверки API (ПЗ №3).
Сервер запускается из корня проекта:

```bash
go run ./cmd/server
# слушает :8080 (или $PORT)
```

> **Windows PowerShell:** перед запуском выполните `chcp 65001`,
> чтобы кириллица в теле запроса не превращалась в `??????`.
> Используйте `curl.exe` (а не алиас `curl` → `Invoke-WebRequest`),
> либо передавайте тело как UTF-8 байты (см. примечание в конце).

---

## 1. `GET /health` — проверка живости

**Запрос:**
```bash
curl -i http://localhost:8080/health
```

**Ожидаемо:**
```
HTTP/1.1 200 OK
Content-Type: application/json
{"status":"ok"}
```

---

## 2. `POST /tasks` — создание задачи (успех)

**Запрос:**
```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Купить молоко"}'
```

**Ожидаемо:**
```
HTTP/1.1 201 Created
Content-Type: application/json
{"id":1,"title":"Купить молоко","done":false}
```

---

## 3. `POST /tasks` — создание второй задачи

**Запрос:**
```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Позвонить маме"}'
```

**Ожидаемо:**
```
HTTP/1.1 201 Created
Content-Type: application/json
{"id":2,"title":"Позвонить маме","done":false}
```

---

## 4. `GET /tasks` — список задач

**Запрос:**
```bash
curl -i http://localhost:8080/tasks
```

**Ожидаемо:**
```
HTTP/1.1 200 OK
Content-Type: application/json
[{"id":1,"title":"Купить молоко","done":false},{"id":2,"title":"Позвонить маме","done":false}]
```

---

## 5. `GET /tasks?q=` — фильтр по подстроке

**Запрос:**
```bash
curl -i "http://localhost:8080/tasks?q=молоко"
```

**Ожидаемо:**
```
HTTP/1.1 200 OK
Content-Type: application/json
[{"id":1,"title":"Купить молоко","done":false}]
```

Проверка регистронезависимости:
```bash
curl -i "http://localhost:8080/tasks?q=МОЛОКО"
# тот же результат
```

Пустой результат:
```bash
curl -i "http://localhost:8080/tasks?q=несуществующее"
# 200 []
```

---

## 6. `GET /tasks/{id}` — задача по id (успех)

**Запрос:**
```bash
curl -i http://localhost:8080/tasks/1
```

**Ожидаемо:**
```
HTTP/1.1 200 OK
Content-Type: application/json
{"id":1,"title":"Купить молоко","done":false}
```

---

## 7. `GET /tasks/{id}` — несуществующий id → 404

**Запрос:**
```bash
curl -i http://localhost:8080/tasks/999
```

**Ожидаемо:**
```
HTTP/1.1 404 Not Found
Content-Type: application/json
{"error":"task not found"}
```

Нечисловой id:
```bash
curl -i http://localhost:8080/tasks/abc
# 404 {"error":"invalid id"}
```

---

## 8. `POST /tasks` — битый JSON → 400

**Запрос:**
```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{bad json}'
```

**Ожидаемо:**
```
HTTP/1.1 400 Bad Request
Content-Type: application/json
{"error":"invalid json: invalid character 'b' looking for beginning of object key string"}
```

---

## 9. `POST /tasks` — неверный `Content-Type` → 400

**Запрос:**
```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: text/plain" \
  -d '{"title":"test"}'
```

**Ожидаемо:**
```
HTTP/1.1 400 Bad Request
Content-Type: application/json
{"error":"Content-Type must be application/json"}
```

---

## 10. `POST /tasks` — пустой `title` → 422

**Запрос:**
```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":""}'
```

**Ожидаемо:**
```
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json
{"error":"title must be 3..140 chars"}
```

Слишком короткий `title`:
```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"ok"}'
# 422 {"error":"title must be 3..140 chars"}
```

---

## 11. *(доп.)* `PATCH /tasks/{id}` — изменить флаг `done`

**Запрос:**
```bash
curl -i -X PATCH http://localhost:8080/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"done":true}'
```

**Ожидаемо:**
```
HTTP/1.1 200 OK
Content-Type: application/json
{"id":1,"title":"Купить молоко","done":true}
```

---

## 12. *(доп.)* `DELETE /tasks/{id}` — удалить задачу

**Запрос:**
```bash
curl -i -X DELETE http://localhost:8080/tasks/2
```

**Ожидаемо:**
```
HTTP/1.1 204 No Content
```

Повторный `DELETE` того же id:
```bash
curl -i -X DELETE http://localhost:8080/tasks/2
# 404 {"error":"task not found"}
```

---

## Примечание: Windows PowerShell и UTF-8

В Windows PowerShell 5.1 встроенный `curl` — это алиас на
`Invoke-WebRequest`, который плохо работает с UTF-8 в теле запроса
(отсюда `??????`). Варианты решения:

**Вариант A — использовать `curl.exe`:**
```powershell
chcp 65001
curl.exe -i -X POST http://localhost:8080/tasks `
  -H "Content-Type: application/json" `
  --data-binary '{"title":"Купить молоко"}'
```

**Вариант B — передать тело как UTF-8 байты:**
```powershell
$body = [System.Text.Encoding]::UTF8.GetBytes('{"title":"Купить молоко"}')
curl.exe -i -X POST http://localhost:8080/tasks `
  -H "Content-Type: application/json" `
  --data-binary $body
```

**Вариант C — перейти на PowerShell 7+ (`pwsh`)** — там UTF-8 по умолчанию.

---

## Сводная таблица

| № | Метод | Путь | Ожидаемый статус | Что проверяет |
|---|-------|------|------------------|---------------|
| 1 | GET | `/health` | 200 | Живость сервера |
| 2 | POST | `/tasks` | 201 | Создание задачи |
| 3 | POST | `/tasks` | 201 | Создание второй задачи |
| 4 | GET | `/tasks` | 200 | Список задач |
| 5 | GET | `/tasks?q=` | 200 | Фильтр по подстроке |
| 6 | GET | `/tasks/1` | 200 | Получение по id |
| 7 | GET | `/tasks/999` | 404 | Несуществующий id |
| 8 | POST | `/tasks` | 400 | Битый JSON |
| 9 | POST | `/tasks` | 400 | Неверный Content-Type |
| 10 | POST | `/tasks` | 422 | Валидация title |
| 11 | PATCH | `/tasks/1` | 200 | *(доп.)* Изменение done |
| 12 | DELETE | `/tasks/2` | 204 | *(доп.)* Удаление |

Обязательный минимум — **запросы 1–10** (10 штук, больше требуемых 6–8).
Запросы 11–12 дают дополнительные баллы.