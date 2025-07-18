cmake_minimum_required(VERSION 3.13)
project(whisper-wasm)

# ✅ 1. Link to whisper.cpp source
add_subdirectory(${CMAKE_SOURCE_DIR}/../../src ${CMAKE_BINARY_DIR}/whisper-build)

# ✅ 2. Include header files
include_directories(${CMAKE_SOURCE_DIR}/../../include)

# === Build libmain ===
set(TARGET libmain)

add_executable(${TARGET}
    emscripten.cpp
)

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
    -s EXPORTED_RUNTIME_METHODS=\"['print', 'printErr', 'ccall', 'cwrap', 'HEAPU8']\" \
    ${EXTRA_FLAGS} \
")

# === Static files ===
set(TARGET whisper.wasm)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${TARGET})

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/index-tmpl.html
    ${CMAKE_CURRENT_BINARY_DIR}/${TARGET}/index.html @ONLY
)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/../helpers.js
    ${CMAKE_CURRENT_BINARY_DIR}/${TARGET}/helpers.js @ONLY
)
