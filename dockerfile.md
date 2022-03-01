dockerfile

```python
FROM python:3.8
ENV PYTHONUNBUFFERED 1
ENV HTTP_PROXY "http://id:passwd@10.19.13.15:3128"
ENV HTTPS_PROXY "http://id:passwd@10.191.13.15:3128"
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
[Docker Practice](https://vuepress.mirror.docker-practice.com/compose/introduction/)
`docker-compose up`启动
