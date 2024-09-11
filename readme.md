# ROS笔记

# 预备知识

## roscore

是一个向节点提供连接信息，以便节点之间可以相互传递信息的服务程序。

是一个帮助节点互相找到的程序。



## ros::ok()

是ROS中的一个函数，用于检测当前ROS系统的状态，主要用于确定节点是否应该继续运行。它是一个布尔函数，返回 `true` 或 `false`，用来检查节点是否还可以正常工作，或者是否应该关闭。

`ros::ok()` 在循环或节点运行时用于检查以下条件：

- 是否已经正确初始化了ROS节点。
- ROS Master 是否还在运行。
- 节点是否接收到终止信号（如用户通过 `Ctrl+C` 发出终止信号）。
- 节点是否调用了 `ros::shutdown()` 函数。

# ROS基础

## 工作空间

**工作空间结构：**

`src`：代码空间，储存所有ros功能包的源码文件。

`build`：编译空间，储存编译过程中产生的缓存信息和中间文件。

`devel`：开发空间，放置编译生成的可执行文件。

`install`安装空间。



**创建工作空间**

先创建工作空间目录，再运行ROS的工作空间初始化命令。

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
catkin_init_workspace
```

创建完成之后在工作空间根目录下编译：

```bash
cd ~/catkin_ws/
catkin_make
```



## 功能包

**创建功能包**：

进入代码空间，创建功能包：

```
cd ~/catkin_ws/src
catkin_create_pkg <pkg_name> [depend1] [depend2] ...
```



## ROS节点

**创建一个ROS节点：**

ROS节点初始化：

```c++
ros::init(argc, argv, "node_name");
```

>`ros::init()` 是每个ROS节点的启动入口。它的作用是初始化该节点，并将其注册到ROS系统中。它在程序的早期执行，用于设置节点的名称并连接到ROS Master。调用 `ros::init()` 是必需的，否则节点无法注册和与其他节点通信。
>
>前两个参数是命令行或launch文件的输入参数，第三个参数是节点名称。

创建节点句柄：

```c++
ros::NodeHandle nh;
```

>**不一定需要**，但这取决于你是否需要与ROS的核心功能交互。如果你只是初始化节点，执行一些不涉及ROS消息传递、参数服务器或者服务调用的任务，那么你不需要创建 `ros::NodeHandle`。

设置循环频率：

```c++
ros::Rate rate(10);
```

创建循环：

```c++
 while (ros::ok())
  {
    // 处理ROS回调队列中的回调函数
    ros::spinOnce();  // 处理回调

    // 控制循环频率为 10 Hz
    loop_rate.sleep();  // 让循环每 0.1 秒执行一次
  }

```

>**`ros::spinOnce()`** 只处理回调队列中的回调函数，不会影响循环的执行频率。
>
>**`loop_rate.sleep()`** 用于控制循环的执行频率，如果没有调用它，即使设定了频率，循环仍然会以全速运行。
>
>因此，结合 `ros::spinOnce()` 和 `loop_rate.sleep()` 是一种常见的做法，既能保证节点以设定频率运行，又能处理来自ROS系统的回调事件。

完整节点样例：

```c++
#include "ros/ros.h"

int main(int argc, char **argv)
{
  ros::init(argc, argv, "node_name");
  ros::NodeHandle nh;

  // 设定一个 10 Hz 的频率
  ros::Rate loop_rate(10);

  while (ros::ok())
  {
    // 处理ROS回调队列中的回调函数
    ros::spinOnce();  // 处理回调

    // 控制循环频率为 10 Hz
    loop_rate.sleep();  // 让循环每 0.1 秒执行一次
  }

  return 0;
}
```



# 话题

话题表示的是一个定义了类型的信息流。实现了一种发布/订阅的通信机制。

## 创建发布者

```c++
#include "ros/ros.h"
#include "std_msgs/String.h"

int main(int argc, char **argv)
{
  // 初始化ROS节点，名称为"talker"
  ros::init(argc, argv, "talker");

  // 创建节点句柄，用于与ROS系统交互
  ros::NodeHandle nh;

  // 创建一个发布者，发布到话题 "chatter"，消息类型为 std_msgs::String
  ros::Publisher chatter_pub = nh.advertise<std_msgs::String>("chatter", 1000);

  // 定义一个频率控制器，设定发布频率为 10Hz
  ros::Rate loop_rate(10);

  int count = 0;
  while (ros::ok())
  {
    // 创建消息对象
    std_msgs::String msg;

    // 定义要发布的内容
    std::stringstream ss;
    ss << "Hello, ROS! Message number: " << count;
    msg.data = ss.str();

    // 打印日志信息到终端
    ROS_INFO("%s", msg.data.c_str());

    // 发布消息到话题
    chatter_pub.publish(msg);

    // 调用spinOnce以处理回调函数（如果有的话）
    ros::spinOnce();

    // 休眠以保证频率为 10Hz
    loop_rate.sleep();
    ++count;
  }

  return 0;
}

