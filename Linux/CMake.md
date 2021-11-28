# Cmake

- cmake 下载地址 http://www.cmake.org/download/

- cmake简介： 它允许开发者编写一种平台无关的CMakeLists.txt文件来定制整个编译流程，然后再根据目标用户的平台 进一步生成所需的本地化的Makefile和工程文件，如Unix的Makefile和Windows的visual studio工程。

  ## 简单的cmake的设置

  ### 项目中只有一个源文件main.cpp：

- 编写CMakeLists.txt文件：

  ```c++
  CMAKE_MINIMUM_REQUIRED (VERSION 2.8)  # CMake 最低版本号要求 
  PROJECT (Demo1)                       # 项目信息
  ADD_EXECUTABLE(Demo main.cc)          # 指定生成目标
  ```

- 在当前目录创建build目录，在build目录下执行cmake .. ,得到Makefile后使用make命令得到Demo1可执行文件。

  ### 多个目录工程：

- 目录结构: ```c++ |-- CMakeLists.txt |-- build |-- include | |-- function.h
  |-- main.cpp

```
* CMakeLists.txt:
```c++
PROJECT (Hello_World)
CMAKE_MINIMUM_REQUIRED (VERSION 3.1)
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/include)            #添加头文件路径
ADD_EXECUTABLE(Demo main.cpp)
```

- 在build目录下生成可执行文件

  ### 生成静态库，链接库

- 目录结构： ```c++ |-- CMakeLists.txt |-- build |-- include |-- lib |-- include | |-- function.hpp
  |-- src | |-- function.cpp | |-- CMakeLists.txt |-- main.cpp

```
* 根目录中的CMakeLists.txt:
```c++
PROJECT (Demo3)
CMAKE_MINIMUM_REQUIRED (VERSION 3.1)
ADD_SUBDIRECTORY(src)                                      #添加lib子目录
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/include)   #添加头文件路径
AUX_SOURCE_DIRECTORY(${CMAKE_SOURCE_DIR}/src SRC)
ADD_EXECUTABLE(Demo main.cpp ${SRC})
LINK_DIRECTORIES(${PROJECT_SOURCE_DIR}/lib)
TARGET_LINK_LIBRARIES(Demo lib)
```

- 子目录中的CMakeLists.txt:

  ```c++
  AUX_SOURCE_DIRECTORY(.SRC)
  INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/include) 
  SET(LIBRARY_OUT_PATH ${PROJECT_SOURCE_DIR}/lib)
  ADD_LIBRATY(lib ${SRC})
  ```

- 在build目录下生成可执行文件

  ## CMake 常用变量和语句

- INCLUDE_DIRECTORIES 指定头文件搜索路径

- LINK_DIRECTORIES 指定库文件搜索路径

- ADD_SUBDIRECTORY 添加子目录

- TARGET_LINK_LIBRARIES 指定文件链接库文件

  ## 指定编译器和编译选项

- CMAKE_CXX_COMPILTER 指定C++编译器

- CMAKE_CXX_FLAGS 指定C++编译选项

- CMAKE_CXX_FLAGS_DEBUG 是除了CMAKE_CXX_FLAGS外，在Debug配置下，额外的参数

- CMAKE_CXX_FLAGS_RELEASE 是除了CMAKE_CXX_FLAGS外，在Release配置下，额外的参数

由于工程项目中各个模块所采用的的编译选项可能存在差异，因此每个模块要配置配置编译选项， 工程为例：

- 目录结构：

  ```c++
  |-- CMakeLists.txt
  |-- build
  |-- include
  |-- lib
  |   |--lib.hpp
  |   |--lib.cpp
  |   |--CMakeLists.txt
  |-- include
  |   |-- hello.hpp       
  |-- src 
  |   |-- hello.cpp 
  `-- main.cpp
  ```

- 根目录中的CMakeLists.txt:

  ```c++
  PROJECT (Demo3)
  CMAKE_MINIMUM_REQUIRED (VERSION 3.1)
  ADD_SUBDIRECTORY(src)                                      #添加lib子目录
  INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/include)   #添加头文件路径
  AUX_SOURCE_DIRECTORY(${CMAKE_SOURCE_DIR}/src SRC)
  ADD_EXECUTABLE(Demo main.cpp ${SRC})
  LINK_DIRECTORIES(${PROJECT_SOURCE_DIR}/lib)
  TARGET_LINK_LIBRARIES(Demo lib)
  SET(CMAKE_CXX_COMPILER      "clang++" )         # 显示指定使用的C++编译器
  SET(CMAKE_CXX_FLAGS   "-std=c++11")             # c++11
  SET(CMAKE_CXX_FLAGS   "-g")                     # 调试信息
  SET(CMAKE_CXX_FLAGS   "-Wall")                  # 开启所有警告
  SET(CMAKE_CXX_FLAGS_DEBUG   "-O0" )             # 调试包不优化
  SET(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG " )   # release包优化
  ```

- 子目录中的CMakeLists.txt: ```c++

  # 在此链接库下编译时可能不需要调试信息

AUX_SOURCE_DIRECTORY(. DIR_LIB_SRCS) ADD_LIBRARY (lib ${DIR_LIB_SRCS}) SET(CMAKE_CXX_COMPILER "clang++" ) # 显示指定使用的C++编译器 SET(CMAKE_CXX_FLAGS "-std=c++11") # c++11 SET(CMAKE_CXX_FLAGS "-Wall") # 开启所有警告 SET(CMAKE_CXX_FLAGS_DEBUG "-O0" ) # 调试包不优化 SET(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG " ) # release包优化 ```

- 在build目录下生成可执行文件