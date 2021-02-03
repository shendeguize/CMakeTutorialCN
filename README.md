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



