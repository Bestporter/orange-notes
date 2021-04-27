dockerfile

```python
FROM python:3.8
ENV PYTHONUNBUFFERED 1
ENV HTTP_PROXY "http://F7688906:LPnXfx61@10.191.131.15:3128"
ENV HTTPS_PROXY "http://F7688906:LPnXfx61@10.191.131.15:3128"
WORKDIR /project
SHELL ["/bin/sh", "-c"]
COPY start.sh /project/
COPY requirements.txt /project/
RUN pip install -r requirements.txt
```

Docker-compose.yml

```python

version: "3"
services:
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/project
    ports:
      - "8001:8000"
```

