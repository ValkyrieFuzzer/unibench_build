FROM valkyrie:latest

# RUN mkdir -p /d/p/angora/taint /d/p/angora/fast & \
#     apt update && apt install -y nasm libglib2.0-dev gtk-doc-tools libtiff-dev libpng-dev tcl-dev
RUN mkdir -p /d/p/angora/taint /d/p/angora/fast /d/p/asan

COPY --from=gclang-bench:latest /d/p/cov/ /d/p/cov
COPY --from=gclang-bench:latest /d/p/bc/ /d/p/bc
COPY --from=gclang-bench:latest /d/p/gclang/ /d/p/gclang
COPY --from=gclang-bench:latest /coverage /coverage

RUN ldd /d/p/gclang/*|grep .so|awk '{print $3}'|grep .so|sort|uniq|sed 's#^/lib#/usr/lib#g'|sed 's#\.so.*$#.so#g'|grep -v libgcc_s.so|grep -v libstdc++.so|grep -v libc.so|grep -v libm.so|grep -v libpthread.so|xargs -i /valkyrie/tools/gen_library_abilist.sh '{}' discard >> /tmp/abilist.txt

ENV ANGORA_TAINT_RULE_LIST=/tmp/abilist.txt

## Break critical edges
RUN for i in exiv2 pdftotext imginfo tiffsplit tiffcp tiff2ps objdump nm readelf size cjpeg djpeg xmllint jhead tcpdump xmlwf readpng file; do opt -break-crit-edges /d/p/bc/${i}.bc -o /d/p/bc/${i}_noce.bc; mv /d/p/bc/${i}_noce.bc /d/p/bc/${i}.bc; done
RUN for i in lame mp3gain wav2swf flvmeta mp42aac cflow infotocap jq mujs sqlite3; do opt -break-crit-edges /d/p/bc/${i}.bc -o /d/p/bc/${i}_noce.bc; mv /d/p/bc/${i}_noce.bc /d/p/bc/${i}.bc; done
ENV CC="$CC -lrt" CXX="$CXX -lrt"


## Fast
RUN $CXX /d/p/bc/exiv2.bc -o /d/p/angora/fast/exiv2 -lz -lpthread -ldl -lexpat & \
    $CXX /d/p/bc/pdftotext.bc -o /d/p/angora/fast/pdftotext & \
    $CC /d/p/bc/imginfo.bc -o /d/p/angora/fast/imginfo -lm & \
    $CC /d/p/bc/tiffsplit.bc -o /d/p/angora/fast/tiffsplit -lz -lm & \
    $CC /d/p/bc/tiffcp.bc -o /d/p/angora/fast/tiffcp -lz -lm & \
    $CC /d/p/bc/tiff2ps.bc -o /d/p/angora/fast/tiff2ps -lz -lm & \
    $CC /d/p/bc/objdump.bc -o /d/p/angora/fast/objdump -fPIC -ldl & \
    $CC /d/p/bc/nm.bc -o /d/p/angora/fast/nm -ldl -lz & \
    $CC /d/p/bc/readelf.bc -o /d/p/angora/fast/readelf & \
    $CC /d/p/bc/size.bc -o /d/p/angora/fast/size -ldl & \
    $CC /d/p/bc/cjpeg.bc -o /d/p/angora/fast/cjpeg & \
    $CC /d/p/bc/djpeg.bc -o /d/p/angora/fast/djpeg & \
    $CC /d/p/bc/xmllint.bc -o /d/p/angora/fast/xmllint -lm & \
    $CC /d/p/bc/jhead.bc -o /d/p/angora/fast/jhead -lm & \
    $CC /d/p/bc/tcpdump.bc -o /d/p/angora/fast/tcpdump  & \
    $CC /d/p/bc/xmlwf.bc -o /d/p/angora/fast/xmlwf  & \
    # $CC /d/p/bc/mutool.bc -o /d/p/angora/fast/mutool -lm -lpthread & \
    $CC /d/p/bc/readpng.bc -o /d/p/angora/fast/readpng -lm -lz & \
    $CC /d/p/bc/file.bc -o /d/p/angora/fast/file -lz & \
    \
    $CC /d/p/bc/lame.bc -o /d/p/angora/fast/lame -lm & \
    $CC /d/p/bc/mp3gain.bc -o /d/p/angora/fast/mp3gain -lm -lpthread -lz & \
    $CC /d/p/bc/wav2swf.bc -o /d/p/angora/fast/wav2swf -lm -lpthread -lz & \
    $CC /d/p/bc/flvmeta.bc -o /d/p/angora/fast/flvmeta -lm -lpthread -lz & \
    $CXX /d/p/bc/mp42aac.bc -o /d/p/angora/fast/mp42aac -lm -lpthread -lz & \
    $CC /d/p/bc/cflow.bc -o /d/p/angora/fast/cflow -lm -lpthread -lz & \
    $CC /d/p/bc/infotocap.bc -o /d/p/angora/fast/infotocap -lm -lpthread -lz & \
    $CC /d/p/bc/jq.bc -o /d/p/angora/fast/jq -lm -lpthread -lz & \
    $CC /d/p/bc/mujs.bc -o /d/p/angora/fast/mujs -lm -lpthread -lz & \
    $CC /d/p/bc/sqlite3.bc -o /d/p/angora/fast/sqlite3 -lm -lpthread -lz -ldl; \
    wait

# Asan
RUN clang++ /d/p/bc/pdftotext.bc -o /d/p/asan/pdftotext  -fsanitize=address & \
    clang /d/p/bc/imginfo.bc -o /d/p/asan/imginfo -lm  -fsanitize=address & \
    clang /d/p/bc/tiffsplit.bc -o /d/p/asan/tiffsplit -lz -lm  -fsanitize=address & \
    clang /d/p/bc/tiffcp.bc -o /d/p/asan/tiffcp -lz -lm  -fsanitize=address & \
    clang /d/p/bc/tiff2ps.bc -o /d/p/asan/tiff2ps -lz -lm  -fsanitize=address & \
    clang /d/p/bc/objdump.bc -o /d/p/asan/objdump -fPIC -ldl  -fsanitize=address & \
    clang /d/p/bc/nm.bc -o /d/p/asan/nm -ldl -lz  -fsanitize=address & \
    clang /d/p/bc/readelf.bc -o /d/p/asan/readelf  -fsanitize=address & \
    clang /d/p/bc/size.bc -o /d/p/asan/size -ldl  -fsanitize=address & \
    clang /d/p/bc/cjpeg.bc -o /d/p/asan/cjpeg  -fsanitize=address & \
    clang /d/p/bc/djpeg.bc -o /d/p/asan/djpeg  -fsanitize=address & \
    clang /d/p/bc/xmllint.bc -o /d/p/asan/xmllint -lm  -fsanitize=address & \
    clang /d/p/bc/jhead.bc -o /d/p/asan/jhead -lm  -fsanitize=address & \
    clang /d/p/bc/tcpdump.bc -o /d/p/asan/tcpdump   -fsanitize=address & \
    clang /d/p/bc/xmlwf.bc -o /d/p/asan/xmlwf   -fsanitize=address & \
    clang /d/p/bc/readpng.bc -o /d/p/asan/readpng -lm -lz  -fsanitize=address & \
    clang /d/p/bc/file.bc -o /d/p/asan/file -lz  -fsanitize=address & \
    clang /d/p/bc/lame.bc -o /d/p/asan/lame -lm  -fsanitize=address & \
    clang /d/p/bc/mp3gain.bc -o /d/p/asan/mp3gain -lm -lpthread -lz  -fsanitize=address & \
    clang /d/p/bc/wav2swf.bc -o /d/p/asan/wav2swf -lm -lpthread -lz  -fsanitize=address & \
    clang /d/p/bc/flvmeta.bc -o /d/p/asan/flvmeta -lm -lpthread -lz  -fsanitize=address & \
    clang++ /d/p/bc/mp42aac.bc -o /d/p/asan/mp42aac -lm -lpthread -lz  -fsanitize=address & \
    clang /d/p/bc/cflow.bc -o /d/p/asan/cflow -lm -lpthread -lz  -fsanitize=address & \
    clang /d/p/bc/infotocap.bc -o /d/p/asan/infotocap -lm -lpthread -lz  -fsanitize=address & \
    clang /d/p/bc/jq.bc -o /d/p/asan/jq -lm -lpthread -lz  -fsanitize=address & \
    clang /d/p/bc/mujs.bc -o /d/p/asan/mujs -lm -lpthread -lz  -fsanitize=address & \
    clang /d/p/bc/sqlite3.bc -o /d/p/asan/sqlite3 -lm -lpthread -lz -ldl -fsanitize=address; \
    wait

## Taint
ENV USE_TRACK=1
RUN $CXX /d/p/bc/exiv2.bc -o /d/p/angora/taint/exiv2 -lz -lpthread -ldl -lexpat & \
    $CXX /d/p/bc/pdftotext.bc -o /d/p/angora/taint/pdftotext & \
    $CC /d/p/bc/imginfo.bc -o /d/p/angora/taint/imginfo -lm & \
    $CC /d/p/bc/tiffsplit.bc -o /d/p/angora/taint/tiffsplit -lz -lm & \
    $CC /d/p/bc/tiffcp.bc -o /d/p/angora/taint/tiffcp -lz -lm & \
    $CC /d/p/bc/tiff2ps.bc -o /d/p/angora/taint/tiff2ps -lz -lm & \
    $CC /d/p/bc/objdump.bc -o /d/p/angora/taint/objdump -ldl & \
    $CC /d/p/bc/nm.bc -o /d/p/angora/taint/nm -ldl -lz & \
    $CC /d/p/bc/readelf.bc -o /d/p/angora/taint/readelf & \
    $CC /d/p/bc/size.bc -o /d/p/angora/taint/size -ldl & \
    $CC /d/p/bc/cjpeg.bc -o /d/p/angora/taint/cjpeg & \
    $CC /d/p/bc/djpeg.bc -o /d/p/angora/taint/djpeg & \
    $CC /d/p/bc/xmllint.bc -o /d/p/angora/taint/xmllint -lm & \
    $CC /d/p/bc/jhead.bc -o /d/p/angora/taint/jhead -lm & \
    $CC /d/p/bc/tcpdump.bc -o /d/p/angora/taint/tcpdump  & \
    $CC /d/p/bc/xmlwf.bc -o /d/p/angora/taint/xmlwf  & \
    # $CC /d/p/bc/mutool.bc -o /d/p/angora/taint/mutool -lm -lpthread & \
    $CC /d/p/bc/readpng.bc -o /d/p/angora/taint/readpng -lm -lz & \
    $CC /d/p/bc/file.bc -o /d/p/angora/taint/file -lz & \
    \
    $CC /d/p/bc/lame.bc -o /d/p/angora/taint/lame -lm & \
    $CC /d/p/bc/mp3gain.bc -o /d/p/angora/taint/mp3gain -lm -lpthread -lz & \
    $CC /d/p/bc/wav2swf.bc -o /d/p/angora/taint/wav2swf -lm -lpthread -lz & \
    $CC /d/p/bc/flvmeta.bc -o /d/p/angora/taint/flvmeta -lm -lpthread -lz & \
    $CXX /d/p/bc/mp42aac.bc -o /d/p/angora/taint/mp42aac -lm -lpthread -lz & \
    $CC /d/p/bc/cflow.bc -o /d/p/angora/taint/cflow -lm -lpthread -lz & \
    $CC /d/p/bc/infotocap.bc -o /d/p/angora/taint/infotocap -lm -lpthread -lz & \
    $CC /d/p/bc/jq.bc -o /d/p/angora/taint/jq -lm -lpthread -lz & \
    $CC /d/p/bc/mujs.bc -o /d/p/angora/taint/mujs -lm -lpthread -lz  & \
    $CC /d/p/bc/sqlite3.bc -o /d/p/angora/taint/sqlite3 -lm -lpthread -lz -ldl; \
    wait
RUN unset USE_TRACK