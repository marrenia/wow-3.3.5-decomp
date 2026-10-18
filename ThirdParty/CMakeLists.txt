add_library(zlib STATIC
    zlib/adler32.c
    zlib/compress.c
    zlib/crc32.c
    zlib/deflate.c
    zlib/gzio.c
    zlib/infback.c
    zlib/inffast.c
    zlib/inflate.c
    zlib/inftrees.c
    zlib/trees.c
    zlib/uncompr.c
    zlib/zutil.c
)

target_include_directories(zlib
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/zlib
)

add_library(blast STATIC
    zlib/contrib/blast/blast.c
)

target_include_directories(blast
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/zlib/contrib/blast
)

add_library(bzip2 STATIC
    bzip2/blocksort.c
    bzip2/bzlib.c
    bzip2/compress.c
    bzip2/crctable.c
    bzip2/decompress.c
    bzip2/huffman.c
    bzip2/randtable.c
)

target_include_directories(bzip2
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/bzip2
)

add_library(lzma STATIC
    lzma/LzmaDecode.c
)

target_include_directories(lzma
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/lzma
)

if (WIN32)
    add_library(nvapi INTERFACE)

    target_include_directories(nvapi
        INTERFACE
            ${CMAKE_CURRENT_SOURCE_DIR}/nvapi
    )

    if (CMAKE_SIZEOF_VOID_P EQUAL 8)
        target_link_libraries(nvapi INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/nvapi/amd64/nvapi64.lib)
    else ()
        target_link_libraries(nvapi INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/nvapi/x86/nvapi.lib)
    endif ()
endif ()

add_library(freetype STATIC
    freetype/src/base/ftsystem.c
    freetype/src/base/ftinit.c
    freetype/src/base/ftdebug.c
    freetype/src/base/ftbase.c
    freetype/src/autohint/autohint.c
    freetype/src/cff/cff.c
    freetype/src/cid/type1cid.c
    freetype/src/psaux/psaux.c
    freetype/src/psnames/psnames.c
    freetype/src/raster/raster.c
    freetype/src/sfnt/sfnt.c
    freetype/src/smooth/smooth.c
    freetype/src/truetype/truetype.c
    freetype/src/type1/type1.c
    freetype/src/winfonts/winfnt.c
)

target_include_directories(freetype
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/freetype/include
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/freetype/src
)

add_library(expat STATIC
    expat/lib/xmlparse.c
    expat/lib/xmlrole.c
    expat/lib/xmltok.c
)

target_compile_definitions(expat
    PUBLIC
        _STATIC
    PRIVATE
        COMPILED_FROM_DSP
        _LIB
)

target_include_directories(expat
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/expat/lib
)

add_library(pcre STATIC
    pcre/pcre.c
    pcre/get.c
    pcre/study.c
    pcre/maketables.c
)

target_compile_definitions(pcre
    PUBLIC
        PCRE_STATIC
    PRIVATE
        SUPPORT_UTF8
        SUPPORT_UCP
)

target_include_directories(pcre
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/pcre
)

add_library(DivxDecoder INTERFACE)

target_include_directories(DivxDecoder
    INTERFACE
        ${CMAKE_CURRENT_SOURCE_DIR}/DivxDecoder
)

