FROM mysql:5.7
LABEL maintainer="farshid.imanipour@gmail.com"

ENV MYSQL_ALLOW_EMPTY_PASSWORD=yes

COPY *.sql /docker-entrypoint-initdb.d/

