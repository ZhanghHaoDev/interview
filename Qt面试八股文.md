#  信号槽相关
## Qt信号和槽的本质是什么
Qt信号和槽是一种用于实现对象间通信的机制。其本质是一种事件驱动的机制，通过信号和槽的连接，当信号被
触发时，槽函数会被自动调用，从而实现对象间的通信和交互。 信号是一种特殊的函数，用于表示某种事件的发生，当事件发生时，信号会被自动发送出去，通知所有连接到该信号的槽函数。 槽是一种特殊的函数，用于处理信号的触发事件，当槽函数被连接到某个信号时，当该信号被触发时，槽函数会自动被调用，从而实现对信号的响应。 信号和槽的连接是通过Qt的元对象系统实现的，每个QObject派生类都有一个元对象，用于存储该类的属性、方法和信号槽信息。通过元对象系统，可以在运行时动态地连接信号和槽，从而实现对象间的通信。 总之，Qt信号和槽是一种事件驱动的机制，通过信号和槽的连接，实现了对象间的通信和交互。 
## 讲述Qt信号槽机制与优势与不足
**优点 ：**

1. 类型安全，需要关联的信号槽的签名必须是等同的。即信号的参数类型和参数个数同接受该信号的槽的参数类型和参数个数相同。若信号和槽签名不一致，编译器会报错。
2. 松散耦合：信号和槽机制减弱了Qt对象的耦合度。激发信号的Qt对象无需知道是那个对象的那个信号槽接收它发出的信号，它只需在适当的时间发送适当的信号即可，而不需要关心是否被接受和那个对象接受了。Qt就保证了适当的槽得到了调用，即使关联的对象在运行时被删除。程序也不会奔溃。
3. 灵活性：一个信号可以关联多个槽，或多个信号关联同一个槽。

**缺点：**
速度较慢。与回调函数相比，信号和槽机制运行速度比直接调用非虚函数慢10倍。
原因：

1. 需要定位接收信号的对象。
2. 安全地遍历所有关联槽。
3. 编组、解组传递参数。
4. 多线程的时候，信号需要排队等待。（然而，与创建对象的new操作及删除对象的delete操作相比，信号和槽的运行代价只是他们很少的一部分。信号和槽机制导致的这点性能损耗，对实时应用程序是可以忽略的。） 
## Qt自定义一个信号槽，触发这个信号，Qt多个信号如何关联一并处理

1. 在发送信号时，也发送一个int类型数字，或者说标志，这样在槽函数触发是可以知道是哪个信号发出的；
2. 在槽函数内有获取发送信号的函数，通过sender（）函数获取发送信号；
## Qt如果一个信号的处理方法一直未被执行有哪些可能性

1. 断开了
2. 连接的时候失败
3. 多线程的时候在排队或者启动锁死了
## 在Qt5的信号处理中如何使用lambda机制（可以代码示例）
信号定义了，但是不写对应槽函数，直接将函数写到槽的位置。
直接就是将对象都不写了，直接写个函数。 
```cpp
connect(musicPlayer,SIGNAL(positionChanged(qint64)),this,SLOT(slotReflushStartTime(qint64)));

connect(musicPlayer,SIGNAL(positionChanged(qint64)),slotReflushStartTime(qint64));
```
## 信号槽的四种写法和五种连接方式
connect(信号发出者，信号，信号接收者，槽，连接方式(隐藏默认自动连接))//五个参数

1. 使用宏定义
```cpp
connect(this,SIGNAL(clicked()),this,SLOT(colse())); //连接方式(隐藏默认自动连接))
```

2. 用函数指针
```cpp
connect(this,&mainwindow::my_signal,this,&mainwindow::my_slot);
```

3. 用重载函数指针Qoverload
```cpp
connect(this,Qoverload<参数>::of(&mainwindow::my_signal),this,Qoverload<参数>::of(&mainwindow::my_slot));
```

4. lambda表达式(匿名函数) 匿名函数代替槽
```cpp
 connect(this,&mainwindow::my_signal,this,[=]{qDebug()<<100;});
 连接方式：自动连接(默认连接方式)
 直接连接(用于单线程,自动匹配)
 队列(用于多线程也可用于单线程,自动匹配)
 阻塞队列(跨线程,多线程)
 唯一连接(跨线程,多线程)
```
## 信号重载了，如何确定连接哪个信号
采用函数指针确定连接哪个信号。
## 槽函数参数、信号的参数
槽函数的参数可以少于信号的参数。

1. 槽函数本身参数比信号的少
2. 槽函数参数带有默认参数
## 槽函数的参数是否可以比信号的参数多？
也可以。唯一的情况就是槽函数参数带有默认参数，除去默认参数外，槽函数的参数必须小于等于信号的参数。 
## Qt信号槽的调用流程

1. MOC查找头文件中的signal与slots，标记出信号槽。将信号槽信息储存到类静态变量staticMetaObject中，并按照声明的顺序进行存放，建立索引
2. connect链接，将信号槽的索引信息放到一个双向链表中，彼此配对。
3. emit被调用，调用信号函数，且传递发送信号的对象指针，元对象指针，信号索引，参数列表到active函数。
4. active函数在双向链表中找到所有与信号对应的槽索引，根据槽索引找到槽函数，执行槽函数。 
## Qt connect的第五个参数（信号槽链接方式）

1. Qt::AutoConnection： 默认值，使用这个值则连接类型会在信号发送时决定。如果接收者和发送者在同一个线程，则自动使用Qt::DirectConnection类型。如果接收者和发送者不在一个线程，则自动使用Qt::QueuedConnection类型。
2. Qt::DirectConnection：槽函数会在信号发送的时候直接被调用，槽函数运行于信号发送者所在线程。效果看上去就像是直接在信号发送位置调用了槽函数。这个在多线程环境下比较危险，可能会造成奔溃
3. Qt::QueuedConnection：槽函数在控制回到接收者所在线程的事件循环时被调用，槽函数运行于信号接收者所在线程。发送信号之后，槽函数不会立刻被调用，等到接收者的当前函数执行完，进入事件循环之后，槽函数才会被调用。多线程环境下一般用这个。
4. Qt::BlockingQueuedConnection：槽函数的调用时机与Qt::QueuedConnection一致，不过发送完信号后发送者所在线程会阻塞，直到槽函数运行完。接收者和发送者绝对不能在一个线程，否则程序会死锁。在多线程间需要同步的场合可能需要这个。
5. Qt::UniqueConnection：这个flag可以通过按位或（|）与以上四个结合在一起使用。当这个flag设置时，当某个信号和槽已经连接时，再进行重复的连接就会失败。也就是避免了重复连接。
## 信号和信号量的区别是什么
**信号**：一种处理异步事件的方式。信号是比较复杂的通信方式，用于通知接收进程有某种事件发生，除了用于进程外，还可以发送信号给进程本身。
**信号量**：进程间通信处理同步互斥的机制。是在多线程环境下使用的一种设施，它负责协调各个线程，以保证它们能够正确，合理的使用公共资源。
## 信号槽是同步的还是异步的？分别如何实现
通常使用的connect，实际上最后一个参数使用的是Qt::AutoConnection类型：Qt支持6种连接方式，其中3中最主要:

1. Qt::DirectConnection（直连方式）（信号与槽函数关系类似于函数调用，同步执行）
  当信号发出后，相应的槽函数将立即被调用。emit语句后的代码将在所有槽函数执行完毕后被执行。
2. Qt::QueuedConnection（排队方式）（此时信号被塞到信号队列里了，信号与槽函数关系类似于消息通信，异步执行）当信号发出后，排队到信号队列中，需等到接收对象所属线程的事件循环取得控制权时才取得该信号，调用相应的槽函数。emit语句后的代码将在发出信号后立即被执行，无需等待槽函数执行完毕。
3. Qt::AutoConnection（自动方式）
 Qt的默认连接方式，如果信号的发出和接收这个信号的对象同属一个线程，那个工作方式与直连方式相同；否则工作方式与排队方式相同。
4. Qt::BlockingQueuedConnection(信号和槽必须在不同的线程中，否则就产生死锁) 这个是完全同步队列只有槽线程执行完成才会返回，否则发送线程也会一直等待，相当于是不同的线程可以同步起来执行。
5. Qt::UniqueConnection 与默认工作方式相同，只是不能重复连接相同的信号和槽，因为如果重复连接就会导致一个信号发出，对应槽函数就会执行多次。
6. Qt::AutoCompatConnection: 工作方式与Qt::AutoConnection一样。
## Qt的信号与槽，有哪几种连接方式，对应的应用场景是什么？
Qt的信号与槽有三种连接方式：

