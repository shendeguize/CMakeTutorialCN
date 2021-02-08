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
  - [Step2: 添加库](#step2-添加库)
  - [Step3: 对库添加使用依赖](#step3-对库添加使用依赖)
  - [Step4: 安装与测试](#step4-安装与测试)
    - [安装规则](#安装规则)
    - [测试支持](#测试支持)
  - [Step5: 增加系统自检](#step5-增加系统自检)


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

## Step2: 添加库
现在我们会向我们的项目中添加一个库.这个库会包含我们计算数字平方根的实现.可执行文件就可以使用库而非编译器提供的平方根函数.

本篇教程里,我们会把库放在一个叫做`MathFunctions`的子文件夹下.这个目录已经包含了一个头文件`MathFunctions.h`,也包含了一个源文件`mysqrt.cxx`. 源文件中包含一个名为`mysqrt`的函数,提供了编译器中`sqrt`相近功能.

把这一行加入`MathFunctions`文件夹的`CMakeLists.txt`中:

```CMake
add_library(MathFunctions mysqrt.cxx)
```

为了使用新的库,我们在顶级的`CMakeLists.txt`中加入`add_subdirectory()`来构建库.我们向可执行文件加入新的库,并将`MathFunctions`添加为包含目录,这样就可以查询得到`mqsqrt.h`头文件了.顶级`CMakeLists.txt`的最后几行应该如下:

```CMake
# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC MathFunctions)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```

接下来我们让MathFunctions库可以作为可选项.尽管本次教程不需要这样,但大型项目中这很常见.第一步是在顶层`CMakeLists.txt`中增加选项:

```CMake
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

这一选项会在`cmake-gui`或`ccmake`中显示,默认值为ON,也可被用户修改.这一选项会被存储在缓存中,用户无需每次都设定.

下一项更改是将MathFunctions库的构建和链接设定成可选的.我们在顶级`CMakeLists.txt`的结尾做如下修改:

```CMake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ${EXTRA_INCLUDES}
                           )
```

变量`EXTRA_LIBS`手机了之后可以在可执行文件中链接的可选库.变量`EXTRA_INCLUDES`也相应的用于收集可选的头文件.在处理很多可选项时,这是一种经典的处理方式.下一步我们会用新方式来做.

相应的源代码改动就比较直接了.首先,在`tutorial.cxx`中,如果需要则包含`MathFunctions.h`:

```C++
#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif
```

然后在同一个文件中,让`USE_MYMATH`变量控制函数的选择:

```C++
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```

因为现在源代码中需要`USE_MYMATH`变量,我们可以在`TutorialConfig.h.in`文件中加入下述这行:

```
#cmakedefine USE_MYMATH
```

**练习**: 为什么我们在选项`USE_MYMATH`后配置`TutorialConfig.h.in`.如果我们把这两条交换会发生什么.

运行`cmake`或者`cmake-gui`来配置项目,然后构建,在运行构建出的可执行文件.

现在让我们更新`USE_MYMATH`的值.最简单的方式是使用`cmake-gui`或终端中的`ccmake`.或者如果想在命令行中修改这一选项:

```
cmake ../Step2 -DUSE_MYMATH=OFF
```

重新构建然后运行.

那个函数结果更好,sqrt还是mysqrt?

## Step3: 对库添加使用依赖

使用依赖运行了对于库或者可执行文件的链接和包含项更好的控制.也提供了对CMake内的可及属性的更充分的控制.控制使用依赖的首要命令包括:
+ [target_compile_definitions](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions)
+ [target_compile_options](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options)
+ [target_include_directories](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories)
+ [target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries)

让我们用现代CMake的方式重构Step2中的使用依赖的部分. 我们首先明确任何链接到MathFunctions的对象都需要包含当前原目录,除了MathFunctions本身.所以这可以作为一个`INTERFACE`使用依赖.

记住`INTERFACE`指的是那些消费者需要而生产者不需要的东西.在`MathFunctions/CMakeLists.txt`的结尾加入:

```CMake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )
```

现在我们已经指定了MathFunctions的使用依赖,我们就可以安全地移除顶级`CMakeLists.txt`文件中的`EXTRA_INCLUDES`变量:

```
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()
```

以及:

```
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

完成后,运行`cmake`或者`cmake-gui`来配置项目并通过在build目录下`cmake --build .`构建运行即可.

## Step4: 安装与测试
现在我们开始给项目添加安装规则和测试支持.

### 安装规则
安装规则非常简单: 对于MathFunctions,我们希望安装库和头文件,对于应用,我们希望安装可执行文件和配置头.

所以在`MathFunctions/CMakeLists.txt`的结尾我们添加:

```CMake
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

在顶层`CMakeLists.txt`的结尾添加:

```
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```

这就是建立一个tutorial的基础本地安装所需要的全部内容.

现在我们运行`cmake`或者`cmake-gui`来配置项目并构建.

然后通过在命令行中运行`cmake`的`install`选项来执行安装步骤(3.15引入,早先版本必须使用make install).对于多配制工具,要记得使用`--config`来指定配置.如果使用IDE,直接构建`INSTALL`目标即可.这一步会安装适合的头文件,库和可执行文件:

```
camke --install .
```

CMake变量`CMAKE_INSTALL_PREFIX`用于确定文件安装的根目录.如果使用`cmake --install`命令,安装前驻可以被`--prefix`参数覆写:

```
cmake --install . --prefix "/home/myuser/installdir"
```

浏览安装目录然后验证安装的Tutorial可以运行.

### 测试支持

接下来让我们测试我们的应用,在顶级`CMakeLists.txt`的结尾,我们可以打开测试功能然后加一些基本测试来验证安装正确.

```CMake
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

第一个测试简单验证了应用运行,没有段错误或其他崩溃发生.并有一个0返回值.这是CTest测试的基本格式.

下一个测试使用了`PASS_REGULAR_EXPRESSION`测试属性来验证测试输出包含特定字符串.用于在参数输入数量不对时打印使用信息.

最后,我们有一个叫做`do_test`的函数来运行应用并验证计算平方根的结果对于给定输出是正确的.对于没错`do_test`的唤起,项目中就会被加入一个带有明早,输入和期待结果的测试.

重新构建应用然后进入二进制目录并运行`ctest`可执行文件:`ctest -N`和`ctest -VV`(译者注:注意是两个V).对于多配制生成器(例如Visual Studio),配置类型必须要指定.为了在DEbug模式下运行测试,例如在构建目录下(而非Debug目录下)使用`ctest -C Debug -VV`.或者在IDE中构建`RUN_TESTS`目标.

## Step5: 增加系统自检

现在我们想项目中增加一些代码,而这些代码依赖的特性可能是目标平台没有的.例如,我们要加入的代码依赖于目标平台是否有`log`和`exp`函数.当然,对于每个平台而言,这些功能都是有的,但是在本篇教程中,我们假定这些功能不是都存在的.

如果平台有`log`和`exp`那么我们就在`mysqrt`函数里使用.我们首先在顶层`CMakeLists.txt`里用`CheckSymbolExists`来测试这些函数的可用性.在一些平台,我们会需要连接到`m`库如果`log`和`exp`没有被招到,就需要在`m`库里再试试.

```CMake
include(CheckSymbolExists)
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
if(NOT (HAVE_LOG AND HAVE_EXP))
  unset(HAVE_LOG CACHE)
  unset(HAVE_EXP CACHE)
  set(CMAKE_REQUIRED_LIBRARIES "m")
  check_symbol_exists(log "math.h" HAVE_LOG)
  check_symbol_exists(exp "math.h" HAVE_EXP)
  if(HAVE_LOG AND HAVE_EXP)
    target_link_libraries(MathFunctions PRIVATE m)
  endif()
endif()
```

现在让我们给`TutorialConfig.h.in`添加一些定义,这样我们就可以在`mysqrt.cxx`里使用了:

```
// does the platform provide exp and log functions?
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```

如果`log`和`exp`在系统上可用,那么我们就在`mysqrt`里使用它们.在`MathFunctions/mysqrt.cxx`里的`mysqrt`里添加下述代码(别忘了返回值之前加`#endif`):

```C++
#if defined(HAVE_LOG) && defined(HAVE_EXP)
  double result = exp(log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  double result = x;
```

我们也需要修改`mysqrt.cxx`来包含`cmath`:

```C++
#include <cmath>
```

运行`cmake`或者`cmake-gui`来配置项目,然后构建并执行Tutorial.

会注意到我们没有使用`log`和`exp`,即使我们认为它们应该是可用的.我们应该很容易发现,我们忘记在`mysqrt.cxx`中包含`TutorialConfig.h`了.

我们也需要更新`MathFunctions/CMakeLists.txt`,这样`mysqrt.cxx`才能够定位文件:

```CMake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${CMAKE_BINARY_DIR}
          )
```

这样更新后,继续并构建项目,然后运行Tutorial.如果`log`和`exp`仍然没有被使用,打开构建目录下的生成的`Tutorial.h`文件,可能他们在当前系统下不可用的.

那个函数目前结果更好呢,sqrt还是mysqrt?

