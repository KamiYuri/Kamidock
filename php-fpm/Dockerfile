#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
FROM php:8.1-fpm
# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt update -yqq && \
    pecl channel-update pecl.php.net && \
    apt install -yqq \
      apt-utils \
      gnupg2 \
      git \
      #
      #--------------------------------------------------------------------------
      # Mandatory Software's Installation
      #--------------------------------------------------------------------------
      #
      # Mandatory Software's such as ("mcrypt", "pdo_mysql", "libssl-dev", ....)
      # are installed on the base image 'laradock/php-fpm' image. If you want
      # to add more Software's or remove existing one, you need to edit the
      # base image (https://github.com/Laradock/php-fpm).
      #
      # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
      libzip-dev zip unzip && \
      docker-php-ext-configure zip  && \
      # Install the zip extension
      docker-php-ext-install zip && \
      php -m | grep -q 'zip'

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------

###########################################################################
# bcmath:
###########################################################################
    # Install the bcmath extension
RUN docker-php-ext-install bcmath
###########################################################################
# xDebug:
###########################################################################
RUN pecl install xdebug-3.1.2 && docker-php-ext-enable xdebug
# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN  sed -i "s/xdebug.remote_host=/xdebug.client_host=/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_connect_back=0/xdebug.discover_client_host=false/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_port=9000/xdebug.client_port=9003/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.profiler_enable=0/; xdebug.profiler_enable=0/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.profiler_output_dir=/xdebug.output_dir=/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_mode=req/; xdebug.remote_mode=req/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_autostart=0/xdebug.start_with_request=yes/" /usr/local/etc/php/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_enable=0/xdebug.mode=debug/" /usr/local/etc/php/conf.d/xdebug.ini 

RUN sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini

###########################################################################
# Mysqli Modifications:
###########################################################################
RUN docker-php-ext-install mysqli

###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "8.1"

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

# Clean up
RUN apt clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Configure non-root user.
ARG PUID=1000
ARG PGID=1000

RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
