FROM coverage-bench:latest

RUN apt-get install wget -y && \
    wget https://github.com/llvm/llvm-project/releases/download/llvmorg-8.0.1/clang+llvm-8.0.1-x86_64-linux-gnu-ubuntu-14.04.tar.xz -O clang+llvm.tar.xz && \
    xz -d clang+llvm.tar.xz && \
    tar xf clang+llvm.tar && \
    rm clang+llvm.tar && \
    mv clang+llvm-8.0.1-x86_64-linux-gnu-ubuntu-14.04 clang+llvm

ENV GOPATH=/go \
    LLVM_HOME=/clang+llvm \
    PATH=/go/bin:/clang+llvm/bin:$PATH \ 
    GIT_SSL_NO_VERIFY=1

RUN apt-get install golang-go -y && \
    mkdir /go && \
    go get github.com/SRI-CSL/gllvm/cmd/...

ENV CC="gclang -fpie -fpic -g" \
    CXX="gclang++ -fpie -fpic -stdlib=libc++ -g"

RUN wget https://github.com/unifuzz/unibench/archive/refs/heads/master.zip && \
    unzip master.zip && \
    mv unibench-master /unibench

RUN cd /unibench && \
    mkdir mp3gain-1.5.2 && cd mp3gain-1.5.2 && mv ../mp3gain-1.5.2.zip ./ && unzip -q mp3gain-1.5.2.zip && rm mp3gain-1.5.2.zip && cd .. &&\
    ls *.zip|xargs -i unzip -q '{}' &&\
    ls *.tar.gz|xargs -i tar xf '{}' &&\
    rm -rf .git/ *.tar.gz *.zip &&\
    mv SQLite-8a8ffc86 SQLite-3.8.9 && mv binutils_5279478 binutils-5279478 && mv libtiff-Release-v3-9-7 libtiff-3.9.7 &&\
    ls -alh

RUN mkdir -p /d/p/gclang /d/p/bc

RUN cd /unibench/exiv2-0.26 && cmake -DEXIV2_ENABLE_SHARED=OFF . &&\
    make -j &&\
    cp bin/exiv2 /d/p/gclang/ &&\
    make clean
RUN get-bc -o /d/p/bc/exiv2.bc /d/p/gclang/exiv2
# San check that this bc works.
RUN $CXX /d/p/bc/exiv2.bc -o /tmp/null -lz -lpthread -ldl -lexpat

RUN cd /unibench/jasper-2.0.12 && cmake -DJAS_ENABLE_SHARED=OFF -DALLOW_IN_SOURCE_BUILD=ON . &&\
    make -j &&\
    cp src/appl/imginfo /d/p/gclang/ &&\
    make clean
RUN get-bc -o /d/p/bc/imginfo.bc /d/p/gclang/imginfo
# San check that this bc works.
RUN $CC /d/p/bc/imginfo.bc -o /tmp/null -lm

RUN cd /unibench/xpdf-4.00 && cmake . &&\
    make -j &&\
    cp xpdf/pdftotext /d/p/gclang/ &&\
    make clean
RUN get-bc -o /d/p/bc/pdftotext.bc /d/p/gclang/pdftotext
# San check that this bc works.
RUN $CXX /d/p/bc/pdftotext.bc -o /tmp/null

#### libtiff 4.2.0
# Compile bintuils using gclang
RUN cd /unibench && rm libtiff* -rf && \
    wget http://download.osgeo.org/libtiff/tiff-4.2.0.tar.gz && \
    tar -zxf tiff-4.2.0.tar.gz && rm tiff-4.2.0.tar.gz &&\
    cd tiff-4.2.0 && \
    ./autogen.sh && ./configure --disable-shared && make -j
# Copy binaries and extract bytecode
RUN cd /unibench/tiff-4.2.0/tools && \
    for i in tiffsplit tiffcp tiff2ps; do cp $i /d/p/gclang/; get-bc -o /d/p/bc/$i.bc /d/p/gclang/$i; done
# San check by compile bytecodes back.
RUN for i in tiffsplit tiffcp tiff2ps; do $CC /d/p/bc/$i.bc -o /tmp/null -lz -lm; done

# We are not using old versions of libtiff anymore.
# RUN cd /unibench/libtiff-3.9.7 && ./autogen.sh && ./configure --disable-shared &&\
#     make -j &&\
#     cp tools/tiffsplit /d/p/gclang/ &&\
#     make clean
# RUN get-bc -o /d/p/bc/tiffsplit.bc /d/p/gclang/tiffsplit && \
#     # San check that this bc works.
#     $CC /d/p/bc/tiffsplit.bc -o /tmp/null -lz -lm

