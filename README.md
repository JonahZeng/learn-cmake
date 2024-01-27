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
- $<0:...> 逻辑false产生空字符串
- $<1:...> 逻辑ture则等于后面的...字符串