if (WOWDECOMP_ARCH_32)
    set(DIVXDECODER_IMPORT_LIB ${CMAKE_CURRENT_BINARY_DIR}/DivxDecoder.lib)

    execute_process(
        COMMAND ${CMAKE_AR} /NOLOGO /MACHINE:X86 /DEF:${CMAKE_CURRENT_SOURCE_DIR}/DivxDecoder/DivxDecoder.def /OUT:${DIVXDECODER_IMPORT_LIB}
        RESULT_VARIABLE DIVXDECODER_RESULT
    )

    if (NOT DIVXDECODER_RESULT EQUAL 0)
        message(FATAL_ERROR "DivxDecoder: creating the import library failed")
    endif ()

    set_property(DIRECTORY APPEND PROPERTY CMAKE_CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/DivxDecoder/DivxDecoder.def)

    target_link_libraries(DivxDecoder
        INTERFACE
            ${DIVXDECODER_IMPORT_LIB}
    )

    set(DIVXDECODER_DLL ${CMAKE_CURRENT_SOURCE_DIR}/DivxDecoder/DivxDecoder.dll)

    if (EXISTS ${DIVXDECODER_DLL})
        add_custom_target(DivxDecoderDll
            COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_SOURCE_DIR}/bin/Win
            COMMAND ${CMAKE_COMMAND} -E copy_if_different ${DIVXDECODER_DLL} ${CMAKE_SOURCE_DIR}/bin/Win/DivxDecoder.dll
        )

        add_dependencies(DivxDecoder DivxDecoderDll)
    else ()
        message(WARNING "DivxDecoder: ${DIVXDECODER_DLL} is missing. Copy DivxDecoder.dll from a 3.3.5a (12340) client there; Win32 builds need it next to the executables.")
    endif ()
else ()
    target_link_libraries(DivxDecoder
        INTERFACE
            XvidDecoder
    )
endif ()

if (WIN32)
    add_library(lgLcd STATIC
        lgLcd/lglcd.cpp
    )

    target_include_directories(lgLcd
        PUBLIC
            ${CMAKE_SOURCE_DIR}
    )
endif ()

# DXVK (Direct3D 9 -> Vulkan), pinned to a release and checked against its published SHA-256.
if (WOWDECOMP_DXVK)
    set(DXVK_VERSION 3.1.1)
    set(DXVK_SHA256 40565b4a724aadc4433fa4e010b4b23916d9b1f1baeee64e17186db94f54e608)
    set(DXVK_ARCHIVE ${CMAKE_CURRENT_BINARY_DIR}/dxvk-${DXVK_VERSION}.tar.gz)
    set(DXVK_ROOT ${CMAKE_CURRENT_BINARY_DIR}/dxvk-${DXVK_VERSION})

    if (WOWDECOMP_ARCH_64)
        set(DXVK_D3D9 ${DXVK_ROOT}/x64/d3d9.dll)
    else ()
        set(DXVK_D3D9 ${DXVK_ROOT}/x32/d3d9.dll)
    endif ()

    if (NOT EXISTS ${DXVK_D3D9})
        file(DOWNLOAD
            https://github.com/doitsujin/dxvk/releases/download/v${DXVK_VERSION}/dxvk-${DXVK_VERSION}.tar.gz
            ${DXVK_ARCHIVE}
            EXPECTED_HASH SHA256=${DXVK_SHA256}
            STATUS DXVK_DOWNLOAD_STATUS
            SHOW_PROGRESS
        )
        list(GET DXVK_DOWNLOAD_STATUS 0 DXVK_DOWNLOAD_RESULT)
        if (NOT DXVK_DOWNLOAD_RESULT EQUAL 0)
            message(FATAL_ERROR "DXVK: download failed (${DXVK_DOWNLOAD_STATUS}). Configure with -DWOWDECOMP_DXVK=OFF to build without it.")
        endif ()
        execute_process(
            COMMAND ${CMAKE_COMMAND} -E tar xzf ${DXVK_ARCHIVE}
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
            RESULT_VARIABLE DXVK_EXTRACT_RESULT
        )
        if (NOT DXVK_EXTRACT_RESULT EQUAL 0 OR NOT EXISTS ${DXVK_D3D9})
            message(FATAL_ERROR "DXVK: extracting ${DXVK_ARCHIVE} failed")
        endif ()
    endif ()

    add_custom_target(DxvkDll
        COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_SOURCE_DIR}/bin/Win
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${DXVK_D3D9} ${CMAKE_SOURCE_DIR}/bin/Win/d3d9.dll
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/dxvk/dxvk.conf ${CMAKE_SOURCE_DIR}/bin/Win/dxvk.conf
        COMMENT "DXVK ${DXVK_VERSION}: copying d3d9.dll and dxvk.conf to bin/Win"
    )
    message(STATUS "DXVK ${DXVK_VERSION}: d3d9.dll will be placed in bin/Win")
endif ()