1. 信号槽的直接连接：使用QObject::connect()函数连接信号和槽，当信号发出时，槽函数自动被调用，适用于信号发出者与槽函数拥有者在同一线程的场景。
2. 信号槽的槽函数链接：使用QObject::connect()函数连接信号和槽函数，当信号发出时，槽函数被调用，适用于信号发出者与槽函数拥有者不在同一线程的场景。
3. 信号槽的信号连接：使用QObject::connect()函数连接信号和信号，当信号发出时，另一个信号也会发出，适用于信号发出者与槽函数拥有者不在同一线程的场景。
## 有没有使用过Qt4？Qt5的信号槽与Qt4相比有什么改进？

1. 编译期：检查信号与槽是否存在，参数类型检查，Q_OBJECT是否存在
2. 信号可以和普通的函数、类的普通成员函数、lambda函数连接（而不再局限于信号函数和槽函数）
3. 参数可以是 typedef 的或使用不同的namespace specifier
4. 可以允许一些自动的类型转换（即信号和槽参数类型不必完全匹配）
# Qt 网络相关
## 描述Qt的TCP通讯流程
**服务端：（QTcpServer）**

1. 创建QTcpServer对象
2. 监听list需要的参数是地址和端口号
3. 当有新的客户端连接成功回发送newConnect信号
4. 在newConnection信号槽函数中，调用nextPendingConnection函数获取新连接QTcpSocket对象
5. 连接QTcpSocket对象的readRead信号
6. 在readRead信号的槽函数使用read接收数据
7. 调用write成员函数发送数据

**客户端：（QTcpSocket）**

1. 创建QTcpSocket对象
2. 当对象与Server连接成功时会发送connected 信号
3. 调用成员函数connectToHost连接服务器，需要的参数是地址和端口号
4. connected信号的槽函数开启发送数据
5. 使用write发送数据，read接收数据
## 描述UDP 之 UdpSocket通讯
UDP（User Datagram Protocol即用户数据报协议）是一个轻量级的，不可靠的，面向数据报的无连接协议。在网络质量令人十分不满意的环境下，UDP协议数据包丢失严重。由于UDP的特性：它不属于连接型协议，因而具有资源消耗小，处理速度快的优点，所以通常音频、视频和普通数据在传送时使用UDP较多，因为它们即使偶尔丢失一两个数据包，也不会对接收结果产生太大影响。所以QQ这种对保密要求并不太高的聊天程序就是使用的UDP协议。
在Qt中提供了QUdpSocket 类来进行UDP数据报（datagrams）的发送和接收。Socket简单地说，就是一个IP地址加一个port端口 。
流程：①创建QUdpSocket套接字对象 ②如果需要接收数据，必须绑定端口 ③发送数据用writeDatagram，接收数据用 readDatagram 。 
## Qt Socket通信的过程
Qt Socket通信的过程主要分为以下几步：

1. 创建Socket：使用QTcpSocket类创建Socket，并初始化连接参数；
2. 连接服务器：使用connectToHost()函数连接服务器；
3. 发送数据：使用write()函数发送数据；
4. 接收数据：使用read()函数接收数据；
5. 断开连接：使用disconnectFromHost()函数断开连接。
# Qt 控件相关
## 描述Qt中的文件流(QTextStream)和数据流(QDataStream)的区别
在Qt中，QTextStream和QDataStream都是用于读写数据的类，但它们的使用场景和读写的数据类型不同。 QTextStream是用于读写文本数据的类，可以将QString、QByteArray等文本数据以类似于流的方式读写到文件、套接字等设备中。QTextStream提供了一系列方便的方法，如readLine()、readAll()、operator<<()、operator>>()等，可以方便地读写文本文件和套接字等设备。 QDataStream是用于读写二进制数据的类，可以将各种类型的二进制数据（如int、float、double、QString等）以二进制方式读写到文件、套接字等设备中。QDataStream通过内部的序列化机制实现了数据的高效读写，可以保证在不同平台上的数据兼容性。QDataStream提供了一系列方便的方法，如writeRawData()、writeInt()、readRawData()、readInt()等，可以方便地读写二进制文件和套接字等设备。 因此，QTextStream和QDataStream的主要区别在于它们所处理的数据类型不同，QTextStream主要处理文本数据，而QDataStream主要处理二进制数据。在使用时，需要根据实际情况选择合适的类进行读写操作。 
## 自定义控件流程
继承需要自定义的控件类，如QPushButton；从外观设计上：QSS、继承绘制函数重绘、继承QStyle相关类重绘、组合拼装等等；从功能行为上：重写事件函数、添加或者修改信号和槽等等。 
## Qt定义面设计类，如果想自定义控件，只能通过写代码的方式吗
不一定。在Qt中，自定义控件可以通过写代码的方式实现，也可以通过Qt Designer的插件机制来实现。
通过写代码的方式，可以继承QWidget或其子类，并在其中实现自己的功能和界面。
可以根据需要添加、删除、修改控件，实现自己的布局和样式，甚至可以处理自定义的事件。通过这种方式，可以实现非常灵活、个性化的控件，但需要一定的编程能力和时间成本。
通过Qt Designer的插件机制，可以扩展Qt Designer的控件库, 添加自定义控件。可以通过继承QDesignerCustomWidgetInterface类,并在其中实现自己的功能和界面，然后将插件编译成动态链接库，就可以在Qt Designer中使用自定义控件了。这种方式相对于写代码的方式，更加便捷和可视化，但需要一定的Q开发经验和插件开发知识。
因此，自定义控件的方式可以根据具体情况选择，根据自己的需求和技术水平选择最适合自己的方式。 
## 什么叫自定义控件
qt本身的控件不能满足需求将原控件功能提升达到要求,此时控件为自定义控件。 
## 你觉得自定义控件的方法主要是哪些？
从外观设计上: QSS、继承绘制函数重绘、继承QStyle相关类重绘 、组合拼装等等
从功能行为上:重写事件函数、添加或者修改信号和槽等等
## QSS平时使用的多吗？能举几个例子

1. 将QSS统一写在一个文件中，通过程序给主窗口加载；
2. 写成一个字符串中，通过程序给主窗口加载；
3. 需要使用的地方，写一个字符串，加载给对象；
4. QT Designer中填写；
## 对QObject的理解

1. QObject 类是Qt 所有类的基类。
2. QObject是Qt对象模型的核心。这个模型的中心要素就是一种强大的叫做信号与槽无缝对象沟通机制。你可以用 connect() 函数来把一个信号连接到槽，也可以用disconnect() 函数来破坏这个连接。为了避免永无止境的通知循环，你可以用blockSignal() 函数来暂时阻塞信号。保护函数 connectNotify() 和 disconnectNotify() 可以用来跟踪连接。
3. 对象树都是通过QObject 组织起来的，当以一个对象作为父类创建一个新的对象时，这个新对象会被自动加入到父类的 children() 队列中。这个父类有子类的所有权。能够在父类的析构函数中自动删除子类。可以通过findChild()和findChildren() 函数来寻找子类。
4. 每个对象都一个对象名称objectName() ，而且它的类名也可以通过metaObject()函数。你可以通过inherits() 函数来决定一个类是否继承其他的类。当一个对象被删除时，它会发射destory() 信号，你可以抓住这个信号避免某些事情。
5. 对象可以通过event() 函数来接收事情以及过滤来自其他对象的事件。就好比installEventFiter() 函数和eventFilter() 函数。childEvent() 函数能够重载实现子对象的事件。
6. QObject还提供了基本的时间支持，QTimer类 提高了更高层次的时间支持。
任何对象要实现信号与槽机制，Q_OBJECT 宏都是强制的。你也需要在源原件上运行元对象编译器。不管是否真正用到信号与槽机制，最好在所有QObject子类使用Q_OBJECT宏，以避免出现一些不必要的错误。
7. 所有的Qt widgets 都是基础QObject。如果一个对象是widget,那么isWidgetType()函数就能判断出。
## Qt 三大核心机制

1. 信号槽
信号槽的五种连接方式（图略）
connect(信号发出者，信号，信号接收者，槽，连接方式(隐藏默认自动连接))//五个参数
2.  元对象系统
元对象系统分为三大类:QObject类、Q_OBJECT宏和元对象编译器moc
Qt的类包含Q_OBJECT宏 moc编译器会对该类编译成标准的C++代码
3. 事件模型
事件的创建：鼠标事件，键盘事件，窗口调整事件，模拟事件
事件的交付：Qt通过调用虚函数QObject::event()来交付事件。
事件循环模型：主事件循环通过调用QCoreApplication::exec()启动，随着QCoreApplication::exit()结束，
本地的事件循环可用利用QEventLoop构建。
一般来说，事件是由触发当前的窗口系统产生的，但也可以通过使用 QCoreApplication::sendEvent()QCoreApplication::postEvent()来手工产生事件。需要说明的是QCoreApplication::sendEvent()会立即发送事件， QCoreApplication::postEvent()则会将事件放在事件队列中分发。
4. 自定义事件
## Qt对象树

