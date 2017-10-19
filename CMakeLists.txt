project(MediaPlayer)
cmake_minimum_required(VERSION 2.8)

#加载自定义模块
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake")
#设置库文件路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
#设置可执行程序路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
#设置子目录
set(SUB_DIR_LIST "MediaPlayer" "libFaad" "AudioDec" "SDLAudioMixer")

#设置需要链接的库
set(LINK_LIB_LIST)

include_directories(src/libFaad)

#安装目录
if(WIN32)
set(INSTALL_PATH_LIB $ENV{HOME}/${CMAKE_PROJECT_NAME}/lib)
set(INSTALL_PATH_INCLUDE $ENV{HOME}/${CMAKE_PROJECT_NAME}/include)
else()
set(INSTALL_PATH_LIB lib)
set(INSTALL_PATH_INCLUDE include)
endif()

foreach(SUB_DIR ${SUB_DIR_LIST})
    #遍历源文件
    aux_source_directory(src/${SUB_DIR} SRC_LIST)
endforeach(SUB_DIR ${SUB_DIR_LIST})

#安装头文件至系统目录
install(FILES src/MediaPlayer/MediaPlayerWrapper.h DESTINATION ${INSTALL_PATH_INCLUDE})

#查找openssl是否安装
find_package(OpenSSL QUIET)
if(OPENSSL_FOUND)
message(STATUS "找到openssl库:\"${OPENSSL_INCLUDE_DIR}\",ENABLE_OPENSSL宏已打开")
include_directories(${OPENSSL_INCLUDE_DIR})
add_definitions(-DENABLE_OPENSSL)
list(APPEND LINK_LIB_LIST ${OPENSSL_LIBRARIES})
endif(OPENSSL_FOUND)

#查找mysql是否安装
find_package(MYSQL QUIET)
if(MYSQL_FOUND)
message(STATUS "找到mysqlclient库:\"${MYSQL_INCLUDE_DIR}\",ENABLE_MYSQL宏已打开")
include_directories(${MYSQL_INCLUDE_DIR})
add_definitions(-DENABLE_MYSQL)
list(APPEND LINK_LIB_LIST ${MYSQL_LIBRARIES})
endif(MYSQL_FOUND)

#查找ffmpeg/libavcodec是否安装
find_package(AVCODEC QUIET)
if(AVCODEC_FOUND)
include_directories(${AVCODEC_INCLUDE_DIR})
list(APPEND  LINK_LIB_LIST ${AVCODEC_LIBRARIES})
message(STATUS "找到libavcodec库:\"${AVCODEC_INCLUDE_DIR}\"")
endif(AVCODEC_FOUND)

#查找ffmpeg/libavcodec是否安装
find_package(AVUTIL QUIET)
if(AVUTIL_FOUND)
include_directories(${AVUTIL_INCLUDE_DIR})
list(APPEND  LINK_LIB_LIST ${AVUTIL_LIBRARIES})
message(STATUS "找到libavutil库:\"${AVUTIL_INCLUDE_DIR}\"")
endif(AVUTIL_FOUND)

#查找SDL2是否安装
find_package(SDL2 QUIET)
if(SDL2_FOUND)
include_directories(${SDL2_INCLUDE_DIR})
list(APPEND  LINK_LIB_LIST ${SDL2_LIBRARY})
message(STATUS "找到SDL2:${SDL2_INCLUDE_DIR},${SDL2_LIBRARY}")
endif(SDL2_FOUND)

#查找MP4V2是否安装
find_package(MP4V2 QUIET)
if(MP4V2_FOUND)
include_directories(${MP4V2_INCLUDE_DIR})
list(APPEND  LINK_LIB_LIST ${MP4V2_LIBRARY})
message(STATUS "找到MP4V2:${MP4V2_INCLUDE_DIR},${MP4V2_LIBRARY}")
endif(MP4V2_FOUND)

#查找ZLMediaKit是否安装
find_package(ZLMEDIAKIT QUIET)
if(ZLMEDIAKIT_FOUND)
message(STATUS "找到ZLMediaKit库:\"${ZLMEDIAKIT_INCLUDE_DIR}\"")
include_directories(${ZLMEDIAKIT_INCLUDE_DIR})
list(APPEND LINK_LIB_LIST ${ZLMEDIAKIT_LIBRARIES})
endif(ZLMEDIAKIT_FOUND)

#查找ZLToolKit是否安装
find_package(ZLTOOLKIT QUIET)
if(ZLTOOLKIT_FOUND)
message(STATUS "找到ZLToolKit库:\"${ZLTOOLKIT_INCLUDE_DIR}\"")
include_directories(${ZLTOOLKIT_INCLUDE_DIR})
list(APPEND LINK_LIB_LIST ${ZLTOOLKIT_LIBRARIES})
endif(ZLTOOLKIT_FOUND)


#打印库文件
message(STATUS "将链接依赖库:${LINK_LIB_LIST}")
#开启RTSP/RTMP之间的互相转换,开启HLS
add_definitions(-DENABLE_RTMP2RTSP -DENABLE_RTSP2RTMP -DENABLE_HLS -DHAVE_CONFIG_H)

#引用头文件路径
include_directories(${PROJECT_SOURCE_DIR}/src)

#使能c++11
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")

if(NOT WIN32)
#关闭过期接口警告
add_compile_options(-Wno-deprecated-declarations)
#关闭__FUNCTION__宏在函数外警告
add_compile_options(-Wno-predefined-identifier-outside-function)
endif(NOT WIN32)


#编译动态库
if(NOT IOS AND NOT ANDROID)
add_library(${CMAKE_PROJECT_NAME}_shared SHARED ${SRC_LIST})
set_target_properties(${CMAKE_PROJECT_NAME}_shared PROPERTIES OUTPUT_NAME "${CMAKE_PROJECT_NAME}")
install(TARGETS ${CMAKE_PROJECT_NAME}_shared  ARCHIVE DESTINATION ${INSTALL_PATH_LIB} LIBRARY DESTINATION ${INSTALL_PATH_LIB})
if(WIN32)
target_link_libraries(${CMAKE_PROJECT_NAME}_shared ${LINK_LIB_LIST} WS2_32 Iphlpapi shlwapi)
else(WIN32)
target_link_libraries(${CMAKE_PROJECT_NAME}_shared ${LINK_LIB_LIST})
endif(WIN32)
endif()

#编译静态库
add_library(${CMAKE_PROJECT_NAME}_static STATIC ${SRC_LIST})
set_target_properties(${CMAKE_PROJECT_NAME}_static PROPERTIES OUTPUT_NAME "${CMAKE_PROJECT_NAME}")
install(TARGETS ${CMAKE_PROJECT_NAME}_static ARCHIVE DESTINATION ${INSTALL_PATH_LIB})

if(WIN32)
#set_target_properties(${CMAKE_PROJECT_NAME}_static PROPERTIES COMPILE_FLAGS "/wd4819 /wd4996" )
#set_target_properties(${CMAKE_PROJECT_NAME}_shared PROPERTIES COMPILE_FLAGS "/wd4819 /wd4996" )
#set_target_properties(${CMAKE_PROJECT_NAME}_shared PROPERTIES LINK_FLAGS "/SAFESEH:NO" )
endif(WIN32)
#测试程序
if(NOT IOS)
add_subdirectory(tests)
endif(NOT IOS)






















