# 基本配置项
moc ui 和 rcc编译开关
SET(CMAKE_AUTOMOC ON)
SET(CMAKE_AUTOUIC ON)
SET(CMAKE_AUTORCC ON)

# 启动C++11标准
SET(CMAKE_CXX_STANDARD 11)

# 包含所有.h文件
* 有些只编写了.h文件，例如常量声明，结构体声明等，请务必配置这一项
SET(CMAKE_INCLUDE_CURRENT_DIR  ON)

# 查找Qt模块
* 首先是CMAKE_PREFIX_PATH，对应各个模块的cmake文件路径，其次FIND_PACKAGE才能生效
SET(CMAKE_PREFIX_PATH <PREFIX_PATH>/lib/cmake)
FIND_PACKAGE(Qt5 COMPONENTS Core Xml Sql Gui Widgets REQUIRED)

# 引入外部头文件和动态链接库
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/../include)
LINK_DIRECTORIES(${PROJECT_SOURCE_DIR}/../lib)

# 统一配置各目录层级的.cpp
* 网上有很多做法是每一个目录编写独立的CMakeLists.txt，但是个人感觉没有单一CMakeLists.txt文件配置方便，
* 特别是如果各个目录间存在依赖的情况下更容易出错
AUX_SOURCE_DIRECTORY(./<mod_1> mod_1_src_list)
AUX_SOURCE_DIRECTORY(./<mod_2> mod_2_src_list)
AUX_SOURCE_DIRECTORY(. src_list)

# 指定最终编译产物的输出路径
* 和使用include和lib作为外部依赖路径类似，我也习惯在src的同级目录分别创建bin和out用来存放最终的编译产物
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/../bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/../out)

# 区分release和debug生成的动态库
* 有时候为了方便调试，我们需要让debug版本的动态以d结尾
SET(CMAKE_DEBUG_POSTFIX d)

# QDebug在release下依然可以输出函数名和行号
* 主要是为了保证日志数据有效
ADD_DEFINITIONS(-DQT_MESSAGELOGCONTEXT)

# 添加链接库
TARGET_LINK_LIBRARIES(${target} Qt5::Sql Qt5::Gui <lib>)

# 输出
ADD_EXECUTABLE(${target} ${SRC_LIST})
ADD_LIBRARY(${target} SHARED ${SRC_LIST})

# 根据release和debug分目录数据产物
* 主要是针对动态库产物的输出，分不同的目录更适合大型项目的统编
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/../debug)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/../release)

# 添加自定义的宏
* 可以在项目中通过条件编译的方式选择不同的配置
OPTION(C_MODE "Use Customize Mode" ON)
IF(C_MODE)

 # 其他指令
ENDIF()
改变代码的编译过程

// 在cmakelists.txt中定义
ADD_DEFINITIONS(-DTEST_DEBUG)

// 配置源码中的条件编译
if def TEST_DEBUG
...
...
else 
...
endif
debug调试

ADD_DEFINITIONS("-Wall -g")
# 添加指定文件
* 一般来说，一个.cpp文件都会有一个.h来对应。编译的时候我们只需要告诉编译器所有的.cpp文件即可。例如：ADD_EXECUTABLE(sth ${cpp})。但是有时候，我们会定义一些结构体或常量，并将他们集中声明在一个.h文件里。
FILE(GLOB HEADER_FILES "*.h")
FILE(GLOB SOURCE_FILES "*.cpp")

# 安装与复制
* 当我们需要在编译完成以后执行copy或install的时候

FILE(COPY ${HEADER_FILES} DESTINATION ${PROJECT_SOURCE_DIR}/../include/${target})

INSTALL(TARGETS mylib
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION libstatic)
		
# 注意事项
# 如果是使用MinGW编译windows下的动态库不需要添加导出类的宏

# LINK_DIRECTORIES 指令必须放在ADD_指令前

# 对多级目录的项目使用cmake做统编，每一个层级的编译应该使用动态库的方式

# 如果你使用的是QtCreator，自定义宏的方式可能不生效，但这并不是cmake的问题

# Windows下如何使用cmake和gcc
# 安装MinGW Installation Manager和CMake的windows安装包，安装gcc编译工具链(mingw32-gcc, mingw32-gcc-g++, mingw32-make ...缺少的依赖可以从MinGW Installation Manager里面安装)

# 配置环境变量

*3.3 mingw32-make.exe 复制后重命名为 cmake.exe

# 指定编译方案：cmake -G "MinGW Makefiles" . （如果编译器为vs的话使用"NMake Makefiles"）

# make 完成编译