1. QT提供了对象树机制，能够自动、有效的组织和管理继承自QObject的对象。
2. 每个继承自QObject类的对象通过它的对象链表(QObjectList)来管理子类对象，当用户创建一个子对象时，其对象链表相应更新子类对象的信息，对象链表可通过children()获取。
3. 当父类对象析构的时候，其对象链表中的所有(子类)对象也会被析构，父对象会自动，将其从父对象列表中删除，QT保证没有对象会被delete两次。开发中手动回收资源时建议使用deleteLater代替delete,因为deleteLater多次是安全的。
## Qt模型
Qt中的View主要有三种QListView，QTreeView, QTabelView
而对应的Model是：QStringListModel, QAbstractItemModel , QStandardItemModel。
抽象 标准
## Qt中的MVD了解吧？
Qt的MVD包含三个部分Model（模型），View（视图），代理（Delegate）。Model否则保存数据，View负责展示数据，Delegate负责Item样式绘制或处理输入。这三部分通过信号槽来进行通信，当Model中数据发生变化时将会发送信号到View，在View中编辑数据时，Delegate负责将编辑状态发送给Model层。基类分别为QAbstractItemModel、QAbstractItemView、QAbstractItemDelegate。Qt中提供了默认实现的MVD类，如QTableWidget、QListWidget、QTreeWidget等。 
## 对Qt元对象系统了解吗？
Qt对标准的C++进行了扩展，如信号槽、对象属性等。Qt的元对象编译系统MOC是一个预处理器，当Qt读取源文件时检测到类中包含有Q_OBJECT宏时，则会创建一个新的文件（生成路径下的moc开头的文件），将源码转换为C++编译器可以识别的代码写入moc开头的文件，然后C++编译器对其进行编译。当你的类需要使用Qt的扩展功能时，如信号槽、对象属性等时，则必须使用MOC，反之如果你的类不使用这些功能的时候不要无畏的使用MOC增大源码体积。使用MOC系统的方法：
**继承QObject。
类中添加Q_OBJECT宏。**
## QObject是否是线程安全的
QObject及其所有子类都不是线程安全的（但都是可重入的）。因此，你不能有两个线程同时访问一个QObject对象，除非这个对象的内部数据都已经很好地序列化（例如为每个数据访问加锁）。 
## QObject的线程依附性是否可以改变
调用QObject::moveToThread()函数。该函数会改变一个对象及其所有子对象的线程依附性。
由于QObject本身是线程不安全的，因此moveToThread接口的调用必须在QObject对象所在的线程内调用。
## 如何安全的在另外一个线程中调用QObject对象的接口
QObject被设计成在一个单线程中创建与使用，因此，在一个线程中创建一个对象，而在另外的线程中调用它的函数，这样的行为不能保证工作良好。
使用信号槽的队列连接或者QT的反射系统提供的QMetaObject::invokeMethed的队列连接调用。这要求接口必须是内省的，也就是说这个函数要么是一个槽函数，要么标记有Q_INVOKABLE宏。
将事件提交到接收对象所在线程的事件循环；当事件发出时，响应函数就会被调用。
## QFrame与QWidget的区别
QFrame 和 QWidget 都是 Qt 中的 GUI 组件，但是它们有一些区别：
继承关系：QFrame 继承自 QWidget，所以 QFrame 具有 QWidget 的所有功能。
功能：QFrame 提供了一个简单的框架，可以作为其他控件的容器。它还可以用来绘制简单的图形，如线条。QWidget 没有这样的功能，但是提供了基础的 GUI 组件功能，如设置尺寸和位置等。
外观：QFrame 可以有边框和背景颜色，因此外观更加丰富。QWidget 只有背景颜色，没有边框。
通常，当需要一个简单的框架时，使用 QFrame，当需要基础的 GUI 组件功能时，使用 QWidget。
## show()和exec()的区别
show显示非模态窗口（不影响用户对其他窗口操作），exec显示模态窗口（阻塞其他窗口，必须在当前窗口操作完成后才能访问其他窗口），open半模态（阻塞其他窗口响应，但不影响后续代码执行）
## Qt的D指针（d_ptr）与Q指针（q_ptr）
**D指针**
PIMPL模式，指向一个包含所有数据的私有数据结构体。

1. 私有的结构体可以随意改变，而不需要重新编译整个工程项目
2. 隐藏实现细节
3. 头文件中没有任何实现细节，可以作为API使用
4. 原本在头文件的实现部分转移到乐源文件，所以编译速度有所提高

**Q指针**
私有的结构体中储存一个指向公有类的Q指针。
**总结**

1. Qt中的一个类常用一个PrivateXXX类来处理内部逻辑，使得内部逻辑与外部接口分开，这个PrivateXXX对象通过D指针来访问；在PrivateXXX中有需要引用Owner的内容，通过Q指针来访问。
2. 由于D和Q指针是从基类继承下来的，子类中由于继承导致类型发生变化，需要通过`static_cast`类型转化，所以`DPTR()`与`QPTR()`宏定义实现了转换。
## 了解Qt的QPointer吗？
QPointer只能用于指向QObject及派生类的对象。当一个QObject或派生类对象被删除后，QPointer能自动将其内部的指针设置为0，这样在使用QPointer之前就可以判断一下是否有效乐。
QPointer对象超出作用域时，并不会删除它指向的内存对象。
## 了解Qt的QSharedPointer吗？
用于实现数据的隐式共享。Qt中大量使用了隐式共享与写时拷贝技术，例如：
```cpp
QString str1 = "abc";
QString str2 = str1;
str2[2] = "X"; 
```
第二行执行完后，str2和str1指向同一片内存数据。第三句执行时，Qt会为str2的内部数据重新分配内存。这样做的好处是可以有效地减少大片数据拷贝的次数，提高程序的运行效率。
Qt中隐式共享和写时拷贝就是利用QSharedDataPointer和QSharedData这两个类实现的。
## 什么是Qml
QML是语言的名称（就像C++，那是一些其他的语言......)
QML代表Qt Meta Language或Qt Modelling Language，是一种人机界面标记语言。
QtQuick是QML的一个工具包，允许用QML语言扩展图形界面（还有其他的QML工具包，有些是图形化的，如Sailfish Silica或BlackBerry Cascade，还有一些是非图形化的，如QBS，它是QMake/CMake/make的替代品...）。
QtQuick 1.X变成了基于Qt4.X，使用QPainter/QGraphicsView API来吸引场景。QtQuick 2.X与Qt5.0一起推出，主要基于Scene Graph，这是一个OpenGLES2抽象层，经过了相当的优化。
随着Qt5.1的推出，Scene Graph变得更适合应用多线程（QtQuick 2.1）和Qt5.2。
## 使用样式表要注意的点
父控件采用样式表设置属性后，该属性会传递到其子控件上，除非子控件使用同样的方法修改属性。如：利用样式表设置父控件最小高度为x，则子控件的最小高度也为x，即使用setFixHeight()修改也无法消除，只能通过样式表重新设置子控件的高度才有效。因此一般只对子控件使用样式设置。 
## QApplication的主要作用是什么
QApplication对象管理QtGui应用程序的控制流程和主要的设置参数
## Qt都提供哪些标准对话框以供使用，他们实现什么功能
9个QColorDialog颜色对话框，能够允许用户选择颜色、QErrorMessage显示错误信息、QFileDialog文件对话框，能够允许用户选的一个或者多个文件以及目录、QFontDialog字体对话框,允许用户选择/设置字体、QInputDialog输入对话框,允许用户进行简单的输入、QPageSetupDialog叶设置对话框，配置与页相关的打印机选项、QProgressDialog进度对话框指示一个长时间操作的工作进度,以提示用户该操作是否已经停止QPrintDialog打印对话框，配置打印机，可以允许用户选择可用的打印机、QMessageBox。 
## QMainForm是从哪里派生的？
```cpp
QMainWindow::QWidget::QObject
```
## Qwidget、Qobejct实现了哪些功能
**QObject**

