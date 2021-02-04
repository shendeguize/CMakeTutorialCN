# CMake教程
Update: 2021.02.03

Translator: [shendeguize@github](
https://github.com/shendeguize)

Origin: [https://cmake.org/cmake/help/latest/guide/tutorial/index.html](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

Version: 3.19.4

Link: [https://github.com/shendeguize/CMakeTutorialCN](https://github.com/shendeguize/CMakeTutorialCN)

本翻译囿于水平，可能有不准确的地方，欢迎指出，谢谢大家

*如有引用,请注明出处*

## 目录
- [CMake教程](#cmake教程)
  - [目录](#目录)
  - [介绍](#介绍)
  - [Step1: 一个基本出发点](#step1-一个基本出发点)
    - [添加版本号 & 配置头文件](#添加版本号--配置头文件)
    - [指定C++标准](#指定c标准)
    - [构建与测试](#构建与测试)


## 介绍
这份渐进式的CMake教程覆盖了构建系统时CMake来处理的一些常见的问题.在一个样例项目中来探讨不同主题是如何结合应用是非常有助于理解的.教程文档和源码可以从CMake源路径下的`Help/guide/tutorial`文件夹中获得(译者注:已同步至当前Git).每一步都有其独立的子目录,这些子目录也包含可以作为出发点的代码.教程样例是循序渐进的,每一步都提供了前一步的完整解决方案.

## Step1: 一个基本出发点
最基础的项目是基于源代码的一个可执行构建.对于简单项目.三行的`CMakeLists.txt`就满足了全部需要的内容.这就是这篇教程的开始点.在`Step1`路径下创建一个`CMakeLists.txt`文件如下:  
```CMake
cmake_minimum_required(VERSION 3.10)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial tutorial.cxx)
```

注意在`CMakeLists.txt`文件中的命令都使用了小写.CMake支持大小写混用命令.`tutorial.cxx`的源代码在`Step1`文件夹下,可用以计算平方根.

### 添加版本号 & 配置头文件
首个添加的特性是给我们的项目和可执行文件提供版本号.尽管我们可以在源代码中添加版本号,但是使用`CMakeLists.txt`是更灵活的方式.

首先,修改`CMakeLists.txt`,使用[`project()`](https://cmake.org/cmake/help/latest/command/project.html#command:project)命令来设定项目名和版本号.

```CMake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)
```

然后制定一个头文件来将版本号传递到源码里:

```CMake
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

因为指定的文件会被写入二进制结构中,我们必须将这一目录添加到搜索include文件的列表中.在`CMakeLists.txt`文件结尾写入:

```CMake
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

使用你喜欢的ide,在源路径下创建`TutorialConfig.h.in`,并写入下述内容:

```
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当CMake生成这个头文件时,`@Tutorial_VERSION_MAJOR@`和`@Tutorial_VERSION_MINOR@`的值会被自定替换.

接下来调整`tutorial.cxx`来包含头文件`TutorialConfig.h`.

最后,更新`tutorial.cxx`如下以打印可执行文件名和版本号:

```C
    if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
                << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
    }
```

### 指定C++标准
接下来,我们通过在`tutorial.cxx`中将`atof`替换为`std::stod`来给我们的项目增加一些C++11特性.同时,移除`#include <cstdlib>`.

```C++
const double inputValue = std::stod(argv[1]);
```

我们需要在CMake代码中显式地声明以使用正确的配置.最简单的方式是在CMake中通过使用`CMAKE_CXX_STANDARD`以启用对特定版本C++标准的支持.对于本篇教程.将`CMakeLists.txt`中的`CMAKE_CXX_STANDARD`设为11,`CMAKE_CXX_STANDARD_REQUIRED`设为`True`.并将`CMAKE_CXX_STANDARD`声明置于`add_executable`前.

```CMake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

### 构建与测试
运行`cmake`可执行文件,或者`cmake-gui`来配置项目,然后使用所选的构建工具来构建它.

例如,从命令行中,我们要进入`Help/guide/tutorial`目录下并建立一个build目录:

```Shell
mkdir Step1_build
```

之后,进入build目录,然后运行CMake来配置项目,并生成原生构建系统:

```Shell
cd Step1_build
cmake ../Step1
```

然后调用这个构建系统来实际编译/链接项目:

```Shell
cmake --build .
```

最后,尝试用下述命令来使用新构建的`Tutorial`:

```
Tutorial 4294967296
Tutorial 10
Tutorial
```


