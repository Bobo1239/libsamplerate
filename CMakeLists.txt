cmake_minimum_required(VERSION 3.1)
project(libsamplerate LANGUAGES C)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

set(SAMPLERATE_SRC
	${PROJECT_SOURCE_DIR}/src/samplerate.c
	${PROJECT_SOURCE_DIR}/src/src_linear.c
	${PROJECT_SOURCE_DIR}/src/src_sinc.c
	${PROJECT_SOURCE_DIR}/src/src_zoh.c)

if(WIN32)
	set(SAMPLERATE_SRC
		${SAMPLERATE_SRC}
		${PROJECT_SOURCE_DIR}/Win32/libsamplerate-0.def)
endif()

include(TestBigEndian)
test_big_endian(CPU_IS_BIG_ENDIAN)
if(CPU_IS_BIG_ENDIAN)
	set(CPU_IS_LITTLE_ENDIAN 0)
else()
	set(CPU_IS_LITTLE_ENDIAN 1)
endif()

include(CheckFunctionExists)
check_function_exists(alarm HAVE_ALARM)
check_function_exists(lrint HAVE_LRINT)
check_function_exists(lrintf HAVE_LRINTF)
check_function_exists(signal HAVE_SIGNAL)

include(CheckIncludeFiles)
check_include_files(stdint.h HAVE_STDINT)
check_include_files(sys/times.h HAVE_SYS_TIMES)
check_include_files(alsa/asoundlib.h HAVE_ALSA_ASOUNDLIB)

include(CheckSymbolExists)
check_symbol_exists(SIGALRM signal.h HAVE_SIGALRM)

include(CheckTypeSize)
check_type_size("int" SIZEOF_INT)
check_type_size("long" SIZEOF_LONG)

find_package(Sndfile)
set(HAVE_SNDFILE SNDFILE_FOUND)

configure_file(${PROJECT_SOURCE_DIR}/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)

# FIXME: name must be libsamplerate-0 to create a dll
add_library(libsamplerate STATIC ${SAMPLERATE_SRC})

target_include_directories(libsamplerate PUBLIC
	${PROJECT_SOURCE_DIR}/src
	${CMAKE_CURRENT_BINARY_DIR})
