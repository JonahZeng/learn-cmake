add_library(MathFunctions_5 STATIC MathFunctions.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions_5
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if (USE_MYMATH)
  target_compile_definitions(MathFunctions_5 PRIVATE "USE_MYMATH")

  # library that just does sqrt
  add_library(SqrtLibrary_5 STATIC
              mysqrt.cxx
              )

  # link SqrtLibrary to tutorial_compiler_flags
  target_link_libraries(SqrtLibrary_5 PUBLIC tutorial_compiler_flags_5)

  target_link_libraries(MathFunctions_5 PRIVATE SqrtLibrary_5)
endif()

# link MathFunctions to tutorial_compiler_flags
target_link_libraries(MathFunctions_5 PUBLIC tutorial_compiler_flags_5)

# TODO 1: Create a variable called installable_libs that is a list of all
# libraries we want to install (e.g. MathFunctions and tutorial_compiler_flags)
# Then install the installable libraries to the lib folder.
# Hint: Use the TARGETS and DESTINATION parameters
set(installable_libs MathFunctions_5 tutorial_compiler_flags_5)
if(TARGET SqrtLibrary_5)
  list(APPEND installable_libs SqrtLibrary_5)
endif()
install(TARGETS ${installable_libs} DESTINATION lib)

# TODO 2: Install the library headers to the include folder.
# Hint: Use the FILES and DESTINATION parameters
install(FILES MathFunctions.h DESTINATION include)