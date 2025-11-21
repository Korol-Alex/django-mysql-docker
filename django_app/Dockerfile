# Базовий образ Python
FROM python:3.11-slim

# Забороняємо створення .pyc файлів та буферизацію stdout/stderr
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Робоча директорія всередині контейнера
WORKDIR /app

# Встановлюємо системні залежності для mysqlclient та компіляції Python пакетів
RUN apt-get update && apt-get install -y \
    build-essential \
    default-libmysqlclient-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Копіюємо requirements.txt і встановлюємо Python пакети
COPY requirements.txt .
RUN pip install --no-cache-dir --disable-pip-version-check -r requirements.txt

# Копіюємо весь код проекту в контейнер
COPY . .

# Команда запуску Django на всіх інтерфейсах контейнера
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
