---
title: ROS 笔记（1）
date: 2018-03-18 09:40:43
tags:
---
下文是 A-Systematic-Approach-to-Learning-Robot-Programming-with-ROS 一书的学习笔记。
<!--more-->
# 第二章

## 消息
在工作空间（`mkdir ros`），新建包 `catkin_package_create exa`。
之后就要把 CMakeList.txt 的内容全部删除。

在 exa 文件夹下新建一个 msg 文件夹存放存放 ExampleMessage.msg：
```C
Header header
int32 demo_int
float64 demo_double
```
可以通过 `rosmsg show std_msgs/Header` 查看 Header 的详情：
```C
uint32 seq
time stamp
string frame_id
```

首先编写 `example_ros_message_publisher.cpp`：
```C
#include "ros/ros.h"
#include "exa/ExampleMessage.h"

int main(int argc, char **argv){
    ros::init(argc, argv, "example_ros_message_publisher");
    ros::NodeHandle nh;
    ros::Publisher my_publisher_object = nh.advertise<exa::ExampleMessage>("exampler_topic",1);
    exa::ExampleMessage my_new_message;
    ros::Rate naptime(1.0);
    my_new_message.header.stamp = ros::Time::now();
    my_new_message.header.seq = 0;
    my_new_message.header.frame_id = "base_frame";
    my_new_message.demo_int = 1;
    my_new_message.demo_double;
    double sqrt_arg;
    while(ros::ok()){
        my_new_message.header.seq++;
        my_new_message.header.stamp = ros::Time::now();
        my_new_message.demo_int *= 2.0;
        sqrt_arg = my_new_message.demo_double;
        my_new_message.demo_double = sqrt(sqrt_arg);
        my_publisher_object.publish(my_new_message);
        naptime.sleep();
    }
}
```

**重点**是编写 CMakeList.txt，我在学习的过程中遇到了许多的坑：
```C
cmake_minimum_required(VERSION 2.8.3)
project(exa)
// 如果 package 中没有 roscpp，就会出现 include ros.h 失败的情况
find_package(catkin REQUIRED COMPONENTS roscpp std_msgs genmsg)

// DIRECTORY 后面的变量是 msg 文件夹，FILES 是 msg 文件
add_message_files(DIRECTORY msg FILES ExampleMessage.msg)

// 如果没有生成消息，include "exa/ExampleMessage.h" 会失败，
// 因为这个 h 文件是根据 msg 文件生成的
generate_messages(DEPENDENCIES std_msgs)

catkin_package(CATKIN_DEPENDS message_runtime std_msgs)

// 如果没有下面的这个，也会出现 include ros.h 失败的情况
include_directories(include ${catkin_INCLUDE_DIRS})

// 链接不能够省略，否则无法生成最后的可执行文件
add_executable(example_ros_message_publisher src/example_ros_message_publisher.cpp)

// 如果没有下面会出现由 msg 生成的 h 文件出现比 cpp 早，导致 include ros.h 失败的情况
add_dependencies(example_ros_message_publisher ${${PROJECT_NAME}_EXPORTED_TARGETS})
target_link_libraries(example_ros_message_publisher ${catkin_LIBRARIES})
```

> `generate_messages` 由 msg 生成的 h 文件在 ./devel/include/exa 里面。

package.xml 的编写比较简单，就是注意加上  `<build_depend>message_generation</build_depend>` 和  `<exec_depend>message_runtime</exec_depend>`就可以了。

在工作空间编译 `catkin_make`，如果之后出现下面这个问题：
```C
Cannot load message class for .... Are your messages built?
```
清除 `catkin_make clean` 后重新编译。
运行的命令(在工作空间)：
```C
roscore
source ./devel/setup.bash
rosrun exa example_ros_message_publisher // 第一个参数是在 Cmake 定义的，第二个是 init 函数定义的
rostopic list                           // 可以看到 /exampler_topic 的一个 topic，它是在 advertise 函数定义的。
rostopic echo exampler_topic
```

