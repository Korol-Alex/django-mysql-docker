Ось готовий повний файл README для твого проекту, який можна скопіювати цілком:

# Django + MySQL + Docker

Цей проєкт демонструє, як підняти **Django** з **MySQL** в **Docker** контейнерах за допомогою **Docker Compose**.  

---

## 📦 Структура проекту



myproject/
├─ django_app/ # Код Django проекту
│ ├─ myproject/
│ │ ├─ settings.py
│ │ └─ ...
│ └─ manage.py
├─ Dockerfile # Dockerfile для Django
├─ docker-compose.yml # Конфігурація Docker Compose для Django + MySQL
├─ requirements.txt # Python залежності
├─ .gitignore
└─ README.md


---

## 🛠 Технології

- Python 3.11
- Django 4.2
- MySQL 8.0
- Docker + Docker Compose
- PyMySQL (для підключення Django → MySQL)
- mysqlclient (як додатковий драйвер)

---

## ⚙️ Налаштування

### 1. Створення `requirements.txt`



Django>=4.2,<5.0
mysqlclient
PyMySQL>=1.0


---

### 2. Dockerfile для Django

```dockerfile
FROM python:3.11-slim

# Забороняємо створення .pyc файлів та буферизацію stdout/stderr
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Робоча директорія
WORKDIR /app

# Встановлюємо системні залежності для mysqlclient
RUN apt-get update && apt-get install -y \
    build-essential \
    default-libmysqlclient-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Копіюємо requirements і встановлюємо Python пакети
COPY requirements.txt .
RUN pip install --no-cache-dir --disable-pip-version-check -r requirements.txt

# Копіюємо весь код проекту
COPY . .

# Запуск Django development server
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

3. docker-compose.yml
version: '3.9'

services:
  db:
    image: mysql:8.0
    container_name: mysql_container
    restart: always
    environment:
      MYSQL_DATABASE: mydb
      MYSQL_USER: myuser
      MYSQL_PASSWORD: mypass
      MYSQL_ROOT_PASSWORD: rootpass
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

  web:
    build: .
    container_name: django_container
    restart: always
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      DB_HOST: db
      DB_NAME: mydb
      DB_USER: myuser
      DB_PASSWORD: mypass

volumes:
  db_data:

4. Django settings.py для MySQL
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': '3306',
    }
}


Для PyMySQL додайте на початку settings.py:

import pymysql
pymysql.install_as_MySQLdb()

5. Запуск проекту

Збірка і запуск контейнерів:

docker-compose up --build -d


Перевірка логів Django:

docker-compose logs web --tail 20


Застосування міграцій:

docker exec -it django_container python manage.py migrate


Перевірка підключення до MySQL з Django контейнера:

docker exec -it django_container python
>>> import os
>>> import pymysql
>>> conn = pymysql.connect(
...     host=os.environ.get('DB_HOST'),
...     user=os.environ.get('DB_USER'),
...     password=os.environ.get('DB_PASSWORD'),
...     database=os.environ.get('DB_NAME')
... )
>>> print("Connected!")
>>> conn.close()

6. .gitignore
__pycache__/
*.pyc
*.pyo
*.pyd
*.sqlite3
.env
venv/
django_app/venv/
*.egg-info/
*.log
db_data/

💡 Поради

Кожен новий проект на окремому EC2 краще тримати в окремому Git репозиторії.

Для безпечної роботи з GitHub використовуйте SSH ключі замість пароля.

Дані бази зберігайте у Docker volume (db_data) щоб вони не губились при перезапуску контейнера.

✅ Результат

Після запуску ви отримаєте:

Django проект на http://<EC2-IP>:8000/

MySQL база доступна з Django через PyMySQL

Можливість швидко розгортати проект на іншому сервері за допомогою Docker Compose