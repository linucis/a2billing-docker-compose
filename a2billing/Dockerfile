FROM debian:jessie
LABEL maintainer="farshid.imanipour@gmail.com"

RUN apt-get update && apt-get install -y curl wget git unzip asterisk mysql-client apache2 php5 php5-common php5-cli libapache2-mod-php5 php5-mysql vim-tiny php-pear php-db php5-gd php5-curl php-soap php5-mcrypt openssh-server subversion

RUN mkdir -p /usr/share/a2billing/latest/

WORKDIR /usr/share/a2billing/

RUN wget -O master.tar.gz --no-check-certificate https://codeload.github.com/Star2Billing/a2billing/tar.gz/master && \
    tar zxf master.tar.gz && \
    mv a2billing-master/* /usr/share/a2billing/latest/ && \
    rm -rf a2billing-master master.tar.gz

RUN chmod u+xwr /usr/share/a2billing/latest/admin/templates_c && \
    chmod a+w /usr/share/a2billing/latest/admin/templates_c && \
    chmod u+xwr /usr/share/a2billing/latest/agent/templates_c && \
    chmod a+w /usr/share/a2billing/latest/agent/templates_c && \
    chmod u+xwr /usr/share/a2billing/latest/customer/templates_c && \
    chmod a+w /usr/share/a2billing/latest/customer/templates_c

RUN chown -Rf www-data:www-data /usr/share/a2billing/latest/admin/templates_c && \
    chown -Rf www-data:www-data /usr/share/a2billing/latest/customer/templates_c && \
    chown -Rf www-data:www-data /usr/share/a2billing/latest/agent/templates_c

RUN cp  /usr/share/a2billing/latest/a2billing.conf /etc/a2billing.conf

WORKDIR /etc/apache2/sites-enabled/

RUN wget https://raw.github.com/Star2Billing/a2billing/develop/addons/apache2/a2billing_admin.conf && \
    wget https://raw.github.com/Star2Billing/a2billing/develop/addons/apache2/a2billing_customer.conf

RUN ln -s /usr/share/a2billing/latest/AGI/a2billing.php /usr/share/asterisk/agi-bin/a2billing.php && \
    chown asterisk:asterisk /usr/share/asterisk/agi-bin/a2billing.php && \
    chmod +x /usr/share/asterisk/agi-bin/a2billing.php

WORKDIR /usr/share/a2billing/latest/addons/sounds

RUN ./install_a2b_sounds.sh

RUN chown -R asterisk:asterisk /usr/share/asterisk/

WORKDIR /etc/asterisk

RUN wget -O extensions_a2billing.conf https://raw.github.com/Star2Billing/a2billing/develop/addons/asterisk-conf/extensions_a2billing_1_8.conf

RUN echo "Adding A2Billing extensions to /etc/asterisk/extensions.conf" && \
    echo ' \
    ; include a2billing extension \
    #include extensions_a2billing.conf \
    ' >> /etc/asterisk/extensions.conf

RUN sed -i "s/localhost/a2sql/g" /etc/a2billing.conf && \
    sed -i "s/a2billing_dbuser/a2billinguser/g" /etc/a2billing.conf && \
    sed -i "s/a2billing_dbpassword/a2billing/g" /etc/a2billing.conf && \
    sed -i "s/a2billing_dbname/mya2billing/g" /etc/a2billing.conf && \
    sed -i "s/port =/port = 3306/g" /etc/a2billing.conf

ENV APACHE_DOCUMENT_ROOT /var/www/
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -i 's/\(^upload_max_filesize = \).*/\120M/' /etc/php5/apache2/php.ini && \
    cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf_orig && \
    sed -i 's/^\(User\|Group\).*/\1 asterisk/' /etc/apache2/apache2.conf && \
    sed -i 's/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

RUN mkdir /var/www/a2billing && \
    chown www-data:www-data /var/www/a2billing && \
    mkdir -p /var/lib/a2billing/script && \
    mkdir -p /var/run/a2billing && \
    ln -s /usr/share/a2billing/latest/admin /var/www/a2billing/admin && \
    ln -s /usr/share/a2billing/latest/agent /var/www/a2billing/agent && \
    ln -s /usr/share/a2billing/latest/customer /var/www/a2billing/customer && \
    ln -s /usr/share/a2billing/latest/common /var/www/a2billing/common

WORKDIR /usr/share/a2billing/latest/

RUN curl -sS https://getcomposer.org/installer | php && \
    php composer.phar update    && \
    php composer.phar install

RUN mkdir /bootstrap

ADD start.sh /bootstrap/start.sh
RUN chmod 755 /bootstrap/start.sh && chown -R www-data:www-data /var/www/

EXPOSE 80
EXPOSE 5060
ENTRYPOINT ["/bootstrap/start.sh"]
