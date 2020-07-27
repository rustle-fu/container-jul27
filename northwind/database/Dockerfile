FROM mysql@sha256:bf17a7109057494c45fba5aab7fc805ca00ac1eef638dfdd42b38d5a7190c9bb

ENV MYSQL_ROOT_PASSWORD=changeit MYSQL_DATABASE=northwind MYSQL_PORT=3306

## Add scripts that are executed when the mysql container starts
ADD 0-northwind-default-current-timestamp.sql /docker-entrypoint-initdb.d
ADD 1-northwind-data.sql /docker-entrypoint-initdb.d

## Healthcheck
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
        CMD mysqladmin ping -uroot -p${MYSQL_ROOT_PASSWORD} || exit 1

EXPOSE ${MYSQL_PORT}
VOLUME /var/lib/mysql

