cmake_minimum_required(VERSION 3.18.1)

project("dpt")

set(DPT_NATIVE_SOURCE
        dpt.cpp
        dpt_hook.cpp
        dex/MultiDexCode.cpp
        dex/CodeItem.cpp
        dpt_jni.cpp
        dpt_util.cpp
        dex/dex_file.cpp
        reflect/dalvik_system_BaseDexClassLoader.cpp
        reflect/dalvik_system_DexPathList.cpp
        reflect/java_io_File.cpp
        reflect/java_util_ArrayList.cpp
        reflect/java_lang_Class.cpp
        reflect/android_app_ActivityThread.cpp
        reflect/android_content_pm_ApplicationInfo.cpp
        reflect/android_app_Application.cpp
        reflect/android_app_LoadedApk.cpp
        reflect/android_app_ContextImpl.cpp
        )
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror")
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")


option(MZ_LZMA "Enables LZMA & XZ compression" OFF)
option(MZ_ZSTD "Enables ZSTD compression" OFF)
option(MZ_OPENSSL "Enables OpenSSL for encryption" OFF)

include_directories(./ )
include_directories(./common)
include_directories(./bhook/include)
include_directories(./minizip-ng)

# 添加库
add_library(dpt
             SHARED
        ${DPT_NATIVE_SOURCE})


include_directories(
        dobby
)

add_subdirectory(bhook)
add_subdirectory(minizip-ng)

# 使用预编译库
add_library(local_dobby STATIC IMPORTED)
set_target_properties(local_dobby PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/../../../libs/${ANDROID_ABI}/libdobby.a)

# 寻找已存在的库
find_library(log-lib log)
find_library(android-lib android)

option (DEBUG_MODE "Build as debug" OFF)
if(${DEBUG_MODE})
    add_definitions(-DDEBUG)
    message("WARNNING:opened log")
else()
    message("closed log")
endif()

target_link_libraries( dpt
            ${log-lib}
        MINIZIP::minizip
        local_dobby
        bytehook
        ${android-lib}
        )


ENABLE_LANGUAGE(ASM)