# Definitions for all external bundled libraries

# Suppress warnings from external libraries
if (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    add_compile_options(/W0)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/CMakeModules)
include(DownloadExternals)
include(ExternalProject)

# Boost
if (NOT USE_SYSTEM_BOOST)
    set(BOOST_ROOT "${CMAKE_SOURCE_DIR}/externals/boost")
    set(Boost_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/externals/boost")
    set(Boost_NO_SYSTEM_PATHS ON)
    add_library(boost INTERFACE)
    target_include_directories(boost SYSTEM INTERFACE ${Boost_INCLUDE_DIR})

# Boost::serialization
    file(GLOB boost_serialization_SRC "${CMAKE_SOURCE_DIR}/externals/boost/libs/serialization/src/*.cpp")
    add_library(boost_serialization STATIC ${boost_serialization_SRC})
    target_link_libraries(boost_serialization PUBLIC boost)
endif()

# Add additional boost libs here; remember to ALIAS them in the root CMakeLists!

# Catch
add_library(catch-single-include INTERFACE)
target_include_directories(catch-single-include INTERFACE catch/single_include)

# Crypto++
if (NOT USE_SYSTEM_CRYPTOPP)
    add_subdirectory(cryptopp)
else()
    find_library(CRYPTOPP_LIBS cryptopp REQUIRED)
endif()

# Xbyak
if (ARCHITECTURE_x86_64)
    if (NOT USE_SYSTEM_XBYAK)
        add_library(xbyak INTERFACE)
        target_include_directories(xbyak SYSTEM INTERFACE ./xbyak/xbyak)
        target_compile_definitions(xbyak INTERFACE XBYAK_NO_OP_NAMES)
    else()
        find_path(XBYAK_INCLUDE xbyak.h PATH_SUFFIXES xbyak REQUIRED)
    endif()
endif()

# Dynarmic
if (ARCHITECTURE_x86_64 OR ARCHITECTURE_ARM64)
    set(DYNARMIC_TESTS OFF)
    set(DYNARMIC_NO_BUNDLED_FMT ON)
    set(DYNARMIC_FRONTENDS "A32")
    add_subdirectory(dynarmic)
endif()

# libfmt
if (NOT USE_SYSTEM_FMT)
    add_subdirectory(fmt)
    add_library(fmt::fmt ALIAS fmt)
else()
    find_package(fmt REQUIRED)
endif()

# getopt
if (MSVC)
    add_subdirectory(getopt)
endif()

# Glad
add_subdirectory(glad)

# inih
if (NOT USE_SYSTEM_INIH)
    add_subdirectory(inih)
else()
    find_library(INIH_LIBS INIReader REQUIRED)
endif()

# MicroProfile
add_library(microprofile INTERFACE)
target_include_directories(microprofile INTERFACE ./microprofile)

# Nihstro
add_library(nihstro-headers INTERFACE)
target_include_directories(nihstro-headers INTERFACE ./nihstro/include)

# Open Source Archives
add_subdirectory(open_source_archives)

# SoundTouch
add_subdirectory(soundtouch)
# The SoundTouch target doesn't export the necessary include paths as properties by default
target_include_directories(SoundTouch INTERFACE ./soundtouch/include)

# Teakra
if (NOT USE_SYSTEM_TEAKRA)
    add_subdirectory(teakra EXCLUDE_FROM_ALL)
else()
    find_library(TEAKRA_LIBS teakra REQUIRED)
endif()

# Zstandard
if (NOT USE_SYSTEM_ZSTD)
    add_subdirectory(zstd/build/cmake EXCLUDE_FROM_ALL)
    target_include_directories(libzstd_static INTERFACE $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/externals/zstd/lib>)
else()
    find_library(ZSTD_LIBS zstd REQUIRED)
endif()

# ENet
if (NOT USE_SYSTEM_ENET)
    add_subdirectory(enet)
    target_include_directories(enet INTERFACE ./enet/include)
else()
    find_library(ENET_LIBS enet REQUIRED)
endif()

# Cubeb
if (ENABLE_CUBEB)
    if (NOT USE_SYSTEM_CUBEB)
        set(BUILD_TESTS OFF CACHE BOOL "")
        add_subdirectory(cubeb EXCLUDE_FROM_ALL)
    else()
        find_package(cubeb REQUIRED)
    endif()
endif()

# DiscordRPC
if (USE_DISCORD_PRESENCE)
    add_subdirectory(discord-rpc EXCLUDE_FROM_ALL)
    target_include_directories(discord-rpc INTERFACE ./discord-rpc/include)
endif()

if (ENABLE_WEB_SERVICE)
    # LibreSSL
    set(LIBRESSL_SKIP_INSTALL ON CACHE BOOL "")
    add_subdirectory(libressl EXCLUDE_FROM_ALL)
    target_include_directories(ssl INTERFACE ./libressl/include)
    target_compile_definitions(ssl PRIVATE -DHAVE_INET_NTOP)

    # JSON
    add_library(json-headers INTERFACE)
    target_include_directories(json-headers INTERFACE ./json)

    # lurlparser
    add_subdirectory(lurlparser EXCLUDE_FROM_ALL)

    if(ANDROID)
        add_subdirectory(android-ifaddrs)
    endif()

    # httplib
    add_library(httplib INTERFACE)
    target_include_directories(httplib INTERFACE ./httplib)
    target_compile_options(httplib INTERFACE -DCPPHTTPLIB_OPENSSL_SUPPORT)

    # cpp-jwt
    add_library(cpp-jwt INTERFACE)
    target_include_directories(cpp-jwt INTERFACE ./cpp-jwt/include)
endif()

# lodepng
add_subdirectory(lodepng)
