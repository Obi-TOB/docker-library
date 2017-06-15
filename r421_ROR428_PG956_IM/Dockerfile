FROM eclipse/stack-base:ubuntu

################################## Postgres SQL SETUP ###########################################

USER root

# From_old_image
# && apt-get -y -q install python-software-properties software-properties-common \
# && apt-get -y -q install  postgresql-9.5 postgresql-client-9.3 postgresql-contrib-9.3
# Install postgres and related pg gem requirements
RUN apt-get update && apt-get -y install postgresql postgresql-contrib libpq-dev build-essential libreadline-dev

# install some software
RUN apt-get update && apt-get -y install mc vim

USER postgres

RUN /etc/init.d/postgresql start \
    && psql --command "ALTER USER postgres WITH PASSWORD 'postgres';" 

USER root

# Adjust PostgreSQL configuration so that remote connections to the
# database are possible.
# RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/9.5/main/pg_hba.conf
RUN sed -i -e '/local\s*all\s*all\s*peer/d' /etc/postgresql/9.5/main/pg_hba.conf \
	&& sed -i -e '/local\s*all\s*postgres\s*peer/d' /etc/postgresql/9.5/main/pg_hba.conf \
    && echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/9.5/main/pg_hba.conf \
	&& echo "local   all             all                                     md5" >> /etc/postgresql/9.5/main/pg_hba.conf \
	&& echo "local   all             postgres                                     md5" >> /etc/postgresql/9.5/main/pg_hba.conf

# And add ``listen_addresses`` to ``/etc/postgresql/9.5/main/postgresql.conf``
RUN echo "listen_addresses='*'" >> /etc/postgresql/9.5/main/postgresql.conf

# Expose the PostgreSQL port
EXPOSE 5432

RUN mkdir -p /var/run/postgresql && chown -R postgres /var/run/postgresql

# Add VOLUMEs to allow backup of config, logs and databases
#VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql"]

# Set the default command to run when starting the container
# CMD ["/usr/lib/postgresql/9.5/bin/postgres", "-D", "/var/lib/postgresql/9.5/main", "-c", "config_file=/etc/postgresql/9.5/main/postgresql.conf"]

USER user
CMD sudo service postgresql start && tail -F /var/log/mysql/error.log


################################## IMAGE_MAGICK SETUP #######################################

USER root

RUN apt-get -y install imagemagick

#################################### END IMAGE_MAGICK #######################################

# Copyright (c) 2012-2016 Codenvy, S.A.
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Eclipse Public License v1.0
# which accompanies this distribution, and is available at
# http://www.eclipse.org/legal/epl-v10.html
# Contributors:
# Codenvy, S.A. - initial API and implementation

ENV RAILS_VERSION 4.2.8
ENV RUBY_MAJOR 2.4
ENV RUBY_VERSION 2.4.1
ENV RUBY_DOWNLOAD_SHA256 a330e10d5cb5e53b3a0078326c5731888bb55e32c4abfeb27d9e7f8e5d000250
ENV RUBYGEMS_VERSION 2.6.12

USER root
# skip installing gem documentation
RUN mkdir -p /usr/local/etc \
    && echo 'install: --no-document' >> /usr/local/etc/gemrc \
    && echo 'update: --no-document' >> /usr/local/etc/gemrc
USER user

# some of ruby's build scripts are written in ruby
# we purge this later to make sure our final image uses what we just built
RUN set -ex \
    && buildDeps=' \
	bison \
	libgdbm-dev \
	ruby \
    ' \
    && sudo apt-get update \
    && sudo apt-get install -y --no-install-recommends make gcc zlib1g-dev autoconf build-essential libssl-dev libsqlite3-dev $buildDeps \
    && sudo rm -rf /var/lib/apt/lists/* \
    && sudo curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
    && echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
    && sudo mkdir -p /usr/src/ruby \
    && sudo tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1 \
    && sudo rm ruby.tar.gz 

USER root
RUN cd /usr/src/ruby \
    && { sudo echo '#define ENABLE_PATH_CHECK 0'; echo; cat file.c; } > file.c.new && mv file.c.new file.c \
    && autoconf \
    && ./configure --disable-install-doc 
USER user

RUN cd /usr/src/ruby \
    && sudo make -j"$(nproc)" \
    && sudo make install \
    && sudo apt-get purge -y --auto-remove $buildDeps \
    && sudo gem update --system $RUBYGEMS_VERSION \
    && sudo rm -r /usr/src/ruby


ENV BUNDLER_VERSION 1.14.6

RUN sudo gem install bundler --version "$BUNDLER_VERSION"

ENV RAKE_VERSION 12.0.0

RUN sudo gem install rake --version "$RAKE_VERSION"

# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
    BUNDLE_BIN="$GEM_HOME/bin" \
    BUNDLE_SILENCE_ROOT_WARNING=1 \
    BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $BUNDLE_BIN:$PATH
RUN sudo mkdir -p "$GEM_HOME" "$BUNDLE_BIN" \
    && sudo chmod 777 "$GEM_HOME" "$BUNDLE_BIN"

RUN sudo apt-get update && sudo apt-get install -y nodejs --no-install-recommends && sudo rm -rf /var/lib/apt/lists/*

# see http://guides.rubyonrails.org/command_line.html#rails-dbconsole
RUN sudo apt-get update && sudo apt-get install -y mysql-client postgresql-client sqlite3 --no-install-recommends && sudo rm -rf /var/lib/apt/lists/*

RUN sudo gem install rails --version "$RAILS_VERSION"

EXPOSE 3000
