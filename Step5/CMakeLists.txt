cmake_minimum_required(VERSION 3.15)

# set the project name and version
project(Tutorial_5 VERSION 1.0)

# specify the C++ standard
add_library(tutorial_compiler_flags_5 INTERFACE)
target_compile_features(tutorial_compiler_flags_5 INTERFACE cxx_std_11)

# add compiler warning flags just when building this project via
# the BUILD_INTERFACE genex
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags_5 INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial_5 tutorial.cxx)

target_link_libraries(Tutorial_5 PUBLIC MathFunctions_5 tutorial_compiler_flags_5)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial_5 PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

# TODO 3: Install Tutorial in the bin directory
# Hint: Use the TARGETS and DESTINATION parameters
install(TARGETS Tutorial_5 DESTINATION bin)

# TODO 4: Install TutorialConfig.h to the include directory
# Hint: Use the FILES and DESTINATION parameters
install(FILES ${PROJECT_BINARY_DIR}/TutorialConfig.h DESTINATION include)

# TODO 5: Enable testing
enable_testing()
# TODO 6: Add a test called Runs which runs the following command:
# $ Tutorial 25
add_test(NAME Runs COMMAND Tutorial_5 25)

# TODO 7: Add a test called Usage which runs the following command:
# $ Tutorial
# Make sure the expected output is displayed.
# Hint: Use the PASS_REGULAR_EXPRESSION property with "Usage.*number"
add_test(NAME Usage COMMAND Tutorial_5)
set_tests_properties(Usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

# TODO 8: Add a test which runs the following command:
# $ Tutorial 4
# Make sure the result is correct.
# Hint: Use the PASS_REGULAR_EXPRESSION property with "4 is 2"
add_test(NAME StandardUse COMMAND Tutorial_5 4)
set_tests_properties(StandardUse PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2")
# TODO 9: Add more tests. Create a function called do_test to avoid copy +
# paste. Test the following values: 4, 9, 5, 7, 25, -25 and 0.0001.
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endfunction(do_test target arg result)

# do a bunch of result based tests
do_test(Tutorial_5 4 "4 is 2")
do_test(Tutorial_5 9 "9 is 3")
do_test(Tutorial_5 5 "5 is 2.236")
do_test(Tutorial_5 7 "7 is 2.645")
do_test(Tutorial_5 25 "25 is 5")
do_test(Tutorial_5 -25 "-25 is (-nan|nan|0)")
do_test(Tutorial_5 0.0001 "0.0001 is 0.01")