1. 信号和槽的非常强大的机制,使用connect()把信号和槽连接起来并且可以用disconnect()来破坏这种连接。为了避免从不结束的通知循环，你可以调用blockSignals()临时地阻塞信号。保护函数connectNotify()和disconnectNotify()使跟踪连接成为可能。
2. QObject可以通过event()接收事件并且过滤其它对象的事件。详细情况请参考installEventFilter()和eventFilter()。一个方便的处理者，childEvent()，能够被重新实现来捕获子对象事件。
3. 最后但不是最不重要的一点，QObject提供了Qt中最基本的定时器，关于定时器的高级支持请参考QTimer
4. 注意Q_OBJECT宏对于任何实现信号、槽和属性的对象都是强制的。
5. 所有的Qt窗口部件继承了QObject。方便的函数isWidgetType()返回这个对象实际上是不是一个窗口部件。它比inherits(“QWidget” )快得多。

**QWidget**

1. QWidget类是所有用户界面对象的基类。
2. Widget是用户界面的基本单元：它从窗口系统接收鼠标，键盘和其他事件，并在屏幕上绘制自己。每个Widget都是矩形的，它们按照Z-order进行排序。
## Qt设计界面有哪些方式？

1. 手工编写创建界面的代码：此方法比较复杂，不够直观；
2. 使用Qt Designer界面编辑器设计：可直接拖放控件、设置控件的属性，简单、直观、易于操作；
3. 动态加载UI文件并生成界面：此方法很灵活，当需要更改界面时只需更改.UI文件即可，无需重新编译程序。
## QWidget和QML的技术本质和使用上，有什么区别？ 
QWidget是一种基于C++的桌面应用程序开发技术，主要用于开发桌面应用程序，它是一种面向对象的技术，可以使用C++语言来实现用户界面的设计和编程。 QML是一种基于JavaScript的应用程序开发技术，主要用于开发桌面应用程序和移动应用程序，它是一种基于声明式的技术，可以使用JavaScript语言来实现用户界面的设计和编程。 两者的本质有所不同，QWidget是基于C++的，QML是基于JavaScript的；使用上也有所不同，QWidget是面向对象的，QML是基于声明式的。
## 你能用几种方法修改QPushButton的大小，文字颜色等属性。

1. 使用Qt Designer设计师：可以使用Qt Designer设计师来调整QPushButton的大小、文字颜色等属性。
2. 使用Qt的API：可以使用Qt的API来调整QPushButton的大小、文字颜色等属性，例如：setMinimumSize()、setMaximumSize()、setStyleSheet()等。
3. 使用CSS：可以使用CSS语法来调整QPushButton的大小、文字颜色等属性，例如：QPushButton {width: 100px; height: 50px; color: red;}。
## 常用的Qt布局有几种，如何自适应缩放？
Qt布局有以下几种：

1. 绝对布局：使用绝对位置和尺寸来定位和调整控件的大小；
2. 盒子布局：使用排列和填充来定位和调整控件的大小；
3. 栅格布局：使用表格排列和调整控件的大小；
4. 流式布局：使用流动排列和调整控件的大小；
5. 堆栈布局：使用堆栈排列和调整控件的大小。

Qt支持自适应缩放，可以使用Qt的布局管理器来实现。例如，可以使用QGridLayout管理器来控制窗口的尺寸和位置，使其能够根据窗口大小的变化而自动调整控件的大小和位置。
## Qt中的兄弟窗口，想刷新重叠部分，请问流程是什么样的，刷新的顺序是什么样的？

1. 调用QWidget的update函数，更新当前窗口的所有内容；
2. 调用QWidget的updateGeometry函数，更新当前窗口的geometry；
3. 调用QWidget的update函数，更新当前窗口的子窗口；
4. 调用QWidget的updateGeometry函数，更新当前窗口的子窗口的geometry；
5. 调用QWidget的update函数，更新兄弟窗口；
6. 调用QWidget的updateGeometry函数，更新兄弟窗口的geometry；
7. 调用QWidget的update函数，更新兄弟窗口的子窗口；
8. 调用QWidget的updateGeometry函数，更新兄弟窗口的子窗口的geometry；
9. 调用QWidget的repaint函数，刷新重叠部分。
10. 刷新的顺序是：更新当前窗口、更新当前窗口的子窗口、更新兄弟窗口、更新兄弟窗口的子窗口、刷新重叠部分。
## Qt如何操作数据库
Qt操作数据库主要是使用Qt的QSqlDatabase类，它提供了一系列的函数来连接、操作、查询和管理数据库。在Qt中，可以使用QSqlQuery类来查询数据库，QSqlTableModel类来操作数据库表，QSqlRelationalTableModel类来管理关系数据库表，QSqlError类来检测错误，QSqlDriver类来检查数据库驱动程序，QSqlIndex类来创建索引，QSqlRecord类来操作数据库记录，QSqlResult类来执行查询等等。
## Qt Remote Object的序列化与反序列化
Qt Remote Objects是一个基于Qt的远程对象系统，它可以让你在不同的进程之间共享对象。它使用Qt的序列化技术来序列化和反序列化远程对象，以便在不同的进程之间发送和接收。
Qt的序列化技术使用QDataStream类来序列化和反序列化基本的C++数据类型，QVariant和QObject的子类。它也支持自定义类型的序列化，只要实现QDataStream的operator<<和operator>>运算符重载。
Qt Remote Objects使用QDataStream来序列化和反序列化远程对象。当客户端请求远程对象时，Qt Remote Objects将使用QDataStream将远程对象序列化，然后发送给客户端。当客户端收到序列化的远程对象时，它将使用QDataStream将远程对象反序列化，然后可以访问远程对象的属性和方法。
## Qt 中的容器类包括
QList：动态数组，支持随机访问和快速插入、删除操作。
QVector：类似 QList，但具有更好的性能。
QLinkedList：双向链表，支持快速插入、删除操作。
QHash：哈希表，支持快速查找、插入、删除操作。
QMap：基于红黑树的映射表，支持快速查找、插入、删除操作。
## Qt中的模型视图框架是什么？
模型视图框架是Qt中用于显示数据的一种机制。该框架将数据模型和视图分离，使得数据的表示和显示可以独立地进行管理。数据模型提供了数据的接口，视图则负责绘制和交互。Qt中提供了多种类型的模型视图类库，包括QAbstractItemModel、QStandardItemModel、QListView、QTableView等。 
## Qt中的插件是什么？
插件是Qt中用于扩展应用程序功能的一种机制。插件可以是动态链接库或静态链接库，包含了一些特定的功能。Qt提供了QPluginLoader类和QFactoryInterface类用于管理插件。通过插件，可以将应用程序的功能分解为多个独立的部分，方便开发和维护。
## Qt中的样式表是什么？
样式表是Qt中用于定制界面风格的一种机制。样式表使用CSS语法，可以定义界面元素的属性、颜色、字体等。Qt中的样式表可以应用于整个应用程序或特定的控件，使得应用程序的界面可以与众不同。Qt还提供了QStyle类和QStyleFactory类，用于管理系统默认样式和自定义样式。 
## 什么是Qt的MVC架构？
Qt的MVC架构是一种基于模型、视图和控制器的设计模式，它将应用程序的数据、用户界面和业务逻辑分离开来，使得各个部分之间的耦合度更低，易于维护和扩展。 
# Qt 内存相关
## 详解Qt中的内存管理机制

