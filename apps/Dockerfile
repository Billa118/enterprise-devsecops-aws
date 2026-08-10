FROM python:3.11-slim-bookworm

WORKDIR /app

COPY requirements.txt .

RUN apt-get update \
    && apt-get upgrade -y \
    && pip install --no-cache-dir --upgrade pip setuptools wheel \
    && pip install --no-cache-dir -r requirements.txt \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
