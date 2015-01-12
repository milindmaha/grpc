# Base Dockerfile for gRPC PHP.
#
# Includes PHP installation dependencies, things that are unlikely to vary.
FROM grpc/base

# Install RVM dependencies and other packages
RUN apt-get update && apt-get install -y \
    autoconf \
    automake \
    bison \
    curl \
    g++ \
    gawk \
    gcc \
    groff \
    libc6-dev \
    libffi-dev \
    libgdbm-dev \
    libncurses5-dev \
    libreadline6-dev \
    libsqlite3-dev \
    libssl-dev \
    libtool \
    libyaml-dev \
    make \
    patch \
    procps \
# TODO(mlumish): Uncomment these lines when building against them works
#    php5-common \
#    php5-cli \
#    php5-dev \
#    php-pear \
    pkg-config \
    procps \
    sqlite3 \
    zlib1g-dev

# Install the version of PHP gRPC is tested against
ENV DEBIAN_FRONTEND noniteractive
RUN apt-get update && apt-get install -y libxml2 libxml2-dev  # used by PHP
RUN cd /var/local \
  && curl -o php-5.5.17.tar.gz http://php.net/distributions/php-5.5.17.tar.gz \
  && tar -xf php-5.5.17.tar.gz \
  && cd php-5.5.17 \
  && ./configure --with-zlib=/usr --with-libxml-dir=ext/libxml \
  && make && make install

# Start the daemon that allows access to the protected git-on-borg repos
RUN /var/local/git/gcompute-tools/git-cookie-authdaemon

# Download the patched PHP protobuf so that PHP gRPC clients can be generated
# from proto3 schemas.
RUN git clone https://team.googlesource.com/one-platform-grpc-team/grpc-php-protobuf-php /var/local/git/protobuf-php

# Install ruby (via RVM) as ruby tools are dependencies for building Protobuf
# PHP extensions.
RUN gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3  # Needed for RVM
RUN \curl -sSL https://get.rvm.io | bash -s stable --ruby
ENV PATH /usr/local/rvm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# ronn: a ruby tool used to convert markdown to man pages, used during the
# install of Protobuf extensions
#
# rake: a ruby version of make used to build the PHP Protobuf extension
RUN rvm all do gem install ronn rake

# Install the patched PHP protobuf so that PHP gRPC clients can be generated
# from proto3 schemas.
RUN cd /var/local/git/protobuf-php \
  && rvm all do rake pear:package version=1.0 \
  && pear install Protobuf-1.0.tgz

# Install PHPUnit, used to run the PHP unit tests
RUN wget https://phar.phpunit.de/phpunit.phar \
  && chmod +x phpunit.phar \
  && mv phpunit.phar /usr/local/bin/phpunit

RUN git clone https://team.googlesource.com/one-platform-grpc-team/grpc /var/local/git/grpc
RUN cd /var/local/git/grpc \
  && git submodule update --init --recursive

RUN make static_c shared_c -j12 -C /var/local/git/grpc

# Define the default command.
CMD ["bash"]