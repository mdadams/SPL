# CMAKE_CXX_STANDARD requires version 3.1
cmake_minimum_required(VERSION 3.1)

set(CMAKE_MODULE_PATH
  ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/build/cmake/modules")

# This include should be placed as early as possible.
include(InSourceBuild)

################################################################################

project(SPL LANGUAGES CXX C)

################################################################################

set(SPL_VERSION_MAJOR 1)
set(SPL_VERSION_MINOR 2)
set(SPL_VERSION_PATCH 0)
set(SPL_VERSION
  "${SPL_VERSION_MAJOR}.${SPL_VERSION_MINOR}.${SPL_VERSION_PATCH}")

################################################################################

option(SPL_ROUNDING_MATH "Add -frounding-math option to compiler" true)

################################################################################

include(GNUInstallDirs)
include(CheckIncludeFiles)
include(CheckFunctionExists)
include(CheckCXXSourceCompiles)
include(CTest)
include(EnableAssertions)

################################################################################

if (NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING
	  "Select the type of build (e.g., Debug, Release, RelWithDebInfo, MinSizeRel)."
	  FORCE)
endif()
message("CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE}")

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU" AND SPL_ROUNDING_MATH)
	add_definitions("-frounding-math")
	message("adding -frounding-math to compiler flags")
endif()

set(Boost_USE_MULTITHREADED OFF)
find_package(CGAL REQUIRED)
set(SPL_HAVE_CGAL 1)
find_package(GMP REQUIRED)
find_package(Boost REQUIRED thread)
find_package(OpenGL REQUIRED)
find_package(GLUT_hacked REQUIRED)
find_package(Doxygen)
find_package(LATEX)

find_package(SndFile)
if (SNDFILE_FOUND)
	set(SPL_HAVE_LIBSNDFILE 1)
else()
	set(SPL_HAVE_LIBSNDFILE 0)
	set(SNDFILE_INCLUDE_DIRS "")
	set(SNDFILE_LIBRARIES "")
endif()
find_program(SH_PROGRAM sh)

check_include_files(sys/time.h SPL_HAVE_SYS_TIME_H)
check_function_exists(getrusage SPL_HAVE_GETRUSAGE)
check_function_exists(gettimeofday SPL_HAVE_GETTIMEOFDAY)

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
	set(SPL_HAVE_PROCFS 1)
else()
	set(SPL_HAVE_PROCFS 0)
endif()

check_cxx_source_compiles("
	#include <tr1/cmath>
	int main() {
		double x = std::tr1::cyl_bessel_i(0.0, 1.0);
	}
" SPL_HAVE_TR1_BESSEL)

include_directories(
  ${SPL_INCLUDE_DIR}
  ${CGAL_INCLUDE_DIR}
  ${Boost_INCLUDE_DIRS}
  ${GLUT_INCLUDE_DIR}
  ${OPENGL_INCLUDE_DIR}
  ${GMP_INCLUDE_DIR}
  ${SNDFILE_INCLUDE_DIRS}
)

################################################################################

configure_file(
	${CMAKE_CURRENT_SOURCE_DIR}/bin/test_wrapper.in
	${CMAKE_CURRENT_BINARY_DIR}/bin/test_wrapper
)

add_subdirectory(src/lib)
add_subdirectory(src/test)
add_subdirectory(src/demo)
add_subdirectory(doc)
