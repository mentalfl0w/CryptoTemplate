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

find_package(Doxygen REQUIRED)
if (DOXYGEN_FOUND)
  set(DOXYGEN_PROJECT_NAME ${PROJECT_NAME})
  set(DOXYGEN_PROJECT_BRIEF "")
  set(DOXYGEN_OUTPUT_LANGUAGE "English")
  set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/.cmake/Doxyfile.in)
  set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)
  configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
  add_custom_target(${PROJECT_NAME}-docs ALL
    COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generating API documentation with Doxygen"
    VERBATIM)
else ()
  message(WARNING "${PROJECT_NAME}: Doxygen not found, skip generating documentation")
endif ()

include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
