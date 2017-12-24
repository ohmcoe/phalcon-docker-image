FROM php:5.6-fpm
MAINTAINER Sakarin Kammanee <Sakarin@coe.phuket.psu.ac.th>

RUN sed -i 's|httpredir|ftp.jp|g' /etc/apt/sources.list

RUN apt-get update &&  \
    apt-get install -y --no-install-recommends \
    gcc \ 
    g++ \
    libpcre3-dev \
    #require for soap
    libxml2-dev \ 
    #require for zip
    zlib1g-dev \
    #require for gd
    libjpeg62-turbo-dev \
    libpng12-dev \
    libfreetype6-dev \
    #other
    git \
    libmcrypt-dev && \
    docker-php-ext-install iconv mcrypt mbstring && \
    #docker-php-ext-install pdo pdo_mysql && \
    docker-php-ext-install pdo pdo_mysql && \
    docker-php-ext-install soap zip xml && \
    docker-php-ext-configure gd --enable-gd-native-ttf --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install gd

#Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

#imap
RUN apt-get -y install --no-install-recommends libssl-dev libc-client2007e-dev libkrb5-dev &&\
    docker-php-ext-configure imap --with-imap-ssl --with-kerberos && \
    docker-php-ext-install imap 

#mongodb
RUN apt-get install -y --no-install-recommends libssl-dev && \
    pecl install mongo && \
    echo "extension=mongo.so" > /usr/local/etc/php/conf.d/mongo.ini 

#custom
RUN echo "post_max_size=21M" > /usr/local/etc/php/conf.d/custom.ini &&  \
    echo "upload_max_filesize=20M" >> /usr/local/etc/php/conf.d/custom.ini && \
    echo "session.gc_maxlifetime=86400" >> /usr/local/etc/php/conf.d/custom.ini 

#get phalcon source
RUN git clone --depth=1 git://github.com/phalcon/cphalcon.git /root/buildphalcon

#compile phalcon
RUN cd /root/buildphalcon/build/php5/64bits && \
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

RUN cat /usr/src/php/php.ini-production | sed 's|^;\(date.timezone.*\)|\1 \"Asia\/Bangkok\"|' > /usr/local/etc/php/php.ini

#clean image
RUN rm -Rf /root/buildphalcon 

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR /var/www
