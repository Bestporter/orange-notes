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
## docker file
```docker

FROM python:3.6.8

ENV PYTHONUNBUFFERED 1

RUN mkdir -p /code
COPY ./requirements.txt /code

WORKDIR /code

RUN sed -i "s/archive.ubuntu./mirrors.aliyun./g" /etc/apt/sources.list
RUN sed -i "s/deb.debian.org/mirrors.aliyun.com/g" /etc/apt/sources.list

RUN apt-get clean && apt-get -y update && \
    apt-get -y install libsasl2-dev python-dev libldap2-dev libssl-dev libsnmp-dev
RUN pip3 install --index-url https://mirrors.aliyun.com/pypi/simple/ --no-cache-dir -r requirements.txt

COPY ./* /code/
```
## docker compose 
```docker
version: '3'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    image: web
    container_name: web
    hostname: web
    restart: always
    command: python /code/manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/web
    ports:
      - "8000:8000"
    depends_on:
      - mysql  

  mysql:
    image: mysql
    container_name: mysql
    hostname: mysql
    restart: always
    command: --default-authentication-plugin=mysql_native_password --mysqlx=0
    ports:
      - 3306:3306
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_HOST=localhost 
      - MYSQL_PORT=3306 
      - MYSQL_DATABASE=dev
      - MYSQL_USER=dev
      - MYSQL_PASSWORD=123456
      - MYSQL_ROOT_PASSWORD=123456
```
