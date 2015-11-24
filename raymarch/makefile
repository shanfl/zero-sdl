TARGET=target-undefined
EXE_SUFFIX=

ifeq ($(platform),windows)
    TARGET=target-windows
    EXE_SUFFIX=.exe
	PLATFORM_LDFLAGS=${BASE_PWD}/bgfx/.build/win64_mingw-gcc/bin/libbgfxDebug.a \
	                 -lgdi32 -lpsapi -lm -luser32 -lwinmm -limm32 -lole32 -loleaut32 -lshell32 -lversion -luuid \
	                 -mwindows -mconsole -m64 -Wl,--gc-sections -static -static-libgcc -static-libstdc++
					 
else ifeq ($(platform), osx)
    TARGET=target-osx
	PLATFORM_LDFLAGS=${BASE_PWD}/bgfx/.build/osx64_clang/bin/libbgfxDebug.a \
	                 -framework Cocoa -framework OpenGL
	                 
else 
    $(error You should define a supported platform: platform=windows or platform=osx)
endif

BASE_PWD=${shell pwd}/..
SDL2_DIR=${BASE_PWD}/sdl2-prefix
SHADERC=${BASE_PWD}/bgfx/.build/win64_mingw-gcc/bin/shadercDebug
TARGET_BINARY_NAME=raymarch
TARGET_BINARY=${TARGET_BINARY_NAME}${EXE_SUFFIX}

CSOURCES=${BASE_PWD}/common-src/start.cpp \
         src/drawing.cpp \
         src/bgfx_utils.cpp \
         src/event_handler.cpp

ifdef debug
	DEBUG_RELEASE=-O0 -g3
else
	DEBUG_RELEASE=-O3
endif

CFLAGS:=$(CFLAGS) -Wall -Wextra -fno-rtti -Wshadow -Wunused-value -fdata-sections -ffunction-sections -msse2 -Wundef
CXXFLAGS:=$(CXXFLAGS) $(CFLAGS) -fno-exceptions -std=c++0x
LDFLAGS:=${LDFLAGS} ${shell ../sdl2-prefix/bin/sdl2-config --libs} ${PLATFORM_LDFLAGS}

all: binary shaders

shaders: shader_dx11

shader_dx11: ${TARGET}/dx11/vs_raymarching.bin ${TARGET}/dx11/fs_raymarching.bin

${TARGET}/dx11/vs_raymarching.bin: src/vs_raymarching.sc
	mkdir -p ${TARGET}/dx11/ && \
	${SHADERC} -f src/vs_raymarching.sc -o ${TARGET}/dx11/vs_raymarching.bin --type vertex --platform windows -p vs_4_0 -O 3 -i ${BASE_PWD}/bgfx/src/

${TARGET}/dx11/fs_raymarching.bin: src/fs_raymarching.sc
	mkdir -p ${TARGET}/dx11/ && \
	${SHADERC} -f src/fs_raymarching.sc -o ${TARGET}/dx11/fs_raymarching.bin --type fragment --platform windows -p ps_4_0 -O 3 -i ${BASE_PWD}/bgfx/src/

binary: ${TARGET}/${TARGET_BINARY}

${TARGET}/${TARGET_BINARY}: ${CSOURCES}
	mkdir -p ${TARGET} && \
	g++ ${CXXFLAGS} ${DEBUG_RELEASE} \
		-D__STDC_LIMIT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DWIN32 -DENTRY_CONFIG_USE_SDL=1 \
		${CSOURCES} \
		${BASE_PWD}/bgfx/3rdparty/ib-compress/indexbuffercompression.cpp \
		${BASE_PWD}/bgfx/3rdparty/ib-compress/indexbufferdecompression.cpp \
		-I${BASE_PWD}/common-src \
		-I${BASE_PWD}/bx/include \
		-I${BASE_PWD}/bx/include/compat/mingw \
		-I${BASE_PWD}/bgfx/include \
		-I${BASE_PWD}/bgfx/3rdparty \
		-I${BASE_PWD}/sdl2-prefix/include \
		-L${BASE_PWD}/sdl2-prefix/lib \
		${LDFLAGS} \
		-o ${TARGET}/${TARGET_BINARY} && \
	strip ${TARGET}/${TARGET_BINARY} --strip-all
	
clean:
	rm -rf ${TARGET}