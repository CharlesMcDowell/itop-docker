FROM phusion/baseimage:bionic-1.0.0
MAINTAINER Vladimir Kunin <vladimir@knowitop.ru>

ARG DEBIAN_FRONTEND=noninteractive
ARG ITOP_VERSION=3.0.0-beta
ARG ITOP_FILENAME=iTop-3.0.0-beta-7312.zip

RUN apt-get install -y software-properties-common \
    && add-apt-repository -y ppa:ondrej/php \
    && apt-get update \
    && apt-get install -y \
        apache2 \
        mysql-server \
        php5 php5-mysql php5-ldap php5-mcrypt php5-cli php5-soap \
        graphviz \
        curl \
        unzip\
        git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && update-alternatives --set php /usr/bin/php7.3

# Get prerequisites software
RUN yum install httpd \
    && yum install mysql mysql-server \
    && yum install php php-mysql php-mcrypt php-xml php-cli php-soap

# Get iTop
RUN rm -rf /var/www/html/* \
    && mkdir -p /tmp/itop \
    && curl -SL -o /tmp/itop/itop.zip https://sourceforge.net/projects/itop/files/itop/$ITOP_VERSION/$ITOP_FILENAME/download \
    && unzip /tmp/itop/itop.zip -d /tmp/itop/ \
    && mv /tmp/itop/web/* /var/www/html \
    && rm -rf /tmp/itop



# Copy services, configs and scripts
COPY service /etc/service/
COPY artifacts/scripts /
COPY artifacts/itop-cron.logrotate /etc/logrotate.d/itop-cron
COPY artifacts/apache2.fqdn.conf /etc/apache2/conf-available/fqdn.conf
COPY run.sh /run.sh
RUN chmod +x -R /etc/service \
    && chmod +x /*.sh \
    && a2enconf fqdn

# Create shortcuts for the right management scripts, get latest Russian translations and fix rights
RUN ln -s /make-itop-config-writable.sh /usr/local/bin/conf-w \
    && ln -s /make-itop-config-read-only.sh /usr/local/bin/conf-ro \
    && /update-russian-translations.sh \
    && chown -R www-data:www-data /var/www/html

EXPOSE 3306
RUN /usr/local/bin/conf-w 
HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1

ENTRYPOINT ["/run.sh"]