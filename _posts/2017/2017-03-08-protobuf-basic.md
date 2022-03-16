---
title: protobuf入门
categories:
  - Blog
tags:
  - protobuf
description: how to compile and use protobuf
---


# protobuf的编译安装和使用

-------------------


## 编译与安装

### 下载地址

    https://github.com/google/protobuf/archive/v2.6.1.zip
    或者github： https://github.com/google/protobuf/

### 安装

安装都是老步骤了：

    tar xvf protobuf-2.6.1.tar.gz
    cd protobuf-2.6.1/
    ./configure
    make
    sudo make install

如果你是从github clone或者下载的zip包，你需要在./configure之前运行./autogen.sh
autogen.sh运行会需要一些依赖
比如gtest等，需要手动下载解压当protobuf目录下
```
unzip gtest-1.7.0.zip
mv gtest-1.7.0 gtest
```

推荐下载[带configure的zip包](https://github.com/google/protobuf/archive/v2.6.1.zip)

安装完成后就可以运行protoc这个我们安装的命令了

```
//查看protobuf的版本
protoc --version
```

可能会出现libprotobuf.so.9等库文件找不到，那么执行下面的操作：
```
su
vi /etc/ld.so.conf
//在上面的文件中加入一行内容：/usr/local/lib
//执行ldconfig使得路径生效
ldconfig
```


## **使用**


### **定义通信协议**

**文件名：im.user.proto**

文件内容：
```
package im;
message user
{
        required int32 id = 1;
        required string user_name = 2;
        required string password = 3;
        optional int32 client_type = 4;
}
```

协议文件定义了一个叫user的message，包含id, user_name, password 和 client_type
具体如下：

![pb_user](/images/pb_user.png)

required表示这个字段必须存在

optional表示这个字段是可选的，可以不存在

int32 string 为字段的类型

### **生存协议文件对应的c++代码**

```
protoc -I=./ --cpp_out=./ im.user.proto
```

**-I** 指示输入的协议文件的路径，当前为当前路径
**--cpp_out** 指定生成c++的代码并放置在当前路径中
**im.user.proto** 为要编译成代码的协议文件

运行后可以看到当前目录下生成了两个文件：
```
im.user.pb.cc
im.user.pb.h
```

如果字段为xxx那么我们可以使用的一些api
设置xxx字段
user.set_xxx(...)
清楚字段
user.clear_xxx()
是否存在字段
user.has_xxx()

### **使用protobuf通信的c++代码**
<br>
**文件名：main.cpp**
文件内容：
```
#include <iostream>
#include <fstream>
//包含生成的协议头文件
#include "im.user.pb.h"
using namespace std;
int main()
{
		//定义协议消息user
        im::user user;
        //设置user的属性内容
        user.set_id(1);
        user.set_user_name("tom");
        user.set_password("123");
        user.set_client_type(0);
	    //模拟输出端
        fstream output("./stream_file", ios::out | ios::trunc | ios::binary);
        // 把user序列化到输出流中，实际这里是输出到一个文件中
        user.SerializeToOstream(&output);
        output.close();
		//模拟接收端
        im::user user2;
        fstream input("./stream_file", ios::in | ios::binary);
        //从输入流中parse出一个user到user2
        user2.ParseFromIstream(&input);
        //输出接收到的信息
        cout << "id\tuser\tpasswd\tclient" << endl
                 << "-------------------------------\n"
                 << user2.id()
                 << "\t" << user2.user_name()
                 << "\t" << user2.password()
                 << "\t" <<user2.client_type() << endl;

        return 0;

}

```

### **编译并运行**
```
g++ main.cpp im.user.pb.cc -lprotobuf -o main
```
运行和结果
```
[vonway@localhost ~]$./main
id      user    passwd  client
-------------------------------
1       tom     123     0

```

上面的例子一次只能发送一个消息，如果发送和接受多个，输出结果只有最后一个的消息，其他消息都会报错，无法识别。

### **连续发送和接收多个消息**

要连续发送和接收需要在每次发送了先发送消息的大小即size，接收时先接收size，然后从输入流中读取size大小的内容到buffer
最后从buffer中parse出此条消息，然后如此循环获取其他消息
具体来看代码：

**mutilmsg.cpp**
```
#include <iostream>
#include <fstream>
//协议头文件
#include "im.user.pb.h"
using namespace std;
int main()
{
        im::user user,user1;
        //设置两个user消息并设置内容属性
        user.set_id(1);
        user.set_user_name("tom");
        user.set_password("123");
        user.set_client_type(0);
        user1.set_id(2);
        user1.set_user_name("jim");
        user1.set_password("321");
        user1.set_client_type(1);
		//模拟输出流
        fstream output("./stream_file", ios::out | ios::trunc | ios::binary);
        //输出size
        output << user.ByteSize();
        //输出序列化的内容
        user.SerializeToOstream(&output);
        //输出第二个消息的size
        output << user1.ByteSize();
        //输出序列化的内容
        user1.SerializeToOstream(&output);
        output.close();


        im::user user2,user3;
        //模拟接收端
        fstream input("./stream_file", ios::in | ios::binary);
        int size;
        char * buffer = new char[size+1];
        //从输入流接收size，并读取size个内容到buffer
        input >> size;
        input.read(buffer,size);
        //从buffer中parse出消息实例
        user2.ParseFromArray(buffer,size);
        cout << "id\tuser\tpasswd\tclient" << endl
                 << "-------------------------------\n"
                 << user2.id()
                 << "\t" << user2.user_name()
                 << "\t" << user2.password()
                 << "\t" <<user2.client_type() << endl;
		//获取第二个消息实例
        input >> size;
        input.read(buffer,size);
        user3.ParseFromArray(buffer,size);
        cout << user3.id()
                 << "\t" << user3.user_name()
                 << "\t" << user3.password()
                 << "\t" <<user3.client_type() << endl;
        return 0;

}

```
**编译**
```
g++ mutilmsg.cpp im.user.pb.cc -lprotobuf -o mutilmsg
```
**运行和结果**
```
[vonway@localhost ~]$./mutilmsg
id      user    passwd  client
-------------------------------
1       tom     123     0
2       jim     321     1
```

入门结束

---------
