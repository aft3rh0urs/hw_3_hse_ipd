## Приложение

Учебный HTTP-сервис со счётчиком. 

| Запрос | Результат |
| --- | --- |
| `GET /` | JSON с `greeting`, `count`, `hostname`, `storage`; счётчик не меняется |
| `POST /increment` | Увеличить счётчик на 1, вернуть тот же JSON; тело не требуется |
| `GET /healthz` | HTTP 200, если HTTP-сервер работает |
| `GET /readyz` | HTTP 200 при доступном хранилище, иначе 503 |

При недоступности Redis `GET /` и `POST /increment` тоже возвращают 503.
Приложение не сбрасывает данные и не переключается на SQLite при сбое Redis.
После восстановления Redis следующий запрос снова может выполниться.
Для проверки JSON открывайте адрес в браузере или используйте `curl`.

## Настройки

| Переменная | По умолчанию | Значение |
| --- | --- | --- |
| `PORT` | `8000` | Порт внутри контейнера; приложение слушает `0.0.0.0` |
| `GREETING` | `Hello, Docker!` | Текст поля `greeting` |
| `STORAGE` | `sqlite` | `sqlite` для 4–6 баллов; `redis` для 8–10 |
| `DATA_FILE` | `/data/counter.db` | Путь SQLite, используется только в режиме sqlite |
| `REDIS_URL` | `redis://redis:6379/0` | Адрес Redis, используется только в режиме redis |

SQLite — база в одном файле; при удалении контейнера без volume файл пропадёт.
В режиме Redis приложение не пишет локальные данные; Redis хранит ключ
`docker-hw:counter`. Перенос счётчика из SQLite в Redis не нужен.

Для образа подойдёт официальный `python:3.13-slim`, для хранилища —
`redis:7.4-alpine`. Теги задают семейство версии, но могут обновляться;
для неизменного содержимого существует привязка по digest. Digest в ДЗ
не обязателен. Пакет redis — клиент, сам сервер Redis он не устанавливает.

## Проверка после запуска контейнера

```sh
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS http://127.0.0.1:8080/readyz
```

Первый POST в новом хранилище возвращает `count: 1`. GET счётчик не
увеличивает. В браузере удобнее смотреть GET, для POST используйте curl.

Список команд и их результатов на 6 баллов:

docker build -t cur_app .
docker compose up -d
[+] up 3/3
 ✔ Volume app_counter_volume Created                                                                                                   0.0s
 ✔ Network app_default       Created                                                                                                   0.0s
 ✔ Container app-app-1       Started     
curl -fsS http://127.0.0.1:8080/
{"greeting": "They talk about my onetaps", "count": 0, "hostname": "29d41c4179cd", "storage": "sqlite"}
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "They talk about my onetaps", "count": 1, "hostname": "29d41c4179cd", "storage": "sqlite"}
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "They talk about my onetaps", "count": 2, "hostname": "29d41c4179cd", "storage": "sqlite"}
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "They talk about my onetaps", "count": 3, "hostname": "29d41c4179cd", "storage": "sqlite"}
ctrl + C
docker compose down
[+] down 2/2
 ✔ Container app-app-1 Removed                                                                                                         3.3s
 ✔ Network app_default Removed
docker compose up -d
[+] up 2/2
 ✔ Network app_default Created                                                                                                         0.1s
 ✔ Container app-app-1 Started  
curl -fsS http://127.0.0.1:8080/
{"greeting": "They talk about my onetaps", "count": 3, "hostname": "dd85c0372ca7", "storage": "sqlite"}
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "They talk about my onetaps", "count": 4, "hostname": "dd85c0372ca7", "storage": "sqlite"}
Теперь в .env(которая создавалась cp .env.example .env)
Меняю GREETING на WAZZUP
Меняю APP_PORT на 8888
docker compose down
[+] down 2/2
 ✔ Container app-app-1 Removed                                                                                                         3.3s
 ✔ Network app_default Removed      
docker compose up -d
[+] up 2/2
 ✔ Network app_default Created                                                                                                         0.0s
 ✔ Container app-app-1 Started                          
curl -fsS http://127.0.0.1:8888/
{"greeting": "Wazzup", "count": 4, "hostname": "f27cfeb1b836", "storage": "sqlite"}

Список команд и их результатов на 8 баллов:
docker build -t cur_app
docker compose up -d
[+] up 14/14
 ✔ Image redis:7.4-alpine Pulled                                                                                                       6.5s
 ✔ Network app_default    Created                                                                                                      0.1s
 ✔ Volume app_redis_data  Created                                                                                                      0.0s
 ✔ Container app-redis-1  Healthy                                                                                                      5.4s
 ✔ Container app-app-1    Started       
curl -fsS http://127.0.0.1:8080
{"greeting": "They talk about my onetaps", "count": 0, "hostname": "a10e61ab69aa", "storage": "redis"}
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "They talk about my onetaps", "count": 1, "hostname": "a10e61ab69aa", "storage": "redis"}
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "They talk about my onetaps", "count": 2, "hostname": "a10e61ab69aa", "storage": "redis"}
docker compose down
[+] down 3/3
 ✔ Container app-app-1   Removed                                                                                                       3.4s
 ✔ Container app-redis-1 Removed                                                                                                       0.3s
 ✔ Network app_default   Removed 
docker compose up -d
[+] up 3/3
 ✔ Network app_default   Created                                                                                                       0.0s
 ✔ Container app-redis-1 Healthy                                                                                                       5.4s
 ✔ Container app-app-1   Started
curl -fsS http://127.0.0.1:8080
{"greeting": "They talk about my onetaps", "count": 2, "hostname": "3f4ac441342c", "storage": "redis"}
docker compose stop redis
[+] stop 1/1
 ✔ Container app-redis-1 Stopped   
curl -fsS -X GET http://127.0.0.1:8080/readyz
curl: (22) The requested URL returned error: 503
curl -fsS -X GET http://127.0.0.1:8080/healthz
{"status": "alive"}
curl -fsS -X GET http://127.0.0.1:8080/readyz
curl: (22) The requested URL returned error: 503
curl -fsS -X POST http://127.0.0.1:8080/increment
curl: (22) The requested URL returned error: 503
docker compose start redis
[+] start 1/1
 ✔ Container app-redis-1 Started
curl -fsS -X GET http://127.0.0.1:8080/
{"greeting": "They talk about my onetaps", "count": 2, "hostname": "3f4ac441342c", "storage": "redis"}
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "They talk about my onetaps", "count": 3, "hostname": "3f4ac441342c", "storage": "redis"}