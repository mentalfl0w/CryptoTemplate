cmake_minimum_required(VERSION 3.21)
project(Miracl LANGUAGES CXX C)
set(CMAKE_AUTOMOC OFF)
set(CMAKE_AUTOUIC OFF)
set(CMAKE_AUTORCC OFF)
include(GNUInstallDirs)

set(SOURCE_FILES miraclSource/source/mrcore.c miraclSource/source/mrarth0.c miraclSource/source/mrarth1.c
    miraclSource/source/mrarth2.c miraclSource/source/mralloc.c miraclSource/source/mrsmall.c miraclSource/source/mrio1.c
    miraclSource/source/mrio2.c miraclSource/source/mrgcd.c miraclSource/source/mrjack.c miraclSource/source/mrxgcd.c
    miraclSource/source/mrarth3.c miraclSource/source/mrbits.c miraclSource/source/mrrand.c miraclSource/source/mrprime.c
    miraclSource/source/mrcrt.c miraclSource/source/mrscrt.c miraclSource/source/mrmonty.c miraclSource/source/mrpower.c
    miraclSource/source/mrsroot.c miraclSource/source/mrcurve.c miraclSource/source/mrfast.c miraclSource/source/mrshs.c
    miraclSource/source/mrshs256.c miraclSource/source/mrshs512.c miraclSource/source/mrsha3.c miraclSource/source/mrfpe.c
    miraclSource/source/mraes.c miraclSource/source/mrgcm.c miraclSource/source/mrlucas.c miraclSource/source/mrzzn2.c
    miraclSource/source/mrzzn2b.c miraclSource/source/mrzzn3.c miraclSource/source/mrzzn4.c miraclSource/source/mrecn2.c
    miraclSource/source/mrstrong.c miraclSource/source/mrbrick.c miraclSource/source/mrebrick.c miraclSource/source/mrec2m.c
    miraclSource/source/mrgf2m.c miraclSource/source/mrflash.c miraclSource/source/mrfrnd.c miraclSource/source/mrdouble.c
    miraclSource/source/mrround.c miraclSource/source/mrbuild.c miraclSource/source/mrflsh1.c miraclSource/source/mrpi.c
    miraclSource/source/mrflsh2.c miraclSource/source/mrflsh3.c miraclSource/source/mrflsh4.c)

if(CMAKE_SIZEOF_VOID_P EQUAL 8) # X64 or aarch64
    message(STATUS "MIRACL: 64-bit build")
    if(CMAKE_CXX_COMPILER_ID)
        set(MIRDEF_SUFFIX "hpp")
        list(APPEND SOURCE_FILES miraclSource/source/big.cpp miraclSource/source/zzn.cpp
            miraclSource/source/ecn.cpp miraclSource/source/ec2.cpp miraclSource/source/flash.cpp
            miraclSource/source/crt.cpp)
    else()
        set(MIRDEF_SUFFIX "h64")
    endif()
    if(MSVC)
        set(MIRDEF_SUFFIX "w64")
        set(MRMULDV_SUFFIX "w64")
        set(DIR_FLAG "win64")
    elseif(MINGW)
        set(MIRDEF_SUFFIX "mgwn")
        set(DIR_FLAG "win64")
        set(MRMULDV_SUFFIX "g64")
    else()
        if(APPLE)
            set(DIR_FLAG "macOS")
            set(CMAKE_OSX_ARCHITECTURES "x86_64" CACHE STRING "" FORCE)
        elseif(WIN32)
            set(DIR_FLAG "win64")
        else()
            set(MIRDEF_SUFFIX "linux64")
            set(DIR_FLAG "linux64")
        endif()
        set(MRMULDV_SUFFIX "g64")
    endif()
else()
    message(STATUS "MIRACL: 32-bit build")
    set(MIRDEF_SUFFIX "h32")
    if(MSVC)
        set(MRMULDV_SUFFIX "c")
        set(DIR_FLAG "win32")
    else()
        if(APPLE)
            set(DIR_FLAG "OSX")
            set(CMAKE_OSX_ARCHITECTURES "x86_64" CACHE STRING "" FORCE)
        elseif(WIN32)
            set(DIR_FLAG "win32")
        else()
            set(MIRDEF_SUFFIX "linux32")
            set(DIR_FLAG "linux32")
        endif()
        set(MRMULDV_SUFFIX "gcc")
    endif()
endif()


file(GLOB HEADER_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}/miraclSource/include ${CMAKE_CURRENT_SOURCE_DIR}/miraclSource/include/*.h)
foreach(HEADER_FILE ${HEADER_FILES})
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/miraclSource/include/${HEADER_FILE} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include)
    list(APPEND SOURCE_FILES ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include/${HEADER_FILE})
endforeach()
if(MINGW OR LINUX)
    file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/fixed_headers/mirdef.${MIRDEF_SUFFIX}"
        DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include)
else()
    file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/miraclSource/include/mirdef.${MIRDEF_SUFFIX}"
        DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include)
endif()
file(RENAME ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include/mirdef.${MIRDEF_SUFFIX}
    ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include/mirdef.h)
list(APPEND SOURCE_FILES "${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include/mirdef.h")

file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/miraclSource/source/mrmuldv.${MRMULDV_SUFFIX}"
    DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/source)
file(RENAME ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/source/mrmuldv.${MRMULDV_SUFFIX}
    ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/source/mrmuldv.c)
list(APPEND SOURCE_FILES "${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/source/mrmuldv.c")

# 生成静态库
add_library(${PROJECT_NAME} STATIC ${SOURCE_FILES})

target_include_directories(${PROJECT_NAME} PUBLIC
    ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include
)

if(MIRACL_INSTALL_BINDIR)
    set(CMAKE_INSTALL_BINDIR ${MIRACL_INSTALL_BINDIR})
endif()

if(MIRACL_INSTALL_LIBDIR)
    set(CMAKE_INSTALL_LIBDIR ${MIRACL_INSTALL_LIBDIR})
endif()

if(MIRACL_INSTALL_INCLUDEDIR)
    set(CMAKE_INSTALL_INCLUDEDIR ${MIRACL_INSTALL_INCLUDEDIR})
endif()

install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

file(GLOB HEADER_FILES ${CMAKE_CURRENT_BINARY_DIR}/${DIR_FLAG}/include/*.h)
install(FILES ${HEADER_FILES} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
