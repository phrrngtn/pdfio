cmake_minimum_required(VERSION 3.0)

project(pdfio VERSION "1.1.2" LANGUAGES C)
find_package(ZLIB REQUIRED)

# I would like a vcpkg for this project and I was not able to get a portfile.cmake
# to work with the existing Makefile so providing this bare-bones CMake file instead.

set(SOURCES
        pdfio-aes.c
        pdfio-array.c
        pdfio-common.c
        pdfio-content.c
        pdfio-crypto.c
        pdfio-dict.c
        pdfio-file.c
        pdfio-md5.c
        pdfio-object.c
        pdfio-page.c
        pdfio-rc4.c
        pdfio-sha256.c
        pdfio-stream.c
        pdfio-string.c
        pdfio-token.c
        pdfio-value.c
        ttf.c
)
add_library(pdfio SHARED ${SOURCES} )
set_target_properties(pdfio PROPERTIES PUBLIC_HEADER "pdfio.h;pdfio-content.h")

# Don't seem to be able to get testpdfio to build correctly with dynamic linking
# Coming up with errors about missing symbols including
#  _pdfioCryptoRC4Init
# I see it defined with a small 't' in libpdfio.so (meaning it is not public)
# It looks like this has something to do with _PDFIO_INTERNAL in pdfio-private.h
# so it may beq quite intentional behavior.
set_property(TARGET pdfio PROPERTY VISIBILITY_INLINES_HIDDEN OFF)

# workaround is to declare a static library and then link that
add_library(pdfio_static STATIC ${SOURCES})

add_executable(pdfiototext pdfiototext.c)
add_executable(testpdfio testpdfio.c)

# requirements copied from README of the project.
set_property(TARGET pdfio PROPERTY C_STANDARD 99)
set_property(TARGET pdfiototext PROPERTY C_STANDARD 99)
set_property(TARGET testpdfio PROPERTY C_STANDARD 99)

# Don't add math-library if we are on WIN32
IF (NOT WIN32)
  # https://stackoverflow.com/a/74867749/40387
  find_library(MATH_LIBRARY m)
  set(LIBRARIES ${ZLIB_LIBRARIES} ${MATH_LIBRARY})
else()
  set(LIBRARIES ${ZLIB_LIBRARIES})
endif()

# Note the workaround by using pdfio_static
target_link_libraries(testpdfio PUBLIC pdfio_static ${LIBRARIES})
target_link_libraries(pdfiototext PUBLIC pdfio ${LIBRARIES})

# leave it up to the calling environment to decide where to put things.
# STUDY: believe that the hidden visibility of some of the symbols is quite intentional
#        so don't bother installing the static library as it is just a workaround.
# TODO: add targets for the documentation, LICENSE and other files.
install(TARGETS pdfio  pdfiototext testpdfio
    PUBLIC_HEADER)