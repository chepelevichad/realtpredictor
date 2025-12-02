# **Программное средство оценки рыночной стоимости недвижимости на основе нейронной сети (RealtPredictor)**

**Описание проекта:** 
Проект представляет собой программное средство для оценки рыночной стоимости недвижимости на основе нейронной сети. Основой проекта является веб-приложение, которое позволяет пользователям вводить параметры объекта недвижимости и получать точную рыночную оценку, сгенерированную нейросетевой моделью. Система также предоставляет аналитические инструменты для риэлторов и администраторов.

**Цель программного средства:** 
Повысить точность оценки рыночной стоимости недвижимости на 15% по сравнению с традиционными методами, сократить время на подготовку отчета для риэлторов с нескольких часов до 10 минут и предоставить клиентам прозрачный и быстрый инструмент для принятия обоснованных решений при сделках с недвижимостью.

**Основные возможности:**
1.  **Оценка недвижимости**: Ввод параметров объекта и получение рыночной стоимости на основе нейросетевой модели.
2.  **Анализ рынка**: Доступ к аналитическим данным и визуализации рыночных трендов.
3.  **Генерация отчетов**: Автоматическое создание подробных PDF-отчетов по результатам оценки.
4.  **Управление моделями и данными**: Интерфейс для администраторов для управления датасетами и жизненным циклом (обучение/архивация) нейросетевых моделей.

**Ссылки на репозитории:**
*   **Серверная часть:** https://github.com/chepelevichad/realtpredictor_server
*   **Клиентская часть:** https://github.com/chepelevichad/realtdredictor_client

---

## **Содержание**

