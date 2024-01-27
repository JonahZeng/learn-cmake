add_library(MathFunctions_3 STATIC MathFunctions.cxx)

# TODO 1: State that anybody linking to MathFunctions needs to include the
# current source directory, while MathFunctions itself doesn't.
# Hint: Use target_include_directories with the INTERFACE keyword
target_include_directories(MathFunctions_3 INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions_3 PRIVATE "USE_MYMATH")

  # library that just does sqrt
  add_library(SqrtLibrary_3 STATIC
              mysqrt.cxx
              )
  target_link_libraries(SqrtLibrary_3 PUBLIC tutorial_compiler_flags)

  # TODO 7: Link SqrtLibrary to tutorial_compiler_flags

  target_link_libraries(MathFunctions_3 PRIVATE SqrtLibrary_3)
  target_link_libraries(MathFunctions_3 PUBLIC tutorial_compiler_flags)
endif()

# TODO 6: Link MathFunctions to tutorial_compiler_flags
