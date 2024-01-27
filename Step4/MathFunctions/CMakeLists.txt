# create the MathFunctions library
add_library(MathFunctions_4 STATIC MathFunctions.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions_4
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions_4 PRIVATE "USE_MYMATH")

  # library that just does sqrt
  add_library(SqrtLibrary_4 STATIC
              mysqrt.cxx
              )

  # link SqrtLibrary to tutorial_compiler_flags
  target_link_libraries(SqrtLibrary_4 PUBLIC tutorial_compiler_flags_4)

  target_link_libraries(MathFunctions_4 PRIVATE SqrtLibrary_4)
endif()

# link MathFunctions to tutorial_compiler_flags
target_link_libraries(MathFunctions_4 PUBLIC tutorial_compiler_flags_4)
