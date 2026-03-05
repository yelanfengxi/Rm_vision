cmake_minimum_required(VERSION 3.8) # 设定cmake的最小可用版本，若无此及以上版本则发出报错
project(hik_camera) # 设定项目名称 例如此处 hik_camera为功能包名称

## Use C++14
set(CMAKE_CXX_STANDARD 14) # 设定C++14作为编译用的版本？
set(CMAKE_CXX_STANDARD_REQUIRED ON) # 是否强制要求 ON是

## By adding -Wall and -Werror, the compiler does not ignore warnings anymore,
## enforcing cleaner code.
add_definitions(-Wall -Werror) # CMake函数 用来向预处理器添加编译选项 -Wall即Warning All 开启大部分警告 -Werror 即Warning as Error 将警告作为错误，强制修复错误

## Export compile commands for clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(ament_cmake_auto REQUIRED) # 寻找依赖功能包？
ament_auto_find_build_dependencies() # 自动查找package.xml文件中构建需要的依赖 ament_cmake是基础功能包

ament_auto_add_library(${PROJECT_NAME} SHARED 
  src/hik_camera_node.cpp
)# 链接库文件？此命令的机理不明白

target_include_directories(${PROJECT_NAME} PUBLIC hikSDK/include)# 链接头文件 到哪？

if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64") # 如果CMake编译环境是X86_64架构的话 如下
  target_link_directories(${PROJECT_NAME} PUBLIC hikSDK/lib/amd64)
  install(
    DIRECTORY hikSDK/lib/amd64/
    DESTINATION lib
  ) # install做什么？
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64") # 如果是aarch64架构如下
  target_link_directories(${PROJECT_NAME} PUBLIC hikSDK/lib/arm64)
  install(
    DIRECTORY hikSDK/lib/arm64/
    DESTINATION lib
  )
else() # 如果都不是 则报错说不支持 XX架构
  message(FATAL_ERROR "Unsupport host system architecture: ${CMAKE_HOST_SYSTEM_PROCESSOR}!")
endif() # 停止选择结构 如果没有endif可以吗？ endif是C++14独有的？

target_link_libraries(${PROJECT_NAME}
  FormatConversion
  MediaProcess
  MvCameraControl
  MVRender
  MvUsb3vTL
)# 链接这些头文件 到哪？

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN hik_camera::HikCameraNode
  EXECUTABLE ${PROJECT_NAME}_node
) # 这个函数是做什么的？ 其中的参数含义？

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  list(APPEND AMENT_LINT_AUTO_EXCLUDE
    ament_cmake_copyright
    ament_cmake_cpplint
    ament_cmake_uncrustify
  )# BUILD_TESTING参数从哪来？ 什么时候此函数会被执行
  ament_lint_auto_find_test_dependencies()
endif()

ament_auto_package(
  INSTALL_TO_SHARE
  config
  launch
) # 自动寻找功能包？ 做什么？
