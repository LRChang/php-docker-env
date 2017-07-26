FROM hub.c.163.com/library/mysql:5.7.18
RUN { \
    echo '[mysqld]'; \
    echo 'character-set-server = utf8'; \
    echo 'collation-server = utf8_unicode_ci'; \
    echo 'skip-character-set-client-handshake'; \
} > /etc/mysql/conf.d/charset.cnf