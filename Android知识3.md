* Httpclient 与 httpurlclient 之间的区别

  Http Client适用于web浏览器，拥有大量灵活的API，实现起来比较稳定，且其功能比较丰富，提供了很多工具，封装了http的请求头，参数，内容体，响应，还有一些高级功能，代理、COOKIE、鉴权、压缩、连接池的处理。正是因为如此，API 比较多难以修改，android 在 6.0 中抛弃了它，转而用 okhttp

  HttpURLConnection对于大部分功能都进行了包装，Http Client的高级功能代码会较复杂，另外，HttpURLConnection在Android 2.3中增加了一些Https方面的改进(包括Http Client，两者都支持https)。且在Android 4.0中增加了response cache。

  - **功能用法对比**
    - 从功能上对比，HttpClient库要丰富很多，提供了很多工具，封装了http的请求头，参数，内容体，响应，还有一些高级功能，代理、COOKIE、鉴权、压缩、连接池的处理。
    - HttpClient高级功能代码写起来比较复杂，对开发人员的要求会高一些，而HttpURLConnection对大部分工作进行了包装，屏蔽了不需要的细节，适合开发人员直接调用。
    - 另外，HttpURLConnection在2.3版本增加了一些HTTPS方面的改进，4.0版本增加一些响应的缓存。
  - **性能对比**
    - HttpUrlConnection直接支持GZIP压缩；HttpClient也支持，但要自己写代码处理。
    - HttpUrlConnection直接支持系统级连接池，即打开的连接不会直接关闭，在一段时间内所有程序可共用；HttpClient当然也能做到，但毕竟不如官方直接系统底层支持好。
    - HttpUrlConnection直接在系统层面做了缓存策略处理（4.0版本以上），加快了重复请求的速度。
    - [这篇文章](http://wj98127.iteye.com/blog/617014)对两者的速度做了一个对比，做法是两个类都使用默认的方法去请求百度的网页内容，测试结果是使用httpurlconnection耗时47ms，使用httpclient耗时641ms。httpURLConnection在速度有比较明显的优势，当然这跟压缩内容和缓存都有直接关系。

* APK 瘦身

  * 移除冗余的代码 用 proguard 来检查
  * 移除无用的资源 用 lint 工具
  * 对资源文件进行取舍
  * 优化压缩图片

* Android 横竖屏切换

  onSaveInstanceState-->onPause-->onStop-->onDestroy-->onCreate-->onStart-->onRestoreInstanceState-->onResume-->

  * 不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次

  * 设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次

  * 设置Activity的android:configChanges="orientation|keyboardHidden"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

* 如何保证线程同步的安全性

  问：synchronized即可修饰非静态方式，也可修饰静态方法，还可修饰代码块，有何区别
  答：synchronized修饰非静态同步方法时，锁住的是当前实例；synchronized修饰静态同步方法时，锁住的是该类的Class对象；synchronized修饰静态代码块时，锁住的是`synchronized`关键字后面括号内的对象。

  问：既然锁和synchronized即可保证原子性也可保证可见性，为何还需要volatile？
  答：synchronized和锁需要通过操作系统来仲裁谁获得锁，开销比较高，而volatile开销小很多。因此在只需要保证可见性的条件下，使用volatile的性能要比使用锁和synchronized高得多。

  ​

* ontouchevent 和 ontouch 有什么区别

  onTouchListener的onTouch方法优先级比onTouchEvent高，会先触发。假如onTouch方法返回false会接着触发onTouchEvent，反之onTouchEvent方法不会被调用。内置诸如click事件的实现等等都基于onTouchEvent，假如onTouch返回true，这些事件将不会被触发。

  * OnTouchEvent()方法：

   onTouchEvent是手机屏幕事件的处理方法，是获取的对屏幕的各种操作，比如向左向右滑动，点击返回按钮等等。属于一个宏观的屏幕触摸监控。onTouchEvent方法是override 的Activity的方法。重新了Activity的onTouchEvent方法后，当屏幕有touch事件时，此方法就会别调用。

  * OnTouch方法

  onTouch()是OnTouchListener接口的方法，它是获取某一个控件的触摸事件，因此使用时，必须使用setOnTouchListener绑定到控件，然后才能鉴定该控件的触摸事件。当一个View绑定了OnTouchLister后，当有touch事件触发时，就会调用onTouch方法。通过getAction()方法可以获取当前触摸事件的状态：

    * ACTION_DOWN：表示按下了屏幕的状态。
    * ACTION_MOVE ：表示为移动手势
    * ACTION_UP：表示为离开屏幕
    * ACTION_CANCEL ：表示取消手势，不会由用户产生，而是由程序产生的

* Android 中如何避免 oom

  * 使用更加轻量级的数据结构 ArrayMap/SparseArray 来替代传统的 HashMap
  * 避免在 Android 中使用枚举,以及减少 Bitmap 所占的内存
  * 缩放图片比例以及尽量不要去加载大图
  * 在ListView和GridView等显示大量图片的控件里面需要使用LRU机制缓存Bitmap.

* SparseArray 的特点

  SparseArray只能存储key为int类型的数据，同时，SparseArray在存储和读取数据时候，使用的是二分查找法
  SparseArray比HashMap更省内存，在某些条件下性能更好，主要是因为它避免了对key的自动装箱（int转为Integer类型），它内部则是通过两个数组来进行数据存储的，一个存储key，另外一个存储value，为了优化性能，它内部对数据还采取了压缩的方式来表示稀疏数组的数据，从而节约内存空间

* ArrayMap 的特点

  这个api的资料在网上可以说几乎没有，然并卵，只能看文档了 
  ArrayMap是一个<**key,value**>映射的[数据结构](http://lib.csdn.net/base/datastructure)，它设计上更多的是考虑内存的优化，内部是使用两个数组进行数据存储，一个数组记录key的hash值，另外一个数组记录Value值，它和SparseArray一样，也会对key使用二分法进行从小到大排序，在添加、删除、查找数据的时候都是先使用二分查找法得到相应的index，然后通过index来进行添加、查找、删除等操作，所以，应用场景和SparseArray的一样，如果在数据量比较大的情况下，那么它的性能将退化至少50%。


* Java 静态方法可以被重写吗？

  如果从重写方法会有什么特点来看，我们是不能重写静态方法的。虽然就算你重写静态方法，编译器也不会报错。也就是说，如果你试图重写静态方法，Java不会阻止你这么做，但你却得不到预期的结果（重写仅对非静态方法有用）。重写指的是根据运行时对象的类型来决定调用哪个方法，而不是根据编译时的类型。让我们猜一猜为什么静态方法是比较特殊的？因为它们是类的方法，所以它们在编译阶段就使用编译出来的类型进行绑定了。使用对象引用来访问静态方法只是Java设计者给程序员的自由。我们应该直接使用类名来访问静态方法，而不要使用对象引用来访问。


* 什么是三级缓存

  * 网络缓存, 不优先加载, 速度慢,浪费流量
  * 本地缓存, 次优先加载, 速度快
  * 内存缓存, 优先加载, 速度最快

* Android 中的 IPC 机制

  * 使用文件
  * 使用 messager
  * 使用 socket
  * 使用 contentprovider
  * 使用 bundle
  * 使用 AIDL

* AIDL 机制，其实就是 android 中的 ipc 机制

  所谓AIDL：Android Interface Definition Language，是一种Android接口定义语言，用于编写Android进程间通信代码。也就是说AIDL只是一个实现进程间通信的一个工具，真正实现Android进程间通信机制的其实是幕后“主谋”Binder机制。所有有关AIDL实现进程间通信都是依赖于Android的Binder机制，那么这个Binder机制到底是个什么东西呢？

  * AIDL适用场景

  官方文档特别提醒我们何时使用AIDL是必要的：只有你允许客户端从不同的应用程序为了进程间的通信而去访问你的service，以及想在你的service处理多线程。具体的不同场景下的AIDL设计规则如下：

  如果不需要进行不同应用程序间的并发通信(IPC)，通过实现一个Binder对象来创建接口；或者你想进行IPC，但不需要处理多线程的，则使用Messenger对象来创建接口。无论如何，在使用AIDL前，必须要理解如何绑定service——bindService。

  * AIDL开发步骤

  AIDL接口文件，和普通的接口内容没有什么特别，只是它的扩展名为.aidl，保存在src目录下。如果其他应用程序需要IPC，则那些应用程序的src也要带有这个文件。Android SDK tools就会在gen目录自动生成一个IBinder接口文件。service必须适当地实现这个IBinder接口。那么客户端程序就能绑定这个service并在IPC时从IBinder调用方法。每个aidl文件只能定义一个接口，而且只能是接口的声明和方法的声明。

  换言之，AIDL文件的作用即是在任何需要跨进程调用的方法都需要声明在涉及的每一个服务或者进程中。

* service 生命周期

  Service对象不能自己启动，需要通过某个Activity、Service或者其他Context对象来启动。启动的方法有两种，Context.startService和Context.bindService()。两种方式的生命周期是不同的，具体如下所示。

  Context.startService方式的生命周期： 
  启动时，startService –> onCreate() –> onStart() 
  停止时，stopService –> onDestroy()

  Context.bindService方式的生命周期： 
  绑定时,bindService  -> onCreate() –> onBind() 
  解绑定时,unbindService –>onUnbind() –> onDestory()


* srcoll 工作原理

  Scroller的工作原理：Scroller本身并不能实现view的滑动，它需要配合**view的computeScroll方法**才能完成弹性滑动的效果，它不断地让view重绘，而每一次重绘距滑动起始时间会有一个时间间隔，通过这个时间间隔Scroller就可以得出view的当前的滑动位置，知道了滑动位置就可以通过scrollTo方法来完成view的滑动。就这样，view的每一次重绘都会导致view进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动，这就是Scroller的工作原理。

* view 的滑动冲突的解决

  * 常见的滑动冲突的场景：

  1.外部滑动方向和内部滑动方向不一致，例如viewpager中包含listview；

  2.外部滑动方向和内部滑动方向一致，例如viewpager的单页中存在可以滑动的bannerview；

  3.上面两种情况的嵌套，例如viewpager的单个页面中包含了bannerview和listview。

  * 滑动冲突处理规则

  可以根据滑动距离和水平方向形成的夹角；或者根绝水平和竖直方向滑动的距离差；或者两个方向上的速度差等

  * 解决方式

  1.外部拦截法：点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，如果不需要就不拦截。该方法需要重写父容器的`onInterceptTouchEvent`方法，在内部做相应的拦截即可，其他均不需要做修改。

  2.内部拦截法：父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交给父容器来处理。这种方法和Android中的事件分发机制不一致，需要配合`requestDisallowInterceptTouchEvent`方法才能正常工作。


* 属性动画

  新引入的属性动画机制已经不再是针对于View来设计的了，也不限定于只能实现移动、缩放、旋转和淡入淡出这几种动画操作，同时也不再只是一种视觉上的动画效果了。它实际上是一种不断地对值进行操作的机制，并将值赋值到指定对象的指定属性上，可以是任意对象的任意属性。所以我们仍然可以将一个View进行移动或者缩放，但同时也可以对自定义View中的Point对象进行动画操作了。我们只需要告诉系统动画的运行时长，需要执行哪种类型的动画，以及动画的初始值和结束值，剩下的工作就可以全部交给系统去完成了。

* **APK 打包流程**

  1、打包资源文件，生成R.java文件

  2、处理aidl文件，生成相应java 文件

  3、编译工程源代码，生成相应class 文件

  4、转换所有class文件，生成classes.dex文件

  5、打包生成apk

  6、对apk文件进行签名

  7、对签名后的apk文件进行对其处理

* Java 中的异常

  - **检查性异常：**最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。例如要打开一个不存在文件时，一个异常就发生了，这些异常在编译时不能被简单地忽略。

  - **运行时异常：** 运行时异常是可能被程序员避免的异常。与检查性异常相反，运行时异常可以在编译时被忽略。

  - **错误：** 错误不是异常，而是脱离程序员控制的问题。错误在代码中通常被忽略。例如，当栈溢出时，一个错误就发生了，它们在编译也检查不到的。

  - finally 关键字用来创建在 try 代码块后面执行的代码块。

    无论是否发生异常，finally 代码块中的代码总会被执行。

    在 finally 代码块中，可以运行清理类型等收尾善后性质的语句。

  - **throws 用于抛出方法层次的异常**， 
    并且直接由些方法调用异常处理类来处理该异常， 
    所以它常用在方法的后面。比如 
    public static void main(String[] args) throws SQLException

    **throw 用于方法块里面的代码**，比throws的层次要低，比如try...catch ....语句块，表示它抛出异常， 
    但它不会处理它， 而是由方法块的throws Exception来调用异常处理类来处理。

    throw用在程序中，明确表示这里抛出一个异常。   
    throws用在方法声明的地方，表示这个方法可能会抛出某异常。

    throw是抛出一个具体的异常类，产生一个异常。
    throws则是在方法名后标出该方法会产生何种异常需要方法的使用者捕获并处理。

* Butterknife 最新的实现原理，主要是利用 APT 在编译期生成相关代码  https://joyrun.github.io/2016/07/19/AptHelloWorld/

  APT(Annotation Processing Tool)是一种处理注释的工具,它对源代码文件进行检测找出其中的Annotation，使用Annotation进行额外的处理。
  Annotation处理器在处理Annotation时可以根据源文件中的Annotation生成额外的源文件和其它的文件(文件具体内容由Annotation处理器的编写者决定),APT还会编译生成的源文件和原来的源文件，将它们一起生成class文件。


* view 动画(视图动画) 的缺点

  * 只针对 View 对象进行动画

  * 然后补间动画还有一个缺陷，就是它只能够实现移动、缩放、旋转和淡入淡出这四种动画操作

  * 就是它只是改变了View的显示效果而已，而不会真正去改变View的属性。什么意思呢？比如说，现在屏幕的左上角有一个按钮，然后我们通过补间动画将它移动到了屏幕的右下角，现在你可以去尝试点击一下这个按钮，点击事件是绝对不会触发的，因为实际上这个按钮还是停留在屏幕的左上角，只不过补间动画将这个按钮绘制到了屏幕的右下

* Java 当中的 clone 和 new 对象有什么区别？

  那么这两种方式有什么相同和不同呢？ new操作符的本意是分配内存。程序执行到new操作符时， 首先去看new操作符后面的类型，因为知道了类型，才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。而clone在第一步是和new相似的， 都是分配内存，调用clone方法时，分配的内存和源对象（即调用clone方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，clone方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部。

  
* 七大设计原则：

  * 单一职责原则【SINGLE RESPONSIBILITY PRINCIPLE】：一个类负责一项职责.
  * 里氏替换原则【LISKOV SUBSTITUTION PRINCIPLE】：继承与派生的规则.
  * 依赖倒置原则【DEPENDENCE INVERSION PRINCIPLE】：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。即针对接口编程，不要针对实现编程.
  * 接口隔离原则【INTERFACE SEGREGATION PRINCIPLE】：建立单一接口，不要建立庞大臃肿的接口，尽量细化接口，接口中的方法尽量少.
  * 迪米特法则【LOW OF DEMETER】：低耦合，高内聚.
  * 开闭原则【OPEN CLOSE PRINCIPLE】：一个软件实体如类、模块和函数应该对扩展开放，对修改关闭.


* 进程和线程的区别

    进程，是并发执行的程序在执行过程中分配和管理资源的基本单位，是一个动态概念，竟争计算机系统资源的基本单位。每一个进程都有一个自己的地址空间，即进程空间或（虚空间）。进程空间的大小 只与处理机的位数有关，一个 16 位长处理机的进程空间大小为 216 ，而 32 位处理机的进程空间大小为 232 。进程至少有 5 种基本状态，它们是：初始态，执行态，等待状态，就绪状态，终止状态。
    线程，在网络或多用户环境下，一个服务器通常需要接收大量且不确定数量用户的并发请求，为每一个请求都创建一个进程显然是行不通的，——无论是从系统资源开销方面或是响应用户请求的效率方面来看。因此，操作系统中线程的概念便被引进了。线程，是进程的一部分，一个没有线程的进程可以被看作是单线程的。线程有时又被称为轻权进程或轻量级进程，也是 CPU 调度的一个基本单位。
    进程的执行过程是线状的，尽管中间会发生中断或暂停，但该进程所拥有的资源只为该线状执行过程服务。一旦发生进程上下文切换，这些资源都是要被保护起来的。这是进程宏观上的执行过程。而进程又可有单线程进程与多线程进程两种。我们知道，进程有 一个进程控制块 PCB ，相关程序段 和 该程序段对其进行操作的数据结构集 这三部分，单线程进程的执行过程在宏观上是线性的，微观上也只有单一的执行过程；而多线程进程在宏观上的执行过程同样为线性的，但微观上却可以有多个执行操作（线程），如不同代码片段以及相关的数据结构集。线程的改变只代表了 CPU 执行过程的改变，而没有发生进程所拥有的资源变化。出了 CPU 之外，计算机内的软硬件资源的分配与线程无关，线程只能共享它所属进程的资源。与进程控制表和 PCB 相似，每个线程也有自己的线程控制表 TCB ，而这个 TCB 中所保存的线程状态信息则要比 PCB 表少得多，这些信息主要是相关指针用堆栈（系统栈和用户栈），寄存器中的状态数据。进程拥有一个完整的虚拟地址空间，不依赖于线程而独立存在；反之，线程是进程的一部分，没有自己的地址空间，与进程内的其他线程一起共享分配给该进程的所有资源。

    线程可以有效地提高系统的执行效率，但并不是在所有计算机系统中都是适用的，如某些很少做进程调度和切换的实时系统。使用线程的好处是有多个任务需要处理机处理时，减少处理机的切换时间；而且，线程的创建和结束所需要的系统开销也比进程的创建和结束要小得多。最适用使用线程的系统是多处理机系统和网络系统或分布式系统。
    总结：
    * 关系
    一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行.
    相对进程而言，线程是一个更加接近于执行体的概念，它可以与同进程中的其他线程共享数据，但拥有自己的栈空间，拥有独立的执行序列。

    * 区别
    进程和线程的主要差别在于它们是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。

    1) 简而言之,一个程序至少有一个进程,一个进程至少有一个线程.
    2) 线程的划分尺度小于进程，使得多线程程序的并发性高。
    3) 另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。
    4) 线程在执行过程中与进程还是有区别的。
    每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。
    5) 从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。
    但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。
    * 优缺点
    线程和进程在使用上各有优缺点：线程执行开销小，但不利于资源的管理和保护；而进程正相反。同时，线程适合于在SMP机器上运行，而进程则可以跨机器迁移。

