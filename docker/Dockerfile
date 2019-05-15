FROM centos:7.4.1708

MAINTAINER reoring <reoring@c16e.com>

ENV APACHE_DOCUMENT_ROOT /var/www/html/public

RUN	curl --silent --location https://rpm.nodesource.com/setup_9.x | bash - && \
		yum -y install gcc-c++ make nodejs npm

RUN  \
	set -eux && \
	yum -y update && \
	yum -y reinstall ca-certificates.noarch && \
	yum -y install deltarpm && \
	yum -y install epel-release && \
	rpm --import http://rpms.famillecollet.com/RPM-GPG-KEY-remi && \
	rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm && \
	yum-config-manager --enable extras && \
	yum-config-manager --enable epel && \
	yum-config-manager --enable remi && \
	yum-config-manager --disable remi-php55 && \
	yum-config-manager --disable remi-php56 && \
	yum-config-manager --disable remi-php70 && \
	yum-config-manager --enable remi-php71 && \
	rpm -ivh https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm && \
	yum-config-manager --enable pgdg96 && \
	curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | tee /etc/yum.repos.d/yarn.repo

RUN yum install -y wget rpm-build && \
    yum groupinstall -y "Development tools" && \
    wget http://vault.centos.org/5.11/os/SRPMS/kcc-2.3-24.2.2.src.rpm && \
    rpmbuild --rebuild kcc-2.3-24.2.2.src.rpm && \
		rpm -ivh /root/rpmbuild/RPMS/x86_64/kcc-2.3-24.2.2.x86_64.rpm && \
		yum groupremove -y "Development tools" && \
		yum remove -y wget rpm-build

