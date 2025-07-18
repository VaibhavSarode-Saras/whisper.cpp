cmake_minimum_required(VERSION 3.13)
project(whisper-wasm)

# ✅ Manually include whisper.cpp headers
include_directories(${CMAKE_SOURCE_DIR}/../../)
link_directories(${CMAKE_SOURCE_DIR}/../../build)

# Output folder for WASM artifacts
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/build/bin)

#
# libmain (WASM build target)
#
set(TARGET libmain)

add_executable(${TARGET}
    emscripten.cpp
)

# ✅ Link whisper (make sure libwhisper.a exists in build/)
target_link_libraries(${TARGET} PRIVATE whisper)

# 🧠 Optional flags
unset(EXTRA_FLAGS)

if (WHISPER_WASM_SINGLE_FILE)
    set(EXTRA_FLAGS "-s SINGLE_FILE=1")
    message(STATUS "Embedding WASM inside main.js")

    add_custom_command(
        TARGET ${TARGET} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/libmain.js
        ${CMAKE_BINARY_DIR}/build/whisper.wasm/main.js
    )
endif()

# 🧠 Emscripten-specific linking
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
# Static files output (index.html + helpers.js)
#
set(STATIC_TARGET whisper.wasm)

file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/build/${STATIC_TARGET})

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/index-tmpl.html
    ${CMAKE_BINARY_DIR}/build/${STATIC_TARGET}/index.html @ONLY
)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/../helpers.js
    ${CMAKE_BINARY_DIR}/build/${STATIC_TARGET}/helpers.js @ONLY
)
