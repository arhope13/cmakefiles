# enable testing
enable_testing()

# set the minimum required version
cmake_minimum_required(VERSION 3.30.5)

# make sure that CMake uses C++20
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS, OFF)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# define constants
set(PROJ_NAME "[project name here]")
set(BIN_NAME "[name of compiled executable here]")

# set the project name
project(${PROJ_NAME} LANGUAGES CXX)

# include GNUInstallDirs
include(GNUInstallDirs)

# set the release type
set(CMAKE_BUILD_TYPE Release)

# load the source files
file(GLOB_RECURSE ALL_SRCS ${CMAKE_SOURCE_DIR}/src/*.cpp)
file(GLOB_RECURSE LIB_SRCS ${CMAKE_SOURCE_DIR}/src/*.cpp)
list(REMOVE_ITEM LIB_SRCS ${CMAKE_SOURCE_DIR}/src/main.cpp)

# 

# set the build targets
add_library(${BIN_NAME}_test STATIC ${LIB_SRCS})
add_executable(${BIN_NAME} ${ALL_SRCS})

# add the install target for the library
install(TARGETS ${BIN_NAME}
	DESTINATION $ENV{DESTDIR}${CMAKE_INSTALL_PREFIX}/bin)

# load GoogleTest
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
	RESULT_VARIABLE result
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download)

# make sure GoogleTest loaded successfully
if(result)
	message(FATAL_ERROR "CMake step for GoogleTest failed: ${result}")
endif()

# build GoogleTest
execute_process(COMMAND ${CMAKE_COMMAND} --build .
	RESULT_VARIABLE result
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download)

# make sure that GoogleTest was built successfully
if(result)
	message(FATAL_ERROR "Build step for GoogleTest failed: ${result}")
endif()

# prevent overridding Orion Engine's compiler/linker settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# add GoogleTest to the build, which defines gtest and gtest_main
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/googletest-src
		${CMAKE_CURRENT_BINARY_DIR}/googletest-build
		EXCLUDE_FROM_ALL)

# include the GoogleTest include files on versions below CMake 2.8.11
if(CMAKE_VERSION VERSION_LESS 2.8.11)
	include_directories("${gtest_SOURCE_DIR}/include")
endif()

# load the test source file
file(GLOB TESTS ${CMAKE_SOURCE_DIR}/tests/main.cpp)

# create the test executable
add_executable(runTests ${TESTS})

# link it with gtest and orion
target_link_libraries(runTests gtest ${BIN_NAME}_test)

# and add the test to ctest
add_test(NAME EngineTests COMMAND runTests)

# end of file