RUN \
  yum -y install \
    nkf \
    convmv \
    kcc \
    mysql \
    httpd \
    mod_ssl \
    php \
    php-cli \
    php-mbstring \
    php-mcrypt \
    php-tidy \
    php-xml \
    php-gd \
    php-intl \
    php-redis \
    php-pear \
    php-mysql \
    php-soap \
    php-bcmath \
    php-pecl-zendopcache \
    msmtp \
    php-pecl-imagick \
    php-pecl-memcache \
    php-pecl-memcached \
    php-pecl-zip \
    php-pecl-xdebug \
    php-pecl-ssh2 \
		yarn \
    && yum -y update bash \
    && rm -rf /var/cache/yum/* \
    && yum clean all


###
### Install Tools
###
RUN yum -y update && \
  yum -y install \
    ImageMagick \
    postgresql96 \
    vim
#
# UTC Timezone & Networking
#
RUN ln -sf /usr/share/zoneinfo/UTC /etc/localtime \
  && echo "NETWORKING=yes" > /etc/sysconfig/network

#
# Global Apache configuration changes
#
RUN sed -i \
  -e 's~^ServerSignature On$~ServerSignature Off~g' \
  -e 's~^ServerTokens OS$~ServerTokens Prod~g' \
  -e 's~^#ExtendedStatus On$~ExtendedStatus On~g' \
  -e 's~^DirectoryIndex \(.*\)$~DirectoryIndex \1 index.php~g' \
  -e 's~^NameVirtualHost \(.*\)$~#NameVirtualHost \1~g' \
  -e 's~^    AllowOverride None$~    AllowOverride All~g' \
  /etc/httpd/conf/httpd.conf

#
# Disable Apache directory indexes
#
RUN sed -i \
  -e 's~^IndexOptions \(.*\)$~#IndexOptions \1~g' \
  -e 's~^IndexIgnore \(.*\)$~#IndexIgnore \1~g' \
  -e 's~^AddIconByEncoding \(.*\)$~#AddIconByEncoding \1~g' \
  -e 's~^AddIconByType \(.*\)$~#AddIconByType \1~g' \
  -e 's~^AddIcon \(.*\)$~#AddIcon \1~g' \
  -e 's~^DefaultIcon \(.*\)$~#DefaultIcon \1~g' \
  -e 's~^ReadmeName \(.*\)$~#ReadmeName \1~g' \
  -e 's~^HeaderName \(.*\)$~#HeaderName \1~g' \
  /etc/httpd/conf/httpd.conf

#
# Disable Apache language based content negotiation
#
RUN sed -i \
  -e 's~^LanguagePriority \(.*\)$~#LanguagePriority \1~g' \
  -e 's~^ForceLanguagePriority \(.*\)$~#ForceLanguagePriority \1~g' \
  -e 's~^AddLanguage \(.*\)$~#AddLanguage \1~g' \
  /etc/httpd/conf/httpd.conf

#
# Disable all Apache modules and enable the minimum
#
RUN sed -i \
  -e 's~^\(LoadModule .*\)$~#\1~g' \
  -e 's~^#LoadModule mime_module ~LoadModule mime_module ~g' \
  -e 's~^#LoadModule log_config_module ~LoadModule log_config_module ~g' \
  -e 's~^#LoadModule setenvif_module ~LoadModule setenvif_module ~g' \
  -e 's~^#LoadModule status_module ~LoadModule status_module ~g' \
  -e 's~^#LoadModule authz_host_module ~LoadModule authz_host_module ~g' \
  -e 's~^#LoadModule dir_module ~LoadModule dir_module ~g' \
  -e 's~^#LoadModule alias_module ~LoadModule alias_module ~g' \
  -e 's~^#LoadModule expires_module ~LoadModule expires_module ~g' \
  -e 's~^#LoadModule deflate_module ~LoadModule deflate_module ~g' \
  -e 's~^#LoadModule headers_module ~LoadModule headers_module ~g' \
  -e 's~^#LoadModule alias_module ~LoadModule alias_module ~g' \
  /etc/httpd/conf/httpd.conf

#
# Global PHP configuration changes
#
RUN sed -i \
    -e 's~^;date.timezone =$~date.timezone = Asia/Tokyo~g' \
    -e 's~^;user_ini.filename =$~user_ini.filename =~g' \
    -e 's~^sendmail_path = /usr/sbin/sendmail -t -i$~sendmail_path = /usr/bin/msmtp -C /etc/msmtprc -t -i~g' \
    /etc/php.ini

# Add msmtp example configuration
#
RUN curl http://msmtp.sourceforge.net/doc/msmtprc.txt -o /etc/msmtprc


# Configure document root
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/httpd/conf/*.conf

RUN sed -ri -e 's/memory_limit = 128M/memory_limit = 1024M/' /etc/php.ini


# Purge
RUN rm -rf /sbin/sln \
    ; rm -rf /usr/{{lib,share}/locale,share/{man,doc,info,gnome/help,cracklib,il8n},{lib,lib64}/gconv,bin/localedef,sbin/build-locale-archive} \
    ; rm -rf /var/cache/{ldconfig,yum}/*


COPY config/etc/httpd/conf /etc/httpd/conf
COPY config/etc/httpd/conf.d /etc/httpd/conf.d
COPY config/etc/httpd/conf.modules.d /etc/httpd/conf.modules.d

RUN echo "xdebug.profiler_enable = 0" >> /etc/php.d/15-xdebug.ini
RUN echo "xdebug.profiler_output_name = xdebug.out.%t" >> /etc/php.d/15-xdebug.ini
RUN echo "xdebug.profiler_output_dir = /var/www/html/profile" >> /etc/php.d/15-xdebug.ini
RUN echo "xdebug.profiler_enable_trigger = 1" >> /etc/php.d/15-xdebug.ini

# Install Composer
RUN yum -y --disablerepo=epel --disablerepo=nodesource --disablerepo=pgdg96 --disablerepo=yarn reinstall ca-certificates.noarch
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    composer config -g repos.packagist composer https://packagist.jp && \
		composer global require hirak/prestissimo
ENV COMPOSER_ALLOW_SUPERUSER=1

RUN yum -y reinstall glibc-common
RUN localedef -v -c -i ja_JP -f UTF-8 ja_JP.UTF-8; echo "";

env LANG=ja_JP.UTF-8
RUN rm -f /etc/localtime
RUN ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

EXPOSE 80 443

WORKDIR /var/www/html

CMD /usr/sbin/httpd -c "ErrorLog /dev/stdout" -DFOREGROUND
