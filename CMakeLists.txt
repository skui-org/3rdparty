##
#
# The MIT License (MIT)
#
# Copyright (c) 2017-2019 Ruben Van Boxem
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

cmake_minimum_required(VERSION 3.5)
project(3rdparty)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
include(toolchain_options)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)

# packaging
if(NOT CPack_CMake_INCLUDED)
  include(CPack)
endif()

# Miscellaneous files
add_custom_target(miscellaneous_3rdparty SOURCES
    appveyor.yml
    README.md
    .travis.yml)

# libexpat
if(NOT EXPAT_FOUND)
  set(expat_sources libexpat/xmlparse.c
                    libexpat/xmlrole.c
                    libexpat/xmltok.c)
  add_library(expat STATIC ${expat_sources})
  target_include_directories(expat INTERFACE ${CMAKE_CURRENT_LIST_DIR}/libexpat)
  target_compile_definitions(expat PRIVATE XML_STATIC
                                           $<$<NOT:$<PLATFORM_ID:Windows>>:HAVE_MEMMOVE>
                                           $<$<NOT:$<PLATFORM_ID:Windows>>:XML_DEV_URANDOM>
                                           $<$<PLATFORM_ID:Windows>:WIN32>)
  target_compile_options(expat PRIVATE $<$<C_COMPILER_ID:MSVC>:/wd4311>
                                       $<$<PLATFORM_ID:MinGW>:-Wno-pointer-to-int-cast>)
  set_target_properties(expat PROPERTIES FOLDER 3rdparty/expat)
endif()