1. 所有继承自QOBJECT类的类，如果在new的时候指定了父亲，那么它的清理时在父亲被delete的时候delete的，所以如果一个程序中，所有的QOBJECT类都指定了父亲，那么他们是会一级级的在最上面的父亲清理时被清理，而不用自己清理；
2. 程序通常最上层会有一个根的QOBJECT，就是放在setCentralWidget（）中的那个QOBJECT，这个QOBJECT在 new的时候不必指定它的父亲，因为这个语句将设定它的父亲为总的QAPPLICATION，当整个QAPPLICATION没有时它就自动清理，所以也无需清理。9这里QT4和QT3有不同，QT3中用的是setmainwidget函数，但是这个函数不作为里面QOBJECT的父亲，所以QT3中这个顶层的QOBJECT要自行销毁）。
3. 这是有人可能会问那如果我自行delete掉这些QT接管负责销毁的指针了会出现什么情况呢，如果时这样的话，正常情况下QT的拥有这个对象的那个父亲会知道这件事情，它会直到它的儿子被你直接DELETE了，这样它会将这个儿子移出它的列表，并且重新构建显示内容，但是直接这样做时有风险的！也就是要说的下一条。
4. 当一个QOBJECT正在接受事件队列时如果中途被你DELETE掉了，就是出现问题了，所以QT中建议大家不要直接DELETE掉一个 QOBJECT，如果一定要这样做，要使用QOBJECT的deleteLater()函数，它会让所有事件都发送完一切处理好后马上清除这片内存，而且就算调用多次的deletelater也不会有问题。
5. QT不建议在一个QOBJECT 的父亲的范围之外持有对这个QOBJECT的指针，因为如果这样外面的指针很可能不会察觉这个QOBJECT被释放，会出现错误，如果一定要这样，就要记住你在哪这样做了，然后抓住那个被你违规使用的QOBJECT的destroyed（）信号，当它没有时赶快置零你的外部指针。当然我认为这样做是及其麻烦也不符合高效率编程规范的，所以如果要这样在外部持有QOBJECT的指针，建议使用引用或者用智能指针，如QT就提供了智能指针针对这些情况，见***一条。***
6. QT中的智能指针封装为QPointer类，所有QOBJECT的子类都可以用这个智能指针来包装，很多用法与普通指针一样，可以详见QT assistant
7. 通过调查这个QT的内存管理功能，发现了很多东西，现在觉得虽然这个QT弄的有点小复杂，但是使用起来还是很方便的，
要说的是某些内存泄露的检测工具会认为QT的程序因为这种方式存在内存泄露，发现时大可不必理会。
## Qt的智能指针，QSharePoint和shared_ptr有什么区别，weak_ptr呢？
Qt智能指针是一种特殊的指针，它可以指向另一个指针。它可以用来创建复杂的数据结构，如链表或树结构。
QSharePoint是一种智能指针，它可以自动管理指向的对象的内存分配和释放，从而实现自动内存管理。
shared_ptr也是一种智能指针，它可以跟踪指向的对象的引用计数，从而保证在没有任何引用的情况下，可以自动释放指向的对象。
weak_ptr是一种特殊的shared_ptr，它可以指向shared_ptr指向的对象，但不会增加指向对象的引用计数。它可以用来避免循环引用导致的内存泄漏问题。
在Qt中，指针指针（Pointer to Pointer）是一种指向指针的指针，通常用于动态分配内存或者多级指针操作。而QSharedPointer和std::shared_ptr都是C++11中的智能指针，用于管理动态内存，可以避免内存泄漏和空悬指针等问题。它们的主要区别如下：

1. QSharedPointer是Qt框架提供的智能指针，而std::shared_ptr是C++11标准库提供的智能指针。
2. QSharedPointer可以与QObject一起使用，可以自动处理QObject的引用计数，当QObject被删除时，QSharedPointer会自动释放对它的引用。而std::shared_ptr无法处理QObject的引用计数，需要手动管理。
3. QSharedPointer可以使用qSharedPointerCast进行类型转换，可以方便地将一个QSharedPointer转换为另一个QSharedPointer。而std::shared_ptr只能使用std::dynamic_pointer_cast进行类型转换。
4. QSharedPointer的默认删除器是delete，而std::shared_ptr的默认删除器是std::default_delete。

相比之下，weak_ptr则是用来解决shared_ptr循环引用问题的。当两个或多个shared_ptr相互引用时，会形成循环引用，导致内存泄漏。此时，可以使用weak_ptr来打破其中一个shared_ptr的引用，避免循环引用。weak_ptr是一种弱引用，它不会增加内存对象的引用计数，只是用来观察对象是否已经被释放，可以通过lock方法获取其对应的shared_ptr。
# Qt 线程进程相关
## 多线程使用使用方法
**方法一：**

1. 创建一个类从QThread类派生
2. 在子线程类中重写 run 函数, 将处理操作写入该函数中 
3. 在主线程中创建子线程对象, 启动子线程, 调用start()函数

**方法二：**

1. 将业务处理抽象成一个业务类, 在该类中创建一个业务处理函数
2. 在主线程中创建一QThread类对象 
3. 在主线程中创建一个业务类对象 
4. 将业务类对象移动到子线程中 
5. 在主线程中启动子线程 
6. 通过信号槽的方式, 执行业务类中的业务处理函数

**多线程使用注意事项**

1. 业务对象, 构造的时候不能指定父对象
2. 子线程中不能处理ui窗口(ui相关的类)
3. 子线程中只能处理一些数据相关的操作, 不能涉及窗口
## 多线程下，信号槽分别在什么线程中执行，如何控制
可以通过connect的第五个参数进行控制信号槽执行时所在的线程
connect有几种连接方式，直接连接和队列连接、自动连接
**直接连接（Qt::DirectConnection）：**信号槽在信号发出者所在的线程中执行
**队列连接 (Qt::QueuedConnection)**：信号在信号发出者所在的线程中执行，槽函数在信号接收者所在的线程中执行
**自动连接 (Qt::AutoConnection)**：多线程时为队列连接函数，单线程时为直接连接函数。
## Qt线程同步的方法有哪些？

1. 互斥量（QMutex）
       QMutex m_Mutex;        m_Mutex.lock();        m_Mutex.unlock();
2. 互斥锁（QMutexLocker）
        QMutexLocker mutexLocker(&m_Mutex);    
        从声明处开始（在构造函数中加锁），出了作用域自动解锁（在析构函数中解锁）。
3. 等待条件（QWaitCondition）
QWaitCondtion m_WaitCondition; m_WaitConditon.wait(&m_muxtex, time);                 
m_WaitCondition.wakeAll();
4. QReadWriteLock类
》一个线程试图对一个加了读锁的互斥量进行上读锁，允许；
》一个线程试图对一个加了读锁的互斥量进行上写锁，阻塞；
》一个线程试图对一个加了写锁的互斥量进行上读锁，阻塞；
》一个线程试图对一个加了写锁的互斥量进行上写锁，阻塞。
读写锁比较适用的情况是：需要多次对共享的数据进行读操作的阅读线程。
QReadWriterLock 与QMutex相似，除了它对 "read","write"访问进行区别对待。它使得多个读者可以共时访问数据。使用QReadWriteLock而不是QMutex，可以使得多线程程序更具有并发性。
5. 信号量QSemaphore
但是还有些互斥量（资源）的数量并不止一个，比如一个电脑安装了2个打印机，我已经申请了一个，但是我不能霸占这两个，你来访问的时候如果发现还有空闲的仍然可以申请到的。于是这个互斥量可以分为两部分，已使用和未使用。
6. QReadLocker便利类和QWriteLocker便利类对QReadWriteLock进行加解锁
## Qt的多线程，哪些是只有Qthread能实现，QtConcurrent办不到的？

1. QThread可以使用信号和槽机制，而QtConcurrent不支持。
2. QThread可以设置线程优先级，而QtConcurrent不支持。
3. QThread可以实现跨平台多线程，而QtConcurrent只能在支持C++11的平台上实现。
4. QThread可以实现更复杂的多线程任务，而QtConcurrent只能实现简单的多线程任务。
## 什么是UI线程，UI线程阻塞后会怎样？
UI线程是Android应用程序中的主线程，它负责绘制UI界面，处理用户交互，以及调度其他相关任务。如果UI线程被阻塞，用户界面将会停止响应，甚至可能会出现应用程序崩溃的情况。 
## Qt编程当中，多线程的两种使用方法？

1. 继承QThread类 继承QThread类是一种常见的多线程编程方法。该方法需要定义一个新类，继承自QThread类，并重写run()函数，run()函数中包含了需要在新线程中执行的代码。在主线程中创建该类的实例对象，调用start()函数启动新线程。 示例代码：
```cpp
class MyThread : public QThread
{
    Q_OBJECT
public:
    void run() override
    {
        // 在新线程中执行的代码
    }
};
MyThread thread;
thread.start();
```

2. 继承QObject类，使用QThread对象 继承QObject类，使用QThread对象是另一种常见的多线程编程方法。该方法需要定义一个新类，继承自QObject类，并在该类中定义需要在新线程中执行的槽函数。在主线程中创建QThread对象，将新类的实例对象移动到该QThread对象所在的线程中，然后启动该QThread对象。 示例代码：
```cpp
class MyObject : public QObject
{
    Q_OBJECT
public slots:
    void doWork()
    {
        // 在新线程中执行的代码
    }
};
QThread thread;
MyObject *obj = new MyObject();
obj->moveToThread(&thread);
QObject::connect(&thread, &QThread::started, obj, &MyObject::doWork);
thread.start();
```
# Qt 事件相关
## Qt程序是事件驱动的，事件到处都可以遇到。能说说平时经常使用到哪些事件吗？

