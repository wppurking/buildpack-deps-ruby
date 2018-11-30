FROM ubuntu:16.04
LABEL maintainer="wppurking@gmail.com"

# refs: https://github.com/GoogleCloudPlatform/ruby-docker/blob/master/ruby-base/Dockerfile
# Install dependencies for Ruby
# Also installs dependencies for the following common gems:
#
# gems             dependencies
# ------------------------------------------------------------------
# curb             libcurl3, libcurl3-gnutls, libcurl4-openssl-dev
# pg               libpq-dev
# rmagick          imagemagick, libmagickwand-dev
# nokogiri         libxml2-dev, libxslt-dev
# sqlite3          libsqlite3-dev
# mysql2           libmysqlclient-dev
# paperclip        file
# charlock_holmes  libicu-dev
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		bzip2 curl git vim apt-utils autoconf \
        build-essential \
        ca-certificates file \
        libcurl3 libcurl3-gnutls libcurl4-openssl-dev \
        libffi-dev \
        libgdbm-dev \
        libgmp-dev \
        libicu-dev \
		libgdbm3 \
        imagemagick libmagickwand-dev \
        libjemalloc-dev libjemalloc1 \
        libpq-dev libmysqlclient-dev libsqlite3-dev \
        libxml2-dev libxslt-dev \
        libncurses5-dev \
        libqdbm-dev \
        libreadline6-dev \
        libssl-dev \
        libyaml-dev \
        libz-dev

# skip installing gem documentation
RUN mkdir -p /usr/local/etc \
	&& { \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 2.5
ENV RUBY_VERSION 2.5.1
ENV RUBY_DOWNLOAD_SHA256 dac81822325b79c3ba9532b048c2123357d3310b2b40024202f360251d9829b1
ENV RUBYGEMS_VERSION 2.7.6
ENV BUNDLER_VERSION 1.16.1

# some of ruby's build scripts are written in ruby
# we purge this later to make sure our final image uses what we just built
RUN set -ex \
	&& buildDeps=' \
		bison \
		libbz2-dev \
		libglib2.0-dev \
		libncurses-dev \
		libreadline-dev \
		ruby \
	' \
	&& apt-get install -y --no-install-recommends $buildDeps \
	&& curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
	&& echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
	&& mkdir -p /usr/src/ruby \
	&& tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1 \
	&& rm ruby.tar.gz \
	&& cd /usr/src/ruby \
	&& { echo '#define ENABLE_PATH_CHECK 0'; echo; cat file.c; } > file.c.new && mv file.c.new file.c \
	&& autoconf \
	&& ./configure --with-jemalloc --disable-install-doc \
	&& make -j"$(nproc)" \
	&& make install \
	&& apt-get purge -y --auto-remove $buildDeps \
	&& gem update --system $RUBYGEMS_VERSION \
	&& rm -r /usr/src/ruby
RUN gem install bundler --version "$BUNDLER_VERSION"

# install nodejs
RUN curl --silent --location https://deb.nodesource.com/setup_8.x | bash -
RUN apt-get install -y nodejs
RUN npm i -g yarn

# Clean up apt
RUN apt-get clean && rm -f /var/lib/apt/lists/*_*
# clean up for docker squash
RUN   rm -fr /usr/share/man &&\
      rm -fr /usr/share/doc &&\
      rm -fr /usr/share/vim/vim74/tutor &&\
      rm -fr /usr/share/vim/vim74/doc &&\
      rm -fr /usr/share/vim/vim74/lang &&\
      rm -fr /usr/local/share/doc &&\
      rm -fr /usr/local/share/ruby-build &&\
      rm -fr /root/.gem &&\
      rm -fr /root/.npm &&\
      rm -fr /tmp/* &&\
      rm -fr /usr/share/vim/vim74/spell/en*

# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
	BUNDLE_BIN="$GEM_HOME/bin" \
	BUNDLE_SILENCE_ROOT_WARNING=1 \
	BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $BUNDLE_BIN:$PATH
RUN mkdir -p "$GEM_HOME" "$BUNDLE_BIN" \
	&& chmod 777 "$GEM_HOME" "$BUNDLE_BIN"

# Common environment variables for framework configuration
ENV RACK_ENV=production \
    RAILS_ENV=production \
    APP_ENV=production \
    RAILS_SERVE_STATIC_FILES=true \
    RAILS_LOG_TO_STDOUT=true

# Initialize entrypoint
WORKDIR /app
EXPOSE 3000
ENV PORT=3000
ENTRYPOINT []
CMD [ "irb" ]