# ⚡ TG Runner Orchestrator

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7+-red?logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

**State-Machine Orchestrator для запуска Telegram-ботов в Docker**

[Быстрый старт](#-быстрый-старт) •
[Архитектура](#️-архитектура) •
[Конфигурация](#️-конфигурация)

</div>

---

## 🚀 Быстрый старт

### Вариант 1: Только Orchestrator (воркер на другом сервере)

```bash
git clone https://github.com/TimaxLacs/tg-runner-orchestrator.git
cd tg-runner-orchestrator

# Запускаем Redis + Orchestrator
docker-compose up -d

# Проверяем
curl http://localhost:8000/_public/status
# {"status": "ok"}
```

Затем на **другом сервере** запустите воркер:
```bash
git clone https://github.com/TimaxLacs/tg-runner-worker.git
cd tg-runner-worker
pip install -e .

export ORCHESTRATOR_URL=http://ORCHESTRATOR_SERVER_IP:8000
export WORKER_TOKEN=test-worker-token
python -m bot_runner_worker
```

---

### Вариант 2: Всё на одном сервере

```bash
git clone https://github.com/TimaxLacs/tg-runner-orchestrator.git
cd tg-runner-orchestrator

# Клонируем воркер в поддиректорию
git clone https://github.com/TimaxLacs/tg-runner-worker.git bot_runner_worker

# Запускаем полный стек
docker-compose -f docker-compose.full.yml up -d

# Проверяем
curl http://localhost:8000/_public/status
```

---

## 🏗️ Архитектура

```
┌─────────────────┐     ┌─────────────────────┐     ┌──────────────────┐
│   tg-runner     │────▶│  tg-runner-         │────▶│  tg-runner-      │
│   (CLI)         │     │  orchestrator       │     │  worker          │
│                 │     │  (этот репо)        │     │  (другой сервер) │
└─────────────────┘     └─────────────────────┘     └──────────────────┘
       │                         │                          │
       │                         │                          ▼
       │                    ┌────┴────┐              ┌──────────────┐
       │                    │  Redis  │              │   Docker     │
       │                    └─────────┘              │  Containers  │
       │                                             └──────────────┘
       │
   pip install tg-runner-cli
```

---

## ⚙️ Конфигурация

### Переменные окружения

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `REDIS_HOST` | Хост Redis | `localhost` |
| `REDIS_PORT` | Порт Redis | `6379` |
| `CLIENT_TOKEN` | Токен для CLI/клиентов | `test-client-token` |
| `GLOBAL_WORKER_TOKEN` | Токен для воркеров | `test-worker-token` |
| `API_PORT` | Порт API | `8000` |
| `LOG_LEVEL` | Уровень логов | `INFO` |

### Использование своих токенов

```bash
# Создайте .env файл
cat > .env << EOF
CLIENT_TOKEN=my-secret-client-token
WORKER_TOKEN=my-secret-worker-token
EOF

# Запустите с этими токенами
docker-compose up -d
```

---

## 📁 Файлы docker-compose

| Файл | Описание |
|------|----------|
| `docker-compose.yml` | Redis + Orchestrator (воркер отдельно) |
| `docker-compose.full.yml` | Полный стек включая воркер |

---

## 🔗 Экосистема TG Runner

| Компонент | Описание | Установка |
|-----------|----------|-----------|
| [tg-runner-orchestrator](https://github.com/TimaxLacs/tg-runner-orchestrator) | Этот репозиторий | `docker-compose up` |
| [tg-runner-worker](https://github.com/TimaxLacs/tg-runner-worker) | Worker для Docker | `pip install -e .` |
| [tg-runner-cli](https://github.com/TimaxLacs/tg-runner-cli) | CLI | `pip install tg-runner-cli` |
| [tg-runner-worker-sdk](https://github.com/TimaxLacs/tg-runner-worker-sdk) | SDK для воркеров | `pip install tg-runner-worker-sdk` |

---

## 🧪 Тестирование

После запуска orchestrator + worker:

```bash
# Установите CLI
pip install tg-runner-cli

# Настройте
export TG_RUNNER_URL=http://localhost:8000
export TG_RUNNER_TOKEN=test-client-token

# Запустите бота
tg-runner start my-bot --simple bot.py -r "aiogram>=3.0" -e "BOT_TOKEN=123:ABC"

# Проверьте
tg-runner list
tg-runner logs my-bot
tg-runner stop my-bot
```

---

## 📄 Лицензия

MIT License

---

<div align="center">

Made with ❤️ by [TimaxLacs](https://github.com/TimaxLacs)

</div>
