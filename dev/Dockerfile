FROM debian:latest

ARG OPENGFX_VERSION="0.5.2"
ARG FREEPLAY_VERSION="0.22.2"

# Get things ready
RUN mkdir -p /config \
    && mkdir /tmp/build \
    && useradd -d /config -u 911 -s /bin/false openttd \
    && chown -R openttd:openttd /config

# Install some build dependencies (we remove these later to save space)
RUN apt-get update && \
    apt-get install -y \
    unzip \
    g++ \
    make \
    patch \
    subversion \
    zlib1g-dev \
    liblzma-dev \
    liblzo2-dev \
    pkg-config \
    wget

RUN mkdir -p /openttd

# Build OpenTTD itself
WORKDIR /openttd
RUN wget https://github.com/Ryccoo/OpenTTD-patches/archive/freeplay-$FREEPLAY_VERSION.tar.gz -O freeplay.tar.gz
RUN tar xfz freeplay.tar.gz
WORKDIR /openttd/OpenTTD-patches-freeplay-$FREEPLAY_VERSION/

RUN mkdir -p ///usr/local/share/games/openttd/data
RUN /openttd/OpenTTD-patches-freeplay-$FREEPLAY_VERSION/configure \
    --enable-dedicated \
    --binary-dir=bin \
    --personal-dir=/ \
    —-enable-debug
RUN make -j5
RUN make install

# Grab OpenGFX as tagged
ADD https://binaries.openttd.org/extra/opengfx/$OPENGFX_VERSION/opengfx-$OPENGFX_VERSION-all.zip opengfx.zip
RUN unzip opengfx.zip \
    && tar -xf opengfx-$OPENGFX_VERSION.tar -C /usr/local/share/games/openttd/baseset/ \
    && rm -rf opengfx-$OPENGFX_VERSION.tar opengfx.zip

# Add the entrypoint
ADD entrypoint.sh /usr/local/bin/entrypoint
RUN chmod +x /usr/local/bin/entrypoint

# Expose the volume
VOLUME /config

# Expose the gameplay port
EXPOSE 3979/tcp
EXPOSE 3979/udp

# Expose the admin port
EXPOSE 3977/tcp

# Tidy up after ourselves
# note: we don't remove libraries and compilers otherwise bad linking things happen
#RUN apt-get remove -y \
#    make \
#    patch \
#    subversion

RUN rm -r /tmp/build

# Finally, let's be OpenTTD!
USER openttd
WORKDIR /config
CMD /usr/local/bin/entrypoint