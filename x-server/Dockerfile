FROM php:7.3-fpm
LABEL "MAINTAINER"="Sakarin Kammanee <Sakarin@coe.phuket.psu.ac.th>"

ENV TZ=Asia/Bangkok
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update &&  \
    apt-get install -y --no-install-recommends \
    gcc \
    make \
    re2c \
    libpcre3-dev \
    libxml2-dev \
	zlib1g-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libfreetype6-dev \
    git \
    libmcrypt-dev \
    libldap2-dev  \
    libzip-dev \
    librabbitmq-dev && \
    #other plugins
    docker-php-ext-configure ldap --with-libdir=/lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap && \
    docker-php-ext-install iconv mbstring && \
    docker-php-ext-install pdo pdo_mysql && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install zip xml && \
    docker-php-ext-configure gd --enable-gd-native-ttf --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install gd && \
    docker-php-ext-install zip


#Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

#PHP-PSR
RUN git clone https://github.com/jbboehr/php-psr.git /root/psr && \
    cd /root/psr && \
    phpize && \
    ./configure && \
    make && \
    make test && \
    make install && \
    echo "extension=psr.so" > /usr/local/etc/php/conf.d/29-psr.ini && \
    cd && rm -Rf /root/psr

#rabbit mq
RUN pecl install amqp && \
echo "extension=amqp.so" > /usr/local/etc/php/conf.d/amqp.ini

#custom
RUN echo "post_max_size=21M" > /usr/local/etc/php/conf.d/custom.ini &&  \
    echo "upload_max_filesize=20M" >> /usr/local/etc/php/conf.d/custom.ini && \
    echo "session.gc_maxlifetime=86400" >> /usr/local/etc/php/conf.d/custom.ini 

#get phalcon source
RUN git clone --depth=1 --branch v3.4.3 git://github.com/phalcon/cphalcon.git /root/buildphalcon

#compile phalcon
RUN cd /root/buildphalcon/build/php7/64bits && \
    phpize && export CFLAGS="-O2 -s" && \
    ./configure && make -j 8 && make install && \
    echo "extension=phalcon.so" > /usr/local/etc/php/conf.d/30-phalcon.ini && \
    rm -Rf /root/buildphalcon

#phalcon dev tool
RUN cd /root && \
    git clone https://github.com/phalcon/phalcon-devtools.git && \
    cd /root/phalcon-devtools && \
    ln -s ~/phalcon-devtools/phalcon.php /usr/bin/phalcon && \
    chmod ugo+x /usr/bin/phalcon

RUN echo TLS_REQCERT allow >> /etc/ldap/ldap.conf

#patch mssql
RUN echo "deb http://deb.debian.org/debian jessie main" | tee -a /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y libssl1.0.0


#clean
RUN rm -Rf /root/buildphalcon 

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


WORKDIR /var/www

CMD ["php-fpm"]
