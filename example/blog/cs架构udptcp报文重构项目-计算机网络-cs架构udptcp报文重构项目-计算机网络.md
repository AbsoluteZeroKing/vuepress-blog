---
title: cs架构udptcp报文重构项目-计算机网络
date: 2020-04-30 14:14:58.0
updated: 2021-02-18 16:01:09.243
url: https://leesdog.space/archives/cs架构udptcp报文重构项目-计算机网络
summary: 整个系统需要完成一个迷你魔兽的搭建，包括游戏逻辑与客户端服务器的网络交互
categories: 
tags: 
- 计算机网络
- UDP 
- TCP 
- Java
- 报文重构
---

## 计算机网络课程设计

### 题目描述

> 1A1B:假想作为“暴雪”公司的首席工程师，你被分配了一个任务——为新的线上竞技游戏设计并实现网络协议。过去的经验告诉你，对项目采用不断演化的开发方法是个好主意，所以你决定首先实施其框架版本“迷你魔兽世界（tiny World of Warcraft ，TWW)”。通过框架的实现，你期望能证实客户端-服务器架构及协议是正确的。
> 2A：你在项目1中实现的“迷你魔兽世界（tiny World of Warcraft ，TWW)”大获成功。此成功带来大量新增用户,从而服务器过载。为了克服此问题，你决定重构你的客户机和服务器以获得所需要的可扩展性。 
> 现在，需要使用多台服务器。每台服务器仅处理地牢(dungeon)的特定地理区域。为了避免数据不一致性的问题，客户端软件要从指定位置取出玩家的状态，并在登陆时提供给合适的服务器。而当玩家从地牢的一块区域移动到另一块区域时，客户端软件要确保其连接到正确的服务器。最后，当玩家登出时, 客户端要把更新的状态信息存储到指定位置。我们将使用跟踪器（tracker)来给客户端提供执行这些任务的必要信息。 
> 在本项目中，你需要修改客户机和服务器以完成前述目标。你还要实现tracker. 


### 系统需求概述

整个系统需要完成一个迷你魔兽的搭建，包括游戏逻辑与客户端服务器的网络交互；在2A中我们需要完成地图的分离处理与交互，在2B中我们需要完成P2P的实现

