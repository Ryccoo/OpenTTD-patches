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


RUN apt-get -y install build-essential pkg-config mingw-w64
RUN i686-w64-mingw32-gcc -v
RUN i686-w64-mingw32-g++ -v

WORKDIR /openttd
RUN wget http://zlib.net/zlib-1.2.11.tar.gz -O zlib.tar.gz
RUN tar xfz zlib.tar.gz
WORKDIR /openttd/zlib-1.2.11
RUN echo `pwd`
RUN sed -e s/"PREFIX ="/"PREFIX = i686-w64-mingw32-"/ -i win32/Makefile.gcc
RUN make -f win32/Makefile.gcc
RUN BINARY_PATH=/usr/local/i686-w64-mingw32/bin \
    INCLUDE_PATH=/usr/local/i686-w64-mingw32/include \
    LIBRARY_PATH=/usr/local/i686-w64-mingw32/lib \
    make -f win32/Makefile.gcc install

WORKDIR /openttd
RUN wget http://downloads.sourceforge.net/project/libpng/libpng16/1.6.34/libpng-1.6.34.tar.gz -O libpng.tar.gz
RUN tar xfz libpng.tar.gz
WORKDIR /openttd/libpng-1.6.34
RUN ./configure \
    --host=i686-w64-mingw32 \
    --prefix=/usr/local/i686-w64-mingw32 \
    CPPFLAGS="-I/usr/local/i686-w64-mingw32/include" \
    LDFLAGS="-L/usr/local/i686-w64-mingw32/lib"

RUN make
RUN make install

WORKDIR /openttd
RUN wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz -O lzo
RUN tar xfz lzo
WORKDIR /openttd/lzo-2.10
RUN ./configure \
    --host=i686-w64-mingw32 \
    --prefix=/usr/local/i686-w64-mingw32 \
    --enable-static \
    CPPFLAGS="-I/usr/local/i686-w64-mingw32/include" \
    LDFLAGS="-L/usr/local/i686-w64-mingw32/lib"
RUN make
RUN make install

WORKDIR /openttd
RUN wget http://downloads.sourceforge.net/project/freetype/freetype2/2.8/freetype-2.8.tar.gz -O freetype
RUN tar xfz freetype
WORKDIR /openttd/freetype-2.8
RUN ./configure \
    --host=i686-w64-mingw32 \
    --prefix=/usr/local/i686-w64-mingw32 \
    --enable-static \
    CPPFLAGS="-I/usr/local/i686-w64-mingw32/include" \
    LDFLAGS="-L/usr/local/i686-w64-mingw32/lib" \
    PKG_CONFIG_LIBDIR=/usr/local/i686-w64-mingw32/lib/pkgconfig
RUN make
RUN make install


WORKDIR /openttd
RUN wget http://tukaani.org/xz/xz-5.2.3.tar.gz -O xz
RUN tar xfz xz
WORKDIR /openttd/xz-5.2.3
RUN ./configure \
    --host=i686-w64-mingw32 \
    --prefix=/usr/local/i686-w64-mingw32 \
    --enable-static --disable-threads \
    CPPFLAGS="-I/usr/local/i686-w64-mingw32/include" \
    LDFLAGS="-L/usr/local/i686-w64-mingw32/lib"
RUN make
RUN make install

WORKDIR /openttd
RUN wget http://download.icu-project.org/files/icu4c/58.1/icu4c-58_1-src.tgz -O icu
RUN tar xfz icu
WORKDIR /openttd
RUN mkdir -p icu/build
WORKDIR /openttd/icu/build
RUN ../source/configure
RUN make -j`nproc`

#ENV LIBS="-lmsvcr100"
WORKDIR /openttd
RUN mkdir -p icu/build-cross
WORKDIR /openttd/icu/build-cross
RUN env CPPFLAGS="-DU_USE_STRTOD_L=0 -I/usr/local/i686-w64-mingw32/include" \
    LDFLAGS="-L/usr/local/i686-w64-mingw32/lib" \
    PKG_CONFIG_LIBDIR=/usr/local/i686-w64-mingw32/lib/pkgconfig \
    ../source/runConfigureICU MinGW \
    --host=i686-w64-mingw32 \
    --prefix=/usr/local/i686-w64-mingw32 \
    --enable-static --disable-shared --disable-strict \
    --with-cross-build=`pwd`/../build
RUN make
RUN make install

WORKDIR /openttd/build
#RUN env PKG_CONFIG_LIBDIR=/usr/local/i686-w64-mingw32/lib/pkgconfig \
#    ./configure \
#    --host=i686-w64-mingw32 \
#    --prefix=/usr/local/i686-w64-mingw32 \
#    --enable-static --static-icu \
#    --with-lzo2=/usr/local/i686-w64-mingw32/lib/liblzo2.a \
#    CFLAGS="-I/usr/local/i686-w64-mingw32/include" \
#    LDFLAGS="-L/usr/local/i686-w64-mingw32/lib"
#RUN make -j5

RUN mkdir -p ///usr/local/share/games/openttd/data

# Build OpenTTD itself
WORKDIR /openttd
RUN wget https://github.com/Ryccoo/OpenTTD-patches/archive/freeplay-0.22.1.tar.gz -O freeplay.tar.gz
RUN tar xfz freeplay.tar.gz
WORKDIR /openttd/OpenTTD-patches-freeplay-0.22.1/
RUN env PKG_CONFIG_LIBDIR=/usr/local/i686-w64-mingw32/lib/pkgconfig \
    ./configure \
    --host=i686-w64-mingw32 \
    --prefix=/usr/local/i686-w64-mingw32 \
    --enable-static --static-icu \
    --with-lzo2=/usr/local/i686-w64-mingw32/lib/liblzo2.a \
    CFLAGS="-I/usr/local/i686-w64-mingw32/include" \
    LDFLAGS="-L/usr/local/i686-w64-mingw32/lib"
RUN make -j5

# Grab OpenGFX as tagged
ADD https://binaries.openttd.org/extra/opengfx/$OPENGFX_VERSION/opengfx-$OPENGFX_VERSION-all.zip opengfx.zip
RUN unzip opengfx.zip \
    && tar -xf opengfx-$OPENGFX_VERSION.tar -C /usr/local/share/games/openttd/baseset/ \
    && rm -rf opengfx-$OPENGFX_VERSION.tar opengfx.zip