FROM unifuzz_base

RUN git clone --depth=1 https://github.com/mrash/afl-cov.git /afl-cov
RUN apt-get install lcov -y

# RUN git clone --depth=1 https://github.com/UNIFUZZ/unibench /coverage

RUN wget https://github.com/unifuzz/unibench/archive/refs/heads/master.zip && \
    unzip master.zip && \
    mv unibench-master /coverage

RUN cd /coverage && \
    mkdir mp3gain-1.5.2 && cd mp3gain-1.5.2 && mv ../mp3gain-1.5.2.zip ./ && unzip -q mp3gain-1.5.2.zip && rm mp3gain-1.5.2.zip && cd .. &&\
    ls *.zip|xargs -i unzip -q '{}' &&\
    ls *.tar.gz|xargs -i tar xf '{}' &&\
    rm -rf .git/ *.tar.gz *.zip &&\
    mv SQLite-8a8ffc86 SQLite-3.8.9 && mv binutils_5279478 binutils-5279478 && mv libtiff-Release-v3-9-7 libtiff-3.9.7 &&\
    ls -alh

ENV CC="gcc -fprofile-arcs -ftest-coverage" CXX="g++ -fprofile-arcs -ftest-coverage"

RUN mkdir -p /d/p/cov

RUN cd /coverage/exiv2-0.26 && cmake -DEXIV2_ENABLE_SHARED=OFF . &&\
    make -j &&\
    cp bin/exiv2 /d/p/cov/ &&\
    make clean

RUN cd /coverage/jasper-2.0.12 && cmake -DJAS_ENABLE_SHARED=OFF -DALLOW_IN_SOURCE_BUILD=ON . &&\
    make -j &&\
    cp src/appl/imginfo /d/p/cov/ &&\
    make clean

RUN cd /coverage/xpdf-4.00 && cmake . &&\
    make -j &&\
    cp xpdf/pdftotext /d/p/cov/ &&\
    make clean

#### libtiff 4.2.0
# Compile bintuils using cov
RUN cd /coverage && rm libtiff* -rf && \
    wget http://download.osgeo.org/libtiff/tiff-4.2.0.tar.gz && \
    tar -zxf tiff-4.2.0.tar.gz && rm tiff-4.2.0.tar.gz &&\
    cd tiff-4.2.0 && \
    ./autogen.sh && ./configure --disable-shared && make -j
RUN cd /coverage/tiff-4.2.0/tools && \
    for i in tiffsplit tiffcp tiff2ps; do cp $i /d/p/cov/; done

#### binutils 2.35
# Compile bintuils using cov
RUN cd /coverage && rm binutils* -rf && \
    wget https://ftp.gnu.org/gnu/binutils/binutils-2.35.tar.gz && \
    tar -zxf binutils-2.35.tar.gz && rm binutils-2.35.tar.gz && \
    cd binutils-2.35 && ./configure --disable-shared --disable-libstdcxx --disable-gdb && \
    make -j
RUN cd /coverage/binutils-2.35/binutils && \
    cp nm-new /d/p/cov/nm && \
    for i in objdump size readelf; do cp $i /d/p/cov/$i; done

#### libjpeg-ijg v9d
# Compile bintuils using cov
RUN cd /coverage && \
    wget https://www.ijg.org/files/jpegsrc.v9d.tar.gz && \
    tar -zxf jpegsrc.v9d.tar.gz && rm jpegsrc.v9d.tar.gz && \
    cd jpeg-9d/ && \
    ./configure --disable-shared && make -j
RUN cd /d/p/cov && \
    for i in cjpeg djpeg; do cp /coverage/jpeg-9d/$i .; done

#### libxml2 2.9.10
# Compile bintuils using cov
RUN cd /coverage && \
    git clone --depth 1 -b v2.9.10  https://gitlab.gnome.org/GNOME/libxml2.git libxml2.9.10 && \
    cd libxml2.9.10 && \
    ./autogen.sh && make distclean && \
    ./configure --disable-shared --with-threads=off --with-http=no --with-ftp=no --with-python=no --without-zlib --without-lzma && \
    make -j
RUN cp /coverage/libxml2.9.10/xmllint /d/p/cov/xmllint

#### jhead 3.0.4
# Compile bintuils using cov
RUN cd /coverage && rm jhead* -rf && \
    wget --no-check-certificate https://www.sentex.ca/~mwandel/jhead/jhead-3.04.tar.gz && \
    tar -zxf jhead-3.04.tar.gz && rm jhead-3.04.tar.gz && \
    cd jhead-3.04 && make -j
RUN cp /coverage/jhead-3.04/jhead /d/p/cov/jhead

