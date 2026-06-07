
cmake_minimum_required(VERSION 3.26)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

project(my_hammered_game VERSION 0.1.0 LANGUAGES CXX)


include(cmake/CPM.cmake)


CPMAddPackage(
    NAME hammered
    GITHUB_REPOSITORY OneBogdan01/hammered
    GIT_TAG main
    OPTIONS
        "HM_BUILD_DEMOS OFF"
)


file(GLOB_RECURSE GAME_SOURCES CONFIGURE_DEPENDS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.hpp
)

add_executable(game ${GAME_SOURCES})
target_link_libraries(game PRIVATE hammered-engine)
set_target_properties(game PROPERTIES
    CXX_STANDARD 23
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)


if(IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/assets)
    add_custom_target(game_copy_assets ALL
        COMMAND ${CMAKE_COMMAND} -E copy_directory_if_different
            ${CMAKE_CURRENT_SOURCE_DIR}/assets
            $<TARGET_FILE_DIR:game>/assets
        COMMENT "Copying game assets"
    )
    add_dependencies(game game_copy_assets)
endif()


# `emcmake cmake -B build-web && cmake --build build-web`.
if(EMSCRIPTEN)
    set_target_properties(game PROPERTIES SUFFIX ".html" OUTPUT_NAME "index")
    target_link_options(game PRIVATE
        -sALLOW_MEMORY_GROWTH=1
        -sSTACK_SIZE=1mb
        -sEXPORTED_RUNTIME_METHODS=cwrap
        --shell-file ${hammered_SOURCE_DIR}/shell.html
    )
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        target_link_options(game PRIVATE
            -sASSERTIONS=2 -sSTACK_OVERFLOW_CHECK=2 -sSAFE_HEAP=1 -sGL_ASSERTIONS=1
        )
    endif()
endif()