官方解释：您可以使用消息包提供给脚本和程序。如果你这样做，消息生成目标需要在任何依赖它们的程序之前构建。每个直接或间接使用您的消息头的目标都必须声明一个明确的依赖关系
```C
add_dependencies(your_program $ {$ {PROJECT_NAME} _EXPORTED_TARGETS})
your_program 是 add_executable 生成的可执行文件
```
您的构建目标也使用从其他catkin包导入的消息或服务头，请同样声明这些相关性：
```C
add_dependencies(your_program $ {catkin_EXPORTED_TARGETS})
```


之后的一个教程是发布长度不等的消息，也就是将 msg 文件内容改为：`float64[] dbl_vel` 就可以了。
接着的步骤都是差不多的，列一下 `vector_publisher.cpp` 的代码：
```C
#include "ros/ros.h"
#include "exa/VecOfDoubles.h"
int main(int argc, char **argv){
    ros::init(argc, argv, "vector_publisher");
    ros::NodeHandle nh;
    // msg 生成的 h 文件依旧是属于 exa 包的，所以命名空间为 exa
    ros::Publisher my_pusblisher = nh.advertise<exa::VecOfDoubles>("vector_topic",1);

    exa::VecOfDoubles vec_msg;
    double counter = 0;
    ros::Rate naptime(1.0);

    vec_msg.dbl_vec.resize(3);
    vec_msg.dbl_vec[0] = 1.414;
    vec_msg.dbl_vec[1] = 2.718;
    vec_msg.dbl_vec[2] = 3.1416;
    
    // 增加元素要用跟 vector 类似的方式
    vec_msg.dbl_vec.push_back(counter);
    while(ros::ok()){
        counter += 1.0;
        vec_msg.dbl_vec.push_back(counter);
        my_pusblisher.publish(vec_msg);
        naptime.sleep();
    }
}
```
而在 `catkin_package()` 中加入 `custom_msgs`。


----------

## 服务

如果一个节点是 publisher，他是不清楚 subscriber，它只知道它要发布的主题。
如果需要双向、一对一、可靠的交流，这时候就需要 client 和 server 了。
client 发送 request，server 返回 response。

第一步是在包目录新建 srv 文件夹，文件夹下新建一个 ExampleServiceMsg.srv 的一个文件：
```C
string name
---
bool on_the_list
bool good_guy
int32 age
string nickname
```
显然在 `---` 上面的是 request，下面的是 response。
这里的 package.xml 也要加入 `message_generation` 的 build 和 exec 的 depend。

之后在 src 文件夹新建一个 `example_ros_service.cpp` 的文件：
```C++
#include "ros/ros.h"
#include "example_ros_service/ExampleServiceMsg.h"
#include "iostream"
#include "string"
using namespace std;

bool callback(example_ros_service::ExampleServiceMsgRequest &request, 
                example_ros_service::ExampleServiceMsgResponse &response){
    ROS_INFO("callback activated");
    string in_name(request.name);
    response.on_the_list = false;
    if(in_name.compare("Bob") == 0){
        ROS_INFO("asked about Bob");
        response.age = 32;
        response.good_guy = false;
        response.on_the_list = true;
        response.nickname = "BobTheTerrible";
    }
    if(in_name.compare("Ted")==0){
        ROS_INFO("asked about Ted");
        response.age = 21;
        response.good_guy = true;
        response.on_the_list = true;
        response.nickname = "Tedy";
    }
    return true;
}

int main(int argc, char **argv){
    ros::init(argc, argv, "example_ros_service");
    ros::NodeHandle nh;
    ros::ServiceServer service = nh.advertiseService("lookup_by_name", callback);
    ROS_INFO("Ready to lookup names");
    ros::spin();
    return 0;
}
```
从上面可以看出 service 不像 msg 一样使用 time loop，而是使用 `ros::spin()`，Service 会 sleep 直到有一个 request。


