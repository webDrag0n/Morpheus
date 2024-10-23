# Morpheus

## Introduction

**本项目的目标是开发一个专注于人-机协同任务的机器人仿真环境**，集成ROS（Robot Operating System）、Unity和MuJoCo的仿真平台，具备高精度的物理仿真、灵活的环境建模、传感器模拟以及基于大语言模型的AI动作绑定和人机协作。通过容器化简化部署，并集成Hololens 2进行场景可视化和人体动作捕捉。该平台还支持通过照片生成场景并扩展类似场景，以及优化人机协作的数据集生成流程。

![Morpheus愿景](H:\ProjectsPerformance\Morpheus\README.assets\Morpheus愿景.png)

![Morpheus关键组件](H:\ProjectsPerformance\Morpheus\README.assets\Morpheus关键组件.png)

## ROS2 EndPoint Interface

[GitHub Repo](https://github.com/webDrag0n/MorpheusROS2EndPoint)

## Status

- ### Roadmap
	- ✅ MuJoCo
		- Unity端插件部署完成：((66f11252-eeb3-43d1-8b00-d9f84883f836))
	- ✅ [[Unitree MuJoCo]]
		- 部署完成
	- ✅ [[Unitree sdk2]] [[Unitree sdk2 python]]
	- ✅ ML-Agent
	- ✅ ROS Plugin：Unity-Robotics-Hub
		- ROS2（foxy）与Unity通信完成测试，相关过程记录在 [[Unity-Robotics-Hub]]
	- ▶️ Isaac Sim RL Sim2Sim测试
		- 环境部分部署完成，但是还有缺失依赖待解决
	- ▶️Hololens 2 连接Unity
		- Microsoft-MRTK3.0 OpenXR技术栈

## How to build

- #Unity #机器人仿真
- ## 安装
	- Git：[官方Git页面](https://github.com/Unity-Technologies/Unity-Robotics-Hub)
	- 首先使用[鱼香ROS](https://fishros.org.cn/forum/)教程安装ROS2-foxy桌面版环境，如遇到问题也可改为安装基础版，但是桌面版有更全面的功能，对于其他ROS2开发可能有用。
		- [一键安装教程](https://fishros.org.cn/forum/topic/20/%E5%B0%8F%E9%B1%BC%E7%9A%84%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E7%B3%BB%E5%88%97)
		  ```bash
		  wget http://fishros.com/install -O fishros && . fishros
		  ```
	- [安装ros_unity_integration与demo](https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/tutorials/ros_unity_integration/setup.md)
		-
		  ```bash
		  git clone https://github.com/Unity-Technologies/Unity-Robotics-Hub.git
		  # 注意最后的branch指定参数
		  git clone https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git -b main-ros2
		  mkdir workspace_colcon
		  cd workspace_colcon
		  mkdir src
		  cp -r ../ROS-TCP-Endpoint ../Unity-Robotics-Hub ./src
		  ```
		- ros_tcp_endpoint目录下：
		  ```bash
		  # conda 环境会扰乱ros2引用
		  conda deactivate
		  
		  # 一定要执行两次
		  # --symlink-install 更快迭代
		  colcon build --symlink-install
		  source install/setup.bash
		  colcon build --symlink-install
		  source install/setup.bash
		  ```
		- 如安装过程中报`No module named 'em'`错误，请通过`pip3 install empy==3.3.2`安装`em`模块，⚠️注意版本号不对也有可能报错，其他缺少模块报错只需缺什么装什么即可。
		- 测试
		  ```bash
		  # conda 环境会扰乱ros2引用
		  conda deactivate
		  
		  ros2 run ros_tcp_endpoint default_server_endpoint --ros-args -p ROS_IP:=0.0.0.0
		  ```
		- 注意如果报错某package not found或import error，很有可能是没有一开始就关闭conda环境，可以尝试删除除src以外所有生成文件重新构建。
	- Unity端插件安装
		
		- 将项目目录下的MuJoCo.zip复制至用户目录（Windows：C:/Users/<UserName>）并解压，文件夹命名为`MuJoCo`
		- 安装ROS-TCP-Connector（Unity端插件）
		  - 在unity package manager中选择`install from git`并填入链接`https://github.com/Unity-Technologies/ROS-TCP-Connector.git?path=/com.unity.robotics.ros-tcp-connector`
		- 安装URDF-Importer
		  - 官方教程：[Importing a Niryo One Robot using URDF Importer](https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/tutorials/urdf_importer/urdf_tutorial.md)
		  - 在unity package manager中选择`install from git`并填入链接`https://github.com/Unity-Technologies/URDF-Importer.git?path=/com.unity.robotics.urdf-importer#v0.5.2`，v0.5.2为版本号，请注意选择。
- ## 增加模块
	- 在`<workspace>/src/unity_robotics_demo/unity_robotics_demo/`目录下添加新publisher，如`h1_control_publisher.py`
	- 更改`<workspace>/src/unity_robotics_demo/setup.py`，在`entry_points`字段下以
	  ```python
	  entry_points={'console_scripts': ['h1_control_publisher = unity_robotics_demo.h1_control_publisher:main'], ...}
	  ```
	  的格式注册新模块  
	- 在`<workspace>/src/unity_robotics_demo/unity_robotics_demo_msgs/msg/`目录下添加新消息，如`H1ControlCommand.msg`
	- 更改`<workspace>/src/unity_robotics_demo/unity_robotics_demo_msgs/CMakeLists.txt`，在`rosidl_generate_interfaces`字段下以
	  ```cmake
	  rosidl_generate_interfaces(${PROJECT_NAME}
	  	"msg/H1ControlCommand.msg"
	      ...
	      "srv/xxx.srv"
	      DEPENDENCIES builtin_interfaces geometry_msgs std_msgs
	  )
	  ```
	  格式注册新消息  
	- 最后执行`<workspace>/start_compile.sh`或
	  ```bash
	  cd <workspace>
	  source install/setup.bash
	  source build
	  source install/setup.bash
	  ```
	  完成编译  
- ## 运行
	- 运行前需要重新编译工作区，以防有更改在增加模块阶段忘记编译
	  ```bash
	  cd <workspace>
	  ./start_compile.sh
	  ./start_ros_tcp_endpoint.sh 
	  ./start_h1_publisher.sh
	  ```
