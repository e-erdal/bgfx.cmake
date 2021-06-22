if(NOT IS_DIRECTORY ${BX_DIR})
	message(SEND_ERROR "BX directory (which is '${BX_DIR}') does not exist. Aborted.")
	return()
endif()

# Main source, including "amalgamated"
file(GLOB BX_SOURCE ${BX_DIR}/src/*.cpp)
# Amalgamated source files, does not include main source
file(GLOB BX_AMALGAMATED_SOURCE ${BX_DIR}/src/amalgamated.**)

if (NOT BX_AMALGAMATED)
    # We are not using amalgamated source so remove it
    list(REMOVE_ITEM BX_SOURCE "${BX_AMALGAMATED_SOURCE}")
else()
    # We are using amalgamated source, just set it as BX_SOURCE
    message("Using amalgamated build.")
    set(BX_SOURCE BX_AMALGAMATED_SOURCE)
endif()

# Initialize bx, does not support shared library option
add_library(bx STATIC ${BX_SOURCE})

if (CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)")
    set(X86 TRUE)
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "(x64)|(X64)")
    set(X64 TRUE)
endif()

message("Current compiler for bx: '${CMAKE_CXX_COMPILER_ID}' using ${CMAKE_SYSTEM_PROCESSOR} arch.")

# Include directories of bx, also includes it's dependencies
# Params from https://github.com/bkaradzic/bx/blob/master/scripts/toolchain.lua

if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC") # MSVC*
    target_include_directories(bx PUBLIC ${BX_DIR}/include ${BX_DIR}/3rdparty ${BX_DIR}/include/compat/msvc)

    target_compile_definitions(bx PUBLIC 
        WIN32 
        _WIN32 
        _HAS_EXCEPTIONS=0 
        _SCL_SECURE=0 
        _SECURE_SCL=0 
        _SCL_SECURE_NO_WARNINGS 
        _CRT_SECURE_NO_WARNINGS
        _CRT_SECURE_NO_DEPRECATE
    )

    target_compile_options(bx PUBLIC
        /wd4201
        /wd4324
        /Ob2
    )

    target_link_options(bx PUBLIC 
        /ignore:4221
    )

    if (X64)
        target_compile_definitions(bx PUBLIC _WIN64)
    endif()

elseif(WIN32 AND CMAKE_CXX_COMPILER_ID MATCHES "Clang") # Clang on WIN32
    target_include_directories(bx PUBLIC ${BX_DIR}/include ${BX_DIR}/3rdparty ${BX_DIR}/include/compat/msvc)

    target_compile_definitions(bx PUBLIC 
        WIN32 
        _WIN32 
        _HAS_EXCEPTIONS=0 
        _SCL_SECURE=0 
        _SECURE_SCL=0 
        _SCL_SECURE_NO_WARNINGS 
        _CRT_SECURE_NO_WARNINGS
        _CRT_SECURE_NO_DEPRECATE
    )

elseif(MINGW) # MINGW*
    target_include_directories(bx PUBLIC ${BX_DIR}/include ${BX_DIR}/3rdparty ${BX_DIR}/include/compat/mingw)

    target_compile_definitions(bx PUBLIC 
        WIN32
        MINGW_HAS_SECURE_API=1
    )

    target_compile_options(bx PUBLIC
        -Wunused-value
        -fdata-sections
        -ffunction-sections
        -msse2
        -Wunused-value
        -Wundef
    )

    target_link_options(bx PUBLIC 
        -Wl
        --gc-sections
        -static
        -static-libgcc
        -static-libstdc++
    )

    if (X64)
        target_compile_definitions(bx PUBLIC -m64)
    elseif (X86)        
        target_compile_definitions(bx PUBLIC -m32)
    endif()

elseif(${CMAKE_SYSTEM_NAME} MATCHES "iOS") # IOS*
    target_include_directories(bx PUBLIC ${BX_DIR}/include ${BX_DIR}/3rdparty ${BX_DIR}/include/compat/ios)

    target_compile_options(bx PUBLIC
        -Wfatal-errors
        -Wunused-value
        -Wundef
    )

    target_link_options(bx PUBLIC 
        -lc++
    )
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Mac OS X") # OSX*
    target_include_directories(bx PUBLIC ${BX_DIR}/include ${BX_DIR}/3rdparty ${BX_DIR}/include/compat/osx)

    target_compile_options(bx PUBLIC
        -Wfatal-errors
        -Wunused-value
        -Wundef
    )

else()
    message(SEND_ERROR "This compiler is not implemented!")
endif()

# Global macros
target_compile_definitions(bx PUBLIC __STDC_LIMIT_MACROS __STDC_FORMAT_MACROS __STDC_CONSTANT_MACROS)

# Additional configuration checks
if(BX_CONFUG_DEBUG)
	target_compile_definitions(bx PRIVATE BX_CONFIG_DEBUG=1) # Enables BX_TRACE, etc...
endif()

set_target_properties(bx PROPERTIES FOLDER "BGFXCMake")