最后就是 Cmake：
```C
# 如果没有 genmsg，会出现 unknow cmake command add_service_files 的错误
find_package(catkin REQUIRED COMPONENTS roscpp std_msgs genmsg) 

add_service_files(DIRECTORY srv FILES ExampleServiceMsg.srv)
# 从 srv 文件生成对应的 h 文件
generate_messages(DEPENDENCIES std_msgs)

catkin_package(CATKIN_DEPENDS std_msgs message_runtime)

include_directories(include ${catkin_INCLUDE_DIRS})

add_executable(example_ros_service src/example_ros_service.cpp)

# ${PROJECT_NAME}_gencpp 是 srv 文件生成的 h 文件的目录
add_dependencies(example_ros_service example_ros_service_gencpp)
target_link_libraries(example_ros_service ${catkin_LIBRARIES})
```
启动服务：
```C
roscore
source ./devel/setup.bash
rosrun example_ros_servcie example_ros_service
# 测试
rosservice list
# 新建一个窗口，
cd ros
# 如果没有 source，会提示 cannot load XX.h
source ./devel/setup.bash
rosservice call lookup_by_name 'Ted'
```


## client

```C
#include "ros/ros.h"
#include "example_ros_service/ExampleServiceMsg.h"
#include "iostream"
#include "string"
using namespace std;

int main(int argv, char **argv){
    ros::init(argc,argv,"example_ros_client");
    ros::NodeHandle nh;
    ros::ServiceClient client = nh.serviceClient<example_ros_service::ExampleServiceMsg>("lookup_by_name");
    example_ros_service::ExampleServiceMsg srv;
    bool found_on_list = false;
    string in_name;
    while(ros::ok()){
        cout << endl;
        cout << "enter a name (x to quit)";
        cin >> in_name;
        if(in_name.compare('x'))  return 0;
        srv.request.name = in_name;
        if(client.call(srv)){
            if(srv.response.on_the_list){
                cout << srv.request.name << "is known as" << srv.response.nickname << endl;
                cout << "He is" << srv.response.age << "years old" <<endl;
                if(srv.response.good_guy) ...
            }else ...
        }
        else{
            ROS_ERROR("Failed to call service lookup_by_name");
            return 1;
        }
    }
    return 0;
}
```
可以知道 ExampleService.msg 文件拥有 3 个类 ExampleServiceMsg、ExampleServiceMsgRequest、ExampleServiceMsgResponse。

----------