#### binutils 2.35
# Compile bintuils using gclang
RUN cd /unibench && rm binutils* -rf && \
    wget https://ftp.gnu.org/gnu/binutils/binutils-2.35.tar.gz && \
    tar -zxf binutils-2.35.tar.gz && rm binutils-2.35.tar.gz && \
    cd binutils-2.35 && ./configure --disable-shared --disable-libstdcxx --disable-gdb && \
    make -j
# Copy binaries and extract bytecode
RUN cd /unibench/binutils-2.35/binutils && \
    cp nm-new /d/p/gclang/nm && \
    for i in objdump size readelf; do cp $i /d/p/gclang/$i; done && \
    cd /d/p/gclang && \
    for i in objdump size readelf nm; do get-bc -o /d/p/bc/$i.bc $i; done
# San check by compile bytecodes back.
RUN $CC /d/p/bc/objdump.bc -o /tmp/null -ldl && \
    $CC /d/p/bc/nm.bc -o /tmp/null -ldl -lz && \
    $CC /d/p/bc/readelf.bc -o /tmp/null && \
    $CC /d/p/bc/size.bc -o /tmp/null -ldl

# We are not using old versions of binutils anymore.
# RUN cd /unibench/binutils-2.28 && ./configure --disable-shared &&\
#     make -j &&\
#     cp binutils/objdump /d/p/gclang/ &&\
#     make clean
# RUN get-bc -o /d/p/bc/objdump.bc /d/p/gclang/objdump && \
#     $CC /d/p/bc/objdump.bc -o /tmp/null -ldl
#
# RUN cd /unibench/binutils-5279478 &&\
#     ./configure --disable-shared &&\
#     for i in bfd libiberty opcodes libctf; do cd $i; ./configure --disable-shared && make -j; cd ..; done  &&\
#     cd binutils  &&\
#     ./configure --disable-shared &&\
#     make nm-new &&\
#     cp nm-new /d/p/gclang/nm &&\
#     cd .. && make distclean
# RUN get-bc -o /d/p/bc/nm.bc /d/p/gclang/nm && \
#     $CC /d/p/bc/nm.bc -o /tmp/null -ldl -lz

#### libjpeg-ijg v9d
# Compile bintuils using gclang
RUN cd /unibench && \
    wget https://www.ijg.org/files/jpegsrc.v9d.tar.gz && \
    tar -zxf jpegsrc.v9d.tar.gz && rm jpegsrc.v9d.tar.gz && \
    cd jpeg-9d/ && \
    ./configure --disable-shared && make -j
# Copy binaries and extract bytecode
RUN cd /d/p/gclang && \
    for i in cjpeg djpeg; do cp /unibench/jpeg-9d/$i .; get-bc -o /d/p/bc/$i.bc $i; done
# San check by compile bytecodes back.
RUN for i in cjpeg djpeg; do $CC /d/p/bc/$i.bc -o /tmp/null; done

#### libxml2 2.9.10
# Compile bintuils using gclang
RUN cd /unibench && \
    git clone --depth 1 -b v2.9.10  https://gitlab.gnome.org/GNOME/libxml2.git libxml2.9.10 && \
    cd libxml2.9.10 && \
    ./autogen.sh && make distclean && \
    ./configure --disable-shared --with-threads=off --with-http=no --with-ftp=no --with-python=no --without-zlib --without-lzma && \
    make -j
# Copy binaries and extract bytecode
RUN cp /unibench/libxml2.9.10/xmllint /d/p/gclang/xmllint && \
    get-bc -o /d/p/bc/xmllint.bc /d/p/gclang/xmllint
# San check by compile bytecodes back.
RUN $CC /d/p/bc/xmllint.bc -o /tmp/null -lm

#### jhead 3.0.4
# Compile bintuils using gclang
RUN cd /unibench && rm jhead* -rf && \
    wget --no-check-certificate https://www.sentex.ca/~mwandel/jhead/jhead-3.04.tar.gz && \
    tar -zxf jhead-3.04.tar.gz && rm jhead-3.04.tar.gz && \
    cd jhead-3.04 && make -j