1. 常见的QT事件类型如下:
2. 键盘事件: 按键按下和松开    鼠标事件: 鼠标移动,鼠标按键的按下和松开
3. 拖放事件: 用鼠标进行拖放    滚轮事件: 鼠标滚轮滚动
4. 绘屏事件: 重绘屏幕的某些部分    定时事件: 定时器到时
5. 焦点事件: 键盘焦点移动   进入和离开事件: 鼠标移入widget之内,或是移出
6. 移动事件: widget的位置改变    大小改变事件: widget的大小改变
7. 显示和隐藏事件: widget显示和隐藏    窗口事件: 窗口是否为当前窗口
## 知道QT事件机制有几种级别的事件过滤吗？能大致描述下吗？
根据对Qt事件机制的分析, 我们可以得到5种级别的事件过滤,处理办法. 以功能从弱到强, 排列如下:

1. 重载特定事件处理函数.
最常见的事件处理办法就是重载象mousePressEvent(), keyPressEvent(), paintEvent() 这样的特定事件处理函数
2. 重载event()函数.
通过重载event()函数,我们可以在事件被特定的事件处理函数处理之前(象keyPressEvent())处理它. 比如, 当我们想改变tab键的默认动作时,一般要重载这个函数. 在处理一些不常见的事件(比如:LayoutDirectionChange)时,evnet()也很有用,因为这些函数没有相应的特定事件处理函数. 当我们重载event()函数时, 需要调用父类的event()函数来处理我们不需要处理或是不清楚如何处理的事件.
3. 在Qt对象上安装事件过滤器.
安装事件过滤器有两个步骤: (假设要用A来监视过滤B的事件)
首先调用B的installEventFilter( const QOject *obj ), 以A的指针作为参数. 这样所有发往B的事件都将先由A的eventFilter()处理.
然后, A要重载QObject::eventFilter()函数, 在eventFilter() 中书写对事件进行处理的代码.
4. 给QAppliction对象安装事件过滤器.
一旦我们给qApp(每个程序中唯一的QApplication对象)装上过滤器,那么所有的事件在发往任何其他的过滤器时,都要先经过当前这个 eventFilter(). 在debug的时候,这个办法就非常有用, 也常常被用来处理失效了的widget的鼠标事件,通常这些事件会被QApplication::notify()丢掉. ( 在QApplication::notify() 中, 是先调用qApp的过滤器, 再对事件进行分析, 以决定是否合并或丢弃)
5. 继承QApplication类,并重载notify()函数.
Qt 是用QApplication::notify()函数来分发事件的.想要在任何事件过滤器查看任何事件之前先得到这些事件,重载这个函数是唯一的办法. 通常来说事件过滤器更好用一些, 因为不需要去继承QApplication类. 而且可以给QApplication对象安装任意个数的事件。
## Qt事件循环
Qt的主事件循环能够从事件队列中获取本地窗口系统事件，然后判断事件类型，并将事件分发给特定的接收对象。
主事件循环通过调用QCoreApplication::exec()启动，随着QCoreApplication::exit()结束，本地的事件循环可用利用QEventLoop构建。 
# Qt 面向对象相关
## Qt创建对象的方式

1. 使用Qt自带的构造函数，如QWidget，QPushButton，QDialog等。
2. 使用Qt的meta-object系统，如QMetaObject::newInstance，QMetaObject::invokeMethod等。

这两种方式的区别在于，第一种方式是使用Qt自带的构造函数，它可以直接创建Qt对象，但是不能实现动态创建，也不能调用它们的函数或者访问它们的成员变量。
第二种方式是使用Qt的meta-object系统，它可以实现动态创建Qt对象，可以调用它们的函数或者访问它们的成员变量。
# 上机实操相关
## 请写一个调用消息对话框提示报错的程序
```cpp
QMessageBox::waring(his,tr("警告"), tr(“用户名或密码错误!”), QMessageBox :Yes)
```
## Qt5实现一个文件对话框
```cpp
#include <QFileDialog>
QString file_name=QFileDialog::getOpenFileName(this,"请选择需要打开的文件：",".","*.txt *.png");  //打开文件对话框
        //参数1 父控件
        //参数2 标题
        //参数3  默认路径
        //参数4 过滤文件格式
        //返回值  文件全路径---"D:/ss/注意事项.txt"
     qDebug()<<file_name;
```
## 用Qt实现一个三角形的按钮，会如何实现？
首先，我们需要使用Qt的QPushButton类来创建一个按钮，然后设置按钮的样式，使其可以显示出一个三角形的形状。
```cpp
1. 创建QPushButton类的实例，并设置按钮的样式：

QPushButton *triangleButton = new QPushButton();
triangleButton->setStyleSheet("QPushButton{border-image:url(:/images/triangle.png);}");

2. 设置按钮的大小：

triangleButton->setFixedSize(QSize(30, 30));

3. 连接按钮的点击信号和槽函数：

connect(triangleButton, SIGNAL(clicked()), this, SLOT(onTriangleButtonClicked()));

4. 实现槽函数：

void onTriangleButtonClicked()
{
    // 在这里实现点击三角形按钮时要执行的操作
}
```
## Qt如何实现类似QQ登录窗口的翻转

1. 首先，创建一个QPropertyAnimation对象，并设置动画的目标对象、属性和时间曲线：
```cpp
QPropertyAnimation *animation = new QPropertyAnimation(this, "geometry");
animation->setDuration(500);
animation->setEasingCurve(QEasingCurve::OutExpo);
```

2. 然后，设置动画的起始值和结束值：
```cpp
//设置起始值
QRect startRect(0, 0, width(), height());
animation->setStartValue(startRect);

//设置结束值
QRect endRect(width(), 0, -width(), height());
animation->setEndValue(endRect);
```

3. 最后，启动动画：
```cpp
animation->start();
```
## Qt窗口圆角如何实现
在Qt中实现窗口圆角，可以使用Qt的样式表实现，如下所示：
```cpp
QWidget {
    border-radius: 10px;
}
// 可以使用如下代码来应用样式表：
QFile file("style.qss");
file.open(QFile::ReadOnly);
QString styleSheet = QLatin1String(file.readAll());
qApp->setStyleSheet(styleSheet);
```
## Qt如何实现QQ两个客户端的私聊功能？
Qt可以通过使用Qt的网络模块来实现QQ两个客户端的私聊功能。Qt的网络模块提供了一系列的网络协议，可以用于实现QQ两个客户端之间的私聊功能。
具体的实现步骤如下：

1. 建立两个客户端的网络连接：首先，使用Qt的网络模块建立两个客户端之间的网络连接，以便进行私聊。
2. 实现私聊功能：使用Qt的网络模块实现两个客户端之间的私聊功能，以便交换文字、图片等信息。
3. 实现断开连接：在两个客户端之间的私聊结束后，可以使用Qt的网络模块实现断开连接，以便释放资源。
## 描述过程，如何实现一个自定义按钮，使其在光标进入，按下，离开三种状态下显示不同的图片 

1. 在项目中创建三张图片，分别表示光标进入，按下，离开三种状态下的图片。
2. 创建一个自定义按钮类，在该类中重写onMouseEnter，onMouseDown，onMouseLeave三个事件，分别设置按钮的图片为对应的三张图片。
3. 在需要使用自定义按钮的地方，实例化自定义按钮类，并将其添加到页面中，即可实现光标进入，按下，离开三种状态下显示不同的图片。
## Qt当中如何读写文件？
在Qt中，可以使用QFile类和QTextStream类来读写文件。
```cpp
QFile file("test.txt");
if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
    return;
QTextStream in(&file);
while (!in.atEnd()) {
    QString line = in.readLine();
    // 处理每一行数据
}
file.close();

2. 写文件：
QFile file("test.txt");
if (!file.open(QIODevice::WriteOnly | QIODevice::Text))
    return;
QTextStream out(&file);
out << "Hello world" << endl;
out << "Qt is awesome" << endl;
file.close();
```
## Qt中事件过滤处理方法
Qt中的事件过滤器可以用于对某个对象的事件进行拦截和处理。事件过滤器是一个QObject对象，它可以安装到任何QObject派生类中，并且可以监听该对象的所有事件。当事件发生时，事件过滤器可以拦截并处理该事件，也可以将该事件转发给原始的事件接收者对象进行处理。 事件过滤器的处理方法如下：

