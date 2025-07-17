cmake_minimum_required(VERSION 3.13)
project(whisper-wasm)

#
# libmain (WASM build target)
#
set(TARGET libmain)

add_executable(${TARGET}
    emscripten.cpp
)

# Commented out because it's missing in the repo
# include(DefaultTargetOptions)

target_link_libraries(${TARGET} PRIVATE
    whisper
)

unset(EXTRA_FLAGS)

if (WHISPER_WASM_SINGLE_FILE)
    set(EXTRA_FLAGS "-s SINGLE_FILE=1")
    message(STATUS "Embedding WASM inside main.js")

    add_custom_command(
        TARGET ${TARGET} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
        ${CMAKE_BINARY_DIR}/bin/libmain.js
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/whisper.wasm/main.js
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
    -s EXPORTED_RUNTIME_METHODS=\"['print', 'printErr', 'ccall', 'cwrap', 'HEAPU8']\" \
    ${EXTRA_FLAGS} \
")

#
# whisper.wasm static files
#
set(TARGET whisper.wasm)

# Create directory to avoid configure_file error
file(MAKE_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${TARGET})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/index-tmpl.html  ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${TARGET}/index.html @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../helpers.js    ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${TARGET}/helpers.js @ONLY)
