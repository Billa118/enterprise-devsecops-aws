FROM python:3.11-slim-bookworm AS builder

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.11-slim-bookworm

WORKDIR /app

COPY --from=builder /install /usr/local

RUN rm -rf /usr/local/lib/python3.11/site-packages/pip /usr/local/lib/python3.11/site-packages/pip-*.dist-info /usr/local/lib/python3.11/site-packages/setuptools /usr/local/lib/python3.11/site-packages/setuptools-*.dist-info /usr/local/lib/python3.11/site-packages/wheel /usr/local/lib/python3.11/site-packages/wheel-*.dist-info /usr/local/lib/python3.11/site-packages/_distutils_hack /usr/local/lib/python3.11/site-packages/pkg_resources

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
