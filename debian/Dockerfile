FROM debian:bullseye-slim as builder
RUN apt update && apt install -y \
    wget \
    tar \
    xz-utils \
    nasm \
    yasm \
    make \
    cmake \
    autoconf \
    git \
    libtool \
    gcc \
    g++ \
    mingw-w64-tools
ENV CC gcc
ENV CXX g++
ENV LDFLAGS "-L/opt/out/lib"
ENV PKG_CONFIG_PATH "/opt/out/lib/pkgconfig"
WORKDIR /opt
# SDL2
FROM builder as sdl2
RUN wget https://www.libsdl.org/release/SDL2-2.0.14.tar.gz -O- | tar -xz
RUN mv SDL2-2.0.14 SDL2 
WORKDIR /opt/SDL2
RUN ./configure --prefix=/opt/out --enable-static --disable-shared  \
    && make -j 4 && make install

# x264
FROM builder as x264
RUN git clone https://code.videolan.org/videolan/x264.git
WORKDIR /opt/x264
RUN ./configure --prefix=/opt/out   --enable-static \
    && make -j 4 && make install

# x265
FROM builder as x265
RUN git clone https://github.com/videolan/x265.git
WORKDIR /opt/x265/source
RUN cmake . -DCMAKE_INSTALL_PREFIX=/opt/out -DENABLE_SHARED=OFF \
    && make -j 4 && make install

# fdk-aac
FROM builder as fdk-aac
RUN git clone https://github.com/mstorsjo/fdk-aac.git
WORKDIR /opt/fdk-aac
RUN autoreconf -fiv \
    && ./configure --prefix=/opt/out --enable-static --disable-shared  \
    && make -j 4 && make install

# opus
FROM builder as opus
ENV CFLAGS "-D_FORTIFY_SOURCE=2"
RUN wget https://archive.mozilla.org/pub/opus/opus-1.3.1.tar.gz -O- | tar -xz \
    && mv opus-1.3.1 opus
WORKDIR /opt/opus
RUN ./configure --prefix=/opt/out  --enable-static --disable-shared \
    && make -j 4 && make install

# vpx
FROM builder as vpx
RUN git clone https://chromium.googlesource.com/webm/libvpx
WORKDIR /opt/libvpx
RUN ./configure --prefix=/opt/out \
    && make -j 4 && make install

# lame(mp3)
FROM builder as lame
RUN apt update && apt install -y subversion
RUN svn checkout https://svn.code.sf.net/p/lame/svn/trunk/lame
WORKDIR /opt/lame
RUN ./configure --prefix=/opt/out --enable-static --disable-shared \ 
    && make -j 4 && make install


# ffmpeg
FROM builder as ffmpeg
RUN wget https://ffmpeg.org/releases/ffmpeg-4.3.1.tar.xz -O- | tar -xJ \ 
    && mv ffmpeg-4.3.1 ffmpeg
RUN git clone https://github.com/FFmpeg/nv-codec-headers.git \
    && cd /opt/nv-codec-headers \
    && sed -i -e "s/PREFIX = .*/PREFIX = \/opt\/out/" Makefile \
    && make install
WORKDIR /opt/ffmpeg
COPY --from=sdl2 /opt/out /opt/out
COPY --from=x264 /opt/out /opt/out
COPY --from=x265 /opt/out /opt/out
COPY --from=fdk-aac /opt/out /opt/out 
COPY --from=lame /opt/out /opt/out
COPY --from=opus /opt/out /opt/out
COPY --from=vpx /opt/out /opt/out
ENV LDFLAGS "-pthread -Bstatic,-lfdk-aac,-lSDL2 -L/opt/out/lib "
ENV CFLAGS "-Wl,-Bstatic -lfdk-aac -lmp3lame -lSDL2"
ENV CXXFLAGS "-Wl,-Bstatic -lfdk-aac -lmp3lame -lSDL2"
RUN ./configure \
    --prefix=/opt/out/ \
    --arch=amd64 \
    --pkg-config-flags="--static" \
    --enable-static \
    --enable-gpl \
    --enable-nonfree \
    --enable-nvenc \
    --enable-cuda \
    --enable-cuvid \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libvpx \
    --enable-libmp3lame \
    --enable-libfdk-aac \
    --enable-libopus \
    && make -j 4 && make install

FROM debian:bullseye-slim 
WORKDIR /opt
COPY --from=ffmpeg /opt/out /opt/out
