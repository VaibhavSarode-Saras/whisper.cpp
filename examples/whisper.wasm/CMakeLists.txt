cmake_minimum_required(VERSION 3.13)
project(whisper-wasm)

#
# Add whisper.cpp file (adjust path if needed)
#
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../../..)

#
# main (WASM build target)
#
set(TARGET main)

add_executable(${TARGET}
    emscripten.cpp
    ../../../whisper.cpp
)

target_link_libraries(${TARGET} PRIVATE
    # add more if needed
)

unset(EXTRA_FLAGS)

if (WHISPER_WASM_SINGLE_FILE)
    set(EXTRA_FLAGS "-s SINGLE_FILE=1")
    message(STATUS "Embedding WASM inside main.js")

    add_custom_command(
        TARGET ${TARGET} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
        ${CMAKE_BINARY_DIR}/bin/main.js
        ${CMAKE_CURRENT_BINARY_DIR}/whisper.wasm/main.js
    )
endif()

set_target_properties(${TARGET} PROPERTIES LINK_FLAGS " \
    --bind \
    -s USE_PTHREADS=1 \
    -s PTHREAD_POOL_SIZE_STRICT=0 \
    -s INITIAL_MEMORY=512MB \
    -s MAXIMUM_MEMORY=2000MB \
    -s ALLOW_MEMORY_GROWTH=1 \
    -s FORCE_FILESYSTEM=1 \
    -s EXPORTED_RUNTIME_METHODS=['print','printErr','ccall','cwrap','HEAPU8'] \
    ${EXTRA_FLAGS} \
")

#
# whisper.wasm static files (index + helpers.js)
#
set(STATIC_DIR ${CMAKE_CURRENT_BINARY_DIR}/whisper.wasm)
file(MAKE_DIRECTORY ${STATIC_DIR})

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/index-tmpl.html
    ${STATIC_DIR}/index.html @ONLY
)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/../helpers.js
    ${STATIC_DIR}/helpers.js @ONLY
)