# zlib
if(NOT ZLIB_FOUND)
  set(zlib_sources zlib/adler32.c
                   zlib/compress.c
                   zlib/crc32.c
                   zlib/deflate.c
                   zlib/gzclose.c
                   zlib/gzlib.c
                   zlib/gzread.c
                   zlib/gzwrite.c
                   zlib/infback.c
                   zlib/inffast.c
                   zlib/inflate.c
                   zlib/inftrees.c
                   zlib/trees.c
                   zlib/uncompr.c
                   zlib/zutil.c)
  file(GLOB zlib_headers zlib/*.h)
  add_library(zlib STATIC ${zlib_sources} ${zlib_headers})
  target_include_directories(zlib INTERFACE ${CMAKE_CURRENT_LIST_DIR}/zlib)
  target_compile_options(zlib PRIVATE $<$<C_COMPILER_ID:MSVC>:/wd4996 /wd4267>
                                      $<$<PLATFORM_ID:Darwin>:-Wno-implicit-function-declaration>)
  set_target_properties(zlib PROPERTIES FOLDER 3rdparty/zlib)
endif()

# libpng
if(NOT PNG_FOUND)
  set(libpng_sources libpng/png.c
                     libpng/pngerror.c
                     libpng/pngget.c
                     libpng/pngmem.c
                     libpng/pngpread.c
                     libpng/pngread.c
                     libpng/pngrio.c
                     libpng/pngrtran.c
                     libpng/pngrutil.c
                     libpng/pngset.c
                     libpng/pngtrans.c
                     libpng/pngwio.c
                     libpng/pngwrite.c
                     libpng/pngwtran.c
                     libpng/pngwutil.c)
  file(GLOB png_headers libpng/*.h)
  if(ARM OR ARM64)
    set(libpng_sources ${libpng_sources}
                       arm_init.c
                       filter_neon_intrinsics.c)
  elseif(X86)
    set(libpng_sources ${libpng_sources}
                       intel_init.c
                       filter_sse2_intrinsics.c)
    set(libpng_defines PNG_INTEL_SSE)
  endif()
  file(GLOB libpng_headers libpng/*.h)
  add_library(png STATIC ${libpng_sources} ${libpng_headers})
  target_include_directories(png INTERFACE ${CMAKE_CURRENT_LIST_DIR}/libpng)
  target_compile_definitions(png PRIVATE ${libpng_defines}
                                         $<$<C_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>)
  target_link_libraries(png PUBLIC zlib)
  set_target_properties(png PROPERTIES FOLDER 3rdparty/png)
endif()

# giflib
if(NOT GIF_FOUND)
  set(giflib_sources giflib/dgif_lib.c
                     giflib/egif_lib.c
                     giflib/gif_err.c
                     giflib/gif_font.c
                     giflib/gif_hash.c
                     giflib/gifalloc.c
                     giflib/quantize.c)
  file(GLOB giflib_headers giflib/*.h)
  add_library(gif STATIC ${giflib_sources} ${giflib_headers})
  target_include_directories(gif INTERFACE ${CMAKE_CURRENT_LIST_DIR}/giflib)
  target_compile_options(gif PRIVATE $<$<C_COMPILER_ID:MSVC>:/wd4996 /wd4018 /wd4267 /wd4028 /wd4244>)
  set_target_properties(gif PROPERTIES FOLDER 3rdparty/gif)
endif()

# libjpeg-turbo
if(NOT LibJpegTurbo_FOUND)
  set(jpeg-turbo_sources libjpeg-turbo/jaricom.c
                         libjpeg-turbo/jcapimin.c
                         libjpeg-turbo/jcapistd.c
                         libjpeg-turbo/jcarith.c
                         libjpeg-turbo/jccoefct.c
                         libjpeg-turbo/jccolor.c
                         libjpeg-turbo/jcdctmgr.c
                         libjpeg-turbo/jchuff.c
                         libjpeg-turbo/jcinit.c
                         libjpeg-turbo/jcmainct.c
                         libjpeg-turbo/jcmarker.c
                         libjpeg-turbo/jcmaster.c
                         libjpeg-turbo/jcomapi.c
                         libjpeg-turbo/jcparam.c
                         libjpeg-turbo/jcphuff.c
                         libjpeg-turbo/jcprepct.c
                         libjpeg-turbo/jcsample.c
                         libjpeg-turbo/jdapimin.c
                         libjpeg-turbo/jdapistd.c
                         libjpeg-turbo/jdarith.c
                         libjpeg-turbo/jdcoefct.c
                         libjpeg-turbo/jdcolor.c
                         libjpeg-turbo/jddctmgr.c
                         libjpeg-turbo/jdhuff.c
                         libjpeg-turbo/jdicc.c
                         libjpeg-turbo/jdinput.c
                         libjpeg-turbo/jdmainct.c
                         libjpeg-turbo/jdmarker.c
                         libjpeg-turbo/jdmaster.c
                         libjpeg-turbo/jdmerge.c
                         libjpeg-turbo/jdphuff.c
                         libjpeg-turbo/jdpostct.c
                         libjpeg-turbo/jdsample.c
                         libjpeg-turbo/jerror.c
                         libjpeg-turbo/jfdctflt.c
                         libjpeg-turbo/jidctflt.c
                         libjpeg-turbo/jidctfst.c
                         libjpeg-turbo/jidctint.c
                         libjpeg-turbo/jidctred.c
                         libjpeg-turbo/jmemmgr.c
                         libjpeg-turbo/jmemnobs.c
                         libjpeg-turbo/jquant1.c
                         libjpeg-turbo/jquant2.c
                         libjpeg-turbo/jutils.c
                         )
  file(GLOB jpeg-turbo_headers libjpeg-turbo/*.h)
  add_library(jpeg-turbo STATIC ${jpeg-turbo_sources} ${jpeg-turbo_headers})
  target_include_directories(jpeg-turbo INTERFACE ${CMAKE_CURRENT_LIST_DIR}/libjpeg-turbo)
  target_compile_definitions(jpeg-turbo PRIVATE $<$<C_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>)
  set_target_properties(jpeg-turbo PROPERTIES FOLDER 3rdparty/jpeg-turbo)
endif()

# Skia
if(NOT SKIA_FOUND)
  set(SKIA_STATIC TRUE)
  add_subdirectory(skia)
  set_target_properties(skia PROPERTIES FOLDER 3rdparty)
endif()

# OpenGL API headers for platforms that don't supply their own
add_library(opengl-api INTERFACE)
target_include_directories(opengl-api INTERFACE ${CMAKE_CURRENT_LIST_DIR}/opengl-api)

add_library(vulkan-api INTERFACE)
target_include_directories(vulkan-api INTERFACE ${CMAKE_CURRENT_LIST_DIR}/vulkan-api)

# Boost Spirit X3 and dependencies
if(NOT Boost_FOUND)
  add_library(boost INTERFACE)
  target_include_directories(boost SYSTEM INTERFACE ${CMAKE_CURRENT_LIST_DIR}/boost)
endif()

# install rules
install(TARGETS expat zlib png gif jpeg-turbo opengl-api skia boost
        ARCHIVE DESTINATION "."
        RUNTIME DESTINATION "."
        LIBRARY DESTINATION ".")
install(DIRECTORY "libexpat/" DESTINATION "libexpat"
                             FILES_MATCHING PATTERN "*.h")
install(DIRECTORY "zlib/" DESTINATION "zlib"
                         FILES_MATCHING PATTERN "*.h")
install(DIRECTORY "libpng/" DESTINATION "libpng"
                           FILES_MATCHING PATTERN "*.h")
install(DIRECTORY "giflib/" DESTINATION "giflib"
                           FILES_MATCHING PATTERN "*.h")
install(DIRECTORY "libjpeg-turbo/" DESTINATION "libjpeg-turbo"
                                  FILES_MATCHING PATTERN "*.h")
install(DIRECTORY "opengl-api/" DESTINATION "opengl-api")
install(DIRECTORY "vulkan-api/" DESTINATION "vulkan-api")
install(DIRECTORY "skia/include" DESTINATION "skia")
install(DIRECTORY "skia/third_party" DESTINATION "skia")
install(DIRECTORY "boost/" DESTINATION "boost")

# Miscellaneous files
add_custom_target(3rdparty_misc SOURCES
    README.md
    appveyor.yml
    .travis.yml)
set_target_properties(3rdparty_misc PROPERTIES FOLDER 3rdparty)
