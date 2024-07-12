cmake_minimum_required(VERSION 3.10)
project(quirc VERSION 1.2 LANGUAGES C CXX)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(BUILD_TESTS "Build test programs" OFF)
option(BUILD_DEMOS "Build demo programs" OFF)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)

# Library source files
set(QUIRC_SRC
    lib/decode.c
    lib/identify.c
    lib/quirc.c
    lib/version_db.c
)

# Create quirc library
add_library(quirc ${QUIRC_SRC})
target_include_directories(quirc PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/lib)
target_compile_options(quirc PRIVATE -Wall -O3)

if(NOT BUILD_SHARED_LIBS)
    set_target_properties(quirc PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

# Install library and header
include(GNUInstallDirs)
install(TARGETS quirc
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(FILES lib/quirc.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

if(BUILD_TESTS)
    # QR Test
    add_executable(qrtest tests/dbgutil.c tests/qrtest.c)
    target_link_libraries(qrtest quirc m jpeg png)

    # Find packages for other tests if needed
    find_package(SDL)
    find_package(SDL_gfx)
    find_package(OpenCV)

    if(SDL_FOUND AND SDL_gfx_FOUND)
        add_executable(inspect tests/dbgutil.c tests/inspect.c)
        target_link_libraries(inspect quirc m jpeg png SDL::SDL SDL_gfx::SDL_gfx)
    endif()

    if(OpenCV_FOUND)
        add_executable(inspect_opencv tests/dbgutil.c tests/inspect_opencv.cpp)
        target_link_libraries(inspect_opencv quirc m jpeg png ${OpenCV_LIBS})
    endif()
endif()

if(BUILD_DEMOS)
    # Demo executables
    set(DEMO_SRC
        demo/camera.c
        demo/mjpeg.c
        demo/convert.c
        demo/dthash.c
        demo/demoutil.c
    )

    if(SDL_FOUND AND SDL_gfx_FOUND)
        add_executable(quirc-demo ${DEMO_SRC} demo/demo.c)
        target_link_libraries(quirc-demo quirc m jpeg SDL::SDL SDL_gfx::SDL_gfx)
    endif()

    if(OpenCV_FOUND)
        add_executable(quirc-demo-opencv ${DEMO_SRC} demo/demo_opencv.cpp)
        target_link_libraries(quirc-demo-opencv quirc m ${OpenCV_LIBS})
    endif()

    add_executable(quirc-scanner ${DEMO_SRC} demo/scanner.c)
    target_link_libraries(quirc-scanner quirc m jpeg)
endif()