```

>**`ros::Publisher chatter_pub = nh.advertise<std_msgs::String>("chatter", 1000);`**
>
>- 创建一个发布者 `chatter_pub`，发布到名为 `"chatter"` 的话题。消息类型为 `std_msgs::String`，缓冲区大小为1000。
>
>**`chatter_pub.publish(msg);`**
>
>- 将消息 `msg` 发布到 `chatter` 话题上。



## 创建订阅者

```c++
#include "ros/ros.h"
#include "std_msgs/String.h"

// 回调函数，当订阅者接收到消息时调用
void chatterCallback(const std_msgs::String::ConstPtr& msg)
{
  // 打印接收到的消息内容
  ROS_INFO("I heard: [%s]", msg->data.c_str());
}

int main(int argc, char **argv)
{
  // 初始化ROS节点，节点名称为"listener"
  ros::init(argc, argv, "listener");

  // 创建节点句柄，用于与ROS系统交互
  ros::NodeHandle nh;

  // 创建一个订阅者，订阅话题 "chatter"，消息类型为 std_msgs::String
  // 当收到消息时，调用 chatterCallback 回调函数
  ros::Subscriber sub = nh.subscribe("chatter", 1000, chatterCallback);

  // ros::spin() 保持程序的运行，并调用回调函数处理接收到的消息
  ros::spin();

  return 0;
}
```

>**`ros::Subscriber sub = nh.subscribe("chatter", 1000, chatterCallback);`**
>
>- 创建一个订阅者 `sub`，订阅名为 `"chatter"` 的话题。消息类型为 `std_msgs::String`，并将回调函数 `chatterCallback` 绑定到该订阅者。当接收到消息时，ROS会调用这个回调函数处理消息。
>
>**`void chatterCallback(const std_msgs::String::ConstPtr& msg)`**
>
>- 这是一个回调函数，当订阅者收到消息时，它会被调用。通过 `msg->data.c_str()` 访问消息的内容并将其打印出来。



## 自定义话题消息

1. 创建msg

   在软件包中新建文件夹，新建消息文件，例如：

   ```bash
   cd ~/catkin_ws/src/my_custom_msgs_pkg
   mkdir msg
   cd msg
   touch Person.msg
   ```

   在 `Person.msg` 中定义自定义消息的字段和类型：

   ```msg
   string name   # 姓名
   int32 age     # 年龄
   float32 height  # 身高 (单位: 米)
   ```

   >每个字段的类型和名称必须与ROS支持的基本类型一致，如 `string`, `int32`, `float32`, `bool` 等

2.  **修改 `CMakeLists.txt` 和 `package.xml`**

   修改 `package.xml`，添加对 `message_generation` 和 `message_runtime` 的依赖，以便处理自定义消息的生成。

   ```xml
   <build_depend>message_generation</build_depend>
   <exec_depend>message_runtime</exec_depend>
   ```

   在 `CMakeLists.txt` 中，找到并修改 `find_package` 部分，确保添加了 `message_generation` 作为依赖：

   ```cmake
   find_package(catkin REQUIRED COMPONENTS
     roscpp
     std_msgs
     message_generation
   )
   ```

   确保 `catkin_package` 中包含 `message_runtime`：

   ```cmake
   catkin_package(
     CATKIN_DEPENDS message_runtime roscpp std_msgs
   )
   ```

   找到 `add_message_files`，添加自定义消息：

   ```cmake
   add_message_files(
     FILES
     Person.msg
   )
   ```

   找到 `generate_messages`，并确保它正确配置：

   ```cmake
   generate_messages(
     DEPENDENCIES
     std_msgs
   )
   ```

   >**`add_message_files()`**
   >
   >`add_message_files()` 命令的作用是**声明需要编译的自定义消息文件**。它告诉ROS构建系统（`catkin`）有哪些 `.msg` 文件需要生成对应的代码文件（如C++头文件或Python类），并为这些自定义消息生成相应的接口。
   >
   >#### 具体作用：
   >
   >- 你在这个命令中列出的 `.msg` 文件会被编译成C++、Python等语言中可用的消息类型。
   >- ROS会根据 `.msg` 文件的定义，自动生成这些消息的序列化和反序列化代码，这样你可以在你的节点代码中轻松使用这些自定义消息。
   >
   >
   >
   >### **`generate_messages()`**
   >
   >`generate_messages()` 命令的作用是**生成所需的消息、服务和动作的代码**。它根据 `add_message_files()` 中声明的 `.msg` 文件，生成C++、Python等语言的消息代码，使这些消息可以在节点中使用。
   >
   >#### 具体作用：
   >
   >- 它会生成对应的头文件（C++）或类（Python），你可以在ROS节点中引用并使用这些消息。
   >- 该命令中的 `DEPENDENCIES` 部分用于指定自定义消息所依赖的其他消息包。通常，自定义消息依赖标准消息包（如 `std_msgs`），因为你可能在自定义消息中使用内置类型或其他ROS消息包的类型。

3. 验证

   回到工作空间根路径下，使用catkin_make编译，编译之后可以使用：

   ```
   rosmsg show Person
   ```

   查看定义的Person消息类型。



## 话题工具

**rostopic echo**

显示在某个话题上发布的数据。

用法：

```bash
rostopic echo [topic]
```



**rostopic list**

能够列出当前已被订阅和发布的所有话题。

用法：

```bash
rostopic list -v #列出所有发布和订阅的主题及其类型的详细信息
```



**rostopic type**

用来查看所发布话题的消息类型。

用法：

```bash
rostopic type [topic]
```



**rostopic pub**

可以把数据发布到当前某个正在广播的话题上。

用法：

```bash
rostopic pub [topic] [msg_type] [args]
```

默认情况下，`rostopic pub` 只发布一条消息。如果你想持续发布消息，可以使用 `-r` 选项来指定发布频率。

```bash
rostopic pub -r 1 /chatter std_msgs/String "data: 'Hello, ROS continuously!'"
```



**rqt_plot**

可以在滚动时间图上显示发布到某个话题上的数据。

```bash
$ rosrun rqt_plot rqt_plot
```



# 服务

## 创建自定义服务

1. 创建srv目录，用于存放自定义的服务文件：

   ```bash
   cd ~/catkin_ws/src/my_service_pkg
   mkdir srv
   ```

2. 创建服务文件

   在 `srv` 目录中创建一个服务文件。例如，我们创建一个 `AddTwoInts.srv` 文件，它接收两个整数并返回它们的和。

   ```bash
   touch srv/AddTwoInts.srv
   ```

   在 `AddTwoInts.srv` 文件中定义请求和响应的结构：

   ```
   int64 a
   int64 b
   ---
   int64 sum
   ```

   >这个文件的定义分为两部分，`---` 之前是请求的字段，之后是响应的字段。在这个例子中，服务将接收两个整数 `a` 和 `b`，并返回它们的和 `sum`。

3. 修改 `CMakeLists.txt` 和 `package.xml`

   在 `package.xml` 中，添加对 `message_generation` 和 `message_runtime` 的依赖：

   ```
   <build_depend>message_generation</build_depend>
   <exec_depend>message_runtime</exec_depend>
   ```

   在 `CMakeLists.txt` 中，我们需要配置服务文件的生成，找到并修改 find_package 部分：
   确保 message_generation 已经作为依赖添加：

   ```
   find_package(catkin REQUIRED COMPONENTS
     roscpp
     rospy
     std_msgs
     message_generation
   )
   ```

   添加 `add_service_files` 来声明 `.srv` 文件：

   ```
   add_service_files(
     FILES
     AddTwoInts.srv
   )
   ```

   生成服务代码：

   ```
   generate_messages(
     DEPENDENCIES
     std_msgs
   )
   ```

   确保 `catkin_package` 中包含 `message_runtime`：

   ```
   catkin_package(
     CATKIN_DEPENDS message_runtime roscpp rospy std_msgs
   )
   ```



## 编写服务端节点

流程如下：

- 初始化ROS节点
- 创建Server实例
- 循环等待服务请求，进入回调函数
- 回调函数中完成服务并反馈应答数据

```c++
#include "ros/ros.h"
#include "my_service_pkg/AddTwoInts.h"