# Copy binaries and extract bytecode
RUN cp /unibench/jhead-3.04/jhead /d/p/gclang/jhead && \
    get-bc -o /d/p/bc/jhead.bc /d/p/gclang/jhead
# San check by compile bytecodes back.
RUN $CC /d/p/bc/jhead.bc -o /tmp/null -lm

# We are not using old versions of jhead anymore.
# RUN cd /unibench/jhead-3.00 &&\
#     make -j &&\
#     cp jhead /d/p/gclang/ &&\
#     make clean
# RUN get-bc -o /d/p/bc/jhead.bc /d/p/gclang/jhead && \
#     # San check that this bc works.
#     $CC /d/p/bc/jhead.bc -o /tmp/null -lm

#### libpcap 1.9.1 / tcpdump 4.9.3
# Compile bintuils using gclang
RUN cd /unibench && rm libpcap* tcpdump* -rf && \
    wget https://www.tcpdump.org/release/libpcap-1.9.1.tar.gz && \
    tar -zxf libpcap-1.9.1.tar.gz && rm libpcap-1.9.1.tar.gz && \
    cd libpcap-1.9.1 && \
    ./configure --disable-shared && make -j && \
    cd .. && \
    wget https://www.tcpdump.org/release/tcpdump-4.9.3.tar.gz && \
    tar -zxf tcpdump-4.9.3.tar.gz && rm tcpdump-4.9.3.tar.gz && \
    cd tcpdump-4.9.3 && \
    ./configure --disable-shared --with-cap-ng=no && make -j
RUN cp /unibench/tcpdump-4.9.3/tcpdump /d/p/gclang/tcpdump && \
    get-bc -o /d/p/bc/tcpdump.bc /d/p/gclang/tcpdump
RUN $CC /d/p/bc/tcpdump.bc -o /tmp/null

# We are not using old versions of libpcap/tcpdump anymore.
# RUN cd /unibench/libpcap-1.8.1 && ./configure --disable-shared &&\
#     make -j &&\
#     cd /unibench/tcpdump-4.8.1 && ./configure &&\
#     make -j &&\
#     cp tcpdump /d/p/gclang/ &&\
#     make clean && cd /unibench/libpcap-1.8.1 && make clean
# RUN get-bc -o /d/p/bc/tcpdump.bc /d/p/gclang/tcpdump && \
#     $CC /d/p/bc/tcpdump.bc -o /tmp/null -ldl


#### libexpat R_2_2_10
RUN cd /unibench && \
    wget https://github.com/libexpat/libexpat/archive/R_2_2_10.tar.gz && \
    tar -zxf R_2_2_10.tar.gz && rm R_2_2_10.tar.gz && \
    cd libexpat-R_2_2_10/expat && \
    ./buildconf.sh && \
    ./configure --disable-shared && make -j
RUN cp /unibench/libexpat-R_2_2_10/expat/xmlwf/xmlwf /d/p/gclang/ && \
    get-bc -o /d/p/bc/xmlwf.bc /d/p/gclang/xmlwf
RUN $CC /d/p/bc/xmlwf.bc -o /tmp/null

#### mupdf 1.18
RUN cd /unibench && \
    wget https://mupdf.com/downloads/archive/mupdf-1.18.0-source.tar.gz && \
    tar -zxf mupdf-1.18.0-source.tar.gz && rm mupdf-1.18.0-source.tar.gz && \
    cd mupdf-1.18.0-source/ && \
    make USE_SYSTEM_LIBS=no HAVE_X11=no HAVE_GLUT=no HAVE_OBJCOPY=no -j
RUN cp /unibench/mupdf-1.18.0-source/build/release/mutool /d/p/gclang/ && \
    get-bc -o /d/p/bc/mutool.bc /d/p/gclang/mutool
RUN $CC /d/p/bc/mutool.bc -o /tmp/null -lm -lpthread

#### libpng 1.6.37
COPY readpng /unibench/readpng
RUN cd /unibench && \
    wget https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz && \
    tar -zxf libpng-1.6.37.tar.gz && rm libpng-1.6.37.tar.gz && \
    cd libpng-1.6.37/ && \
    ./configure --disable-shared && make -j && make install && \
    cd ../readpng && cp ../libpng-1.6.37/.libs/libpng16.a . && make
RUN cp /unibench/readpng/readpng /d/p/gclang/ && \
    get-bc -o /d/p/bc/readpng.bc /d/p/gclang/readpng
