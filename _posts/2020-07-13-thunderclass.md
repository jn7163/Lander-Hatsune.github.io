---
title: 雷课堂（THUnderClass）——清华大学2020C++大作业个人项目记录与总结
layout: post
categories: Projects
tag: 
    - Tsinghua
    - C/C++
    - THUnderClass
permalink: /THUnderClass
---
<!-- more -->
# 声明 #

- 本文是博主个人对清华大学2020年自动化系《C++程序设计与训练》课程（以下简称“课程”）大作业（以下简称“大作业”）——雷课堂项目的完成记录与总结。
- 博主仅为一名修习了此课程的学生，以个人名义记录。个人能力浅薄、经验欠缺，言辞、代码若有不妥以至纰漏处，欢迎批评指正！
- 本文将仅对大作业的完成过程和个人的粗浅思考过程进行记述，不再对大作业本身进行评价。
- 关于此大作业相当丰富的一簇信息可见[知乎](https://www.zhihu.com/question/389457315)。
- 待老师发布参考代码`ThunderClassDemo`时，本人已完成了需求中管理员的大部分功能，故未强行换用老师的代码。换句话说，在完成此项目时，我未直接使用老师的代码，但在实现部分功能时参考了老师的解决方案。
- 个人本项目的仓库（包含源码）已在开发中同步托管至[GitHub](https://github.com/Lander-Hatsune/THUnder-class)。

# 正文 #

## 设计 ##

其实最开始没听“老人言”，没规划大局，学了点Qt的基本知识就直接上手想一个功能写一个功能了。写到后面，愈发出现了“牵一发动全身”的问题，才腾出时间来认真做了思考。现将分析需求的思考所得+实操中的微调总地列写下方。

### 架构总述 ###

模仿知名会议软件，在征得老师同意后，采用了**客户端-服务端**分离架构。

- 客户端包括了教师、学生、管理员三个模式，主要处理窗口信号，将必要的信息发送服务端，以及接收处理服务端发出的信息。
- 服务端主要负责信息的接收、处理、转发，且能操作数据库，用以实现管理员的功能。

（一大好处：服务端犯不着用GUI，调试方便，我是真的有点头疼GUI这东西）

（于是只有客户端有界面，只有客户端按要求需要“前后端分离”，只有客户端动用MVC模式）

在客户端子项目里，我为了方便区分文件（懒得命名）把MVC的三部分类直接放在了三个文件夹里，目录如下。

#### /THUnderClient ####

```
/THUnderClient
├── controller
│   ├── adminop.cpp
│   ├── adminop.h
│   ├── loginop.cpp
│   ├── loginop.h
│   ├── stuop.cpp
│   ├── stuop.h
│   ├── teacherop.cpp
│   └── teacherop.h
├── definitions.h// 通用的一些常量、信息头等等定义
├── main.cpp
├── model
│   ├── adminclient.cpp
│   ├── adminclient.h
│   ├── client.cpp
│   ├── client.h// 用户类基类
│   ├── record.cpp
│   ├── record.h// 签到记录、课堂注意力记录类，（说白了是个struct）
│   ├── Socket.cpp
│   ├── Socket.h// GitHub上找到的现成项目，来源见后文
│   ├── stuclient.cpp
│   ├── stuclient.h
│   ├── teacherclient.cpp
│   └── teacherclient.h
├── THUnderClient.pro// qmake项目文件
├── THUnderClient.pro.user
└── view
    ├── adminmainpage.cpp
    ├── adminmainpage.h
    ├── adminmainpage.ui// 管理员窗口
    ├── ansprobwindow.cpp
    ├── ansprobwindow.h
    ├── ansprobwindow.ui// 学生模式答题窗口
    ├── loginpage.cpp
    ├── loginpage.h
    ├── loginpage.ui// 登录窗口
    ├── pushprobdialog.cpp
    ├── pushprobdialog.h
    ├── pushprobdialog.ui// 教师模式推送题目窗口
    ├── recordwindow.cpp
    ├── recordwindow.h
    ├── recordwindow.ui// 教师模式查看课堂记录窗口
    ├── stumainpage.cpp
    ├── stumainpage.h
    ├── stumainpage.ui// 学生模式主窗口
    ├── teachermainpage.cpp
    ├── teachermainpage.h
    └── teachermainpage.ui// 教师模式主窗口
```

#### /THUnderServer ####

```
/THUnderServer
├── build
│   ├── clients.db// 帐号信息数据库
│   ├── CMakeCache.txt
│   ├── CMakeFiles ...
│   ├── cmake_install.cmake
│   ├── Makefile
│   ├── sqlitesh.exe// Sqlite3 shell（用于调试）
│   └── THUnderServer.exe// 项目可执行文件
├── cmakelists.txt// cmake项目文件
├── dboperator.cpp
├── dboperator.h// 数据库操作类（说白了只是一批函数）
├── definitions.h// 通用的一些常量、信息头等等定义
├── main.cpp
├── server.cpp
├── server.h
├── socket// GitHub上找到的现成项目，来源见后文
│   ├── Socket.cpp
│   └── Socket.h
├── sqlite// Sqlite3，来源见后文
│   ├── shell.c// Sqlite3 shell的main所在，未包含于项目中
│   ├── sqlite3.c
│   ├── sqlite3ext.h
│   └── sqlite3.h
├── user.cpp
└── user.h// 用户类（说白了是个struct）
```

### 需求分析 & 实现方案 ###

参照大作业需求文档`2020C++大作业“雷课堂”V2.pdf`。

1. **用户登陆**：根据用户名密码登陆软件，三次密码输入错误自动退出雷课堂软件。根据账号类型（教师/学生）不同自动切换功能。必须包含一个账户名为Admin，密码为 Admin 的管理员账号，此账号仅能用于管理教师和学生账户的增删改。（不需考虑如何在增删改用户和密码后通知该账户持有者。毕竟我们有微信）

    - 这是我遇到的第一个难题：我压根不会GUI开发。看了一些[Qt的教学文章](http://c.biancheng.net/qt/)，仍然不清楚一个GUI程序中信息的流向。最后还是通读了一个Qt官方的，也是Qt Creator自带的一个Example，[Calculator Builder](https://doc.qt.io/qt-5/qtdesigner-calculatorbuilder-example.html)，这才真正明白了`Button_Signal -> On_Button_Clicked_Slot -> UI_Class`这样的信息流动机制。着手写了个加减乘除计算器练了练手，自以为有点能耐，用的了控件了。
    - 登录功能
    ![登录界面](/figures/thunder-login.png)
        - 会用控件，那登录功能本身就不必赘述了。
        - 需要解决的问题是Socket不会用。老师的几节辅导课都在讲原理，然后在CSDN上找了一份不错的代码一点一点改、封装。我有点等不及，跑去GitHub上搜了一下，果然有，一个叫[Socket.cpp](https://github.com/ReneNyffenegger/Socket.cpp)的仓库，包含了作者封装好的Socket、SocketClient、SocketServer三个类、按两种方式Send和Recieve的共四个方法、还有四个简单应用实例，美滋滋。clone下来简单读了读，跑了跑实例，感觉良好。大大缩减了我费劲去写去调试Socket的时间（然而后面还真的碰到问题自己调试了好久）。还有一点是这个项目全部使用了TCP协议，虽然我有听说大文件要用UDP，但是之后做了声音和图像的传输测试，感觉还好，况且作业本身并未要求性能，我便未再追求。
        - 这样一来登录功能剩下的就是用户名密码信息的编码、信息头的选取了，我一拍脑袋，选了`:`作为特殊符号，信息头用了类似`:CT:`这样的表示，内部信息的分割也由`:`完成。（这么想想好像万一谁取个用户名叫CT我就很容易爆炸）
    - 管理员功能：
    ![管理员界面](/figures/thunder-admin.png)
        - 想搞票大的（懒得自己编码文本数据库），没有用数据库API的经验，上网搜了搜，听说Sqlite处理百万级以下的数据量还是得心应手的，而且比MySql用起来简单？于是跑去[官网](https://www.sqlite.org/)下载了Source code（大作业要求“源码级引用”）。
        - 网上参照了很多篇博客，学习了Sqlite3的C-API，总结写了一篇博客（[见此处](https://lander-hatsune.github.io/sqlite-c-api)）。把这些基本的数据库操作指令封装一下成为`dboperator`类，由服务器调用，实现管理员的功能。
       
2. **语音设备选择和切换**：教师开始上课前/学生加入课堂前，应可自主选择语音输入和播放设备；并可在课程持续期间随时切换语音设备。

    - 这是遇到的第二个难关：我压根不会和WinAPI打交道。思来想去，我想到要是用Qt自己的QAudio录音，那就不是一般的方便了（至于为什么能调“第三方类库”，我写在界面类里好不好嘛～）。和老师交涉后，取得了许可（~~不过会不会扣分老师没明说，哭了~~）。
    - 于是学QAudio。网上一番搜寻未找到很值得参照的完整的项目，于是又想起来Qt的Examples，Qt Creator里一搜，果然有：[Audio Input Example](https://doc.qt.io/qt-5/qtmultimedia-multimedia-audioinput-example.html)、[Audio Output Example](https://doc.qt.io/qt-5/qtmultimedia-multimedia-audiooutput-example.html)、[Audio Devices Example](https://doc.qt.io/qt-5/qtmultimedia-multimedia-audiodevices-example.html)、[Audio Recorder Example](https://doc.qt.io/qt-5/qtmultimedia-multimedia-audiorecorder-example.html)，全家桶。啃英文说明、啃代码，一番下来，算是会用了QAudio的API，实现了单机上的一系列语音功能。
    - (由于和功能4，语音直播，很接近，在这里一并说明。)
    - 网络传输的话，读取的QByteArray用`toStdString`转string，加上一个信息头，我本以为就可以send了。没想到我借用的Socket.cpp那项目里，判断一条信息终止看的是一个`\n`。在send一条信息时末尾添一个换行符，receive信息时，就以换行符为止。要命的是声音不长眼，音频数据里多的是`\n`，这怎么传？试了两三种方案（包括最扯淡的把所有`\n`都+1、用三个`\n`标志结尾等），最后还是选了一个测试下还挺稳健的方案，就是用`*MK*`表示结尾（当然也可以更长更特殊），为此对原Socket.cpp项目也做了一定量的改动（改进？嘿嘿）。学生接到语音信息就照着Qt的Example，输出即可。8000的波特率，16位采样，远程答辩的网速下也表现优秀。
    
3. **共享屏幕**：教师在上课过程中，可共享整个屏幕或某个窗口内容给全体同学（包括但不限于 PPT 和代码编辑器）；可随时切换共享源、停止或再次开始共享屏幕。 
    - 如果重复上一个功能的逻辑，调Qt的API的话，共享屏幕（不要求压制推流的话不就是截图嘛）倒是不难实现，找了几篇相关的博客学习了一下特定窗口的截图、图片的压缩，然后万能`toStdString`，加信息头，发送。
    - 然后又参照老师的`ThunderClassDemo_Worked`示范项目中用QLabel展示QPixmap的方案，学生模式接收图像文件，转QPixmap，贴到QLabel里，大功告成。
    - 不要求清晰度，直接转jpg，压到20%，远程答辩的网速下也表现优秀（逃
    
4. **语音直播**：开始上课时，自动开始语音采集，并实时的通过网络传送给所有已经连接到本课堂的学生。 

    见2.语音设备选择和切换。

5. **随机语音提问**：教师可一键（单次鼠标点击或单次快捷键）在全体在线同学中随机选择一名。被选中的同学的麦克风将被自动打开，并发送给教师和其余全体同学。教师可再次一键结束此次语音提问。

    教师客户端发一条命令到服务端，服务端随机抽取幸运观众，发命令到指定的学生客户端，其接收此命令，即打开麦克风（虽然写起来简单但这个功能我真的无力吐槽）。
    
6. **在线发题**：教师可在上课过程中多次动态编辑并向全体同学发送单选/多选题，并实时统计个选项选择人数、选择每个选项的同学名单、每位同学作答的耗时。教师亦可随时中断发题，但仍需统计上述信息。

    编辑题目：
    ![编辑题目](/figures/thunder-pushprob.png)
    
    
    作答题目：
    ![作答题目](/figures/thunder-ansprob.png)
    - 稍微纠结了一下时间，但为了简洁、方便调试，还是考虑新建一个窗口编辑题目，题目和答案收集好，按`:`间隔开，加信息头发送。
    - 同学收到题目，新建窗口展示题目，并供作答。（这里我不经意犯了一个费了很大劲才找到的错误：**在线程函数里对窗口进行操作**似乎是不妥的，调试无果，经学姐和老师指正才了解到问题所在，引老师的话说，“Qt**好像要求它所有的界面的操作都要在它自己的界面线程里**”）
    - 教师若点击收题，即发送命令，经转发到学生处，收到命令，客户端即关闭答题窗口，不得作答。
    - 学生主动作答后，教师收到信息，统计入表格中，我对表格（QTable）的操作是在[官方文档](https://doc.qt.io/qt-5/qtablewidget.html)学会的（已经新建了一行还得挨个新建Item这我是没想到的）。
    - 事后看到孙恒学长在知乎上的回答，惊悉自己没有考虑到网络延迟，没有用时间戳，又是一个失误。
    
7. **在线答题**：学生在收到试题时，应弹出置顶窗口显示题目和选项，并开始计时。直到学生提交答案或教师中断发题时，才关闭窗口，并将答案和耗时反馈给教师。

    见6.在线发题。

以下的几个功能比较琐碎，有的也有些互相包含的倾向，况且我的实现比较简单，就不再详细描述。

8. **学生签到**：进入课堂时自动签到。而教师可收到何时学生签到和退出课堂的信息。（多次签到和退出均需记录） 

9. **注意力**：课堂持续期间，学生签到后，“雷课堂软件处于焦点窗口状态的时长”与学生在线时长的百分比，将在下课时反馈给教师做记录。

10. **上课/下课**：上课时，教师端开始随时接收用户登录请求，并根据用户名密码自动决定是否允许学生端连入。一旦允许连入，之后的语音、屏幕共享均、语音提问、在线答题信息均会传送给该同学。教师下课时，应在接收了全体在线同学的注意力数据后再断开与学生端的网络连接，之后自动生成全部课上统计信息，以文件形式存储并在教师端界面上显示。

    这里的统计信息我同样借用表格进行了展示，但出于时间原因（和莫名的完美主义不想简单的输出成文本文件），没有完成“以文件形式存储”这一功能。以我的理解，这个是唯一一个未能实现的需求。
    
11. **进入课堂/退出课堂**：在输入了教师端的 IP 地址（或 IP 和端口号）后，连接到教师端，实现进入课堂功能并开始网络数据通信。如在 30 秒内不能连接到教师端，应弹出提示。在主动退出课堂或直接关闭了软件时，应向教师端发送注意力数据，再断开与教师端的网络连接。（不需考虑如何获取教师端 IP 和端口号，毕竟我们有课程微信群）

12. **麦克风管制**：除非收到教师语音提问，否则麦克风时刻处于静音状态。

### 幕后的几处细节 ###

- 最重要的一个前文不怎么提到的“细节”，就是服务端的设计。

    在这个项目里，服务端承担了大多数的信息、命令的处理、转发工序。
    - 信息的识别与客户端采用同样的信息头编码。
    - 精确的转发依靠一份用户列表，包含了所有登录的User的所需信息。
    - 还是想再说一句，个人感觉没有GUI，开发和调试都轻松许多。
<br>
<br>
- 其次，是多线程编程的知识和技术。

    我完全没有接触过多线程编程，也很遗憾出于时间原因没能完整学习多线程的相关知识。但在开发雷课堂时为实现高效的网络传输，参照Socket.cpp项目，我的确多次应用了多线程技术。
    
- DbTranslator

    由于相当部分的同学采用老师的`ThunderClassDemo`项目作为起点，老师便**建议性地**发布了以`ThunderClassDemo`的文本数据库（`UserInformation.txt`）格式排列的答辩用账户信息，为了给老师少添点麻烦，我试着写了一个读取`UserInformation.txt`，写入我采用的`Clients.db`的“翻译机”，唤为DbTranslator。
    
- 中文

    非常可悲，我犯了懒，打一开始就没有想到该有中文支持，到最后面对一份中文用户名的`UserInformation.txt`，人都傻了。急忙和老师交涉一下，由于中文支持也是需求外的事，还是放了我一条生路，允许我自行提供账户，我费了些功夫把`UserInformation.txt`里九十多个账户改成了拼音。（结果还是麻烦老师了）

# 小结 #

总而言之，这是一份有着不少瑕疵但差强人意的答卷，也是时间限制下我尽自己所能的一次尝试。

由于开发期间还在学期中，又夹杂了考试，真实用时其实大多分配在考试周后的一个礼拜，集中在三五天内，具体不太可计了。

最终所有的代码长度似乎为163034行，显然不对劲。检查了一下，除去Sqlite3的几个硕大文件（158907），行数大概在**4127行**左右。也超过我以往的项目代码长度数倍了。

![](/figures/thunder-cloc.png)<br>
![](/figures/thunder-cloc-sqlite.png)<br>

以这篇博客纪念一下第一次的这样大而完整（虽然不周全）的个人项目，收获良多，预计对未来继续和计算机打交道的日子会有很大帮助。

照例感谢一下为我提供帮助的老师、助教、同学、各位博主、各位开发者，感谢你们的分享！

以上。