1. [Архитектура](#архитектура)
	1. [C4-модель](#c4-модель)
	2. [Схема данных](#схема-данных)
2. [Функциональные возможности](#функциональные-возможности)
	1. [Диаграмма вариантов использования](#диаграмма-вариантов-использования)
	2. [User-flow диаграммы](#user-flow-диаграммы)
3. [Детали реализации](#детали-реализации)
	1. [UML-диаграммы](#uml-диаграммы)
	2. [Спецификация API](#спецификация-api)
	3. [Безопасность](#безопасность)
	4. [Оценка качества кода](#оценка-качества-кода)
4. [Тестирование](#тестирование)
	1. [Unit-тесты](#unit-тесты)
	2. [Интеграционные тесты](#интеграционные-тесты)
5. [Установка и запуск](#установка-и-запуск)
	1. [Манифесты для сборки docker образов](#манифесты-для-сборки-docker-образов)
	2. [Манифесты для развертывания k8s кластера](#манифесты-для-развертывания-k8s-кластера)
6. [Лицензия](#лицензия)
7. [Контакты](#контакты)

---
## **Архитектура**

### C4-модель

**Контейнерный уровень архитектуры ПС**

![C4 Container Diagram](https://github.com/user-attachments/assets/64110863-93b9-4d89-902c-2b28fb0646b4)

Программное средство оценки рыночной стоимости недвижимости на основе нейронной сети «RealtPredictor» представлено как специализированная информационная система, обеспечивающая взаимодействие между тремя основными группами пользователей:

*   **Клиенты**: Вводят ключевые параметры объекта (адрес, площадь, количество комнат и т.д.) через веб-интерфейс (Vue.js) и получают результат оценки и отчеты.
*   **Риэлторы и оценщики**: Используют систему как профессиональный инструмент для быстрой оценки объектов и анализа рыночных трендов.
*   **Администраторы**: Обеспечивают техническую поддержку, управляют учетными записями и контролируют обучение нейронных сетей.

**Стек технологий:**
*   **Frontend**: Vue.js SPA.
*   **Backend**: Python (Django Rest Framework / FastAPI).
*   **Database**: PostgreSQL / MySQL.
*   **ML Engine**: TensorFlow/Scikit-learn.

**Компонентный уровень архитектуры ПС**

![C4 Component Diagram](https://github.com/user-attachments/assets/474dbf7e-66cd-4696-92bc-f30fed17e2c7)

Серверная часть разбита на модули (микросервисы):
*   **Prediction Service**: Валидация данных, препроцессинг и инференс ML-модели.
*   **Auth Service**: Управление пользователями и JWT-токенами.
*   **Data Service**: Управление датасетами.
*   **Report Service**: Генерация PDF-отчетов.

### Схема данных

Схема базы данных спроектирована в соответствии с **Третьей нормальной формой (3НФ)**. Данные нормализованы, транзитивные зависимости исключены путем вынесения справочников (`districts`, `cities`, `building_types`).

**Основные сущности:**
*   `users` — пользователи и роли.
*   `properties` — характеристики недвижимости (площадь, этаж, год постройки).
*   `prediction_requests` — история оценок.
*   `listings` — рыночные данные для обучения.

**Пример SQL-скрипта инициализации:**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role_id INTEGER REFERENCES roles(id)
);

CREATE TABLE properties (
    id SERIAL PRIMARY KEY,
    address VARCHAR(255),
    total_area DECIMAL(10, 2),
    floor INTEGER,
    total_floors INTEGER,
    district_id INTEGER REFERENCES districts(id),
    build_year INTEGER
);

CREATE TABLE prediction_requests (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    request_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    predicted_price DECIMAL(12, 2)
);
```

Полный скрипт доступен в файле: `/database/init.sql`.

---

## **Функциональные возможности**

### Диаграмма вариантов использования

![Use Case Diagram](<img width="1015" height="719" alt="вариантов использования drawio" src="https://github.com/user-attachments/assets/e30c0553-df6f-47b2-903f-5dea74799c27" />)

Система определяет два ключевых актора:
1.  **Пользователь**: Вводит параметры недвижимости, получает оценку стоимости, просматривает историю запросов.
2.  **Администратор**: Управляет пользователями, просматривает статистику, загружает и активирует новые ML-модели.

### User-flow диаграммы

**Процесс оценки недвижимости (Основной сценарий):**

![User Flow Activity Diagram](<img width="902" height="721" alt="деятельности drawio" src="https://github.com/user-attachments/assets/cef1d2fb-e56f-4b39-bcdc-576c89bf2c82" />)

1.  **Ввод данных**: Пользователь заполняет форму.
2.  **Обработка**: Нажатие кнопки "Оценить" отправляет запрос на сервер.
3.  **Валидация**: Система проверяет корректность данных (например, этаж <= этажности дома).
4.  **Предикт**: Данные проходят предобработку и попадают в ML-модель.
5.  **Результат**: Цена сохраняется в БД и отображается пользователю.

---

## **Детали реализации**

### UML-диаграммы

**1. Диаграмма развертывания (Deployment Diagram)**
![Deployment Diagram](<img width="841" height="681" alt="развертывания drawio" src="https://github.com/user-attachments/assets/d6a25ebe-b4ca-4ff8-b47a-1045c5935c65" />)

Архитектура развертывания включает:
*   **Web Server (Nginx)**: Обратный прокси для статики и перенаправления запросов.
*   **Application Server**: Запускает Python приложение (через WSGI/Gunicorn).
*   **Database Server**: Отдельный узел для СУБД (MySQL/PostgreSQL).

**2. Диаграмма пакетов (Package Diagram)**
![Package Diagram](<img width="702" height="601" alt="пакетов drawio" src="https://github.com/user-attachments/assets/bbd52744-32a6-4da0-829e-0d071580785c" />)

Структура приложения разделена на слои:
*   **Controller/Route**: Обработка входящих HTTP-запросов.
*   **Service**: Бизнес-логика.
*   **Model/ORM**: Взаимодействие с данными.

**3. Диаграмма последовательности (Sequence Diagram)**
![Sequence Diagram](<img width="996" height="701" alt="последовательности drawio" src="https://github.com/user-attachments/assets/bc7c4571-238b-40fe-ba06-97804e8959e5" />)

Детализация процесса оценки:
1.  API получает JSON.
2.  `PredictionService` вызывает валидацию.
3.  `MLModelWrapper` выполняет `preprocess()` и `predict()`.
4.  Результат сохраняется через `DBRepository`.

**4. Диаграмма деятельности (Жизненный цикл модели)**
![Activity Diagram ML](<img width="391" height="661" alt="деятельности процесса drawio" src="https://github.com/user-attachments/assets/7b6e7ba6-c016-4786-ab57-2e0089dbe9d0" />)

Описывает состояния нейросети: от загрузки и обучения до активации в продакшн или архивации.

### Спецификация API

API реализовано на базе **Django REST Framework**. Основные эндпоинты:

*   `POST /api/auth/login/` — Аутентификация (JWT).
*   `POST /api/predict/price/` — Оценка стоимости.
    *   *Input*: `{"total_area": 50, "floor": 3, "district_id": 5, ...}`
    *   *Output*: `{"estimated_price": 75000, "currency": "USD"}`
*   `GET /api/history/` — История запросов пользователя.

### Безопасность

*   **Аутентификация**: JWT (JSON Web Tokens). Access-токены живут короткое время.
*   **Хеширование паролей**: Алгоритм PBKDF2/Argon2.
*   **Валидация**: Строгая проверка типов данных на уровне сериализаторов DRF.
*   **HTTPS**: Все соединения шифруются.

### Оценка качества кода

Качество кода оценено статическими анализаторами (`pylint`, `radon`):

| Метрика | Значение | Интерпретация |
| :--- | :--- | :--- |
| **Maintainability Index (MI)** | 81.2 | Высокая сопровождаемость кода. |
| **Cyclomatic Complexity** | 2.8 | Низкая сложность алгоритмов. |
| **Pylint Score** | 8.90/10 | Соответствие стандартам PEP 8. |
| **Test Coverage** | 78% | Покрытие тестами основных модулей. |

---

## **Тестирование**

### Unit-тесты

Тестирование изолированных функций бизнес-логики (препроцессинг, валидаторы).

**Пример теста нормализации данных (pytest):**
```python
from ml_engine.preprocessing import normalize_input_data

def test_normalize_data_typical_values():
    """Проверка приведения площади и этажа к диапазону 0..1"""
    result = normalize_input_data(
        total_area=50.0,
        floor=5,
        max_area_const=100.0,
        max_floor_const=10
    )
    assert result["area"] == 0.5
    assert result["floor"] == 0.5
```

### Интеграционные тесты

Проверка взаимодействия API, БД и ML-модуля.

**Пример теста эндпоинта оценки:**
```python
import pytest
from rest_framework.test import APIClient

@pytest.mark.django_db
def test_predict_price_endpoint():
    client = APIClient()
    payload = {
        "total_area": 45.0,
        "rooms_count": 2,
        "floor": 3,
        "location_id": 1
    }
    # Эмуляция POST запроса
    response = client.post("/api/predict/price/", payload, format="json")
    
    assert response.status_code == 200
    data = response.json()
    assert "estimated_price" in data
    assert data["estimated_price"] > 0
```

---

## **Установка и запуск**

### Манифесты для сборки docker образов

Проект контейнеризирован. Примеры `Dockerfile`:

**Backend:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "realtpredictor.wsgi:application", "--bind", "0.0.0.0:8000"]
```

**Frontend:**
```dockerfile
# Build Stage
FROM node:16 as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
# Production Stage
FROM nginx:stable-alpine
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Манифесты для развертывания k8s кластера

Пример Deployment для бэкенда в Kubernetes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: realtpredictor-backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: django-app
        image: chepelevichad/realtpredictor-server:latest
        ports:
        - containerPort: 8000
        env:
        - name: DB_HOST
          value: "postgres-service"
```

---

## **Лицензия**

Этот проект лицензирован по лицензии MIT - подробности представлены в файле [LICENSE](LICENSE)

---

## **Контакты**

Автор: vladvladchepelevich@gmail.com
```
