# Руководство для контрибьюторов

Спасибо за интерес к проекту integramDD! Это руководство поможет вам внести свой вклад в развитие проекта.

## Содержание

- [Кодекс поведения](#кодекс-поведения)
- [Как внести вклад](#как-внести-вклад)
- [Настройка окружения для разработки](#настройка-окружения-для-разработки)
- [Стиль кода и требования к форматированию](#стиль-кода-и-требования-к-форматированию)
- [Структура коммитов](#структура-коммитов)
- [Процесс создания Pull Request](#процесс-создания-pull-request)
- [Требования к тестированию](#требования-к-тестированию)
- [Процесс review и merge](#процесс-review-и-merge)
- [Контактная информация](#контактная-информация)

## Кодекс поведения

Участвуя в этом проекте, вы соглашаетесь соблюдать следующие принципы:

- Будьте уважительны и доброжелательны ко всем участникам
- Конструктивная критика приветствуется
- Недопустимы любые формы дискриминации и домогательств
- Фокусируйтесь на том, что лучше для проекта и сообщества

## Как внести вклад

Вы можете помочь проекту следующими способами:

- **Сообщение об ошибках**: Создайте issue с подробным описанием проблемы
- **Предложение новых функций**: Опишите предлагаемую функциональность в issue
- **Улучшение документации**: Исправление опечаток, дополнение примеров
- **Написание кода**: Исправление багов, реализация новых функций
- **Написание тестов**: Увеличение покрытия тестами

### Создание Issue

Перед созданием нового issue:

1. Проверьте, что подобная проблема ещё не зарегистрирована
2. Используйте понятный и описательный заголовок
3. Предоставьте максимум деталей:
   - Шаги для воспроизведения проблемы
   - Ожидаемое поведение
   - Фактическое поведение
   - Версии ПО (Python, PostgreSQL, Docker и т.д.)
   - Логи ошибок (если применимо)

## Настройка окружения для разработки

### Требования

- **OS**: Linux / Windows / macOS
- **Docker**: ≥ 20.10.0
- **Docker Compose**: ≥ 1.29.0
- **Python**: 3.11+ (для локальной разработки без Docker)
- **Git**: для работы с репозиторием
- **Порт 8000**: должен быть свободен

### Установка

1. **Форк и клонирование репозитория**

```bash
# Создайте форк через GitHub UI, затем клонируйте
git clone https://github.com/YOUR_USERNAME/integramDD.git
cd integramDD

# Добавьте upstream репозиторий
git remote add upstream https://github.com/unidel2035/integramDD.git
```

2. **Настройка переменных окружения**

```bash
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=integram
export DB_USER=postgres
export DB_PASSWORD=postgres
```

Или создайте файл `.env` в корне проекта:

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=integram
DB_USER=postgres
DB_PASSWORD=postgres
```

3. **Запуск в Docker**

```bash
docker compose up --build
```

Или (для старого синтаксиса):

```bash
docker-compose up --build
```

4. **Загрузка хранимых процедур**

После того, как контейнеры запущены:

```bash
cd ./.sql_to_load
./upload_methods.sh -u postgres -p postgres -d integram
cd ..
```

5. **Проверка работоспособности**

Откройте в браузере:
```
http://localhost:8000/docs
```

Вы должны увидеть интерактивную документацию Swagger UI.

### Локальная разработка (без Docker)

Если вы предпочитаете работать без Docker:

1. **Установите PostgreSQL 15**

2. **Создайте виртуальное окружение и установите зависимости**

```bash
cd app
python -m venv venv
source venv/bin/activate  # На Windows: venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
```

3. **Инициализируйте базу данных**

```bash
psql -U postgres -d integram -f init-db.sql
cd .sql_to_load
./upload_methods.sh -u postgres -p postgres -d integram
```

4. **Запустите сервер**

```bash
cd app
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

## Стиль кода и требования к форматированию

### Python

Проект следует стандартам **PEP 8** с некоторыми дополнениями:

#### Общие правила

- **Кодировка**: UTF-8
- **Отступы**: 4 пробела (не табы)
- **Максимальная длина строки**: 88 символов (совместимо с Black)
- **Импорты**: сортируются по группам (стандартная библиотека, сторонние пакеты, локальные импорты)

#### Пример правильного оформления

```python
from fastapi import APIRouter, Path, HTTPException, Depends
from fastapi.responses import JSONResponse
from sqlalchemy import text
from typing import List

from app.db.db import engine, load_sql
from app.models.terms import Term, TermCreateRequest
from app.auth.auth import verify_token
from app.logger import setup_logger


router = APIRouter()
logger = setup_logger(__name__)


@router.get(
    "/{db_name}/terms",
    response_model=List[Term],
)
async def get_all_terms(
    db_name: str = Depends(validate_table_exists),
):
    """Fetches all metadata terms from the database.

    This endpoint returns a list of all defined terms.
    Authorization is required.

    Returns:
        List[Term]: A list of term entries.
    """
    try:
        sql = load_sql("get_terms.sql", db=db_name)

        async with engine.connect() as conn:
            result = await conn.execute(text(sql))
            rows = result.mappings().all()

        return JSONResponse([dict(row) for row in rows])
    except SQLAlchemyError as e:
        logger.exception(f"DB error while fetching terms from {db_name}")
        raise HTTPException(status_code=500, detail="Database error")
```

#### Документация кода

- **Docstrings**: используйте docstrings для всех публичных модулей, классов и функций
- **Стиль docstrings**: Google Style или NumPy Style
- **Комментарии**: пишите комментарии для сложной логики

#### Именование

- **Функции и переменные**: `snake_case`
- **Классы**: `PascalCase`
- **Константы**: `UPPER_CASE_WITH_UNDERSCORES`
- **Приватные методы**: начинаются с `_`

### SQL

- Используйте **UPPER CASE** для ключевых слов SQL
- Параметризованные запросы для предотвращения SQL-инъекций
- Используйте осмысленные имена для таблиц и столбцов

### Рекомендуемые инструменты

Хотя проект не требует строгого использования линтеров, рекомендуется:

- **ruff**: для быстрой проверки и форматирования (замена flake8, isort, black)
- **mypy**: для проверки типов (опционально)
- **pytest**: для запуска тестов

Установка:

```bash
pip install ruff mypy
```

Использование:

```bash
# Проверка кода
ruff check .

# Автоматическое исправление
ruff check --fix .

# Форматирование
ruff format .

# Проверка типов
mypy app/
```

## Структура коммитов

Проект использует **Conventional Commits** для единообразия истории изменений.

### Формат сообщения коммита

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Типы коммитов

- **feat**: новая функциональность
- **fix**: исправление ошибки
- **docs**: изменения в документации
- **test**: добавление или изменение тестов
- **refactor**: рефакторинг кода без изменения функциональности
- **style**: форматирование, отсутствующие точки с запятой и т.д.
- **perf**: улучшение производительности
- **chore**: изменения в сборке, зависимостях, инструментах
- **ci**: изменения в CI/CD конфигурации

### Scope (опционально)

Область изменений, например: `api`, `db`, `auth`, `video`, `docs`

### Примеры

```bash
# Новая функциональность
feat(video): Add video streaming API for drone integration

# Исправление бага
fix(db): Update key assignment in _build_reqs_map to handle None field names

# Документация
docs: Add comprehensive API methods documentation for issue #44

# Тесты
test: Add comprehensive rate limiting and security tests

# Рефакторинг
refactor(api): Simplify error handling in terms endpoint
```

### Subject

- Используйте повелительное наклонение ("Add" вместо "Added")
- Не заканчивайте точкой
- Первая буква строчная (кроме имён собственных)
- Максимум 72 символа

### Body (опционально)

Подробное описание изменений:
- Что было изменено и почему
- Какие проблемы решает

### Footer (опционально)

Ссылки на issue или breaking changes:

```
Fixes #123
Closes #456

BREAKING CHANGE: изменён формат API ответа
```

## Процесс создания Pull Request

### 1. Создание ветки

```bash
# Обновите main ветку
git checkout main
git pull upstream main

# Создайте новую ветку для вашей работы
git checkout -b feature/your-feature-name
# или
git checkout -b fix/bug-description
```

### 2. Внесение изменений

- Делайте небольшие, атомарные коммиты
- Следуйте стилю кода проекта
- Добавляйте тесты для новой функциональности
- Обновляйте документацию при необходимости

### 3. Проверка перед отправкой

```bash
# Запустите тесты
pytest tests/

# Проверьте код (если используете ruff)
ruff check .
ruff format .

# Убедитесь, что приложение запускается
docker compose up --build
```

### 4. Отправка изменений

```bash
# Закоммитьте изменения
git add .
git commit -m "feat: Add new feature"

# Отправьте в ваш форк
git push origin feature/your-feature-name
```

### 5. Создание Pull Request

1. Перейдите на GitHub в ваш форк
2. Нажмите **"Compare & pull request"**
3. Заполните шаблон PR:

```markdown
## Описание

Краткое описание изменений и их цель.

## Связанные issue

Fixes #48

## Тип изменений

- [ ] Bug fix (исправление, не ломающее существующую функциональность)
- [ ] New feature (новая функциональность)
- [ ] Breaking change (изменения, нарушающие обратную совместимость)
- [ ] Documentation update (обновление документации)

## Чек-лист

- [ ] Код следует стилю проекта
- [ ] Проведён self-review кода
- [ ] Добавлены комментарии к сложным участкам
- [ ] Обновлена документация
- [ ] Изменения не генерируют новых warnings
- [ ] Добавлены тесты для новой функциональности
- [ ] Все тесты проходят успешно
- [ ] Зависимые изменения были смержены

## Тестирование

Опишите, как были протестированы изменения:

- [ ] Локальное тестирование
- [ ] Unit тесты
- [ ] Integration тесты
- [ ] Manual тестирование

## Скриншоты (если применимо)

Добавьте скриншоты для визуальных изменений.
```

### 6. Требования к PR

- **Заголовок**: следует Conventional Commits (например, `feat: Add new endpoint`)
- **Описание**: чётко объясняет, что и зачем было сделано
- **Тесты**: все новые функции покрыты тестами
- **Документация**: обновлена при необходимости
- **Конфликты**: отсутствуют конфликты с main веткой

## Требования к тестированию

### Структура тестов

```
tests/
├── api/              # Тесты API endpoints
│   └── video/        # Тесты видео API
├── test_term_api.py  # Тесты term endpoints
└── test_object_api.py # Тесты object endpoints
```

### Написание тестов

Проект использует **pytest** с поддержкой асинхронных тестов через **pytest-asyncio**.

#### Пример unit теста с моками

```python
import pytest
from unittest.mock import AsyncMock, MagicMock
from httpx import AsyncClient, ASGITransport

from app.main import app
from app.api import terms


@pytest.fixture
def auth_headers():
    """Returns mock Authorization header for tests."""
    return {"Authorization": "Bearer secret-token"}


@pytest.mark.asyncio
async def test_get_all_terms(monkeypatch, auth_headers):
    """Tests the GET /term endpoint with mocked DB and auth."""
    monkeypatch.setattr(
        terms, "verify_token",
        AsyncMock(return_value={"user_id": 1, "role": "admin"})
    )

    mock_engine = setup_engine_mock([{"id": 64, "val": "test"}])
    monkeypatch.setattr(terms, "engine", mock_engine)

    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as client:
        response = await client.get("/term", headers=auth_headers)

    assert response.status_code == 200
    assert isinstance(response.json(), list)
```

#### Интеграционные тесты

Для интеграционных тестов требуется запущенный сервер:

```python
from httpx import post as httpx_post


def test_create_term_integration():
    """Integration test for POST /term (requires live server)"""
    response = httpx_post(
        "http://localhost:8000/term",
        headers={"Authorization": "Bearer secret-token"},
        json={"val": "test", "t": 3, "mods": {}}
    )
    assert response.status_code == 200
```

### Запуск тестов

```bash
# Все тесты
pytest

# С подробным выводом
pytest -v

# Конкретный файл
pytest tests/test_term_api.py

# Конкретный тест
pytest tests/test_term_api.py::test_get_all_terms

# С покрытием кода
pytest --cov=app tests/
```

### Требования к покрытию

- Новый код должен иметь минимум **80% покрытия** тестами
- Критичная бизнес-логика должна быть покрыта на **100%**
- Добавляйте тесты для edge cases и error handling

## Процесс review и merge

### Review процесс

1. **Автоматические проверки**:
   - CI/CD pipeline должен пройти успешно
   - Все тесты должны быть зелёными

2. **Code Review**:
   - Минимум 1 approve от мэйнтейнера
   - Все комментарии должны быть разрешены
   - Код проверяется на:
     - Соответствие стилю проекта
     - Правильность реализации
     - Наличие тестов
     - Качество документации

3. **Обновления**:
   - Если запрошены изменения, внесите их и обновите PR
   - Отвечайте на комментарии reviewers
   - Держите ветку актуальной с main

### Merge стратегия

- Используется **Squash and Merge** для чистой истории
- Все коммиты в PR объединяются в один при merge
- Сообщение коммита должно следовать Conventional Commits

### После merge

- Ваша ветка будет автоматически удалена
- Изменения попадут в main и будут задеплоены автоматически
- Можете удалить локальную ветку:

```bash
git checkout main
git pull upstream main
git branch -d feature/your-feature-name
```

## Контактная информация

### Вопросы и помощь

Если у вас возникли вопросы или нужна помощь:

1. **GitHub Issues**: создайте issue с меткой `question`
2. **GitHub Discussions**: используйте для обсуждений (если включено)
3. **Pull Request Comments**: задавайте вопросы прямо в PR

### Сообщество

- **Repository**: https://github.com/unidel2035/integramDD
- **Issue Tracker**: https://github.com/unidel2035/integramDD/issues

### Мэйнтейнеры

При необходимости можете упомянуть мэйнтейнеров в issue или PR:
- Проверьте список контрибьюторов в репозитории

---

## Дополнительные ресурсы

- [Документация FastAPI](https://fastapi.tiangolo.com/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Pytest Documentation](https://docs.pytest.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [PEP 8 Style Guide](https://pep8.org/)

---

**Спасибо за ваш вклад в integramDD!** 🎉
