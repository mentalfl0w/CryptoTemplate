cmake_minimum_required(VERSION 3.16)

project(CryptoTemplate LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core)

add_subdirectory(3rdparty/MIRACL-Project)

option(ENABLE_SHA1 OFF)
option(ENABLE_AES OFF)
option(ENABLE_CHACHA20 OFF)
option(ENABLE_SDF OFF)
add_subdirectory(3rdparty/GmSSL)

set(SOURCE source/main.cpp)

add_executable(${PROJECT_NAME}
  ${SOURCE}
)
target_link_libraries(${PROJECT_NAME}
  Qt${QT_VERSION_MAJOR}::Core
  Miracl
  gmssl
)

target_include_directories(${PROJECT_NAME} PRIVATE
  include
  3rdparty/GmSSL/include
)

include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
