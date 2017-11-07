FROM debian:latest


ARG OPENTTD_VERSION="1.7.0"
ARG OPENGFX_VERSION="0.5.2"

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
RUN svn checkout svn://svn.openttd.org/tags/${OPENTTD_VERSION} /openttd/build
WORKDIR /openttd/build

RUN wget https://www.tt-forums.net/download/file.php?id=197853
RUN patch -p 0 -i file.php?id=197853

RUN /openttd/build/configure \
    --enable-dedicated \
    --binary-dir=bin \
    --personal-dir=/ 
RUN make -j5
RUN make install

# Grab OpenGFX as tagged
RUN wget https://binaries.openttd.org/extra/opengfx/$OPENGFX_VERSION/opengfx-$OPENGFX_VERSION-all.zip -O opengfx.zip
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

RUN rm -r /tmp/build

RUN mkdir -p /config

# Finally, let's be OpenTTD!
#USER openttd
#WORKDIR /config
#CMD /usr/local/bin/entrypoint 

RUN apt-get install -y clang
RUN apt-get install -y build-essential pkg-config libsdl1.2-dev subversion patch zlib1g-dev liblzo2-dev liblzma-dev libfontconfig-dev libicu-dev
RUN apt-get install -y git autoconf automake autopoint bash bison bzip2 flex gettext git g++ gperf intltool libffi-dev libgdk-pixbuf2.0-dev libtool libltdl-dev libssl-dev libxml-parser-perl make openssl p7zip-full patch perl pkg-config python ruby scons sed unzip wget xz-utils g++-multilib libc6-dev-i386 libtool-bin

