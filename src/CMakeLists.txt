cmake_minimum_required(VERSION 2.8.11)
project(wamp)
add_definitions(-DWAMP_LIBRARY -DQT_NO_KEYWORDS -DSECRET_WITH_UNSTABLE -DUNICODE)
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

    find_package(LibSecret REQUIRED)
endif()
if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    add_definitions(-DUNICODE)
endif()
find_package(MsgPack REQUIRED)
file(GLOB_RECURSE DATABASE_SRCS
         RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
         ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp
         ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
include_directories(./client ./router ${MSGPACK_INCLUDE_DIRS} ${LIBSECRET_INCLUDE_DIRS})
# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)

set(QT_MIN_VERSION "5.6.0")
find_package(Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED Core Qml)
#find_package(MsgPack REQUIRED)

add_library(wamp SHARED ${DATABASE_SRCS} qmldir)
get_target_property(core_INCLUDE_DIRECTORIES core INCLUDE_DIRECTORIES)
get_target_property(websockets_INCLUDE_DIRECTORIES websockets INCLUDE_DIRECTORIES)
include_directories(${core_INCLUDE_DIRECTORIES} ${websockets_INCLUDE_DIRECTORIES})
set_property(TARGET wamp PROPERTY CXX_STANDARD 14)
add_dependencies(wamp core websockets)
if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    set(ADDITIONAL_LIBS ${ADDITIONAL_LIBS} Advapi32 Secur32)
endif()
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    set(ADDITIONAL_LIBS ${ADDITIONAL_LIBS} ${LIBSECRET_LIBRARIES})
endif()
target_link_libraries(wamp core websockets Qt5::Core Qt5::Qml ${ADDITIONAL_LIBS})
set(WAMP_INSTALL_PATH "bin/plugins/QFlow/Wamp")
install (TARGETS wamp DESTINATION ${WAMP_INSTALL_PATH})
install (FILES qmldir DESTINATION ${WAMP_INSTALL_PATH})