[1A要求文档](https://leesdog.space/upload/2021/02/1.Warcraft%20-ceb4f41cb4d740df801063fb66c5f7a9.doc)

[2A要求文档](https://leesdog.space/upload/2021/02/2A.Warcraft-21e8f623186c4194b2751fe948b2c53a.doc)

[2B要求文档](https://leesdog.space/upload/2021/02/2B.Warcraft-da6ff1c1d2104f198f6036c9c1f37b25.doc)

### 功能性需求

1.	客户端方的协议功能实现：

①登录：通过玩家的账号登陆，如果没有账号，则创建一个并且登陆。

②移动：朝某个方向移动3个距离

③攻击：攻击某个玩家，这个玩家必须在视线范围内

④广播：广播对话消息

⑤登出：退出游戏，并且保存当前玩家的状态


2.	服务器方的协议功能实现：

①登录响应：响应客户端的登陆，如果没有账号，则创建一个并且登陆。

②移动响应：响应客户端的移动，同时广播移动后的消息

③攻击响应：响应客户端的攻击，这个玩家必须在视线范围内

④广播响应：响应客户端的广播，并广播对话消息至所有在线服务器

⑤登出：响应客户端的登出，并且保存当前玩家的状态

3.	Tracker方的协议功能实现

①登录响应：响应客户端的登陆，以名字hash对登录者划分服务器

②移动响应：响应客户端的移动，当跨服后执行logout+新服务器login

③攻击响应：响应客户端的攻击，这个攻击应该不能跨服

④广播响应：响应客户端的广播，广播消息应能达到跨服效果

### 非功能性需求

1.	对重复登录做出反应
2.	对登录名格式错误做出反应
3.	对命令格式错误做出反应
4.	对已有同名角色登录做出反应
5.	对move后的视野判断问题做出解决
6.	对attack攻击无对象或自己做出处理
7.	对attack攻击超范围做出处理
8.	对speak长度进行控制
9.	对执行其他命令时未login做出反应
10.	2A对服务器进行分流分图后的消息传递进行处理

![计图](https://leesdog.space/upload/2021/02/%E8%AE%A1%E5%9B%BE-651a2290113f4e4e85b92a5599071fcc.png)
![计图2A](https://leesdog.space/upload/2021/02/%E8%AE%A1%E5%9B%BE2A-ce71a7a8d1b540d9abea4e814bdde67e.png)


### Day0x01

今天首先在云服上把环境给配好了

因为是准备使用java，首先将服务器转换成Ubuntu的题目规范系统

然后安装java jdk

首先下载了我所使用的jdk，然后通过Winscp传过去

### 解压安装包jdk-8u202-linux-x64-demos.tar.gz

```
 tar -zxvf jdk-8u161-linux-x64-demos.tar.gz
```

### 将解压后的文件移到/usr/lib目录下

切换到/usr/bin目录下

```
 cd /usr/lib
```

并新建jdk目录

```
 sudo mkdir jdk
```

将解压的jdk文件移动到新建的/usr/lib/jdk目录下来

```
 sudo mv ~/jdk1.8.0_161/usr/lib/jdk
```

执行命令后可到 **usr/lib/jdk** 目录下查看是否移动成功

### 配置java环境变量

这里是将环境变量配置在etc/profile，即为所有用户配置JDK环境。

使用命令打开/etc/profile文件

```
 sudo vim /etc/profile
```

在末尾添加以下几行文字：

```
#set java env
export JAVA_HOME=/usr/lib/jdk/jdk1.8.0_161
export JRE_HOME=${JAVA_HOME}/jre    
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib    
export PATH=${JAVA_HOME}/bin:$PATH
```

### 执行命令使修改立即生效

```
 source /etc/profile
```

![](http://image.leesdog.space/os-1.png)

java环境配置成功





然后今天阅读了一下文档的相关信息，首先了解项目，由于我们要实现通信，首先需要定义一下我们传送的包内容的协议格式，我们已经从socket编程中学过，TCP将所有的消息处理成一个的节流。因此，我们需要生成一个消息头来指定每个消息的长度。

首先我们回顾一下TCP协议的报文格式：

![](http://image.leesdog.space/os-2.png)

　●源、目标端口号字段：占16比特。TCP协议通过使用”端口”来标识源端和目标端的应用进程。端口号可以使用0到65535之间的任何数字。在收到服务请求时，操作系统动态地为客户端的应用程序分配端口号。在服务器端，每种服务在”众所周知的端口”（Well-Know Port）为用户提供服务。 

　●顺序号字段：占32比特。用来标识从TCP源端向TCP目标端发送的数据字节流，它表示在这个报文段中的第一个数据字节。 

　●确认号字段：占32比特。只有ACK标志为1时，确认号字段才有效。它包含目标端所期望收到源端的下一个数据字节。 

　●头部长度字段：占4比特。给出头部占32比特的数目。没有任何选项字段的TCP头部长度为20字节；最多可以有60字节的TCP头部。 

　●标志位字段（U、A、P、R、S、F）：占6比特。各比特的含义如下： 

　◆URG：紧急指针（urgent pointer）有效。 

　◆ACK：为1时，确认序号有效。 

　◆PSH：为1时，接收方应该尽快将这个报文段交给应用层。 

　◆RST：为1时，重建连接。 

　◆SYN：为1时，同步程序，发起一个连接。 

　◆FIN：为1时，发送端完成任务，释放一个连接。 

　●窗口大小字段：占16比特。此字段用来进行流量控制。单位为字节数，这个值是本机期望一次接收的字节数。 

　●TCP校验和字段：占16比特。对整个TCP报文段，即TCP头部和TCP数据进行校验和计算，并由目标端进行验证。 

　●紧急指针字段：占16比特。它是一个偏移量，和序号字段中的值相加表示紧急数据最后一个字节的序号。 

　●选项字段：占32比特。可能包括”窗口扩大因子”、”时间戳”等选项。


根据此格式我们结合文档内容进行协议的定义

1、Version：8位。指定协议的版本号。版本号目前固定为4。

2、Total length：16位的网络字节尾数。指定消息的字节长度，包括包头。

3、Msg type：8位。指定消息的类型。在下面的章节会列举消息的类型。

4、Per-msg payload：可变长度，这取决于消息的类型。

![](http://image.leesdog.space/os-3.png)

明天对TCP消息的传递沟通做具体的客户端服务器的实现

---



### Day0X02

今天我们来实际先构建一下客户端与服务器



首先我们抛弃老师文件里的各种要求，单纯实现一个基于双方通信的多客户端与服务端结构出来

通过之前java的web服务器编写，我们可以基本了解到java中传输信息的方式，主要是利用Socket类以及.net包中的ServerSocket 

**服务器端：**

```java
public void initServer() {


    try {
        //创建服务器对象,并指定端口号
        ServerSocket server = new ServerSocket(9090);
        System.out.println("服务器已经建立......");
        while(true){
            Socket socket =server.accept();
            System.out.println("客户端连接进来了......");
            //当有客户端连接进来以后，开启一个线程，用来处理该客户端的逻辑,
            ServerThread st = new ServerThread(socket);
            st.start();
        //添加该客户端到容器中
            list.add(st);
        }
    } catch (Exception e) {

        e.printStackTrace();
    }
}
```

**客户端**：

```java
public void initClient() {
    Socket client = null;
    try {
        client = new Socket("localhost", 9090);
        final InputStream ins = client.getInputStream();
        final OutputStream ous = client.getOutputStream();

    } catch (IOException e) {
        e.printStackTrace();
    } catch (Exception e) {
        e.printStackTrace();
    }

}
```

这样双方的连接就构建上了，接下来是解决基本通信问题，首先是服务器端发送请输入账号密码的消息并等待接收

**服务器端：**

```java
 public void run() {
        try {
            // 获取输入输出流
            ins = socket.getInputStream();
            ous = socket.getOutputStream();
            // 发送消息给客户端
            String msg = "welcome to lee's server !";
            sendMsg(ous, msg);
            // 发送要求登录信息给客户端
            String userinfo = "please input your name and password ，use& to 分割";
            sendMsg(ous, userinfo);
            // 获取客户端输入的用户名
            String userNamekey = readMsg(ins);
            boolean succeessLoginOrNot = loginCheck(userNamekey);
            while (!succeessLoginOrNot) {
                msg="Fail to connect server......" +
                        "please check your name and password and login again....." +
                        "please input your name and password ，use& to 分割";
                sendMsg(ous, msg);
            // 获取客户端输入的用户名
                userNamekey = readMsg(ins);
                succeessLoginOrNot = loginCheck(userNamekey);	//检查数据库
            }
            try {
                ins.close();
                ous.close();
                socket.close();
            //将当前已经关闭的客户端从容器中移除
                MyServer.list.remove(this)
            } catch (IOException e) {
                e.printStackTrace();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
```

**客户端：**

```java
 String result = readMsg(ins);
 HnadleMessage(result);	//处理信息
 public String readMsg(InputStream ins) throws Exception {
        int value = ins.read();
        String str = "";
        while (value != 10) {
		// 代表客户单不正常关闭
            if (value == -1) {
                throw new Exception();
            }
            str = str + (char) value;
            value = ins.read();
        }
        str = str.trim();
        return str;
    }
```

至此，一个基本的双方的多线程通信构建完成



但我们实际上的游戏服务器与客户端之间的通信并非这么简单，虽然我们看似解决了双方的通信问题，但在真实游戏情况下是**三方甚至更多方通信**，也就是服务器同时处理多个客户端消息，并且需要达到一个**异步正确处理消息**的要求，也就是在网络速度不一的情况下，能根据消息的时间戳决定消息的处理结果，例如，一个玩家A攻击了玩家B，A攻击时B本来是在A的攻击范围内的，但由于消息发送较慢，这时B进行了移动且消息更快到达，这就会导致A攻击不到B，但在实际情况下A确实是应该能攻击到的，这种情况也要做一些处理；这些内容的实现我们需要依靠相关协议来实现;同时，这一次我们的消息是基于位的协议构造的消息，而不是简单的socket消息，这也是需要解决的问题



正如我们上次所研究的tcp结构，具体在此项目中，我们所需构建的报文结构如下图

![](http://image.leesdog.space/os-4.png)

其中IP头和TCP头是可以通过javaSocket编程自动完成的，我们所需要实现的就是后面两个东西，结合前面所学到的知识我们可以知道，包头消息的长度均为32位，故之后我们所处理的消息均为以下这种格式

![](http://image.leesdog.space/os-5.png)



具体来讲我们需要处理以下这几种消息

- LOGIN_REQUEST

- LOGIN_REPLY

- Move

- MOVE_NOTIFY

- ATTACK

- ATTACT_NOTIFY

- SPEAK

  我们对这些消息构建一个枚举类型的类，之后根据消息编号进行分类处理

  ```java
  public enum Message {
      LOGIN_REQUEST,
      LIGIN_REPLY,
      MOVE_NOTIFY,
      ATTACT_NOTIFY,
      MOVE,
      ATTACK,
      SPREAK,
      LOGOUT
  }
  ```

  和我们之前写的代码的处理不一样的是，这里我们应该根据tcp包头的消息内容中**消息编号**的不同来调用不同的函数，并将此消息以**函数参数**形式传递给该函数供该函数继续解析，而不是像我们之前那样直接根据传入的字符串进行解析分类，消息编号由于上图协议的规定**必定在**32位信息的**25-32位处**，而其他信息则由不同的函数进行不同的解析方式

明天我们来实现以上消息的解析分类与实现

---



### Day0X03

昨天我们实现了基础的Socket多线程通信，今天我们来实现消息的解析分类与实现



首先是login_request

- LOGIN_REQUEST

  ![](http://image.leesdog.space/os-6.png)

  ![](http://image.leesdog.space/os-7.png)

  根据以上的协议我们可以知道整个tcp正文内我们需要构建的大小是8×4×4位的大小的信息，其中如果我们将所有信息都=协议都看一下的话会发现第一行的内容也就是第一个8×4的内容恒定是：

  - Version：8位。指定协议的版本号。版本号目前固定为4。

  - Total length：16位的网络字节尾数。指定消息的字节长度，包括包头。

  - Msg type：8位。指定消息的类型。

所以为了之后其他消息的实现，我们可以做一些处理：

**客户端：**

```java
while(true)
{
    Scanner sc=new Scanner(System.in);
    String ctype=sc.nextLine();
    MessageType type=judgeMessageType(ctype);
    handleMessage(ins,ous,type,sc);
}
```

同时我们需要重写一下我们之前所写的客户端代码，以适应协议

```java
public static void main(String[]args) {
    Socket client = null;
    try {
        client = new Socket("localhost", 9090);
        System.out.println("客户端已经建立......");
        final InputStream ins = client.getInputStream();
        final OutputStream ous = client.getOutputStream();
        }
```

```java
public static MessageType judgeMessageType(String type)
{
    switch (type)
    {
        case "login":
        {
            return MessageType.LOGIN_REQUEST;
        }
        case "move":
        {
            return MessageType.MOVE;
        }
        case "attack":
        {
            return MessageType.ATTACK;
        }
        case "speak":
        {
            return MessageType.SPREAK;
        }
        case "logout":
        {
            return MessageType.LOGOUT;
        }
        default:
            return MessageType.UNCLEAR_MESSAGE;

    }
}
```

```java
public static void handleMessage(InputStream ins, OutputStream ous, MessageType messageType,Scanner sc)
{
    if(messageType==MessageType.LOGIN_REQUEST)
    {
        System.out.println("请输入账号");
        String account=sc.nextLine();
        login(account,ous);
    }
    else if(messageType==MessageType.UNCLEAR_MESSAGE)
    {
        return;
    }
}
```

最后我们通过handleMessage判断MessageType消息信息的种类，然后调用不同的方法，例如我们现在实现的login：

```java
private static void login(String account, OutputStream ous)
{
    byte name[]=new byte[10];	//最大为9字节，最后一位为空
    char len=0X0010;			//login函数固定消息长度为16位
    char padding=0;				//login函数的padding为16位
    int i=0;
    for(char a:account.toCharArray())
    {
        name[i]=(byte) a;
        i++;
    }

    try {
        ous.write(VERSION);		//将信息写入ous后传递给server
        ous.write(len);
        ous.write(name);
        ous.write(padding);
        ous.flush();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**服务端：**

我们同样需要重写一下服务端程序以适应程序要求

```java
try {
    //创建服务器对象,并指定端口号
    ServerSocket server = new ServerSocket(9090);
    System.out.println("服务器已经建立......");
    while(true)
    {
        Socket socket =server.accept();
        System.out.println("客户端连接进来了......");
        //当有客户端连接进来以后，开启一个线程，用来处理该客户端的逻辑,
        ServerThread st = new ServerThread(socket);
        st.start();
    //添加该客户端到容器中
        list.add(st);
    }
} catch (Exception e) {

    e.printStackTrace();

}
```

同时完成一下与客户端的交互部分：

```java
public void run() {
    try {
        // 获取输入输出流
        ins = socket.getInputStream();
        ous = socket.getOutputStream();

        PrintWriter printWriter=new PrintWriter(ous);
        printWriter.println("请登录");
        ous.flush();
        printWriter.close();
        while(true)
        {
            ins=socket.getInputStream();
            ous=socket.getOutputStream();
            boolean ifError= handleMessage(ins);
            if(ifError==false)
            {
                break;
            }
        }
        try {
            ins.close();
            ous.close();
            socket.close();
        //将当前已经关闭的客户端从容器中移除
            MyServer.list.remove(this);
        } catch (IOException e) {
            e.printStackTrace();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }

}
```

同时由于协议前32位固定，故我们的解析函数可以先进行统一解析：

```java
private boolean handleMessage(InputStream ins) throws IOException {
    byte[] head=new byte[4];
    head= this.ins.readNBytes(4);
    if(head[0]==VERSION)
    {
        short len1= (short) (head[1]<<8);
        short len= (short) (len1+head[2]);
        byte messageType=head[3];
        byte[]content=new byte[len];
        content=this.ins.readNBytes(len);
        if(messageType==1)  //登录
        {
            loginReply(content,ous);
        }
    }
    else
    {
        //versionWrong();
        return false;
    }
    return false;
}
```

当messageType为1时调用loginReply回复客户端消息：

```java
private void loginReply(byte[] content, OutputStream ous) {
    String name="";
    for(int i=0;i<9;i++)    //读取名字
    {
        name+=content[i];
    }
    if(MyServer.namelist.contains(name))
    {
        //构建login_reply errorcode=1的消息
    }
    else
    {
        //构建login_reply errorcode=0的消息
    }

}
```

这样基本的函数交互就完成了，后续我们需要以类似格式完成其他函数的交互

ps：由于今天去老家看望了老人，所以实现内容少于预期，争取明天补回来

---

### Day0x04-06

这几天放假的功夫加上周一，我将之前的1A里的功能全部进行了一些实现，同时修正了一些可能出现的bug,也重构了一些代码，具体情况如下：

首先，针对**服务端**，仔细思考后决定将结构重构

首先定义三个数据结构用于存储玩家信息等

```java
public static ArrayList<Client> clients = new ArrayList<>();        //客户端信息
public static Map<String,Boolean> namelist = new LinkedHashMap<>(); //角色是否在线
public static Map<String,player>playerdata=new LinkedHashMap<>();   //角色的属性，player为自定义类型
```

MyServer先通过main函数调用init

```java
public static void main(String[] args) {
    new MyServer().init();

}
```

init中**多线程创建**服务器对象，同时将服务器存入clients的map中

```java
try {
    //创建服务器对象,并指定端口号
    ServerSocket serverSocket = new ServerSocket(9090);
    Socket socket=null;
    Client client=null;
    System.out.println("服务器已经建立......");
    while(true)
    {
        socket =serverSocket.accept();
        System.out.println("连接上客户端");
        client=new Client(socket);
        clients.add(client);
        new Thread(client).start();
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

单独再写一个Client类继承runnable接口放在上面的Myserver里，**对应一个客户端**，客户端登录后获得人物相关参数

```java
private class Client implements Runnable
```

```java
private String playername=null;
private int hp=0;
private int exp=0;
private int x=0;
private int y=0;
private Socket socket = null;
InputStream inputStream = null;
DataInputStream dataInputStream = null;
OutputStream outputStream = null;
DataOutputStream dataOutputStream = null;
boolean flag = true;
public boolean isLogin=false;
```

然后在构造函数中初始化流参数

```java
public Client(Socket socket)
{
    this.socket=socket;
    try
    {
        inputStream=socket.getInputStream();
        outputStream=socket.getOutputStream();
    } catch (IOException e) { System.out.println("消息接收失败"); e.printStackTrace(); }
    dataOutputStream=new DataOutputStream(outputStream);
    dataInputStream=new DataInputStream(inputStream);
}
```

在run中循环执行read，直到关闭客户端

```java
public void run()
{
    String message;

    while(flag)
    {
        try
        {
            if(dataInputStream!=null)
            {
                message=dataInputStream.readUTF();
                handleMessage(message);
            }
        }  catch (SocketException e) {
            flag = false;
            System.out.println("客户下线");
            clients.remove(this);
            namelist.put(playername,false);
            isLogin=false;
            // e.printStackTrace();
        } catch (EOFException e) {
            flag = false;
            System.out.println("客户下线");
            clients.remove(this);
            namelist.put(playername,false);
            isLogin=false;
            // e.printStackTrace();
        } catch (IOException e) {
            flag = false;
            System.out.println("接受消息失败");
            clients.remove(this);
            namelist.put(playername,false);
            isLogin=false;
            e.printStackTrace();
        }
```



然后针对**客户端**，为了方便之后写成后的调试工作，将大类MyClient继承runnable接口，通过另外的类多次调用他来实现多个玩家的登录模拟，每个客户端有下列参数：

``` java
public static final byte VERSION=0X04;   //版本号
private DataInputStream dataInputStream = null;
private DataOutputStream dataOutputStream = null;
private InputStream inputStream = null;
private OutputStream outputStream = null;
private Socket socket = null;
private boolean flag = false;
private String playername=null;
private int HP;
private int EXP;
private int X;
private int Y;
public  Map<String, player> playerseen=new LinkedHashMap<>();   //能被看到的角色
```

客户端内为了实现**接收消息与发送消息的并发**，我们将这两类操作分离开来分别继承runnable接口

``` java
private class ReciveMessage implements Runnable

private class SendMessage implements Runnable
```

然后对不同操作进行分类处理

接下来对每一个具体的功能进行**从客户端到服务器的交互解析**：



#### Login全过程

- 首先服务器开启等待连接，客户端调用connect函数**连接**，然后**启动接发消息的线程**：

``` java
private void connect() {
    try {
        socket = new Socket("localhost", 9090);
        System.out.println("成功与服务器建议连接");
        outputStream = socket.getOutputStream();
        dataOutputStream = new DataOutputStream(outputStream);
        inputStream = socket.getInputStream();
        dataInputStream = new DataInputStream(inputStream);
    }catch (IOException e) {
        System.out.println("申请链接失败");
        e.printStackTrace();
    }
}
```

``` java
private void init()
{
    connect();//首先进行连接
    new Thread(new ReciveMessage()).start();//启动接收消息的线程
    new Thread(new SendMessage()).start();//启动发送消息的线程

}
```

- 然后ReciveMessage线程阻塞等待接收服务器端消息，sendMessage线程等待命令行输入指令

此时按格式输入： login playername

``` java
private class SendMessage implements Runnable
{
    Scanner sc=null;
    public void run()
    {
        while(true)
        {
            sc=new Scanner(System.in);
            String command=sc.nextLine();
            handleCommand(command);
        }

    }
```

进入到上述handleCommand函数中，对于login：按空格分离命令，获得后面的playername名，然后调用login函数传入名字

```java
private void handleCommand(String command) {
    if(command.contains("login"))
    {
        String[] names=command.split(" ");
        if(names.length>1)
        {
            String name=names[1];
            login(name);
        }
        else
        {
            System.out.println("输出格式有误，格式为: login <player_name>");
        }
        .......
```

login函数按login协议构造报文，注意这里的位操作：

```java
private  void login(String account) {
    byte[] mes=new byte[16];	//一共是16个8位
    mes[0]=VERSION;				//1字节用于存版本号
    mes[1]=0;					//2,3字节存总长度，由于login指令定长，故为此
    mes[2]=0x0010;
    mes[3]=0x01;				//4字节存指令编号，0x01代表login
    int i=4;					//后面存名字，最大为10字节（9字节其实，最后一位必须为空）
    byte[]na=new byte[10];
    for(char c:account.toCharArray())
    {
        byte b=(byte) c;
        mes[i]=b;
        na[i-4]=b;
        i++;
        if(i==14)
            break;
    }
    playername=new String(na);
    sendMessageToServer(new String(mes));	//调用send将构造的报文传到服务端
}
```

最后调用sendmessageToserver传输文本

```java
private void sendMessageToServer(String message) {
    try {
        dataOutputStream.writeUTF(message);
        dataOutputStream.flush();
    } catch (IOException e) {
        System.out.println("发送消息失败");
        e.printStackTrace();
    }
}
```

此时server接收到消息，下图处解除阻塞，调用服务器的handleMessage

```java
public void run()
{
    String message;

    while(flag)
    {
        try
        {
            if(dataInputStream!=null)
            {
                message=dataInputStream.readUTF();	//等待读入客户端发来的数据
                handleMessage(message);
            }
        } 
        ......
```

handleMessage根据接收到的指令进行解析：

```java
public  void handleMessage(String message)
{
    byte[] byteMessage=message.getBytes();	//首先用字节的形式转换信息
    byte version=byteMessage[0];			//得到版本
    char len= (char) ((char) (byteMessage[1]<<8)+byteMessage[2]);//得到长度
    byte msgType=byteMessage[3];			//得到信息类型

    if(version==VERSION)    //版本一致
    {
        //login_request处理
        if(msgType==0x01)
        {
            byte[]tempName=new byte[]{byteMessage[4],byteMessage[5],byteMessage[6],byteMessage[7],byteMessage[8],byteMessage[9],byteMessage[10],byteMessage[11],byteMessage[12],byteMessage[13]};	//获取到名字
            String name=new String(tempName);		//将名字转成string
            if(isLogin==true)//登录阶段以后登录，发送0x0b的1号错误信息
            {
                byte[] mes=new byte[8];
                mes[0]=VERSION;
                mes[1]=0;
                mes[2]=0x0008;
                mes[3]=0x0b;
                mes[4]=1;
                try {
                    sendToClient(new String(mes));
                } catch (IOException e) { e.printStackTrace(); }
                return;
            }

            if(namelist.get(name)!=null)    //信息库存在该角色
            {
                if(namelist.get(name)==true)//该角色在线，发送0x02的1号错误信息
                {
                    byte[] mes=new byte[16];
                    mes[0]=VERSION;
                    mes[1]=0;
                    mes[2]=0x0010;
                    mes[3]=0x02;
                    mes[4]=0x01;
                    try {
                        sendToClient(new String(mes));
                    } catch (IOException e) { e.printStackTrace(); }
                    return;
                }
                else                        //该角色不在线，则从map中读入该角色信息
                {
                    player currentPlayer = playerdata.get(name);
                    byte[] mes=new byte[16];
                    mes[0]=VERSION;
                    mes[1]=0;
                    mes[2]=0x0010;
                    mes[3]=0x02;
                    mes[4]=0x00;
                    hp=currentPlayer.HP;
                    mes[5]= (byte) ((hp&0xff000000)>>24);
                    mes[6]= (byte) ((hp&0xff0000)>>16);
                    mes[7]= (byte) ((hp&0xff00)>>8);
                    mes[8]= (byte) (hp&0xff);

                    exp=currentPlayer.EXP;
                    mes[9]= (byte) ((exp&0xff000000)>>24);
                    mes[10]= (byte) ((exp&0xff0000)>>16);
                    mes[11]= (byte) ((exp&0xff00)>>8);
                    mes[12]= (byte) (exp&0xff);

                    mes[13]= (byte) currentPlayer.x;
                    mes[14]= (byte) currentPlayer.y;
                    playername=name;
                    isLogin=true;	//将此线程对应客户端设置为已登录状态
                    namelist.put(name,true);
                    try { sendToClient(new String(mes));
                    } catch (IOException e) { e.printStackTrace(); }
//将该角色信息以move_notify形式发送给其他角色，参数1代表发送给自己以外其他角色
                    move_notify(playername,1);	
                    for(Map.Entry<String,player>entry:playerdata.entrySet())
                    {
                        move_notify(entry.getKey(),0);	//将其他角色信息以move_notify发送给自己，参数0代表发送给自己
                    }
                    return;
                }
            }
            //没有该角色，则创建该角色，并随机初始数值
            namelist.put(name,true);
            playername=name;
            isLogin=true;
            Random r = new Random();
            hp=100+r.nextInt(20);
            exp=0;
            x=r.nextInt(100);
            y=r.nextInt(100);
            player currentPlayer=new player(hp,exp,x,y);
            playerdata.put(name,currentPlayer);

            byte[] mes=new byte[16];
            mes[0]=VERSION;
            mes[1]=0;
            mes[2]=0x0010;
            mes[3]=0x02;
            mes[4]=0x00;
            mes[5]= (byte) ((hp&0xff000000)>>24);
            mes[6]= (byte) ((hp&0xff0000)>>16);
            mes[7]= (byte) ((hp&0xff00)>>8);
            mes[8]= (byte) (hp&0xff);

            mes[9]= (byte) ((exp&0xff000000)>>24);
            mes[10]= (byte) ((exp&0xff0000)>>16);
            mes[11]= (byte) ((exp&0xff00)>>8);
            mes[12]= (byte) (exp&0xff);

            mes[13]= (byte) x;
            mes[14]= (byte) y;
            try {
                sendToClient(new String(mes));
            } catch (IOException e) { e.printStackTrace(); }

            move_notify(playername,1);//同上
            for(Map.Entry<String,player>entry:playerdata.entrySet())
            {
                move_notify(entry.getKey(),0);
            }
            return;
        }
```



此时服务器接收到客户端的应答消息，reciveMessage解除阻塞，调用客户端的handleMessage处理消息：

```java
private class ReciveMessage implements Runnable
{
    @Override
    public void run() {
        flag = true;
        try {
            while (flag) {
                String message = dataInputStream.readUTF();//读取消息
                handleMessage(message);
            }
            ......
```

```java
public void handleMessage(String message)
{
    byte[] byteMessage=message.getBytes();
    byte version=byteMessage[0];
    char len= (char) ((char) (byteMessage[1]<<8)+byteMessage[2]);
    byte msgType=byteMessage[3];

    if(version==VERSION)    //版本一致
    {
        if(msgType==0x02)//如果接收到的消息是 login_reply
        {
            if(byteMessage[4]==0)	//如果是0号错误信息，则为正确，按发送过来的信息初始化角色信息并提示
            {
                System.out.println("Welcome to the tiny world of warcraft.");
                int hp1=byteMessage[5]<<24;
                int hp2=byteMessage[6]<<16;
                int hp3=byteMessage[7]<<8;
                int hp4=byteMessage[8];
                HP=hp1+hp2+hp3+hp4;
                System.out.println("your HP:"+HP);
                int exp1=byteMessage[9]<<24;
                int exp2=byteMessage[10]<<16;
                int exp3=byteMessage[11]<<8;
                int exp4=byteMessage[12];
                EXP=exp1+exp2+exp3+exp4;
                X=byteMessage[13];
                Y=byteMessage[14];
                System.out.println("your EXP:"+EXP);
                System.out.println("your location: X="+X);
                System.out.println("your location: Y="+Y);
                System.out.println("--------------------------------------------------------------");
            }
            else if(byteMessage[4]==1)	//如果是1号错误信息，说明已有同名角色，按协议提示
            {
                System.out.println("A play with the same name is already in the game.");
            }
        }
        ....
```

至此，完整的login交互完成，下面是动图演示，包含了登录，重复登录处理，多客户端登录请求：

![](http://image.leesdog.space/jwlogin1.gif)



---



#### move全过程

接下来是move，首先是客户端调用move指令，格式为move <north|south|east|west>

和上面login一样，所以这里我就直接进入到处理过程了：

```java
private void handleCommand(String command) {
    ......
    if(command.contains("move"))
    {
        //System.out.println("请输入方向对应的编号，0为North北，1为South南，2为East东，3为West西");
        int direction=0;
        String[] directions=command.split(" ");
        if(directions.length>1)
        {
            String comm=directions[1];
            if(comm.equals("north"))
                direction=0;
            else if(comm.equals("south"))
                direction=1;
            else if(comm.equals("east"))
                direction=2;
            else if(comm.equals("west"))
                direction=3;
            move(direction);	//最终以move指令int传出
        }
        else
        {
            System.out.println("输出格式有误，格式为: move <north|south|east|west>");
        }
    }
    ......
```

```java
private void move(int direction) {
    byte[]mes=new byte[8];//构造一个8字节的报文
    mes[0]=VERSION;
    mes[1]=0;
    mes[2]=0x0008;
    mes[3]=0x03;
    mes[4]= (byte) direction;
    sendMessageToServer(new String(mes));
}
```

消息传到服务器，服务器这边对其做处理：

```java
//move
if(msgType==0x03)	
{
    if(isLogin==true//登录了
    {
        move(byteMessage[4]);	//调用服务器端的move函数
    }
    else	//未登录就使用指令，构建0x0b报文并设置0错误参数返回
    {
        byte[] m=new byte[8];
        m[0]=VERSION;
        m[1]=0;
        m[2]=0x0008;
        m[3]=0x0b;
        m[4]=0;
        try {
            sendToClient(new String(m));
        } catch (IOException e) { e.printStackTrace(); }
    }
    return;
}
```

```java
private void move(byte b) {	//根据不同的int方向进行处理

    if(b==0)//North
    {
        y+=3;
        if(y>99)//过99则-100，因为地图为环形
        {
            y-=100;
        }
        player currentPlayer= playerdata.get(playername);	//获得当前角色的属性信息，并对x，y进行相应修改
        if(currentPlayer!=null)
        {
            currentPlayer.y=y;
        }
    }
    else if(b==1)//South
    {
        y-=3;
        if(y<0)
        {
            y+=100;
        }
        player currentPlayer= playerdata.get(playername);
        if(currentPlayer!=null)
        {
            currentPlayer.y=y;
        }
    }
    else if(b==2)//East
    {
        x-=3;
        if(x<0)
        {
            x+=100;
        }
        player currentPlayer= playerdata.get(playername);
        if(currentPlayer!=null)
        {
            currentPlayer.x=x;
        }
    }
    else if(b==3)//West
    {
        x+=3;
        if(x>99)
        {
            x-=100;
        }
        player currentPlayer= playerdata.get(playername);
        if(currentPlayer!=null)
        {
            currentPlayer.x=x;
        }
    }
    else
    {
        return;
    }
    move_notify(playername,2);	//最后调用move_notify返回消息给客户端
}
```

```java
private void move_notify(String name,int model) {
    byte[] mes=new byte[24];
    mes[0]=VERSION;
    mes[1]=0;
    mes[2]=0x0018;
    mes[3]=0x04;
    int i=4;
    for(char c:name.toCharArray())//存入角色名
    {
        byte b=(byte) c;
        mes[i]=b;
        i++;
        if(i==14)
            break;
    }
    player currentPlayer=playerdata.get(name);	//获得该角色信息并存入
    mes[14]= (byte) currentPlayer.x;
    mes[15]= (byte) currentPlayer.y;
    int chp=currentPlayer.HP;
    int cexp=currentPlayer.EXP;

    mes[16]= (byte) ((chp&0xff000000)>>24);
    mes[17]= (byte) ((chp&0xff0000)>>16);
    mes[18]= (byte) ((chp&0xff00)>>8);
    mes[19]= (byte) (chp&0xff);

    mes[20]= (byte) ((cexp&0xff000000)>>24);
    mes[21]= (byte) ((cexp&0xff0000)>>16);
    mes[22]= (byte) ((cexp&0xff00)>>8);
    mes[23]= (byte) (cexp&0xff);
    try {
        if(model==0)
        sendToClient(new String(mes));
        if(model==1)
            sendToAllClientsOutOfSelf(new String(mes));
        if(model==2)				//此时参数为2，即发送此消息给所有客户端
            sendToAllClients(new String(mes));
    } catch (IOException e) { e.printStackTrace(); }

}
```

```java
private void sendToAllClients(String message) throws IOException {
    for (Client c : clients) {
            outputStream = c.socket.getOutputStream();
            dataOutputStream = new DataOutputStream(outputStream);
            sendToClient(message);
    }
}
```

此时客户端接收到move_notify消息：

```java
......
else if(msgType==0x04)//move_notify
{

    byte[]tempName=new byte[]{byteMessage[4],byteMessage[5],byteMessage[6],byteMessage[7],byteMessage[8],byteMessage[9],byteMessage[10],byteMessage[11],byteMessage[12],byteMessage[13]};//提取名字与坐标
    byte locationX=byteMessage[14];
    byte locationY=byteMessage[15];

    int hp1=byteMessage[16]<<24;	//提取各属性信息
    int hp2=byteMessage[17]<<16;
    int hp3=byteMessage[18]<<8;
    int hp4=byteMessage[19];
    int hp=hp1+hp2+hp3+hp4;

    int exp1=byteMessage[20]<<24;
    int exp2=byteMessage[21]<<16;
    int exp3=byteMessage[22]<<8;
    int exp4=byteMessage[23];
    int exp=exp1+exp2+exp3+exp4;

    if(new String(tempName).equals(playername))//接到自己的数据
    {
        HP=hp;
        EXP=exp;
        X=locationX;
        Y=locationY;
        System.out.println(playername.trim()+ ": location=(" +X+","+Y+"), HP="+HP+", EXP="+EXP);
    }
    else//接收到其他人的数据
    {
        if(locationX<=X+5&&locationX>=X-5&&locationY<=Y+5&&locationY>=Y-5)  //视线范围内的消息才做反应
        {
            player p=new player(hp,exp,locationX,locationY);
            playerseen.put(new String(tempName).trim(),p);
            System.out.println(new String(tempName).trim()+ ": location=(" +locationX+","+locationY+"), HP="+hp+", EXP="+exp);
        }
    }
    return;
}
.......
```

至此，完整的move全过程实现完成，下面是动图演示，包含move各个方向以及部分错误处理；第二张图演示move后发现其他角色：
![1](http://image.leesdog.space/jwmove1.gif)
![](http://image.leesdog.space/jwmove2.gif)





![](http://image.leesdog.space/jwmove3.gif)



---

#### attack全过程

接下来是attack的全过程，首先客户端输入相应指令attack <player_name>

```java
else if(command.contains("attack"))
{
    String people=null;
    String[]attacks=command.split(" ");
    if(attacks.length>1)
    {
        people=attacks[1];
        attack(people);	//获得要攻击的人的名字，调用attack
    }
    else
    {
        System.out.println("输出格式有误，格式为: attack <player_name>");
    }

}
```

调用attack函数

```java
private void attack(String people) {
    if(playerseen.containsKey(people))	//如果自己的可视表里有该角色才能执行攻击操作
    {
        byte[]mes=new byte[16];//构造报文信息
        mes[0]=VERSION;
        mes[1]=0;
        mes[2]=0x0010;
        mes[3]=0x05;
        int i=4;
        for(char c:people.toCharArray())
        {
            byte b=(byte) c;
            mes[i]=b;
            i++;
            if(i==14)
                break;
        }
        sendMessageToServer(new String(mes));//发送至服务器
    }
    else
    {
        System.out.println("The target is not visible.");
    }

}
```

服务器这边进行处理：

```java
......
else if(msgType==0x05)
{
    if(isLogin==true)
    {
        byte[]tempName=new byte[]{byteMessage[4],byteMessage[5],byteMessage[6],byteMessage[7],byteMessage[8],byteMessage[9],byteMessage[10],byteMessage[11],byteMessage[12],byteMessage[13]};
        attack(tempName);
    }
    else
    {
        byte[] m=new byte[8];
        m[0]=VERSION;
        m[1]=0;
        m[2]=0x0008;
        m[3]=0x0b;
        m[4]=0;
        try {
            sendToClient(new String(m));
        } catch (IOException e) { e.printStackTrace(); }
    }
    return;
}
......
```

然后调用服务器的attack

```java
private void attack(byte[] tempName) {
    String people=new String(tempName);
    Random r = new Random();	//生成一个范围内的随机伤害
    int damage=10+r.nextInt(10);
    playerdata.get(playername).EXP+=damage;
    playerdata.get(people).HP-=damage;
    attack_notify(people,damage,playerdata.get(people).HP);
    if(playerdata.get(people).HP<=0)    //死亡
    {
        playerdata.get(people).HP=30+r.nextInt(20);
        playerdata.get(people).x=r.nextInt(100);
        playerdata.get(people).y=r.nextInt(100);
        move_notify(people,2);	//发送该角色新状态消息
    }

}
```

最后调用attack_notify回传消息给客户端：

```java
private void attack_notify(String people,int damage,int HP) {
    byte[]mes=new byte[32];
    mes[0]=VERSION;
    mes[1]=0;
    mes[2]=0x0020;
    mes[3]=0x06;
    int i=4;
    for(char c:playername.toCharArray())
    {
        byte b=(byte) c;
        mes[i]=b;
        i++;
        if(i==14)
            break;
    }
    for(char c:people.toCharArray())
    {
        byte b=(byte) c;
        mes[i]=b;
        i++;
        if(i==24)
            break;
    }
    mes[24]= (byte) damage;
    mes[25]= (byte) ((HP&0xff000000)>>24);
    mes[26]= (byte) ((HP&0xff0000)>>16);
    mes[27]= (byte) ((HP&0xff00)>>8);
    mes[28]= (byte) (HP&0xff);
    try {
        sendToAllClients(new String(mes));
    } catch (IOException e) { e.printStackTrace(); }

}
```

然后客户端这边接收到attack_notify消息,处理发送过来的内容

```java
 else if(msgType==0x06)//attack_notify
                {
                    byte[]attackerName=new byte[]{byteMessage[4],byteMessage[5],byteMessage[6],byteMessage[7],byteMessage[8],byteMessage[9],byteMessage[10],byteMessage[11],byteMessage[12],byteMessage[13]};
                    byte[]victimName=new byte[]{byteMessage[14],byteMessage[15],byteMessage[16],byteMessage[17],byteMessage[18],byteMessage[19],byteMessage[20],byteMessage[21],byteMessage[22],byteMessage[23]};
                    int damage=byteMessage[24];
                    int hp1=byteMessage[25]<<24;
                    int hp2=byteMessage[26]<<16;
                    int hp3=byteMessage[27]<<8;
                    int hp4=byteMessage[28];
                    int hp=hp1+hp2+hp3+hp4;
                    String vic=new String(victimName).trim();
                    String att=new String(attackerName).trim();
                    if(  (playerseen.containsKey(vic)&&(playerseen.containsKey(att))  ) ||  ( att.contains(playername.trim())||vic.contains(playername.trim())  )       )//两者均在视线内或者两者之一是自己
                    {
                        if(hp>0)
                        {
                           // <attacker name> damaged <victim name> by <damage>. <victim name>’s HP is now <HP>.
                            System.out.println(new String(attackerName).trim()+" damage "+new String(victimName).trim()+" by "+damage+". "+new String(victimName).trim()+"'s HP is now "+hp+".");
                        }
                        else
                        {
                            //<attacker name> killed <victim name>.
                            System.out.println(new String(attackerName).trim()+" killed "+new String(victimName).trim()+" . ");
                        }
                    }

                }
```

至此，attack全过程实现完毕，下面是演示动图，包括了基本的攻击，攻击视野外玩家，攻击致死等情况：

![](http://image.leesdog.space/jwmove4.gif)

---

#### speak全过程

speak全过程相对比较简单，不过需要注意的是前面的指令全是定长指令，而speak为非定长指令

首先客户端调用speak获取其后面的内容

```java
else if(command.contains("speak"))
{
    String message=null;
    String[]speaks=command.split(" ");
    if(speaks.length>1)
    {
        message=speaks[1];
        speak(message);
    }
}
```

```java
private void speak(String message) {
    byte[]mes=new byte[message.length()+4];
    mes[0]=VERSION;
    if(message.length()<=255)
    {
        char len= (char) message.length();
        mes[1]= (byte) ((len&0xff00)>>8);	//将长度转换成两个字节表示
        mes[2]= (byte) (len&0xff);
        mes[3]=0x07;
        int i=4;
        for(char c:message.toCharArray())
        {
            byte b=(byte) c;
            mes[i]=b;
            i++;
        }
        sendMessageToServer(new String(mes));
    }

}
```

调用完speak将信息传到服务器，服务器进行处理：

```java
//speak
else if(msgType==0x07)
{
    if(isLogin==true)
    {
        byte[]speakmessage=new byte[len];
        for(int i=0;i<len;i++)
        {
            speakmessage[i]=byteMessage[4+i];	//4为头部的
        }
        speak_notify(speakmessage);
    }
```

然后调用speak_notify

```java
private void speak_notify(byte[] speakmessage) {
    int len=speakmessage.length+14;//构建报文信息，加的14是头部信息4字节+发送者名字10字节
    byte[]mes=new byte[len];
    mes[0]=VERSION;
    mes[1]= (byte) ((len&0xff00)>>8);
    mes[2]= (byte) (len&0xff);
    mes[3]=0x08;
    int i=4;
    for(char c:playername.toCharArray())
    {
        byte b=(byte) c;
        mes[i]=b;
        i++;
        if(i==14)
            break;
    }
    for(;i<len;i++)
    {
        mes[i]=speakmessage[i-14];
    }
    try {
        sendToAllClients(new String(mes));
    } catch (IOException e) { e.printStackTrace(); }
}
```

最后传回客户端进行处理显示：

```java
//speak_notify
else if(msgType==0x08)
{
    byte[]MES=new byte[len-14];
    byte[]speakerName=new byte[]{byteMessage[4],byteMessage[5],byteMessage[6],byteMessage[7],byteMessage[8],byteMessage[9],byteMessage[10],byteMessage[11],byteMessage[12],byteMessage[13]};
    for(int i=14;i<len;i++)
    {
        MES[i-14]=byteMessage[i];
    }
    System.out.println(new String(speakerName).trim()+":"+new String(MES).trim());
}
```

至此，speak全过程实现完毕，下面是动图演示多客户端情况下的信息交互：

![](http://image.leesdog.space/jwmove5.gif)

---

#### logout全过程

首先客户端调用logout指令

```java
else if(command.contains("logout"))
{
    logout();
}
```

按协议构建报文0x09号报文

```java
private void logout() {
    byte[]mes=new byte[4];
    mes[0]=VERSION;
    mes[1]=0;
    mes[2]=0x0004;
    mes[3]=0x09;
    sendMessageToServer(new String(mes));
}
```

转发到服务器，服务器这边进行相关处理：

```java
//logout
else if(msgType==0x09)
{
    if(isLogin==true)
    {
        logout_notify();	//直接调用logout_notify
    }
    else
    {
        byte[] m=new byte[8];
        m[0]=VERSION;
        m[1]=0;
        m[2]=0x0008;
        m[3]=0x0b;
        m[4]=0;
        try {
            sendToClient(new String(m));
        } catch (IOException e) { e.printStackTrace(); }
    }

    return;
}
```

```java
private void logout_notify() {
    byte[]mes=new byte[16];
    mes[0]=VERSION;
    mes[1]=0;
    mes[2]=0x0010;
    mes[3]=0x0a;
    namelist.put(playername,false);//将该玩家设为离线
    isLogin=false;					//将该客户端设为未登录状态
    int i=4;
    for(char c:playername.toCharArray())
    {
        byte b=(byte) c;
        mes[i]=b;
        i++;
        if(i==14)
            break;
    }
    try {
        sendToAllClients(new String(mes));	//告知所有客户端
    } catch (IOException e) { e.printStackTrace(); }
}
```

最后返回到客户端进行广播，注意**删除客户端内该可视角色的信息**

```java
else if(msgType==0x0a)
{
    byte[]loggoutName=new byte[]{byteMessage[4],byteMessage[5],byteMessage[6],byteMessage[7],byteMessage[8],byteMessage[9],byteMessage[10],byteMessage[11],byteMessage[12],byteMessage[13]};
    System.out.println(new String(loggoutName).trim()+"已退出游戏");
    if(playerseen.containsKey(new String(loggoutName).trim()))
    {
        for(Iterator<String> iterator = playerseen.keySet().iterator(); iterator.hasNext(); ) {
            String key = iterator.next();
            if(key.equals(new String(loggoutName).trim()))
            {
                iterator.remove();
            }
        }
    }
}
```

至此，logout全过程实现完毕，下面是动图演示，主要包括客户端的登出以及广播：


![](http://image.leesdog.space/jwlogoutgif)

---

#### invalid_state

在两种情况下会出现上述这种情况，不过前面的动图中其实也已经演示过了，这里单独再演示一遍：

![](http://image.leesdog.space/jwinvalid.gif)

----

到这里，基本上可以说第一部分的内容应该是完成了，其中我也修复了很多bug，也是花了不少的时间，尤其是在attack这部分；但是由于使用的是java，没有办法跑老师的测试代码，这一点我还在想怎么办。。还有就是目前刚刚看了一下2A的内容，明天打算尝试着搞一下2A试试

----

### Day0x07

今天主要先来探究一下2A部分的内容，根据我的一些初步理解（应该算中步理解，因为和最开始的理解已经有很大不同了），这里我们主要处理重构的是login、move与logout这仨方法，同时也要将服务器的一些结构进行更改，开始写代码写的越来越乱，故整理了一下思路，将所有过程全部过了一遍，根据我当前的理解，目前想到的一些比较关键的点是：

- 客户端调用login指令时首先是通过udp 0x00号消息发送给tracker，tracker对**一号指定服务器**发送**getMes tcp**报文（getMes报文需要自构，主要功能就是请求该角色信息，如果存在则返回角色信息，不存在则在一号服务器内创建；这里我思来想去还是确定一个固定的请求客户端，虽然该请求角色的位置判定可能不在该客户端内，但由于服务器整体存有角色信息，依旧可以取到角色的信息，返回以后tracker再根据角色的具体坐标返回相应的服务器信息）

- 生成玩家时玩家的坐标**只能是该服务器管辖范围内坐标**

- 得到对应坐标消息以后根据坐标在tracker里计算出相应对应的服务器，然后从存有xml信息的map中取出该服务器ip，udp信息，根据协议这里我们以udp 0x01消息返回，ip，udp信息给客户端

- 现在整体的login功能全过程应该是：

  1.构建udp 0x00消息给tracker，让tracker发送getMes tcp报文给一号服务器，获得返回的message（带有角色属性信息）后根据x，y坐标返回该角色对应ip，udp端口

  上述整体过程为：**客户端->tracker->服务器->tracker->客户端，客户端获得服务器ip+udp端口**

  2.客户端根据上述获得的端口消息构建udp 0x04消息发送给服务器，服务器返回0x05 udp消息

  上述整体过程为：**客户端->服务器，客户端获得角色属性信息**

  3.客户端根据上述获得的坐标等消息构建udp发送给tracker获得服务器的tcp端口

  上述整体过程为：**客户端->tracker，客户端获得服务器tcp端口**

  4.tcp连接客户端与服务器

  5.根据消息发送tcp login请求，这里的login已经被重构成2A里的新协议要求

- logout处需要**首先进行save操作**，然后返回的消息也需要进行重构

- 这里需要注意的一点是udp消息是**不可靠传输**，且是无需要保持连接的，所以如何实现较为可靠的传输也是一个问题

- tracker与客户端之间的udp连接端口可以**事先就进行约定**，而tracker与服务器之间的udp连接端口由于**约定了传输1号服务器**故也因此可以解决

今天本来是直接写代码的，但是写了半天以后发现了很多问题，于是静下心来决定将整个更改后的传输全过程好好理一下，然后再去写代码，这才得到了上面的一些我的思考，同时由于udp的不可靠性，要想实现可靠传输需要在应用层加上tcp协议层就实现的事情，例如包的编号，重传以及三次握手（协议里已经提到了一些解决方案，例如udp协议都会有一个id，通过每次传输请求后+1的方式也就实现了编号与消息的对应），这一块的内容还是很麻烦的，明天尝试先实现以下udp的可靠传输试一试
