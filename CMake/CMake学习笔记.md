## CMake学习笔记

### 基本的CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.10)
# set the project name
project(Tutorial)
# add the executable
add_exectable(Tutorial tutorial.cxx) 
```

> 同时支持英文的大小写,  * tutorial.cxx * 为源代码

#### 添加版本号和设置头文件

```
cmake_minimum_required(VERSION 3.10)

# set the pooject name and version
project(Tutorial VERSION 1.0)

```

配置头文件，将版本号传递给源代码

``` 
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

由于配置文件会被写入二进制文件目录，因此我们必须将该目录也加入到 *include* 目录中。

```
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```

#### 创建TutorialConfig.h.in

```
//Tutorial 的配置项
#define Tutorial_VERISON_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERISON_MINOR @Tutorial_VERSION_MINOR@
```

> 在Cmake 配置时，会将@Tutorial_VERSION_MAJOR@，@Tutorial_VERSION_MINOR@ 替换

#### 在源文件中包含 ***TutorialConfig.h***

可以在源文件中直接使用 Tutorial_VERISON_MAJOR，Tutorial_VERISON_MINOR

#### 指定 C++ 标准

使用 `CMAKE_CXX_STANDARD` 指定C++标准，使用`CMAKE_CXX_STANDARD_REQUIRED` 开启C++支持。确保`CMAKE_CXX_STANDARD` 声明在 `add_execuable` 之前。

```
cmake_minumum_required(VERSION 3.10)

project(Tutorial VERISON 1.0)

#指定 C++ 标准

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

```

 

#### 构建和测试

命令行构建

- 定位到源代码目录。创建 `build`目录

  ```
  mkdir build
  ```

- 进入 `build` 目录，执行

  ```
  cmake ../
  ```

- 编译和链接

   ```
  cmake --build .
   ```

### 添加库

假设库的头文件和实现文件在 `Lib_Dir`中，`lib.cxx`为库的实现。

1. 在 `Lib_Dir` 中创建 `CmakeLists.txt` 文件并添加：

      ``` 
   add_library(Lib_Dir lib.cxx)
      ```

2. 使用新的库我们需要在顶层的 `CMakelists.txt` 中添加：

   ```
   add_subdirectory(Lib_Dir)
   add_exectuable(Tutorial tutorial.cxx)
   
   target_link_libraries(Tutorial PUBLIC Lib_Dir)
   
   #将库的头文件路径添加 到include 
   target_include_directories( Tutorial PUBLIC
   							"${PORJRCT_BINARY_DIR}"
   							"${PROJECT_SOURCE_DIR}/Lib_dir"
   							)
   ```

3. 设置是否使用指定库

   在顶层的 `CMakeLists.txt` 中设置

   ```
   option(USE_LIB "Use lib_name" ON)
   
   #一定要在option之后
   configure_file(TutorialConfig.h.in TutorialConfig.h)
   
   if(USE_LIB)
   	add_subdirectory(Lib_Dir)
   	list(APPEND EXTRA_LIBS Lib_Dir)
   	list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/Lib_Dir")
   endif()
   
   add_exectuable(Tutorial tutorial.cxx)
   
   target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})
   target_include_directories( Tutorial PUBLIC
   							"${PORJRCT_BINARY_DIR}"
   							"${EXTRA_INCLUDES}"
   							)
   ```

   > `EXTRA_LIBS`,`EXTRA_INCLUDES`指可选的库和头文件

   同时，要修改源文件。条件包含相应的头文件。条件使用库里的函数。例如：

   ```
   #ifdef USE_LIB
   #include "lib.h"
   #endif
   ....
   #ifdef USE_LIB
   	//使用相应的库函数
   #else
   	...
   #endif
   ```

   为何使源文件能够找到对应的 `USE_LIB` 宏。需要修改 `TutorialConfig.h.in`:

   ```
   #cmakedefine USE_LIB
   ```

   ***注意***:可以在配置时，动态指定 `option` 的值；

   ```
   cmake ../ -DUSE_LIB=OFF
   ```

### 添加库的用法要求

更好的控制库或者可执行文件的链接和`include`.同时，更好控制 Cmake 的过度属性。

主要的命令如下：

- `target_compile_definitons()`
- `target_compile_options()`
- `target_include_directories()`
- `target_link_libraries()`

