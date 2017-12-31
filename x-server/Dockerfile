FROM php:fpm
LABEL "MAINTAINER"="Sakarin Kammanee <Sakarin@coe.phuket.psu.ac.th>"

ENV TZ=Asia/Bangkok
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update &&  \
    apt-get install -y --no-install-recommends \
    gcc \
    libpcre3-dev \
    libxml2-dev \
	zlib1g-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libfreetype6-dev \
    git \
    libmcrypt-dev \
    libldap2-dev && \
    #other plugins
    docker-php-ext-configure ldap --with-libdir=/lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap && \
    docker-php-ext-install iconv mbstring && \
    docker-php-ext-install pdo pdo_mysql && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install zip xml && \
    docker-php-ext-configure gd --enable-gd-native-ttf --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install gd


# Install Dependencies
RUN ACCEPT_EULA=Y apt-get install -y \
    locales unixodbc unixodbc-dev \
    && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen

ADD . .

RUN ACCEPT_EULA=Y dpkg -i msodbcsql_17.0.0.5-1_amd64.deb && \
    ACCEPT_EULA=Y dpkg -i mssql-tools_17.0.0.5-1_amd64.deb && \
    rm *.deb

# Install pdo_sqlsrv and sqlsrv from PECL. Replace pdo_sqlsrv-4.1.8preview with preferred version.
RUN pecl install pdo_sqlsrv-5.2.0RC1 sqlsrv-5.2.0RC1 \
    && docker-php-ext-enable pdo_sqlsrv sqlsrv

#Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

#mongodb
RUN apt-get install -y --no-install-recommends libssl-dev && \
    pecl install mongodb && \
    echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/mongo.ini 

#custom
RUN echo "post_max_size=21M" > /usr/local/etc/php/conf.d/custom.ini &&  \
    echo "upload_max_filesize=20M" >> /usr/local/etc/php/conf.d/custom.ini && \
    echo "session.gc_maxlifetime=86400" >> /usr/local/etc/php/conf.d/custom.ini 

#get phalcon source
RUN git clone --depth=1 git://github.com/phalcon/cphalcon.git /root/buildphalcon

#compile phalcon
RUN cd /root/buildphalcon/build/php7/64bits && \
    phpize && export CFLAGS="-O2 -s" && \
    ./configure && make -j 8 && make install && \
    echo "extension=phalcon.so" > /usr/local/etc/php/conf.d/30-phalcon.ini && \
    rm -Rf /root/buildphalcon

#lib mcrypt
RUN pecl install mcrypt-1.0.1 && \ 
    echo "extension=mcrypt.so" >> /usr/local/etc/php/conf.d/custom.ini 

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