## class
新建一个 `example_ros_class.h`:
```C
#ifndef EXAMPLE_ROS_CLASS_H_
#define EXAMPLE_ROS_CLASS_H_
#include "math.h"
#include "stdlib.h"
#include "string"
#include "vector"
#include "ros/ros.h"
#include "std_msgs/Bool.h"
#include "std_msgs/Float32.h"
#include "std_srvs/Trigger.h"

class ExampleRosClass{
public:
    ExampleRosClass(ros::NodeHandle* nodehandle);
private:
    ros::NodeHandle nh_;
    ros::Subscriber minimal_subscriber_;
    ros::ServiceServer mininal_service_;
    ros::Publisher minimal_publisher_;
    double val_from_subscriber_;
    double val_to_remember_;
    void initializeSubscribers();
    void initializePublishers();
    void initializeServices();
    void subsciberCallback(const std_msgs::Float32 &message_holder);
    bool serviceCallback(std_srvs::TriggerRequest &request, std_srvs::TriggerResponse &response);
}; // 注意这里 class 不同于 Java，是需要加上的。
#endif
```
新建一个 `example_ros_class.cpp`：
```C
#include "example_ros_class.h"
// 构造函数，ExampleRosClass 属于 ExampleRosClass 这个包
ExampleRosClass::ExampleRosClass(ros::NodeHandle* nodehandle):nh_(*nodehandle){
    ROS_INFO("in class constructor ExampleRosClass");
    initializeSubscribers();
    initializePublishers();
    initializeServices();
    val_to_remember_ = 0.0;
}

void ExampleRosClass::initializeSubscribers(){
    ROS_INFO("initialize subscribers");
    minimal_subscriber_ = nh_.subscribe("example_class_input_topic",1,&ExampleRosClass::subsciberCallback,this);
}
void ExampleRosClass::initializeServices(){
    ROS_INFO("initialize services");
    mininal_service_ = nh_.advertiseService("example_minimal_service",&ExampleRosClass::serviceCallback,this);
}
void ExampleRosClass::initializePublishers(){
    ROS_INFO("initialize publisher");
    minimal_publisher_ = nh_.advertise<std_msgs::Float32>("example_class_output_topic",1,true);
}
void ExampleRosClass::subsciberCallback(const std_msgs::Float32 &message_holder){
    val_from_subscriber_ = message_holder.data;
    ROS_INFO("my callback activated: received value %f", val_from_subscriber_);
    std_msgs::Float32 output_msg;
    val_to_remember_ += val_from_subscriber_;
    output_msg.data = val_to_remember_;
    minimal_publisher_.publish(output_msg);
}
bool ExampleRosClass::serviceCallback(std_srvs::TriggerRequest &request, std_srvs::TriggerResponse &response){
    ROS_INFO("service callback activated");
    response.success = true;
    response.message = "here is a repsonse string";
    return true;
}

int main(int argc, char **argv){
    ros::init(argc,argv,"ExampleRosClass");
    ros::NodeHandle nh;
    ROS_INFO("main: instantiating an object of type ExampleRosClass");
    ExampleRosClass examplerRosClass(&nh);
    ROS_INFO("let the callbacks do all the work");
    ros::spin();
    return 0;
}
```

> 这里编译总是找不到错误的原因，我把 `add_executable` 删去，`catkin_make`，再将 `add_executable` 加上，又可以显示编译的错误。

运行程序：
```C
source ./devel/setup.bash
rosrun example_ros_service example_ros_class
rosservice call example_minimal_service
// 在 call 的窗口可以看到 response
success: True
message: "here is a repsonse string"
// example_ros_class 窗口可以看到
service callback activated
rostopic pub -r 2 example_class_input_topic std_msgs/Float32 2.0
```

## Library Modules
新建一个 `creating_a_ros_library` 的包，将上一节的 ExampleRosClass.h 移动到 `include/creating_a_ros_library` 里面。复制 ExampleRosClass.cpp 到 src 目录下，同时将 int main 的代码移动到新建文件 `example_ros_class_test_main.cpp` 里面，记得更改两个 cpp 文件对 h 文件的 include 路径。
CMake：
```C
find_package(catkin REQUIRED COMPONENTS roscpp std_msgs)
catkin_package(CATKIN_DEPENDS std_msgs message_runtime)
include_directories(include ${catkin_INCLUDE_DIRS})
// 将 cpp 加入 lib
add_library(example_ros_library src/example_ros_class.cpp)
add_executable(ros_library_test_main src/example_ros_class_test_main.cpp)
// 此处需要链接新建的 lib
target_link_libraries(ros_library_test_main ${catkin_LIBRARIES} example_ros_library)
```
而且这里不再需要 `generate_message`，因此对应的包可以去除。
生成的 lib 在 `~/ros_ws/devel/lib`。


重点是如何在 external package 导入一个 library，首先新建一个叫做 `using_a_library` 的包，其次将 main 的内容复制到新建的 `using_a_library.cpp` 中，最后就是重要的编写 Cmake：
```C
find_package(catkin REQUIRED COMPONENTS creating_a_ros_library roscpp std_msgs std_srvs)
catkin_package(CATKIN_DEPENDS creating_a_ros_library roscpp std_msgs std_srvs)
# 将 example_ros_class.h 的路径包含进来
include_directories(include ${catkin_INCLUDE_DIRS} ~/ros/src/creating_a_ros_library/include/)
add_executable(using_a_library src/using_a_library.cpp)
# 导入对应的 lib 路径，也可以把后面的 creating_a_ros_library 省略
FIND_LIBRARY(mylib_LIBRARIES example_ros_library ./devel/lib/creating_a_ros_library/)
# 根据上面的路径别名链接自己创建的 lib
target_link_libraries(using_a_library ${catkin_LIBRARIES} ${mylib_LIBRARIES})
```

