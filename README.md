# 关于这个教程

这个入门教程是cmake官方step by step入门教程，版本3.28.1，[链接](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

This directory contains source code examples for the CMake Tutorial.
Each step has its own subdirectory containing code that may be used as a
starting point. The tutorial examples are progressive so that each step
provides the complete solution for the previous step.

## step1 A Basic Starting Point
- 掌握configure_file使用
- 掌握.in配置文件书写
- add_executable用法
- target_include_directories用法
- 设置编译标准CXX_STANDARD

## step2 Adding a Library
- add_library用法
- target_link_libraries用法
- option用法
- target_compile_definitions用法

## step3 Adding Usage Requirements for a Library
- CMAKE_CURRENT_SOURCE_DIR 用法如下；
- target_include_directories(target INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}), Remember INTERFACE means things that consumers require but the producer doesn't.
- target_compile_features(target INTERFACE cxx_std_11)作为link接口，要求consumer使用这个feature但producer不要求

## step4 Adding Generator Expressions
- $\<0:...> 逻辑false产生空字符串
- $\<1:...> 逻辑ture则等于后面的...字符串

## step5 Installing and Testing
- install(TARGETS ...)
- install(FILES ...)
- enable_testing()
- add_test(NAME ... COMMAND ...)
- set_test_properties(name PROPERTIES PASS_REGULAR_EXPRESSION ...)
- function(name arg0 arg1 ...)

## step6 Adding Support for a Testing Dashboard
- 编写CTestConfig.cmake
- remove enable_test()
- include(CTest)

## step7 Adding System Introspection

- include(CheckCXXSourceCompiles)

- ```cmake
  check_cxx_source_compiles("
      #include <cmath>
      int main() {
        std::log(1.0);
        return 0;
      }
    " HAVE_LOG)
  ```

## step8 Adding a Custom Command and Generated File

- ```cmake
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    DEPENDS MakeTable
    )
  ```

## step9 Packaging an Installer

- top cmakelists.txt include blow code

```cmake
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_SOURCE_GENERATOR "TGZ")
include(CPack)
```

- run command

```cmake
cpack -G TGZ -C Debug
```

## Step 10 Selecting Static or Shared Libraries

- [`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS) 变量控制add_library()的默认行为，所以只需要在顶层cmakelists.txt中定义这个变量值即可控制所有的子工程lib目标类型
- 对于windows系统来说，如果生成dll, 还需要在头文件中指明导出函数

```c++
#if defined(_WIN32)
#  if defined(EXPORTING_MYMATH)
#    define DECLSPEC __declspec(dllexport)
#  else
#    define DECLSPEC __declspec(dllimport)
#  endif
#else // non windows
#  define DECLSPEC
#endif

namespace mathfunctions {
double DECLSPEC sqrt(double x);
}
```

编译本dll的时候使用\__declspec(dllexport)，使用dll导入的时候使用__declspec(dllimport)

- 对于linux系统来说，还需要注意[`POSITION_INDEPENDENT_CODE`](https://cmake.org/cmake/help/latest/prop_tgt/POSITION_INDEPENDENT_CODE.html#prop_tgt:POSITION_INDEPENDENT_CODE), 也就是gcc经常用的-fPIC选项

```cm
set_target_properties(SqrtLibrary PROPERTIES
                        POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                        )
```

## Step 11 Adding Export Configuration

为了让我们的工程能够被find_package()识别使用，需要导出必要的cmake配置文件：

- 在install目标语句中增加EXPORT目标

```cmake
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs}
        EXPORT MathFunctionsTargets # 导出MathFunctionsTargets
        DESTINATION lib)
# install include headers
install(FILES MathFunctions.h DESTINATION include)
```

- 上层cmakelists.txt install增加.cmake安装

```cmake
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake # 自动生成MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)

include(CMakePackageConfigHelpers)
```

- 头文件目录对install修改为相对目录

```cmake
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include> # 如果是安装后导入，使用相对目录
                           )
```

- 生成xxxConfig.cmake供find_package()识别

  1. 新建一个Config.cmake.in文件
  2. 使用configure_package_config_file

  ```cmake
  @PACKAGE_INIT@
  
  include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
  # Config.cmake.in
  ######################################################################
  # 顶层cmakelists.txt接
  include(CMakePackageConfigHelpers)
   generate the config file that includes the exports
  configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
    INSTALL_DESTINATION "lib/cmake/example"
    NO_SET_AND_CHECK_MACRO
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
    )
  ```

- 生成xxxConfigVersion.cmake

```cmake
# 顶层cmakelists.txt接
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)
```

- 安装xxxConfig.cmake和xxxConfigVersion.cmake

```cmake
# 顶层cmakelists.txt接
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
  DESTINATION lib/cmake/MathFunctions
  )
```

至此完成；

## Step 12 Packaging Debug and Release

**note**: 仅对unix makefile等single-configuration类型生效，对于multi-configuration比如ninja-multi, visual studio等类型不生效；

首先需要让release和debug生成目标文件使用不同的文件名，通常是添加后缀d表示debug；这里需要用到变量`CMAKE_DEBUG_POSTFIX`;

```CMAKE
set(CMAKE_DEBUG_POSTFIX d)

add_library(tutorial_compiler_flags INTERFACE)
```

首先在top-level cmakelists.txt中添加如上定义；

然后，给target添加DEBUG_POSTFIX属性：

```cmake
add_executable(Tutorial tutorial.cxx)
set_target_properties(Tutorial PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})

target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)
# 添加版本号
set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")
set_property(TARGET MathFunctions PROPERTY SOVERSION "1")
```

新建release, debug目录分别生成，编译，安装：

```bash
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

新建一个MultiCPackConfig.cmake文件，

```cmake
include("release/CPackConfig.cmake")

set(CPACK_INSTALL_CMAKE_PROJECTS
    "debug;Tutorial;ALL;/"
    "release;Tutorial;ALL;/"
    )
```

接下来，执行install以后cpack

```bash
cpack --config MultiCPackConfig.cmake
```

