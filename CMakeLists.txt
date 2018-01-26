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

check_type_size("long long" SIZEOF_LONG_LONG)
check_type_size("long"      SIZEOF_LONG)
check_type_size("int"       SIZEOF_INT)
check_type_size("short"     SIZEOF_SHORT)

if(SIZEOF_LONG EQUAL "8")
	set(SIGNED_64_BIT_C_TYPE "long")
	set(SIGNED_64_BIT_PRINTF_MODIFIER "l")
elseif(SIZEOF_LONG_LONG EQUAL "8")
	set(SIGNED_64_BIT_C_TYPE "long long")
	set(SIGNED_64_BIT_PRINTF_MODIFIER "ll")
elseif(SIZEOF_SHORT EQUAL "8")
	set(SIGNED_64_BIT_C_TYPE "short")
	set(SIGNED_64_BIT_PRINTF_MODIFIER "s")
elseif(SIZEOF_INT EQUAL "8")
	set(SIGNED_64_BIT_C_TYPE "int")
	set(SIGNED_64_BIT_PRINTF_MODIFIER "")
else()
	message(FATAL_ERROR "No standard C integer type has size 8")
endif()

check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(sys/bittypes.h HAVE_SYS_BITTYPES_H)
check_function_exists(getopt STDLIB_HAS_GETOPT)

configure_file(config.h.in config.h @ONLY)

add_executable(dbgen
	bcd2.c
	bm_utils.c
	build.c
	driver.c
	load_stub.c
	permute.c
	print.c
	rnd.c
	rng64.c
	speed_seed.c
	text.c
)

add_executable(qgen
	bm_utils.c
	build.c
	permute.c
	qgen.c
	rnd.c
	rng64.c
	speed_seed.c
	text.c
	varsub.c
)

set_property(
	TARGET dbgen qgen
	APPEND PROPERTY COMPILE_DEFINITIONS ${DATABASE} ${WORKLOAD}
)

set_property(
	TARGET dbgen qgen
	APPEND PROPERTY COMPILE_OPTIONS -Wall
)