> 单独编译一个包：`catkin_make --pkg using_a_library`

----------


## Action

Service-client 模式的缺点就是需要等待 service 回应。在等待回应的时候，client 应该可以去完成其他的任务，因此，action 就充当了这个角色。

第一步是定义一个 demo.action 文件：
```C
#goal，client 发送的信息
int32 input
---
#result，回应 client 的信息
int32 output
int32 goal_stamp
---
#feedback，在这个例子中没有使用
int32 fdbk
```
`generate_message` 会根据 `---` 生成对应的 demoAction、demoGoal、demoResult、demoFeedback。

`example_action_server.cpp` 的编写：
```C
#include "ros/ros.h"
#include "actionlib/server/simple_action_server.h"
#include "example_action_server/demoAction.h"

int g_count = 0;
bool g_count_failure = false;

class ExampleActionServer{
private:
    ros::NodeHandle nh_;
    actionlib::SimpleActionServer<example_action_server::demoAction> as_;
    example_action_server::demoGoal goal_;
    example_action_server::demoResult result_;
    example_action_server::demoFeedback feedback_;
    int countdown_val_;
public:
    ExampleActionServer();
    ~ExampleActionServer(void){}
    void executeCB(const actionlib::SimpleActionServer<example_action_server::demoAction>::GoalConstPtr &goal);
};
ExampleActionServer::ExampleActionServer():as_(nh_,"timer_action",
                                    boost::bind(&ExampleActionServer::executeCB,this,_1),false){
    as_.start();
}
void ExampleActionServer::executeCB(const actionlib::SimpleActionServer<example_action_server::demoAction>::GoalConstPtr &goal){
    ROS_INFO("goal input is %d", goal -> input);
    ros::Rate timer(1.0);
    countdown_val_ = goal -> input;
    while(countdown_val_>0){
        ROS_INFO("countdown_val_ = %d",countdown_val_);
        if(as_.isPreemptRequested()){
            result_.output = countdown_val_;
            as_.setAborted(result_);
            return;
        }
        feedback_.fdbk = countdown_val_;
        as_.publishFeedback(feedback_);
        countdown_val_--;
        timer.sleep();
    }
    result_.output = countdown_val_;
    as_.setSucceeded(result_);
}
int main(int argc, char **argv){
    ros::init(argc, argv, "example_action_server");
    ExampleActionServer as_object;
    ros::spin();
    return 0;
}
```
可以看出，action server 主要是 `as_` 为主，`nh_` 已经无太多作用了（主要用于 as 的初始化）。
`boost::bind` 的作用是绑定 server 的回调函数。
`as_.setSucceeded(result_)` 的作用是将成功和 result 返回给 client。返回的信息封装在 demoResult 对象里。