1. 创建一个QObject派生类，实现其eventFilter()函数，该函数会在事件发生时被调用。
```cpp
class MyEventFilter : public QObject
{
Q_OBJECT
public:
explicit MyEventFilter(QObject *parent = nullptr);
protected:
bool eventFilter(QObject *watched, QEvent *event) override;
};
```
2.在需要监听的对象中，调用installEventFilter()函数安装事件过滤器。
```cpp
MyEventFilter *eventFilter = new MyEventFilter;
QLabel *label = new QLabel("Hello, World!");
label->installEventFilter(eventFilter);
```
3.在eventFilter()函数中处理事件，可以通过判断事件类型和事件源来对事件进行不同的处理。
```cpp
bool MyEventFilter::eventFilter(QObject *watched, QEvent *event)
{
    if (watched == label && event->type() == QEvent::MouseButtonPress) {
        QMouseEvent *mouseEvent = static_cast<QMouseEvent *>(event);
        // 处理鼠标事件
        return true;
    }
    return false;
}
```
## Qt 操作INI文件、JSON 文件、XML文件
Qt提供了三种常用的文件格式处理方式，分别是INI文件、JSON文件和XML文件。下面分别介绍如何在Qt中操作这三种文件格式。

1. 操作INI文件 INI文件是一种常用的配置文件格式，在Qt中可以通过QSettings类来读写INI文件。以下是一个例子：
```
QSettings settings("myApp.ini", QSettings::IniFormat);
// 写入配置项
settings.setValue("General/Name", "Tom");
settings.setValue("General/Age", 20);
// 读取配置项
QString name = settings.value("General/Name").toString();
int age = settings.value("General/Age").toInt();
```
2.操作JSON文件 JSON是一种轻量级的数据交换格式，在Qt中可以通过QJsonDocument类和QJsonObject类来读写JSON文件。以下是一个例子：
```
// 读取JSON文件
QFile file("data.json");
if (file.open(QIODevice::ReadOnly)) {
    QJsonParseError error;
    QJsonDocument doc = QJsonDocument::fromJson(file.readAll(), &error);
    if (error.error == QJsonParseError::NoError && doc.isObject()) {
        QJsonObject obj = doc.object();
        QString name = obj.value("name").toString();
        int age = obj.value("age").toInt();
    }
    file.close();
}
// 写入JSON文件
QJsonObject obj;
obj.insert("name", "Tom");
obj.insert("age", 20);
QJsonDocument doc(obj);
QFile file("data.json");
if (file.open(QIODevice::WriteOnly)) {
    file.write(doc.toJson());
    file.close();
}
```
3.操作XML文件 XML是一种常用的文本格式，用于表示结构化的数据，在Qt中可以通过QXmlStreamReader类和QXmlStreamWriter类来读写XML文件。以下是一个例子：
```
// 读取XML文件
QFile file("data.xml");
if (file.open(QIODevice::ReadOnly)) {
    QXmlStreamReader reader(&file);
    while (!reader.atEnd()) {
        reader.readNext();
        if (reader.isStartElement() && reader.name() == "person") {
            QString name = reader.attributes().value("name").toString();
            int age = reader.attributes().value("age").toInt();
            // 处理读取到的数据
        }
    }
    file.close();
}
// 写入XML文件
QFile file("data.xml");
if (file.open(QIODevice::WriteOnly)) {
    QXmlStreamWriter writer(&file);
    writer.setAutoFormatting(true);
    writer.writeStartDocument();
    writer.writeStartElement("persons");
    writer.writeStartElement("person");
    writer.writeAttribute("name", "Tom");
    writer.writeAttribute("age", "20");
    writer.writeEndElement();
    writer.writeEndElement();
    writer.writeEndDocument();
    file.close();
}
```
以上是Qt中操作INI文件、JSON文件和XML文件的简单示例，通过这些方法，我们可以方便地读写和处理各种数据格式的文件。
## QtChart (图表、曲线图、饼状图、柱形、拆线图等) ？
QtChart是Qt自带的图表库，用于绘制各种类型的图表，例如曲线图、饼状图、柱形图、折线图等。使用QtChart可以方便地将数据可视化，并支持用户交互和定制化设置。下面介绍QtChart中常用的几种图表类型及其使用方法。

1. 曲线图 曲线图用于展示一段时间内某个变量的变化趋势，例如温度、湿度、压力等。使用QtChart绘制曲线图的步骤如下：
- 创建QChart对象，并设置标题和坐标轴。
- 创建QLineSeries对象，设置曲线的名称和数据。
- 将QLineSeries对象添加到QChart对象中。
- 创建一个QChartView对象，并将QChart对象设置为其父对象。
- 将QChartView对象添加到布局中。 以下是一个简单的曲线图绘制示例：
```
QChart *chart = new QChart();
chart->setTitle("Temperature");
chart->legend()->hide();
chart->createDefaultAxes();
QLineSeries *series = new QLineSeries();
series->setName("Temperature");
series->append(0, 20);
series->append(1, 22);
series->append(2, 24);
series->append(3, 26);
chart->addSeries(series);
QChartView *chartView = new QChartView(chart);
chartView->setRenderHint(QPainter::Antialiasing);
ui->chartLayout->addWidget(chartView);
```
2.饼状图 饼状图用于展示不同数据之间的比例关系，例如不同国家的GDP占比、不同产品的销售占比等。使用QtChart绘制饼状图的步骤如下：

- 创建QChart对象，并设置标题。
- 创建QPieSeries对象，设置饼状图的名称和数据。
- 将QPieSeries对象添加到QChart对象中。
- 创建一个QChartView对象，并将QChart对象设置为其父对象。
- 将QChartView对象添加到布局中。 以下是一个简单的饼状图绘制示例：
```
QChart *chart = new QChart();
chart->setTitle("Sales");
QPieSeries *series = new QPieSeries();
series->setName("Product");
series->append("Product A", 25);
series->append("Product B", 35);
series->append("Product C", 40);
chart->addSeries(series);
chart->legend()->setAlignment(Qt::AlignRight);
QChartView *chartView = new QChartView(chart);
chartView->setRenderHint(QPainter::Antialiasing);
ui->chartLayout->addWidget(chartView);
```
3.柱形图 柱形图用于展示不同数据之间的数量关系，例如不同月份的销售额、不同省份的人口等。使用QtChart绘制柱形图的步骤如下：

- 创建QChart对象，并设置标题和坐标轴。
- 创建QBarSeries对象，设置柱形图的名称和数据。
- 将QBarSeries对象添加到QChart对象中。
- 创建一个QChartView对象，并将QChart对象设置为其父对象。
- 将QChartView对象添加到布局中。 以下是一个简单的柱形图绘制示例：
```
QChart *chart = new QChart();
chart->setTitle("Sales");
chart->setAnimationOptions(QChart::SeriesAnimations);
QBarSeries *series = new QBarSeries();
series->setName("Monthly Sales");
QBarSet *set1 = new QBarSet("January");
QBarSet *set2 = new QBarSet("February");
QBarSet *set3 = new QBarSet("March");
*set1 << 100 << 200 << 150;
*set2 << 150 << 100 << 250;
*set3 << 200 << 150 << 100;
series->append(set1);
series->append(set2);
series->append(set3);
chart->addSeries(series);
chart->createDefaultAxes();
chart->legend()->setVisible(true);
QChartView *chartView = new QChartView(chart);
chartView->setRenderHint(QPainter::Antialiasing);
ui->chartLayout->addWidget(chartView);
```
4.折线图 折线图用于展示不同数据之间的趋势变化，例如不同时间段内的股票价格、不同季度的GDP增长率等。使用QtChart绘制折线图的步骤与曲线图类似。 以上是QtChart中常用的几种图表类型的绘制示例，通过QtChart，我们可以方便地将数据可视化，让数据更加直观和易于理解。
## Qt 中音频类和视频类分别是什么？
在Qt中，音频类和视频类分别是QAudio和QMediaPlayer。

1. QAudio QAudio类提供了音频的输入和输出功能，可以用于录音、播放音频等操作。使用QAudio需要以下步骤：
- 创建QAudioDeviceInfo对象，获取可用的音频设备信息。
- 创建QAudioFormat对象，设置音频格式。
- 创建QAudioInput或QAudioOutput对象，设置音频设备和音频格式。
- 开始录音或播放音频数据。 以下是一个简单的录音示例：
```
QAudioFormat format;
format.setSampleRate(44100);
format.setChannelCount(2);
format.setSampleSize(16);
format.setCodec("audio/pcm");
format.setByteOrder(QAudioFormat::LittleEndian);
format.setSampleType(QAudioFormat::SignedInt);
QAudioDeviceInfo info(QAudioDeviceInfo::defaultInputDevice());
if (!info.isFormatSupported(format)) {
    qWarning() << "Default format not supported, trying to use nearest format.";
    format = info.nearestFormat(format);
}
QAudioInput *audioInput = new QAudioInput(info, format);
QBuffer *buffer = new QBuffer(this);
buffer->open(QIODevice::ReadWrite);
audioInput->start(buffer);


```
## QML鼠标与事件处理？ QML布局？ Loader 动态加载组件？
在 QML 中，可以通过鼠标与事件处理来响应用户的操作。常见的鼠标事件包括：