// 回调函数，处理来自客户端的请求
bool add(my_service_pkg::AddTwoInts::Request &req,
         my_service_pkg::AddTwoInts::Response &res)
{
  // 计算两个整数的和
  res.sum = req.a + req.b;
  ROS_INFO("Request: a=%ld, b=%ld", (long int)req.a, (long int)req.b);
  ROS_INFO("Sending back response: [%ld]", (long int)res.sum);
  return true;
}

int main(int argc, char **argv)
{
  // 初始化ROS节点
  ros::init(argc, argv, "add_two_ints_server");

  // 创建节点句柄
  ros::NodeHandle nh;

  // 创建一个服务，服务名称为 "add_two_ints"，回调函数为 add
  ros::ServiceServer service = nh.advertiseService("add_two_ints", add);
  ROS_INFO("Ready to add two ints.");
  
  // 循环等待并处理服务请求
  ros::spin();

  return 0;
}
```

>`ros::ServiceServer service = nh.advertiseService("add_two_ints", add);`
>
>- 创建一个名为 `add_two_ints` 的服务，指定回调函数 `add` 来处理请求。
>
>`bool add(my_service_pkg::AddTwoInts::Request &req, my_service_pkg::AddTwoInts::Response &res)`
>
>- 回调函数 `add` 处理来自客户端的请求，计算两个整数的和并返回。
>- 在ROS服务（Service）节点中，**服务端回调函数的返回值类型必须是 `bool`**。这是ROS服务机制的设计要求，用于表明服务是否成功处理了客户端的请求。
>- 在ROS服务中，服务端回调函数的参数是**固定的**。具体来说，回调函数的参数必须是 **服务的请求（`Request`）和响应（`Response`）对象**，这是ROS服务机制的要求，用于处理来自客户端的请求并生成响应。



## 编写客户端节点

```c++
#include "ros/ros.h"
#include "my_service_pkg/AddTwoInts.h"
#include <cstdlib>

