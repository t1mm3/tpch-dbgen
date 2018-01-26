cmake_minimum_required(VERSION 3.0)
include(CheckIncludeFiles)
include(CheckTypeSize)
include(CheckSymbolExists)
include(CheckFunctionExists)

project(tpch-dbgen # TPC-H benchmark data (and query) generator utilities
	VERSION 2.17.3 # taken from the TPC-H benchmark release version
	LANGUAGES C)

set(DATABASE "DB2" CACHE STRING "Command language dialect to target with the qgen query generator (\"DATABASE\"")
set_property(CACHE DATABASE PROPERTY STRINGS INFORMIX DB2 TDAT SQLSERVER SYBASE ORACLE VECTORWISE POSTGRES)

set(WORKLOAD TPCH CACHE STRING "Choice of benchmark / query workload (\"WORKLOAD\")")
set_property(CACHE WORKLOAD PROPERTY STRINGS TPCH)

set(EOL_HANDLING OFF CACHE BOOL "Skip separator after the last field in generated lines?")

check_type_size("long long"        SIZEOF_LONG_LONG)
check_type_size("long"             SIZEOF_LONG)
check_type_size("int"              SIZEOF_INT)
check_type_size("short"            SIZEOF_SHORT)

check_include_file(strings.h       HAVE_STRINGS_H)
check_include_file(inttypes.h      HAVE_INTTYPES_H)
check_include_file(stdint.h        HAVE_STDINT_H)
check_include_file(sys/bittypes.h  HAVE_SYS_BITTYPES_H)

check_function_exists(getopt       HAVE_GETOPT)

configure_file(src/config.h.in src/config.h @ONLY)

add_executable(dbgen
	src/bcd2.c
	src/bm_utils.c
	src/build.c
	src/driver.c
	src/load_stub.c
	src/permute.c
	src/print.c
	src/rnd.c
	src/rng64.c
	src/speed_seed.c
	src/text.c
)

add_executable(qgen
	src/bm_utils.c
	src/build.c
	src/permute.c
	src/qgen.c
	src/rnd.c
	src/rng64.c
	src/speed_seed.c
	src/text.c
	src/varsub.c
)

set_property(
	TARGET dbgen qgen
	APPEND PROPERTY COMPILE_DEFINITIONS ${DATABASE} ${WORKLOAD}
)

set_property(
	TARGET dbgen qgen
	APPEND PROPERTY COMPILE_OPTIONS -Wall
)

