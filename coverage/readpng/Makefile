# The container is responsible for providing me with a libpng16.a, I'm
# not gonna worry about that.
all: readpng.c
	${CC} ${CFLAGS} readpng.c libpng16.a -o readpng ${LDFLAGS} -lz -lm
