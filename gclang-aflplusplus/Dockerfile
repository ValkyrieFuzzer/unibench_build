FROM aflplusplus:latest

RUN mkdir -p /d/p/aflplusplus /d/p/aflplusplus-asan
COPY --from=gclang-bench:latest /d/p/cov/ /d/p/cov
COPY --from=gclang-bench:latest /d/p/bc/ /d/p/bc
COPY --from=gclang-bench:latest /d/p/gclang/ /d/p/gclang
COPY --from=gclang-bench:latest /coverage /coverage

ENV CXX="$CXX -stdlib=libc++"

RUN $CXX /d/p/bc/exiv2.bc -o /d/p/aflplusplus/exiv2 -lz -lpthread -ldl -lexpat
RUN $CXX /d/p/bc/pdftotext.bc -o /d/p/aflplusplus/pdftotext

RUN $CC /d/p/bc/imginfo.bc -o /d/p/aflplusplus/imginfo -lm 
RUN for i in tiffsplit tiffcp tiff2ps; do $CC /d/p/bc/$i.bc -o /d/p/aflplusplus/$i -lz -lm; done
RUN $CC /d/p/bc/objdump.bc -o /d/p/aflplusplus/objdump -ldl && \
    $CC /d/p/bc/nm.bc -o /d/p/aflplusplus/nm -ldl -lz && \
    $CC /d/p/bc/readelf.bc -o /d/p/aflplusplus/readelf && \
    $CC /d/p/bc/size.bc -o /d/p/aflplusplus/size -ldl
RUN for i in cjpeg djpeg; do $CC /d/p/bc/$i.bc -o /d/p/aflplusplus/$i; done
RUN $CC /d/p/bc/xmllint.bc -o /d/p/aflplusplus/xmllint -lm
RUN $CC /d/p/bc/jhead.bc -o /d/p/aflplusplus/jhead -lm
RUN $CC /d/p/bc/tcpdump.bc -o /d/p/aflplusplus/tcpdump 
RUN $CC /d/p/bc/xmlwf.bc -o /d/p/aflplusplus/xmlwf 
RUN $CC /d/p/bc/mutool.bc -o /d/p/aflplusplus/mutool -lm -lpthread
RUN $CC /d/p/bc/readpng.bc -o /d/p/aflplusplus/readpng -lm -lz
RUN $CC /d/p/bc/file.bc -o /d/p/aflplusplus/file -lz

RUN $CC /d/p/bc/lame.bc -o /d/p/aflplusplus/lame -lm
RUN $CC /d/p/bc/mp3gain.bc -o /d/p/aflplusplus/mp3gain -lm -lpthread -lz
RUN $CC /d/p/bc/wav2swf.bc -o /d/p/aflplusplus/wav2swf -lm -lpthread -lz
RUN $CC /d/p/bc/flvmeta.bc -o /d/p/aflplusplus/flvmeta -lm -lpthread -lz
RUN $CXX /d/p/bc/mp42aac.bc -o /d/p/aflplusplus/mp42aac -lm -lpthread -lz
RUN $CC /d/p/bc/cflow.bc -o /d/p/aflplusplus/cflow -lm -lpthread -lz
RUN $CC /d/p/bc/infotocap.bc -o /d/p/aflplusplus/infotocap -lm -lpthread -lz
RUN $CC /d/p/bc/jq.bc -o /d/p/aflplusplus/jq -lm -lpthread -lz
RUN $CC /d/p/bc/mujs.bc -o /d/p/aflplusplus/mujs -lm -lpthread -lz
RUN $CC /d/p/bc/sqlite3.bc -o /d/p/aflplusplus/sqlite3 -lm -lpthread -lz -ldl
## TODO: Compile asan

RUN ls -alh /d/p/*