#### libpcap 1.9.1 / tcpdump 4.9.3
# Compile bintuils using cov
RUN cd /coverage && rm libpcap* tcpdump* -rf && \
    wget https://www.tcpdump.org/release/libpcap-1.9.1.tar.gz && \
    tar -zxf libpcap-1.9.1.tar.gz && rm libpcap-1.9.1.tar.gz && \
    cd libpcap-1.9.1 && \
    ./configure --disable-shared && make -j && \
    cd .. && \
    wget https://www.tcpdump.org/release/tcpdump-4.9.3.tar.gz && \
    tar -zxf tcpdump-4.9.3.tar.gz && rm tcpdump-4.9.3.tar.gz && \
    cd tcpdump-4.9.3 && \
    ./configure --disable-shared --with-cap-ng=no && make -j
RUN cp /coverage/tcpdump-4.9.3/tcpdump /d/p/cov/tcpdump

#### libexpat R_2_2_10
RUN cd /coverage && \
    wget https://github.com/libexpat/libexpat/archive/R_2_2_10.tar.gz && \
    tar -zxf R_2_2_10.tar.gz && rm R_2_2_10.tar.gz && \
    cd libexpat-R_2_2_10/expat && \
    ./buildconf.sh && \
    ./configure --disable-shared && make -j
RUN cp /coverage/libexpat-R_2_2_10/expat/xmlwf/xmlwf /d/p/cov/

#### mupdf 1.18
RUN cd /coverage && \
    wget https://mupdf.com/downloads/archive/mupdf-1.18.0-source.tar.gz && \
    tar -zxf mupdf-1.18.0-source.tar.gz && rm mupdf-1.18.0-source.tar.gz && \
    cd mupdf-1.18.0-source/ && \
    make USE_SYSTEM_LIBS=no HAVE_X11=no HAVE_GLUT=no HAVE_OBJCOPY=no -j
RUN cp /coverage/mupdf-1.18.0-source/build/release/mutool /d/p/cov/

#### libpng 1.6.37
COPY readpng /coverage/readpng
RUN cd /coverage && \
    wget https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz && \
    tar -zxf libpng-1.6.37.tar.gz && rm libpng-1.6.37.tar.gz && \
    cd libpng-1.6.37/ && \
    ./configure --disable-shared && make -j && make install && \
    cd ../readpng && cp ../libpng-1.6.37/.libs/libpng16.a . && make
RUN cp /coverage/readpng/readpng /d/p/cov/

#### FILE5_39
RUN cd /coverage && \
    wget https://github.com/file/file/archive/FILE5_39.tar.gz && \
    tar -zxf FILE5_39.tar.gz && rm FILE5_39.tar.gz && \
    cd file-FILE5_39/ && \
    autoreconf -i && ./configure --disable-shared --disable-libseccomp && \
    make -j && make install && \
    # Magic file: /usr/local/share/misc/magic.mgc
    cp /usr/local/share/misc/magic.mgc /d/p/cov/
RUN cp /coverage/file-FILE5_39/src/file /d/p/cov/

RUN cd /coverage/lame-3.99.5 && ./configure --disable-shared &&\
    make -j &&\
    cp frontend/lame /d/p/cov/ 

RUN cd /coverage/mp3gain-1.5.2 && sed -i 's/CC=/CC?=/' Makefile &&\
    make clean && make -j &&\
    cp mp3gain /d/p/cov/ 
  
RUN cd /coverage/swftools-0.9.2/ && ./configure &&\
    make -j &&\
    cp src/wav2swf /d/p/cov/ 

RUN cd /coverage/flvmeta-1.2.1 && cmake . &&\
    make -j &&\
    cp src/flvmeta /d/p/cov/

RUN cd /coverage/Bento4-1.5.1-628 && cmake . &&\
    make -j &&\
    cp mp42aac /d/p/cov/

RUN cd /coverage/cflow-1.6 && ./configure &&\
    make -j &&\
    cp src/cflow /d/p/cov/

RUN cd /coverage/ncurses-6.1 && ./configure --disable-shared &&\
    make -j &&\
    cp progs/tic /d/p/cov/infotocap

RUN cd /coverage/jq-1.5 && ./configure --disable-shared &&\
    make -j &&\
    cp jq /d/p/cov/

RUN cd /coverage/mujs-1.0.2 &&\
    build=debug make -j &&\
    cp build/debug/mujs /d/p/cov/

#--disable-amalgamation can be used for coverage build
RUN apt install -y tcl-dev &&\
    cd /coverage/SQLite-3.8.9 && ./configure --disable-shared &&\ 
    make -j &&\
    cp sqlite3 /d/p/cov/ 
  
RUN ls -alh /d/p/*