RUN $CC /d/p/bc/readpng.bc -o /tmp/null -lm -lz

#### FILE5_39
RUN cd /unibench && \
    wget https://github.com/file/file/archive/FILE5_39.tar.gz && \
    tar -zxf FILE5_39.tar.gz && rm FILE5_39.tar.gz && \
    cd file-FILE5_39/ && \
    autoreconf -i && ./configure --disable-shared --disable-libseccomp && \
    make -j && make install && \
    # Magic file: /usr/local/share/misc/magic.mgc
    cp /usr/local/share/misc/magic.mgc /d/p/bc/
RUN cp /unibench/file-FILE5_39/src/file /d/p/gclang/ && \
    get-bc -o /d/p/bc/file.bc /d/p/gclang/file
RUN $CC /d/p/bc/file.bc -o /tmp/null -lz

RUN cd /unibench/lame-3.99.5 && ./configure --disable-shared &&\
    make -j &&\
    cp frontend/lame /d/p/gclang/  && \
    get-bc -o /d/p/bc/lame.bc /d/p/gclang/lame
RUN $CC /d/p/bc/lame.bc -o /tmp/null -lm

RUN cd /unibench/mp3gain-1.5.2 && sed -i 's/CC=/CC?=/' Makefile &&\
    make clean && make -j &&\
    cp mp3gain /d/p/gclang/  && \
    get-bc -o /d/p/bc/mp3gain.bc /d/p/gclang/mp3gain
RUN $CC /d/p/bc/mp3gain.bc -o /tmp/null -lm -lpthread -lz

RUN cd /unibench/swftools-0.9.2/ && ./configure &&\
    make -j &&\
    cp src/wav2swf /d/p/gclang/  && \
    get-bc -o /d/p/bc/wav2swf.bc /d/p/gclang/wav2swf
RUN $CC /d/p/bc/wav2swf.bc -o /tmp/null -lm -lpthread -lz

RUN cd /unibench/flvmeta-1.2.1 && cmake . &&\
    make -j &&\
    cp src/flvmeta /d/p/gclang/ && \
    get-bc -o /d/p/bc/flvmeta.bc /d/p/gclang/flvmeta
RUN $CC /d/p/bc/flvmeta.bc -o /tmp/null -lm -lpthread -lz

RUN cd /unibench/Bento4-1.5.1-628 && cmake . &&\
    make -j &&\
    cp mp42aac /d/p/gclang/ && \
    get-bc -o /d/p/bc/mp42aac.bc /d/p/gclang/mp42aac
RUN $CXX /d/p/bc/mp42aac.bc -o /tmp/null -lm -lpthread -lz

RUN cd /unibench/cflow-1.6 && ./configure &&\
    make -j &&\
    cp src/cflow /d/p/gclang/ && \
    get-bc -o /d/p/bc/cflow.bc /d/p/gclang/cflow
RUN $CC /d/p/bc/cflow.bc -o /tmp/null -lm -lpthread -lz

RUN cd /unibench/ncurses-6.1 && ./configure --disable-shared &&\
    make -j &&\
    cp progs/tic /d/p/gclang/infotocap && \
    get-bc -o /d/p/bc/infotocap.bc /d/p/gclang/infotocap
RUN $CC /d/p/bc/infotocap.bc -o /tmp/null -lm -lpthread -lz

RUN cd /unibench/jq-1.5 && ./configure --disable-shared &&\
    make -j &&\
    cp jq /d/p/gclang/ && \
    get-bc -o /d/p/bc/jq.bc /d/p/gclang/jq
RUN $CC /d/p/bc/jq.bc -o /tmp/null -lm -lpthread -lz

RUN cd /unibench/mujs-1.0.2 &&\
    build=debug make -j &&\
    cp build/debug/mujs /d/p/gclang/ && \
    get-bc -o /d/p/bc/mujs.bc /d/p/gclang/mujs
RUN $CC /d/p/bc/mujs.bc -o /tmp/null -lm -lpthread -lz

#--disable-amalgamation can be used for coverage build
RUN apt install -y tcl-dev &&\
    cd /unibench/SQLite-3.8.9 && ./configure --disable-shared &&\ 
    make -j &&\
    cp sqlite3 /d/p/gclang/  && \
    get-bc -o /d/p/bc/sqlite3.bc /d/p/gclang/sqlite3
RUN $CC /d/p/bc/sqlite3.bc -o /tmp/null -lm -lpthread -lz -ldl