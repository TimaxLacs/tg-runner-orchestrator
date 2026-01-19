# ⚡ TG Runner Orchestrator

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7+-red?logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

**State-Machine Orchestrator для запуска Telegram-ботов в Docker**

[Быстрый старт](#-быстрый-старт) •
[Blueprints](#-blueprints) •
[Workers](#-workers) •
[Документация](#-документация)

</div>

---

## ✨ Возможности

| Возможность | Описание |
|-------------|----------|
| 🔄 **State Machine** | Декларативное описание workflow |
| 👷 **Distributed Workers** | Масштабируемая обработка задач |
| 🐳 **Docker Integration** | Запуск ботов в изолированных контейнерах |
| 🔁 **Auto-Retry** | Автоматические повторы при ошибках |
| 📊 **Observability** | Prometheus метрики, OpenTelemetry |
| 🔒 **Security** | Аутентификация, лимиты, квоты |

---

## 🏗️ Архитектура

```
┌─────────────────┐     ┌─────────────────────┐     ┌──────────────────┐
│   tg-runner     │────▶│  tg-runner-         │────▶│  tg-runner-      │
│   (CLI)         │     │  orchestrator       │     │  worker          │
└─────────────────┘     └─────────────────────┘     └──────────────────┘
                                                            │
                                                            ▼
                                                    ┌──────────────┐
                                                    │   Docker     │
                                                    │  Containers  │
                                                    │  (ваши боты) │
                                                    └──────────────┘
```

---

## 📦 Установка

```bash
git clone https://github.com/TimaxLacs/tg-runner-orchestrator.git
cd tg-runner-orchestrator
pip install -e .
```

---

## 🚀 Быстрый старт

### Docker Compose (рекомендуется)

```bash
# Клонируем
git clone https://github.com/TimaxLacs/tg-runner-orchestrator.git
cd tg-runner-orchestrator

# Запускаем
docker-compose -f docker-compose.bot-runner.yml up -d

# Проверяем
curl http://localhost:8000/_public/status
```

### Программно

```python
import asyncio
from avtomatika import OrchestratorEngine, Config
from avtomatika.storage.redis import RedisStorage
from avtomatika.blueprints.bot_runner import blueprint
from redis.asyncio import Redis

async def main():
    config = Config()
    config.CLIENT_TOKEN = "your-token"
    config.GLOBAL_WORKER_TOKEN = "worker-token"
    
    redis = Redis(host="localhost", port=6379)
    storage = RedisStorage(redis)
    
    engine = OrchestratorEngine(config=config, storage=storage)
    engine.register_blueprint(blueprint)
    
    await engine.start()
    print("Orchestrator running on http://0.0.0.0:8000")

asyncio.run(main())
```

---

## 📘 Blueprints

Blueprint — это workflow в виде конечного автомата:

```python
from avtomatika import StateMachineBlueprint, JobContext

workflow = StateMachineBlueprint("my_workflow", api_endpoint="/jobs/my")

@workflow.state("init", is_start=True)
async def start(context: JobContext):
    context.actions.dispatch_task(
        task_type="process",
        params={"data": context.initial_data},
        transitions={"success": "done", "failure": "error"}
    )

@workflow.state("done", is_end=True)
async def done(context: JobContext):
    pass
```

---

## ⚙️ Конфигурация

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `REDIS_HOST` | Хост Redis | `localhost` |
| `REDIS_PORT` | Порт Redis | `6379` |
| `API_PORT` | Порт API | `8000` |
| `CLIENT_TOKEN` | Токен клиентов | — |
| `GLOBAL_WORKER_TOKEN` | Токен воркеров | — |

---

## 🔗 Экосистема TG Runner

| Компонент | Описание |
|-----------|----------|
| [tg-runner-orchestrator](https://github.com/TimaxLacs/tg-runner-orchestrator) | Orchestrator (этот репо) |
| [tg-runner-worker](https://github.com/TimaxLacs/tg-runner-worker) | Worker для Docker |
| [tg-runner-cli](https://github.com/TimaxLacs/tg-runner-cli) | CLI для пользователей |

---

## 📁 Структура

```
tg-runner-orchestrator/
├── src/avtomatika/
│   ├── engine.py           # Движок
│   ├── blueprint.py        # Blueprints
│   ├── executor.py         # Job executor
│   ├── dispatcher.py       # Task dispatcher
│   ├── blueprints/
│   │   └── bot_runner.py   # Bot Runner blueprint
│   └── storage/
├── docs/                   # Документация
├── examples/               # Примеры ботов
├── tests/                  # Тесты
└── docker-compose.bot-runner.yml
```

---

## 📄 Лицензия

MIT License

---

<div align="center">

**[⬆ Наверх](#-tg-runner-orchestrator)**

Made with ❤️ by [TimaxLacs](https://github.com/TimaxLacs)

</div>
