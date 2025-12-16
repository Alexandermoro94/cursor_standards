# Документация по n8n

> Последняя версия документации получена через Context7 MCP из официального репозитория `/n8n-io/n8n-docs`

## Оглавление

1. [Обзор](#обзор)
2. [Установка](#установка)
3. [Начало работы](#начало-работы)
4. [API](#api)
5. [Docker](#docker)
6. [Создание узлов (Nodes)](#создание-узлов-nodes)
7. [Продвинутые возможности AI](#продвинутые-возможности-ai)
8. [Конфигурация](#конфигурация)
9. [Ресурсы](#ресурсы)

---

## Обзор

**n8n** — это расширяемый инструмент автоматизации рабочих процессов (workflow automation), который позволяет подключать что угодно к чему угодно. n8n объединяет гибкость кода со скоростью no-code решений, предлагая более 400 интеграций, встроенный AI и возможность self-hosting.

### Основные возможности

- **Визуальные рабочие процессы**: Создание автоматизаций через последовательность связанных узлов
- **400+ интеграций**: Встроенная поддержка популярных сервисов и приложений
- **AI функциональность**: Встроенные возможности искусственного интеллекта
- **Self-hosting**: Полный контроль над данными и инфраструктурой
- **Fair-code лицензия**: Открытый исходный код с коммерческими опциями

---

## Установка

### Установка через npm (глобально)

```bash
npm install n8n -g
n8n start
```

После установки n8n будет доступен по адресу `http://localhost:5678`

### Установка через Docker

#### Базовая установка с SQLite

```shell
docker volume create n8n_data

docker run -it --rm \
 --name n8n \
 -p 5678:5678 \
 -e GENERIC_TIMEZONE="<YOUR_TIMEZONE>" \
 -e TZ="<YOUR_TIMEZONE>" \
 -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
 -e N8N_RUNNERS_ENABLED=true \
 -v n8n_data:/home/node/.n8n \
 docker.n8n.io/n8nio/n8n
```

#### Установка с PostgreSQL

```shell
docker volume create n8n_data

docker run -it --rm \
 --name n8n \
 -p 5678:5678 \
 -e GENERIC_TIMEZONE="<YOUR_TIMEZONE>" \
 -e TZ="<YOUR_TIMEZONE>" \
 -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
 -e N8N_RUNNERS_ENABLED=true \
 -e DB_TYPE=postgresdb \
 -e DB_POSTGRESDB_DATABASE=<POSTGRES_DATABASE> \
 -e DB_POSTGRESDB_HOST=<POSTGRES_HOST> \
 -e DB_POSTGRESDB_PORT=<POSTGRES_PORT> \
 -e DB_POSTGRESDB_USER=<POSTGRES_USER> \
 -e DB_POSTGRESDB_SCHEMA=<POSTGRES_SCHEMA> \
 -e DB_POSTGRESDB_PASSWORD=<POSTGRES_PASSWORD> \
 -v n8n_data:/home/node/.n8n \
 docker.n8n.io/n8nio/n8n
```

#### Запуск с туннелем (для webhooks)

```shell
docker volume create n8n_data

docker run -it --rm \
 --name n8n \
 -p 5678:5678 \
 -e GENERIC_TIMEZONE="<YOUR_TIMEZONE>" \
 -e TZ="<YOUR_TIMEZONE>" \
 -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
 -e N8N_RUNNERS_ENABLED=true \
 -v n8n_data:/home/node/.n8n \
 docker.n8n.io/n8nio/n8n \
 start --tunnel
```

#### Управление Docker контейнером

```shell
# Найти ID контейнера
docker ps -a

# Остановить контейнер
docker stop <container_id>

# Удалить контейнер
docker rm <container_id>

# Запустить контейнер
docker run --name=<container_name> [options] -d docker.n8n.io/n8nio/n8n
```

#### Обновление Docker образа

```shell
# Последняя стабильная версия
docker pull docker.n8n.io/n8nio/n8n

# Конкретная версия
docker pull docker.n8n.io/n8nio/n8n:1.81.0

# Нестабильная версия (next)
docker pull docker.n8n.io/n8nio/n8n:next
```

### Docker Compose

```shell
# Запуск всех сервисов
docker compose up -d

# Проверка версий Docker
docker --version
docker compose version
```

---

## Начало работы

### Ваш первый workflow

**Workflow** в n8n — это последовательность связанных узлов, которые автоматизируют задачи. Основные концепции:

1. **Trigger узлы**: Запускают workflow (например, Webhook, Schedule, Manual)
2. **Credentials**: Настройка аутентификации для различных сервисов
3. **Обработка данных**: Прохождение данных через workflow
4. **Визуальная логика**: Представление логики через узлы
5. **Выражения (Expressions)**: Динамическое манипулирование данными

### Основные концепции

- **Узлы (Nodes)**: Базовые строительные блоки workflow
- **Соединения (Connections)**: Связи между узлами для передачи данных
- **Выражения**: Использование `{{ }}` для динамических значений
- **Credentials**: Безопасное хранение учетных данных

### Примеры выражений (Expressions)

Выражения в n8n позволяют динамически обращаться к данным и выполнять вычисления:

#### Базовые выражения

**Обращение к данным из предыдущего узла:**
```javascript
{{ $json.customer_name }}
{{ $json["orderStatus"] }}
{{ $json.body.city }}
```

**Создание персонализированного сообщения:**
```expression
Hi {{ $json.customer_name }}. Your description is: {{ $json.customer_description }}
```

**Работа с датами (используя Luxon):**
```javascript
// Дата 7 дней назад
{{ $today.minus(7, 'days') }}

// Разница между датами в месяцах
{{(()=>{  
  let end = DateTime.fromISO('2017-03-13');
  let start = DateTime.fromISO('2017-02-13');
  let diffInMonths = end.diff(start, 'months');
  return diffInMonths.toObject();
})()}}
```

**Обращение к параметрам узла:**
```javascript
{{ $node["HTTP Request"].parameter["headerParametersUi"]["parameter"][0]["value"] }}
```

**Обращение к текущему элементу ввода:**
```javascript
{{ $input.item }}
{{ $input.item.json.name }}
```

**Пагинация (увеличение номера страницы):**
```javascript
{{ $pageCount + 1 }}
```

**Использование внешних секретов:**
```javascript
{{ $secrets.<vault-name>.<secret-name> }}
```

#### Примеры использования в узлах

**Форматирование сообщения для Postbin:**
```javascript
There was a solar flare of class {{$json["classType"]}}
```

**Динамическое сообщение для Discord:**
```javascript
This week we have {{$json["totalBooked"]}} booked orders with a total value of {{$json["bookedSum"]}}
```

---

## API

### n8n Public REST API

n8n предоставляет REST API для программного управления workflows, credentials и выполнениями.

#### Аутентификация

Используйте API ключ в заголовке запроса:

```shell
curl -X 'GET' \
  '<N8N_HOST>:<N8N_PORT>/api/v1/workflows?active=true' \
  -H 'accept: application/json' \
  -H 'X-N8N-API-KEY: <your-api-key>'
```

#### Примеры использования API

**Python:**
```python
import n8n
client = n8n.Client(api_key="your-key")
```

**JavaScript:**
```javascript
const n8n = require('n8n-api');
const client = new n8n.Client({ apiKey: 'your-key' });
```

**cURL для self-hosted:**
```bash
curl -X 'GET' \
  '<N8N_HOST>:<N8N_PORT>/<N8N_PATH>/api/v<version-number>/workflows?active=true' \
  -H 'accept: application/json' \
  -H 'X-N8N-API-KEY: <your-api-key>'
```

**cURL для n8n Cloud:**
```bash
curl -X 'GET' \
  '<your-cloud-instance>/api/v<version-number>/workflows?active=true' \
  -H 'accept: application/json' \
  -H 'X-N8N-API-KEY: <your-api-key>'
```

#### Создание workflow через API

**POST /rest/workflows - Создание нового workflow:**
```json
{
  "name": "My Workflow",
  "nodes": [
    {
      "name": "HTTP Request",
      "type": "main",
      "index": 0
    }
  ],
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "HTTP Request",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "settings": {},
  "staticData": null,
  "tags": []
}
```

**PUT /workflows/{id} - Обновление workflow:**
```json
{
  "name": "Updated Workflow",
  "nodes": [
    {
      "id": "node_1",
      "type": "n8n-nodes-base.start",
      "position": [250, 300]
    }
  ],
  "connections": {},
  "settings": {}
}
```

**Пагинация результатов:**
```bash
curl -X 'GET' \
  '<N8N_HOST>:<N8N_PORT>/api/v<version-number>/workflows?active=true&limit=150&cursor=MTIzZTQ1NjctZTg5Yi0xMmQzLWE0NTYtNDI2NjE0MTc0MDA' \
  -H 'accept: application/json' \
  -H 'X-N8N-API-KEY: <your-api-key>'
```

### Ресурсы для изучения REST API

- [KnowledgeOwl's guide to working with APIs](https://support.knowledgeowl.com/help/working-with-apis)
- [IBM Cloud Learn Hub - What is an Application Programming Interface (API)](https://www.ibm.com/cloud/learn/api)
- [IBM Cloud Learn Hub - What is a REST API?](https://www.ibm.com/cloud/learn/rest-apis)
- [MDN web docs - An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)

---

## Docker

### Переменные окружения в Docker

```bash
docker run -it --rm \
 --name n8n \
 -p 5678:5678 \
 -e N8N_TEMPLATES_ENABLED="false" \
 docker.n8n.io/n8nio/n8n
```

### Мониторинг с Prometheus

n8n предоставляет метрики для Prometheus:

```text
# HELP n8n_scaling_mode_queue_jobs_active Current number of jobs being processed across all workers in scaling mode.
# TYPE n8n_scaling_mode_queue_jobs_active gauge
n8n_scaling_mode_queue_jobs_active 0

# HELP n8n_scaling_mode_queue_jobs_completed Total number of jobs completed across all workers in scaling mode since instance start.
# TYPE n8n_scaling_mode_queue_jobs_completed counter
n8n_scaling_mode_queue_jobs_completed 0

# HELP n8n_scaling_mode_queue_jobs_failed Total number of jobs failed across all workers in scaling mode since instance start.
# TYPE n8n_scaling_mode_queue_jobs_failed counter
n8n_scaling_mode_queue_jobs_failed 0

# HELP n8n_scaling_mode_queue_jobs_waiting Current number of enqueued jobs waiting for pickup in scaling mode.
# TYPE n8n_scaling_mode_queue_jobs_waiting gauge
n8n_scaling_mode_queue_jobs_waiting 0
```

---

## Создание узлов (Nodes)

### Создание нового узла

#### Использование n8n-node CLI

**Глобальная установка CLI:**
```shell
npm install --global @n8n/node-cli
n8n-node --version
```

**Создание проекта без установки:**
```shell
npm create @n8n/node@latest
```

**Создание с шаблоном:**
```shell
npm create @n8n/node@latest n8n-nodes-mynode -- --template declarative/custom
```

#### Структура проекта

После создания проекта установите зависимости:

```shell
npm i
```

### Типы узлов

#### Declarative Style Node

Узлы, созданные в декларативном стиле, используют файлы конфигурации для определения параметров и поведения.

#### Code-based Node

Узлы на основе кода предоставляют больше гибкости для сложной логики.

### Аутентификация в узлах

#### Generic Authentication

Упрощенная настройка аутентификации для методов, где данные отправляются в заголовке, теле или query string:

```javascript
import {
	IAuthenticateGeneric,
	ICredentialType,
	INodeProperties,
} from 'n8n-workflow';

export class AsanaApi implements ICredentialType {
	name = 'asanaApi';
	displayName = 'Asana API';
	documentationUrl = 'asana';
	properties: INodeProperties[] = [
		{
			displayName: 'Access Token',
			name: 'accessToken',
			type: 'string',
			default: '',
		},
	];

	authenticate: IAuthenticateGeneric = {
		type: 'generic',
		properties: {
			headers: {
				Authorization: '=Bearer {{$credentials.accessToken}}',
			},
		},
	};
}
```

### Установка Community Nodes

#### Ручная установка

```sh
# Создать директорию для узлов
mkdir ~/.n8n/nodes
cd ~/.n8n/nodes

# Установить community node
npm i n8n-nodes-nodeName
```

### Документирование узлов

Документация узла должна включать:

- **Описание**: Назначение и функции узла
- **Операции**: Список поддерживаемых операций
- **Credentials**: Информация об аутентификации
- **Примеры**: Примеры использования и шаблоны
- **Связанные ресурсы**: Ссылки на документацию сервиса

---

## Code Node - Работа с кодом

### JavaScript примеры

#### Подсчет количества элементов

```javascript
if (Object.keys(items[0].json).length === 0) {
	return [
		{
			json: {
				results: 0,
			}
		}
	]
}
return [
	{
		json: {
			results: items.length,
		}
	}
];
```

#### Обработка данных заказов

```javascript
let totalBooked = items.length;
let bookedSum = 0;

for(let i=0; i < items.length; i++) {
  bookedSum = bookedSum + items[i].json.orderPrice;
}

return [{json:{totalBooked, bookedSum}}];
```

#### Создание тестовых данных для Merge узла

**Данные о людях:**
```javascript
return [
  {
    json: {
      name: 'Stefan',
      language: 'de'
    }
  },
  {
    json: {
      name: 'Jim',
      language: 'en'
    }
  },
  {
    json: {
      name: 'Hans',
      language: 'de'
    }
  }
];
```

**Данные о приветствиях:**
```javascript
return [
  {
    json: {
      greeting: 'Hello',
      language: 'en'
    }
  },
  {
    json: {
      greeting: 'Hallo',
      language: 'de'
    }
  }
];
```

#### Работа с execution data

```javascript
// Установить одно значение
$execution.customData.set("key", "value");

// Установить весь объект
$execution.customData.setAll({"key1": "value1", "key2": "value2"});

// Получить все данные
var customData = $execution.customData.getAll();

// Получить конкретное значение
var customData = $execution.customData.get("key");
```

### Python примеры

#### Подсчет элементов (Python)

```python
if len(items[0].json) == 0:
	return [
		{
			"json": {
				"results": 0,
			}
		}
	]
else:
	return [
		{
			"json": {
				"results": len(items),
			}
		}
	]
```

#### Работа с execution data (Python)

```python
# Установить одно значение
_execution.customData.set("key", "value");

# Установить весь объект
_execution.customData.setAll({"key1": "value1", "key2": "value2"});

# Получить все данные
customData = _execution.customData.getAll();

# Получить конкретное значение
customData = _execution.customData.get("key");
```

---

## Продвинутые возможности AI

### Обзор

n8n предоставляет расширенные возможности AI для создания чат-ботов, обработки документов и данных. Доступно в Cloud и self-hosted версиях начиная с версии 1.19.4.

### Основные концепции

#### Agents и Chains

**Agents** и **Chains** — фундаментальные концепции в AI, которые помогают структурировать то, как AI системы рассуждают и выполняют задачи.

- **Agents**: Автономные системы, которые могут принимать решения и выполнять действия
- **Chains**: Последовательности операций для обработки данных

#### Contextual Compression Retriever

Узел Contextual Compression Retriever улучшает ответы из векторных хранилищ документов, учитывая контекст запроса. Это повышает релевантность и точность извлеченных документов.

### LangChain

n8n интегрирован с LangChain для работы с AI. Полезные ресурсы:

- [Официальная документация LangChain](https://python.langchain.com/)
- [LangChain JavaScript документация](https://js.langchain.com/)
- [LangChain шаблоны кода](https://github.com/langchain-ai/langchain/tree/master/templates)

### Примеры использования AI

- Создание AI чат-агентов
- Обработка документов с помощью AI
- Интеграция AI в существующие workflows
- Использование инструментов (tools) для расширения возможностей AI агентов

#### Использование функции $fromAI()

Функция `$fromAI()` позволяет AI модели заполнять параметры автоматически:

**Строковый параметр:**
```javascript
$fromAI("name", "The commenter's name", "string", "Jane Doe")
```

**Числовой параметр:**
```javascript
$fromAI("numItemsInStock", "Number of items in stock", "number", 5)
```

**Полный пример со всеми параметрами:**
```javascript
$fromAI("name", "The commenter's name", "string", "Jane Doe")
```

---

## Конфигурация

### Методы конфигурации

#### Переменные окружения

n8n поддерживает множество переменных окружения для настройки поведения:

- `N8N_TEMPLATES_ENABLED`: Включить/выключить шаблоны
- `N8N_RUNNERS_ENABLED`: Включить task runners
- `GENERIC_TIMEZONE`: Часовой пояс
- `DB_TYPE`: Тип базы данных (postgresdb, sqlite)
- `DB_POSTGRESDB_*`: Параметры подключения к PostgreSQL

### OAuth2 Credential Overwrites

Можно настроить глобальные OAuth2 credentials через JSON конфигурацию:

```json
{
    "asanaOAuth2Api": {
        "clientId": "<id>",
        "clientSecret": "<secret>"
    },
    "githubOAuth2Api": {
        "clientId": "<id>",
        "clientSecret": "<secret>"
    }
}
```

### Source Control Environments

n8n поддерживает интеграцию с Git для управления версиями workflows через push/pull операции.

### Настройка таймаутов

**Глобальный таймаут workflow:**
```bash
export EXECUTIONS_TIMEOUT=3600  # 1 час
```

**Максимальный таймаут для отдельного workflow:**
```bash
export EXECUTIONS_TIMEOUT_MAX=7200  # 2 часа
```

---

## Ресурсы

### Официальная документация

- **Основной сайт**: [docs.n8n.io](https://docs.n8n.io/)
- **GitHub репозиторий**: [github.com/n8n-io/n8n-docs](https://github.com/n8n-io/n8n-docs)
- **Workflow шаблоны**: [n8n.io/workflows](https://n8n.io/workflows/)

### Обучение

#### Level One Course

Курс для начинающих, который знакомит с основными концепциями n8n:

- Для тех, кто только начинает использовать n8n
- Помощь в создании первого workflow
- Автоматизация процессов в личной и рабочей жизни

#### Быстрый старт

Для тех, кто хочет быстро начать без подробных объяснений, доступен [quickstart guide](https://docs.n8n.io/try-it-out/tutorial-first-workflow).

### Сообщество

- **Community Nodes**: Расширения от сообщества
- **Примеры workflows**: Коллекции готовых решений
- **Форум**: Поддержка и обсуждения

### Интеграции

n8n поддерживает более 400 интеграций, включая:

- **Приложения**: Google Docs, Slack, GitHub, и многие другие
- **Базы данных**: PostgreSQL, MySQL, MongoDB
- **API**: HTTP Request узел для работы с любыми REST API
- **Триггеры**: Webhooks, Schedule, Email, и другие

### Примеры использования

#### Facebook Graph API

Примеры работы с Facebook Graph API через n8n:

- Получение ленты страницы (GET)
- Публикация поста (POST)
- Загрузка видео
- Удаление поста (DELETE)

#### Работа с базами данных

Примеры работы с PostgreSQL:

```js
// Пример входных данных
[
    {
        "email": "alex@example.com",
        "name": "Alex",
        "age": 21 
    },
    {
        "email": "jamie@example.com",
        "name": "Jamie",
        "age": 33 
    }
]
```

### Пример структуры workflow

Полный пример JSON структуры workflow:

```json
{
  "id": "1012",
  "name": "Nathan's Workflow",
  "active": false,
  "nodes": [
    {
      "parameters": {},
      "name": "Start",
      "type": "n8n-nodes-base.start",
      "typeVersion": 1,
      "position": [130, 640]
    },
    {
      "parameters": {
        "authentication": "headerAuth",
        "url": "https://internal.users.n8n.cloud/webhook/custom-erp",
        "options": {"splitIntoItems": true},
        "headerParametersUi": {
          "parameter": [
            {"name": "unique_id", "value": "recLhLYQbzNSFtHNq"}
          ]
        }
      },
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 1,
      "position": [430, 300],
      "credentials": {"httpHeaderAuth": "beginner_course"}
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {"value1": "={{$json[\"orderStatus\"]}}", "value2": "processing"}
          ]
        }
      },
      "name": "IF",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [630, 300]
    },
    {
      "parameters": {
        "functionCode": "let totalBooked = items.length;\nlet bookedSum = 0;\n\nfor(let i=0; i < items.length; i++) {\n  bookedSum = bookedSum + items[i].json.orderPrice;\n}\nreturn [{json:{totalBooked, bookedSum}}]"
      },
      "name": "Function",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [800, 400]
    }
  ],
  "connections": {
    "HTTP Request": {
      "main": [[[{"node": "IF", "type": "main", "index": 0}]]]
    },
    "IF": {
      "main": [
        [[{"node": "Set", "type": "main", "index": 0}]],
        [[{"node": "Function", "type": "main", "index": 0}]]
      ]
    }
  }
}
```

### Базовый пример workflow на JavaScript

```javascript
const workflow = {
  nodes: [
    { 
      name: 'Start', 
      type: 'n8n-nodes-base.start',
      position: [250, 300]
    },
    {
      name: 'Webhook',
      type: 'n8n-nodes-base.webhook',
      position: [250, 300],
      parameters: {
        httpMethod: "POST",
        path: "api/users"
      }
    }
  ],
  connections: {}
};
```

---

## Заключение

n8n — мощный инструмент для автоматизации рабочих процессов, который сочетает простоту визуального интерфейса с гибкостью программирования. Документация постоянно обновляется, и вы всегда можете найти актуальную информацию на [docs.n8n.io](https://docs.n8n.io/).

### Следующие шаги

1. Создайте свой первый workflow
2. Изучите доступные интеграции
3. Изучите продвинутые возможности AI
4. Присоединитесь к сообществу для обмена опытом

---

*Документация обновлена: 2025*
*Источник: [n8n-io/n8n-docs](https://github.com/n8n-io/n8n-docs)*