* Android 4.4 与 Android 5.0 之间的区别

  * md 设计
  * 在第 4.4 版中，ART 是可选的，默认运行时仍为 Dalvik。对于 Android 5.0，默认运行时现在是 ART。
  * 提醒通知以及整合碎片化

    ​


*   socket 链接过程

    socket 不仅能保证同一台计算机中的不同进程之间的通信，还能保证不同计算机之间的通信，套接字通信

    socket 是对 tcp/ip 协议的封装，socket 本身不是通信协议，而是提供一些调用接口，通过 socket 我们才能使用  tcp/ip。

    * http 链接：就是所谓的短连接，即客户端向服务器发送一次请求，服务端响应后会立即断掉
    * socket 链接：就是长链接，理论上客户端和服务端一旦建立起链接将不会主动断掉。

*   HTTP连接

    HTTP协议即超文本传送协议(Hypertext Transfer Protocol )，是Web联网的基础，也是手机联网常用的协议之一，HTTP协议是建立在TCP协议之上的一种应用。

    HTTP连接最显著的特点是客户端发送的每次请求都需要服务器回送响应，在请求结束后，会主动释放连接。从建立连接到关闭连接的过程称为“一次连接”。

    1）在HTTP 1.0中，客户端的每次请求都要求建立一次单独的连接，在处理完本次请求后，就自动释放连接。 
    2）在HTTP 1.1中则可以在一次连接中处理多个请求，并且多个请求可以重叠进行，不需要等待一个请求结束后再发送下一个请求。 
    由于HTTP在每次请求结束后都会主动释放连接，因此HTTP连接是一种“短连接”，要保持客户端程序的在线状态，需要不断地向服务器发起连接请求。通常的做法是即时不需要获得任何数据，客户端也保持每隔一段固定的时间向服务器发送一次“保持连接”的请求，服务器在收到该请求后对客户端进行回复，表明知道客户端“在线”。若服务器长时间无法收到客户端的请求，则认为客户端“下线”，若客户端长时间无法收到服务器的回复，则认为网络已经断开。

