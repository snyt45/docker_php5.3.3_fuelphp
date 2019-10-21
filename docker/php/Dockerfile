FROM centos:6.9

ENV PHP_VERSION 5.3.3

RUN yum install -y \
        php \
        php-mbstring \
        php-pgsql \
        php-mysql \
        php-gd \
        zip \
        unzip \
        git \
        httpd \
        php-devel \
        httpd-devel \
        php-pear \
        gcc && \
    yum clean all

RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer && \
    curl https://get.fuelphp.com/oil | sh

RUN mkdir /app
WORKDIR /app

EXPOSE 80

CMD ["/usr/sbin/apachectl","-DFOREGROUND"]