int main(int argc, char **argv)
{
  // 初始化ROS节点
  ros::init(argc, argv, "add_two_ints_client");

  // 检查输入参数（要求提供两个整数）
  if (argc != 3)
  {
    ROS_INFO("Usage: add_two_ints_client X Y");
    return 1;
  }

  // 创建节点句柄
  ros::NodeHandle nh;

  // 创建客户端，连接到 "add_two_ints" 服务
  ros::ServiceClient client = nh.serviceClient<my_service_pkg::AddTwoInts>("add_two_ints");

  // 创建服务请求对象并赋值
  my_service_pkg::AddTwoInts srv;
  srv.request.a = atoll(argv[1]);
  srv.request.b = atoll(argv[2]);

  // 请求服务并处理返回结果
  if (client.call(srv))
  {
    ROS_INFO("Sum: %ld", (long int)srv.response.sum);
  }
  else
  {
    ROS_ERROR("Failed to call service add_two_ints");
    return 1;
  }

  return 0;
}
```

>`ros::ServiceClient client = nh.serviceClient<my_service_pkg::AddTwoInts>("add_two_ints");`
>
>- 创建一个服务客户端，连接到 `add_two_ints` 服务。
>
>`client.call(srv);`
>
>- 向服务端发送请求，并获取响应。

## 数据流简要概述

1. **客户端创建请求数据并调用服务**：
   - 客户端通过 `srv.request` 设置请求数据。
   - 调用 `client.call(srv)`，请求被发送到服务端。
2. **服务端接收请求并处理**：
   - 服务端的回调函数接收到请求数据，存储在 `req` 对象中。
   - 服务端处理请求，并将响应结果存储在 `res` 对象中。
3. **服务端回传响应数据**：
   - 服务端回调函数返回 `true`，表示处理成功。
   - `res` 对象中的数据通过ROS机制回传给客户端。
4. **客户端接收响应数据**：
   - 客户端的 `srv.response` 被填充为服务端返回的响应数据。
   - 客户端可以读取 `srv.response` 中的值来获得服务端的处理结果。



# ROS配置主从机

## 主机配置

在主机的`/etc/hosts`文件中，添加从机IP地址及从机hostname：

```
从机IP地址  从机hostname
```



在主机`.bashrc`文件中，添加以下命令：

```bash
export ROS_HOSTNAME=主机hostname
export ROS_MASTER_URI=http://主机ip:11311
export ROS_IP=主机ip
```



## 从机配置

在从机的`/etc/hosts`文件中，添加主机地址及主机hostname：

```bash
主机IP地址  主机hostname
```



在从机`.bashrc`文件中，添加以下命令：

```bash
export ROS_HOSTNAME=从机hostname
export ROS_MASTER_URI=http://主机ip:11311
```

> `/etc/hosts` 文件中的内容确实可以视作一个包含 IP 地址和主机名的简单表格。这个文件通常用于手动配置 IP 地址和主机名之间的映射，以便在本地解析主机名。



## 举例

**主机`firefly`**

1. 打开 `/etc/hosts` 并编辑：

   ```bash
   vim /etc/hosts
   ```

   添加：

   ```bash
   192.168.43.33  ysp-HP
   ```

2. 打开 `.bashrc` 并编辑：

   ```bash
   vim .bashrc
   ```

   添加：

   ```bash
   export ROS_HOSTNAME=firefly
   export ROS_MASTER_URI=http://firefly:11311
   export ROS_IP=firefly
   ```



**从机`ysp-HP`**

1. 打开 `/etc/hosts` 并编辑：

   ```bash
   vim /etc/hosts
   ```

   添加：

   ```
   192.168.0.112  firefly
   ```

2. 打开 `.bashrc` 并编辑：

   ```bash
   export ROS_HOSTNAME=ysp-HP
   export ROS_MASTER_URI=http://firefly:11311
   ```

   