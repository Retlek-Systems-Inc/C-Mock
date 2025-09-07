
cmake_minimum_required(VERSION 3.23)

project(cmock VERSION 0.1.2 LANGUAGES C CXX)

include(GNUInstallDirs)

option(BUILD_TESTING "Build tests for C-Mock" OFF)

# -------------------------------------------------------------------
# If not already declared - get gmock.
if (BUILD_TESTING AND NOT TARGET gmock)
    include(FetchContent)

    FetchContent_Declare( googletest
        GIT_REPOSITORY    https://github.com/google/googletest.git
        GIT_TAG           release-1.17.0
    )

    # Prevent overriding the parent project's compiler/linker
    # settings on Windows
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE) 
    FetchContent_MakeAvailable(googletest)
    set(GTEST_ROOT ${googletest_SOURCE_DIR} CACHE STRING "Root dir for find(GTest)")
endif()

# -------------------------------------------------------------------
add_library(gmock_c INTERFACE)

target_sources(gmock_c
    INTERFACE
        FILE_SET HEADERS TYPE HEADERS
        BASE_DIRS include
        FILES
            include/cmock/cmock-function-class-mockers.h
            include/cmock/cmock-function-mockers.h
            include/cmock/cmock-internal.h
            include/cmock/cmock-spec-builders.h
            include/cmock/cmock.h
)

target_include_directories(gmock_c SYSTEM
    INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

target_link_libraries( gmock_c
    INTERFACE
        gmock
        ${CMAKE_DL_LIBS}
)

# -------------------------------------------------------------------
if (BUILD_TESTING)
    add_subdirectory(test)
endif()

# -------------------------------------------------------------------
# Install the target, exporting headers automatically via FILE_SET
install(TARGETS gmock_c
    EXPORT gmock_cTargets
    FILE_SET HEADERS
)

# Export target for find_package() support
install(EXPORT gmock_cTargets
    NAMESPACE cmock::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/gmock_c
)