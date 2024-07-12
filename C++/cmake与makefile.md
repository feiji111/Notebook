# CMake

# 1. 使用CMake过程中的一些问题记录



## 1.1 -std=c++xx选项与-std=gnu++xx的区别

参考：

- [What are the differences between -std=c++11 and -std=gnu++11?](https://stackoverflow.com/questions/10613126/what-are-the-differences-between-std-c11-and-std-gnu11)
- 



这二者的区别就在于是否支持**GNU extensions**，关于GNU extensions，参考[这里](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Extensions.html)。



## 1.2 CMAKE_BUILD_TYPE选项中的Debug，Release，RelWithDebInfo和MinSizeRel的区别

参考：

- [What are CMAKE_BUILD_TYPE: Debug, Release, RelWithDebInfo and MinSizeRel?](https://stackoverflow.com/questions/48754619/what-are-cmake-build-type-debug-release-relwithdebinfo-and-minsizerel)
- da



## 1.3 在CMakeList.txt中引入OpenCV的总结

在CMake中引入OpenCV有几种方式

```cmake
find_package(OpenCV REQUIRED)

# If the package has been found, several variables will
# be set, you can find the full list with descriptions
# in the OpenCVConfig.cmake file.
# Print some message showing some of them
message(STATUS "OpenCV library status:")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBS}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")

# Add OpenCV headers location to your include paths
include_directories(${OpenCV_INCLUDE_DIRS})

# Declare the executable target built from your sources
add_executable(main main.cpp)

# Link your application with OpenCV libraries
target_link_libraries(main ${OpenCV_LIBS})
```

在第一条`find_package(OpenCV REQUIRED)`成功后，CMake会**自动**设置这几个变量：

- `OpenCV_VERSION`
- `OpenCV_LIBS`
- `OpenCV_INCLUDE_DIRS`



这里涉及到`find_package`的工作原理，参考[这里](#2.2.1-find_package)。



## 1.4 交叉编译



## 1.5 CMakeCache.txt



## 1.6 在CMake与Makefile中指定C和C++标准的版本

有三种方法：

- 直接编辑Makefile，添加选项`CFLAGS += -std=c++11`
- cmake命令行参数(或者在cmake-gui中添加相应参数)，`cmake -DCMAKE_CXX_STANDARD=11 ..`
- 在CMakeLists.txt中，添加这一行`set(CMAKE_CXX_STANDARD 11)`

`set(CMAKE_CXX_STANDARD 11)`一般会与`set(CMAKE_CXX_STANDARD_REQUIRED ON)`配套使用，`set(CMAKE_CXX_STANDARD_REQUIRED ON)`的作用在于当CMake尝试确定可用的C++标准时，如果编译器不支持你指定的C++标准（在这个例子中是C++11），CMake在配置（configure）过程中会报错，而不是默默地降级到编译器所支持的最高C++标准。这就确保了你的代码一定会按照你指定的C++版本进行编译，而不会因为编译器的限制而使用更低的C++版本，这对于需要利用比C++11更高版本C++特性的项目来说非常重要。



## 1.7 CMake变量



## 1.8 头文件目录





# 2. CMake Manual

CMake的主要目的就是为了能够跨平台。不同的Make工具(比如GNU Make，qmake，nmakepmake等等)往往有着不同的规范和标准，因此Makefile的格式也有着区别，因此在不同的Make工具平台，就得专门为其编写相应的Makefile，所以CMake旨在通过编写一种平台无关的CMakeList.txt文件，然后根据相应的平台生成本地化的Makefile，解决跨平台的问题。

## 2.1 cmake-variables

## 2.2. cmake-commands

### 2.2.1 find_package

官方文档的参考，参考[这里](https://cmake.org/cmake/help/latest/command/find_package.html)。

`find_package`函数用于`Find a package (usually provided by something external to the project), and load its package-specific details`。

在这个函数寻找packages，有三种模式：

- **Module mode**
- **Config mode** Config mode下，CMake会寻找`<lowercasePackageName>-config.cmake`或者`<PackageName>Config.cmake`的文件，如果指定了具体的版本，也会寻找`<lowercasePackageName>-config-version.cmake`或者`<PackageName>ConfigVersion.cmake`
- **FetchContent redirection mode**

在不指定具体模式的情况下，会先采用Module mode，然后在采用Config mode。如果要指定具体的模式，可以通过`CONFIG`或者`MODULE`参数来指定。

基本的格式(Basic SIgnature)

```cmake
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [REGISTRY_VIEW  (64|32|64_32|32_64|HOST|TARGET|BOTH)]
             [GLOBAL]
             [NO_POLICY_SCOPE]
             [BYPASS_PROVIDER])
```

完整的格式(Full Signature)

```cmake
find_package(<PackageName> [version] [EXACT] [QUIET]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [CONFIG|NO_MODULE]
             [GLOBAL]
             [NO_POLICY_SCOPE]
             [BYPASS_PROVIDER]
             [NAMES name1 [name2 ...]]
             [CONFIGS config1 [config2 ...]]
             [HINTS path1 [path2 ... ]]
             [PATHS path1 [path2 ... ]]
             [REGISTRY_VIEW  (64|32|64_32|32_64|HOST|TARGET|BOTH)]
             [PATH_SUFFIXES suffix1 [suffix2 ...]]
             [NO_DEFAULT_PATH]
             [NO_PACKAGE_ROOT_PATH]
             [NO_CMAKE_PATH]
             [NO_CMAKE_ENVIRONMENT_PATH]
             [NO_SYSTEM_ENVIRONMENT_PATH]
             [NO_CMAKE_PACKAGE_REGISTRY]
             [NO_CMAKE_BUILDS_PATH] # Deprecated; does nothing.
             [NO_CMAKE_SYSTEM_PATH]
             [NO_CMAKE_INSTALL_PREFIX]
             [NO_CMAKE_SYSTEM_PACKAGE_REGISTRY]
             [CMAKE_FIND_ROOT_PATH_BOTH |
              ONLY_CMAKE_FIND_ROOT_PATH |
              NO_CMAKE_FIND_ROOT_PATH])
```



**路径寻找的过程**

对于Module mode下，首先会在变量[`CMAKE_MODULE_PATH`](https://cmake.org/cmake/help/latest/variable/CMAKE_MODULE_PATH.html#variable:CMAKE_MODULE_PATH)中列出的路径中寻找，然后在[Find Modules](https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html#find-modules)中寻找。



对于Config mode下，寻找的过程更加复杂。