`example_action_client.cpp` 的代码：
```C
#include "ros/ros.h"
#include "actionlib/client/simple_action_client.h"
#include "example_action_server/demoAction.h"
using namespace std;
bool g_goal_active = false;
int g_result_output = -1;
int g_fdbk = -1;
void doneCb(const actionlib::SimpleClientGoalState &state,
                const example_action_server::demoResultConstPtr &result){
    ROS_INFO("doneCb, server responded with state :%s",state.toString().c_str());
    ROS_INFO("output = %d",result->output);
    g_result_output = result->output;
    g_goal_active = false;
}
void feedbackCb(const example_action_server::demoFeedbackConstPtr &fdbk_msg){
    ROS_INFO("feedback status = %d",fdbk_msg->fdbk);
    g_fdbk = fdbk_msg->fdbk;
}
void activeCb(){
    ROS_INFO("Goal just went active");
    g_goal_active = true;
}
int main(int argc, char **argv){
    ros::init(argc,argv,"example_action_client");
    ros::NodeHandle nh;
    ros::Rate main_timer(1.0);
    example_action_server::demoGoal goal;
    actionlib::SimpleActionClient<example_action_server::demoAction> action_client("timer_action",true);
    bool server_exists = action_client.waitForServer(ros::Duration(1.0));
    while(!server_exists){
        ROS_WARN("could not read from server, retrying");
        server_exists = action_client.waitForServer(ros::Duration(1.0));
    }
    int countdown_goal = 1;
    while(countdown_goal>=0){ // 根据 countdown 的值递减，每次递减发送一条信息
        cout<<"enter a desired timer value, in seconds (0 to abort, <0 to quit)";
        cin>>countdown_goal;
        if(countdown_goal==0)
            action_client.cancelGoal();
        if(countdown_goal<0)
            return 0;  // quit this program
        ROS_INFO("sending timer goal = %d seconds to timer action server",countdown_goal);
        goal.input = countdown_goal;
        action_client.sendGoal(goal,&doneCb, &activeCb, &feedbackCb);
    }
    return 0;
}
```
首先 `waitForServer` 尝试连接 server；
再通过 sendGoal 发送给 server，同时还可以传入 callback 函数。发送的信息是使用在 demo.action 封装的 demoGoal。
`activeCb` 是 server 接收到 client 的 goal request 时调用；

> 注意：`as_` 初始化的名字要和 `action_client` 初始化名字一样，这样才能进行通信。发布的 topic 是 `/timer_action/goal`

- client 是不能够用 ctrl+C 退出的，从代码的 return 看出只能够通过输入一个负数退出；
- 之前的 client 是使用 `ros::ok()` 来退出的；
- 而 server 一般是通过 `ros::spin()` 进入循环，所以也可以通过 ctrl+C 退出。


## Parameter Server

> 参数服务器是共享的，多变量的字典，可以通过网络API访问。节点使用参数服务器存储参数，但参数服务器不是为了高性能运算而设计，所以最好用于静态、非二进制数据，比如配置参数。参数服务器使得参数全局可见，便于更改。

当一个 publisher 广播 messages，service 和 action server 接受消息，这种通信是点对点的，但是 parameter server 更像一个 shared memory。
parameter setting 是存储在 YAML 文件中的，可以通过命令行或者 launch 文件读取。
YAML 主要通过 rosparam 控制：
```C
rosparam set /gains "p:1.0 i:2.0"   // 在 server 上放置的参数
roparam list                        // 获取 parameter server 的参数
rosparam get /gains
rosparam dump param_dump            // 讲 server 的参数装载进一个文件 param_dump 里面
rosparam load jnt1_gains . yaml
rosparam get / joint1_gains         // 这里是 load 完成后才可以进行
```
新建一个 `test_parameter.yaml` 的文件：
```C
// 注意 : 后面的空格不能省略
joint1_gains: {p: 4.0, i: 5.0, d: 6.0}
```
新建一个 `load_gains.launch` 文件用于读取 yaml 文件：
```C
<launch>
<rosparam command="load" file="test_param.yaml"/>
</launch>
```
读取数据：`roslaunch load_gains.launch`。
之后就是 `read_parameter_from_node.cpp`：
```C
#include "ros/ros.h"
int main(int argc, char** argv){
    ros::init(argc, argv, "example_parameter_server");
    ros::NodeHandle nh; 
    double P_gain, D_gain, I_gain;
    if(nh.getParam("/joint1_gains/p", P_gain))
        ROS_INFO("proportional gain set to %f",P_gain);
    else
        ROS_WARN("could not find the parameter");
}
```
运行编译的文件前一定要先通过 launch 文件读取数据。
删除参数：
```C
rosparam delete /joint1_gains
```