*   SOCKET原理

    3.1套接字（socket）概念

    套接字（socket）是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的协议，本地主机的IP地址，本地进程的协议端口，远地主机的IP地址，远地进程的协议端口。

    应用层通过传输层进行数据通信时，TCP会遇到同时为多个应用程序进程提供并发服务的问题。多个TCP连接或多个应用程序进程可能需要通过同一个 TCP协议端口传输数据。为了区别不同的应用程序进程和连接，许多计算机操作系统为应用程序与TCP／IP协议交互提供了套接字(Socket)接口。应用层可以和传输层通过Socket接口，区分来自不同应用程序进程或网络连接的通信，实现数据传输的并发服务。

    3.2 建立socket连接 
    建立Socket连接至少需要一对套接字，其中一个运行于客户端，称为ClientSocket ，另一个运行于服务器端，称为ServerSocket 。

    套接字之间的连接过程分为三个步骤：服务器监听，客户端请求，连接确认。

    服务器监听：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求。

    客户端请求：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。

    连接确认：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

*   SOCKET连接与TCP连接

    创建Socket连接时，可以指定使用的传输层协议，Socket可以支持不同的传输层协议（TCP或UDP），当使用TCP协议进行连接时，该Socket连接就是一个TCP连接。

*   Socket连接与HTTP连接

    由于通常情况下Socket连接就是TCP连接，因此Socket连接一旦建立，通信双方即可开始相互发送数据内容，直到双方连接断开。但在实际网络应用中，客户端到服务器之间的通信往往需要穿越多个中间节点，例如路由器、网关、防火墙等，大部分防火墙默认会关闭长时间处于非活跃状态的连接而导致 Socket 连接断连，因此需要通过轮询告诉网络，该连接处于活跃状态。

    而HTTP连接使用的是“请求—响应”的方式，不仅在请求时需要先建立连接，而且需要客户端向服务器发出请求后，服务器端才能回复数据。

    很多情况下，需要服务器端主动向客户端推送数据，保持客户端与服务器数据的实时与同步。此时若双方建立的是Socket连接，服务器就可以直接将数据传送给客户端；若双方建立的是HTTP连接，则服务器需要等到客户端发送一次请求后才能将数据传回给客户端，因此，客户端定时向服务器端发送连接请求，不仅可以保持在线，同时也是在“询问”服务器是否有新的数据，如果有就将数据传给客户端。

    * 数据库提高查询效率

      * 合理使用索引

      索引是数据库中重要的数据结构，它的根本目的就是为了提高查询效率。现在大多数的数据库产品都采用IBM最先提出的ISAM索引结构。索引的使用要恰到好处，其使用原则如下：

      * 在经常进行连接，但是没有指定为外键的列上建立索引，而不经常连接的字段则由优化器自动生成索引。

      * 在频繁进行排序或分组（即进行group by或order by操作）的列上建立索引。

      * 在条件表达式中经常用到的不同值较多的列上建立检索，在不同值少的列上不要建立索引。比如在雇员表的“性别”列上只有“男”与“女”两个不同值，因此就无必要建立索引。如果建立索引不但不会提高查询效率，反而会严重降低更新速度。

      * 如果待排序的列有多个，可以在这些列上建立复合索引（compound index）。

      * 使用系统工具。如Informix数据库有一个tbcheck工具，可以在可疑的索引上进行检查。在一些数据库服务器上，索引可能失效或者因为频繁操作而使得读取效率降低，如果一个使用索引的查询不明不白地慢下来，可以试着用tbcheck工具检查索引的完整性，必要时进行修复。另外，当数据库表更新大量数据后，删除并重建索引可以提高查询速度。

    *  避免或简化排序

      应当简化或避免对大型表进行重复的排序。当能够利用索引自动以适当的次序产生输出时，优化器就避免了排序的步骤。以下是一些影响因素：

    *  索引中不包括一个或几个待排序的列；

    *  group by或order by子句中列的次序与索引的次序不一样；

    *  排序的列来自不同的表。


    为了避免不必要的排序，就要正确地增建索引，合理地合并数据库表（尽管有时可能影响表的规范化，但相对于效率的提高是值得的）。如果排序不可避免，那么应当试图简化它，如缩小排序的列的范围等

