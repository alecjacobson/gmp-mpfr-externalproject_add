
project(example)
cmake_minimum_required(VERSION 3.20)

set(gmp_INSTALL ${PROJECT_BINARY_DIR}/gmp-install)
set(gmp_LIB_DIR ${gmp_INSTALL}/lib)
set(gmp_LIBRARY ${gmp_LIB_DIR}/${CMAKE_STATIC_LIBRARY_PREFIX}gmp${CMAKE_STATIC_LIBRARY_SUFFIX})
set(gmp_INCLUDE_DIR ${gmp_INSTALL}/include)
include(ProcessorCount)
ProcessorCount(Ncpu)
include(ExternalProject)
ExternalProject_Add(gmp
URL  https://gmplib.org/download/gmp/gmp-6.2.1.tar.xz
URL_MD5 0b82665c4a92fd2ade7440c13fcaa42b
UPDATE_DISCONNECTED true  # need this to avoid constant rebuild
PATCH_COMMAND 
  curl "https://gmplib.org/repo/gmp/raw-rev/5f32dbc41afc" "|" git apply -v
CONFIGURE_HANDLED_BY_BUILD ON  # avoid constant reconfigure
CONFIGURE_COMMAND 
  ${PROJECT_BINARY_DIR}/gmp-prefix/src/gmp/configure 
  --disable-debug --disable-dependency-tracking --enable-cxx --with-pic
  --prefix=${gmp_INSTALL}
  --disable-shared
BUILD_COMMAND make -j${Ncpu}
INSTALL_COMMAND make -j${Ncpu} install
TEST_COMMAND ""
BUILD_BYPRODUCTS ${gmp_LIBRARY}
)
ExternalProject_Get_Property(gmp SOURCE_DIR)
set(gmp_LIBRARIES ${gmp_LIBRARY})
add_library(gmp::gmp INTERFACE IMPORTED GLOBAL)
file(MAKE_DIRECTORY ${gmp_INCLUDE_DIR})  # avoid race condition
target_include_directories(gmp::gmp INTERFACE ${gmp_INCLUDE_DIR})
target_link_libraries(gmp::gmp INTERFACE "${gmp_LIBRARIES}")  # need the quotes to expand list
add_dependencies(gmp::gmp gmp)


set(mpfr_INSTALL ${PROJECT_BINARY_DIR}/mpfr-install)
set(mpfr_LIBRARY ${mpfr_INSTALL}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}mpfr${CMAKE_STATIC_LIBRARY_SUFFIX})
set(mpfr_INCLUDE_DIR ${mpfr_INSTALL}/include)
include(ProcessorCount)
ProcessorCount(Ncpu)
include(ExternalProject)
message(STATUS "gmp_LIB_DIR: ${gmp_LIB_DIR}")
ExternalProject_Add(mpfr
DEPENDS gmp
URL  https://ftp.gnu.org/gnu/mpfr/mpfr-4.1.0.tar.xz
URL_MD5 bdd3d5efba9c17da8d83a35ec552baef
UPDATE_DISCONNECTED true  # need this to avoid constant rebuild
CONFIGURE_HANDLED_BY_BUILD ON  # avoid constant reconfigure
CONFIGURE_COMMAND 
  ${PROJECT_BINARY_DIR}/mpfr-prefix/src/mpfr/configure 
  --disable-debug --disable-dependency-tracking  --disable-silent-rules --enable-cxx --with-pic
  --with-gmp-include=${gmp_INCLUDE_DIR} --with-gmp-lib=${gmp_LIB_DIR}
  --disable-shared
  --prefix=${mpfr_INSTALL}
  --disable-shared
BUILD_COMMAND make -j${Ncpu}
INSTALL_COMMAND make -j${Ncpu} install
TEST_COMMAND ""
BUILD_BYPRODUCTS ${mpfr_LIBRARY}
)
#PATCH_COMMAND  curl "https://raw.githubusercontent.com/Homebrew/formula-patches/03cf8088210822aa2c1ab544ed58ea04c897d9c4/libtool/configure-big_sur.diff" "|" sed -e "s/configure.orig/configure/g" "|" git apply -v
ExternalProject_Get_Property(mpfr SOURCE_DIR)
set(mpfr_LIBRARIES ${mpfr_LIBRARY})
add_library(mpfr::mpfr INTERFACE IMPORTED GLOBAL)
file(MAKE_DIRECTORY ${mpfr_INCLUDE_DIR})  # avoid race condition
target_include_directories(mpfr::mpfr INTERFACE ${mpfr_INCLUDE_DIR})
target_link_libraries(mpfr::mpfr INTERFACE "${mpfr_LIBRARIES}")  # need the quotes to expand list
add_dependencies(mpfr::mpfr mpfr)


add_executable(example main.cpp)
target_link_libraries(example gmp::gmp mpfr::mpfr) 