- 点击事件（MouseArea）
- 悬停事件（hover）
- 按下事件（pressed）
- 松开事件（released）
- 移动事件（MouseArea） 在 QML 中，可以使用 MouseArea 来处理鼠标事件。例如：
```
Rectangle {
    width: 100
    height: 100
    color: "blue"
    MouseArea {
        anchors.fill: parent
        onClicked: {
            console.log("Clicked!")
        }
    }
}
```
上面的代码创建了一个蓝色的矩形，并在其上添加了一个 MouseArea 来处理鼠标事件。当用户点击矩形时，会在控制台输出一条消息。
QML布局
在 QML 中，可以使用各种布局来管理控件的位置和大小。常见的布局包括：

- ColumnLayout
- RowLayout
- GridLayout
- StackLayout 以 ColumnLayout 为例：
```
ColumnLayout {
    spacing: 10
    Rectangle {
        width: 100
        height: 50
        color: "red"
    }
    Rectangle {
        width: 100
        height: 50
        color: "green"
    }
    Rectangle {
        width: 100
        height: 50
        color: "blue"
    }
}
```
上面的代码创建了一个 ColumnLayout，并在其中添加了三个矩形。这些矩形会依次排列在垂直方向上，并且它们之间的间距为 10。
Loader 动态加载组件
在 QML 中，可以使用 Loader 组件来动态加载其他组件。例如：
```
Loader {
    sourceComponent: Rectangle {
        width: 100
        height: 100
        color: "red"
    }
}
```
上面的代码创建了一个 Loader，并将其 sourceComponent 属性设置为一个红色的矩形。在运行时，Loader 会动态加载这个矩形，并显示在其内部。可以通过改变 sourceComponent 属性来动态加载不同的组件。
## Qt相机和视频处理技术？
Qt相机和视频处理技术是Qt提供的一组API和库，用于在Qt应用程序中访问摄像头和处理视频。以下是一些常用的Qt相机和视频处理技术：

1. QtMultimedia模块：提供了访问音频、视频、相机等多媒体设备的API。可以用来在Qt应用程序中访问相机，捕捉视频和音频流，以及播放音频和视频文件。
2. QtCamera模块：是QtMultimedia模块的一部分，提供了访问相机的API。可以用来获取相机的设备信息、预览相机的图像、捕捉相机的图像和视频等。
3. QtAV库：是一个基于Qt的多媒体框架，提供了高性能的视频和音频处理能力。可以用来播放各种格式的视频和音频文件，以及进行视频编解码和处理。
4. OpenCV库：是一个开源的计算机视觉库，提供了丰富的图像和视频处理算法。可以用来在Qt应用程序中实现图像和视频的处理，例如人脸识别、目标跟踪、图像滤波等。
5. FFmpeg库：是一个开源的音视频编解码库，支持多种音视频格式和编解码器。可以用来在Qt应用程序中实现音视频的录制、播放和转码等功能。 通过使用这些技术，开发者可以方便地在Qt应用程序中访问相机和处理视频，实现各种有趣的功能，例如人脸识别、视频监控、视频编辑等。
## Qt当中文件对话框、字体对话体、输入对话框、消息对话框应用实战？
以下是四个对话框在Qt中的应用实战：

1. 文件对话框（QFileDialog） QFileDialog是Qt中用于打开和保存文件的对话框类。它可以让用户选择文件的路径和名称，并且支持多种文件格式的过滤。以下是一个文件对话框的应用实例：
```
QString fileName = QFileDialog::getOpenFileName(this, "Open File", QDir::homePath(), "Text Files (*.txt)");
if(!fileName.isEmpty()) {
    QFile file(fileName);
    if(file.open(QIODevice::ReadOnly)) {
        // 读取文件内容
        file.close();
    }
}
```
2.字体对话框（QFontDialog） QFontDialog是Qt中用于选择字体的对话框类。它可以让用户选择字体的名称、大小、颜色等属性。以下是一个字体对话框的应用实例：
```
bool ok;
QFont font = QFontDialog::getFont(&ok, QFont("Helvetica [Cronyx]", 10), this);
if(ok) {
    // 应用选择的字体
    setFont(font);
}
```
3.输入对话框（QInputDialog） QInputDialog是Qt中用于输入文本、数字、列表等的对话框类。它可以让用户输入各种类型的数据，并且支持自定义对话框标题、提示信息等。以下是一个输入对话框的应用实例：
```
QString text = QInputDialog::getText(this, "Input Dialog", "Enter your name:", QLineEdit::Normal, "", &ok);
if(ok && !text.isEmpty()) {
    // 处理用户输入的文本
}
```
4.消息对话框（QMessageBox） QMessageBox是Qt中用于显示消息、提示、警告等的对话框类。它可以让用户选择不同的按钮选项，并且支持自定义对话框标题、提示信息等。以下是一个消息对话框的应用实例：
```cpp
QMessageBox::StandardButton reply;
reply = QMessageBox::question(this, "Message Box", "Are you sure to quit？", QMessageBox::Yes | QMessageBox::No);
if(reply == QMessageBox::Yes) {
    // 处理用户选择
}
```
需要注意的是，四个对话框的应用场景不同，需要根据实际需求进行选择和使用。
## Qt绘制原理双缓冲机制？
Qt绘制原理中的双缓冲机制是指在绘制过程中使用两个缓冲区，一个用于绘制，一个用于显示，从而避免了绘制过程中的闪烁等问题。具体来说，双缓冲机制的实现过程如下：

1. 创建两个缓冲区，一个用于绘制，一个用于显示；
2. 在绘制过程中，将所有的绘制操作都先绘制到绘制缓冲区中；
3. 绘制完成后，将绘制缓冲区中的内容复制到显示缓冲区中；
4. 显示缓冲区中的内容会被显示在屏幕上。 这样，由于绘制操作是在绘制缓冲区中进行的，因此不会直接影响到显示缓冲区中的内容，从而避免了闪烁等问题。 在Qt中，双缓冲机制是由QPainter类和QWidget类共同实现的。QPainter类用于在绘制缓冲区中进行绘制操作，而QWidget类则在显示缓冲区中进行显示操作。具体来说，当QWidget类的paintEvent()函数被触发时，会创建一个QPainter对象，然后在其中调用绘制函数，将所有的绘制操作都绘制到绘制缓冲区中。绘制完成后，QPainter对象会自动将绘制缓冲区中的内容复制到显示缓冲区中，然后显示缓冲区中的内容会被显示在屏幕上。 需要注意的是，双缓冲机制虽然可以避免闪烁等问题，但是也会增加内存的消耗。因此，在实际应用中需要根据具体情况进行选择和使用。
## Graphics View图形视图框架结构？
Graphics View是Qt中用于显示和处理大量图形元素的框架，其主要包括以下几个重要的类和组件：

1. QGraphicsItem和QGraphicsScene：QGraphicsItem类是Graphics View框架中所有图形项的基类，它定义了所有图形项的共同属性和行为。QGraphicsScene类是Graphics View框架中的场景类，它管理着所有的图形项，并且提供了对它们进行操作的接口。
2. QGraphicsView：QGraphicsView类是Graphics View框架中的视图类，它用于在屏幕上显示QGraphicsScene中的内容，并且提供了对场景进行缩放、平移、旋转等操作的接口。
3. QGraphicsWidget：QGraphicsWidget类是Graphics View框架中的窗口类，它可以作为一个图形项添加到QGraphicsScene中，并且支持鼠标事件、键盘事件等交互操作。
4. QGraphicsProxyWidget：QGraphicsProxyWidget类是Graphics View框架中的窗口代理类，它可以将一个QWidget对象转换为一个图形项，然后添加到QGraphicsScene中。
5. QGraphicsItemGroup：QGraphicsItemGroup类是Graphics View框架中的图形项组类，它可以将多个图形项组合成一个整体，从而方便对它们进行操作。
6. QGraphicsEffect：QGraphicsEffect类是Graphics View框架中的特效类，它可以对QGraphicsItem进行图形效果的处理，例如模糊、阴影、发光等。
7. QGraphicsPixmapItem和QGraphicsTextItem：QGraphicsPixmapItem类是Graphics View框架中的图片项类，它可以将一个QPixmap对象显示在场景中。QGraphicsTextItem类是Graphics View框架中的文本项类，它可以将一个字符串显示在场景中。 以上是Graphics View框架的主要组件和类，它们共同构成了Graphics View框架的结构。通过这些组件和类的使用，我们可以方便地实现各种复杂的图形界面和图形效果。