* 消除对大型表行数据的顺序存取

    在嵌套查询中，对表的顺序存取对查询效率可能产生致命的影响。比如采用顺序存取策略，一个嵌套3层的查询，如果每层都查询1000行，那么这个查询就要查询10亿行数据。避免这种情况的主要方法就是对连接的列进行索引。例如，两个表：学生表（学号、姓名、年龄……）和选课表（学号、课程号、成绩）。如果两个表要做连接，就要在“学号”这个连接字段上建立索引。

    ​
    还可以使用并集来避免顺序存取。尽管在所有的检查列上都有索引，但某些形式的where子句强迫优化器使用顺序存取。

* 避免相关子查询
    一个列的标签同时在主查询和where子句中的查询中出现，那么很可能当主查询中的列值改变之后，子查询必须重新查询一次。查询嵌套层次越多，效率越低，因此应当尽量避免子查询。如果子查询不可避免，那么要在子查询中过滤掉尽可能多的行。


* 避免困难的正规表达式
    MATCHES和LIKE关键字支持通配符匹配，技术上叫正规表达式。但这种匹配特别耗费时间。
    另外，还要避免非开始的子串。例如语句：SELECT ＊ FROM customer WHERE zipcode[2，3] >“80”，在where子句中采用了非开始子串，因而这个语句也不会使用索引。

* 使用临时表加速查询
    把表的一个子集进行排序并创建临时表，有时能加速查询。它有助于避免多重排序操作，而且在其他方面还能简化优化器的工作。
    临时表中的行要比主表中的行少，而且物理顺序就是所要求的顺序，减少了磁盘I/O，所以查询工作量可以得到大幅减少。
    注意：临时表创建后不会反映主表的修改。在主表中数据频繁修改的情况下，注意不要丢失数据。
* 用排序来取代非顺序存取
    ​

    ​

    ​

    ​

    ​

    ​

  ​

  ​