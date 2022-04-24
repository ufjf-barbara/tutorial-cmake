cmake_minimum_required (VERSION 3.14)

# Define the project
project(cmake-tutorial)

# GoogleTest requires at least C++11
set(CMAKE_CXX_STANDARD 11)

# Add definition for math library
add_library(math src/math.cc)


# Add definition for the cmake-tutorial binary
add_executable(cmake_tutorial src/main.cc)
target_link_libraries(cmake_tutorial math)

include(FetchContent)
FetchContent_Declare(
  googletest
  # Specify the commit you depend on and update it regularly.
  URL https://github.com/google/googletest/archive/e2239ee6043f73722e7aa812a459f54a28552929.zip
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)


# Test
add_executable(math_test test/math_test.cc)
target_link_libraries(math_test PRIVATE math gtest)

