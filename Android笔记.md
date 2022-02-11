<h1 align="center">Android笔记</h1>

##                                                        MVVM篇

### 1. LiveData的目的

* 在lifecycle的帮助下，规避订阅者回调的生命周期安全问题。

* 遵循“唯一可信源”理念的约束，也即通过“决策逻辑内聚”和“读写分离”来规避“跨域消息同步”等常见下高频存在的可靠一直问题，是团队新手也能不暇思索写出低风险代码。

* 就算不用dataBinding,也能使“单向依赖”成为可能，规避潜在的内存泄漏等问题。

#### 问题一：liveData与ObservableFields

* 两者相同点都属于可观察类，当数据发生改变的时候观察者能收到数据变化通知。

* Livedata属于jetpack包下组件可以与多个jetpack组件相互配合使用如room,workmager等方便简洁。

* liveData可以通过postValue和setvalue无论数据有没有不同，观察者都可以收到通知，ObservableFields只有数据发生改变才会去通知刷新界面。

#### 问题二：LiveData的数据倒灌

* 发生场景在页面二次返回的时候，还是收到了旧数据情景。LiveData中是通过SafeIterAbleMap存储每个观察者对象，实现观察者的缓存提高性能，setValue会判断版本号，数据发生一次改变将版本号+1，在生命周期活跃状态下，将发生改变的数据传递给每个观察者对象。每次observe的时候，mLastVersion的值小于 mVersion的值 是问题产生的根源.

* public enum State {

​       //Activity's {@link android.app.Activity#onDestroy() onDestroy} call

​     ***\*DESTROYED\****,

​     //the state when it is constructed but has not received ----onCreate yet

​     ***\*INITIALIZED\****,

​     //after {@link android.app.Activity#onCreate(android.os.Bundle) onCreate} call;

​     //right before</b> {@link android.app.Activity#onStop() onStop} call.

​    ***\*CREATED\****,

​     //after {@link android.app.Activity#onStart() onStart} call;

​     //right before</b> {@link android.app.Activity#onPause() onPause} call.

​    ***\*STARTED\****,

​    // this state is reached after onResume is called.

​    ***\*RESUMED\****;

 }

### 2. ViewModel目的：

* 让状态管理独立于视图控制器，从而实现“状态管理的分治”、实现“状态管理的一致性”和“状态共享”(跨页面通信)。

* 为状态设置作用域，是状态的共享做到作用域可控。（getActivity()，隶属于该Activity的其他fragment共享ViewModel实例，Fragment.this仅限本Fragment，Fragment离场时ViewModel连同Fragment一起销毁。

* 实现单向依赖，避免内存泄漏。(view单向引用ViewModel,ViewModel不可持有View引用)。

### 3. DataBinding目的：

* 通过基于适配器模式的“数据驱动思想”，规避视图调用的一致性问题。(双向绑定的存在 主要是为了 避免直接调用控件实例来获取数据，来避免视图调用的一致性问题。比如登录的场景，用户名、密码输入完毕点击 “提交”，那么在双向绑定的帮助下，我们只需通过 State-ViewModel 拿到与 用户名、密码 输入框发生双向绑定的 ObservableField 即可。)

* 并且数据驱动顺带免去了因调用视图对象而存在的大量冗余的判空处理与大量样板代码。



##                                                      Kotlin篇

### 简介：

​           与java一样，属于静态类型的编程语言，所有表达式在编译器已经确定好，可以在编译阶段检测出来一些错误。Kotlin的变量声明不要判断变量类型，var接收，类型推导。支持函数式编程(函数可以作为值使用，函数可以作为参数传递，也可以将函数作为返回值，使用lambda表达式代码优雅，简洁)。

### 1. 协程：

​         对线程和Handler的APi的一种封装，是一种优雅处理异步任务的解决方案，协程可以在不同的线程来回切换，这样就可以让代码通过编写的顺序来执行，并且不会阻塞当前线程，省去了再各种耗时操作写回调的情况，有点类似Rxjava的线程调度。https://juejin.cn/post/7052908509117022239   Suspend:当前线程挂起，稍后自动切回原来的线程上。

### 常见操作符号：

* **RunBlocking:** 运行在主线程，会阻塞当前线程执行。

* **Launch: **不会阻塞当前线程，异步执行代码。

* **Async:  **跟launch相似，唯一不同可以有返回值。

### 协程的线程调度器：

* **Dispatchers.Main:**  主线程调度器，会在主线程中执行。

* **Dispatchers.IO:**  工作线程调度器，会在子线程中执行。

* **Dispatchers.Default:**  默认调度器，没有设置调度器时就是这个，等同于Dispatchers.IO。

* **Dispachers.Unconfiend:**  无指定调度器，根据当前执行的环境而定，会在当前的线程上执行。

### 协程的启动模式：

* **CoroutineStart.DEFAULT:**  默认模式，会立即执行。

* **CoroutineStart.LAZY:**  懒加载模式，不会执行，只有手动调用协程的start方法才会执行。

* **CoroutineStart.ATOMIC:**  原子模式，跟CoroutineStart.DEFAULT类似，但是协程开始执行之前 不能被取消。

* **CoroutineStart.UNDISPATCHED: ** 未指定模式，会立即执行协程，会导致原先设置的线程调度器失效，开始执行在原来的线程上，类似于DIspatchers.Unconfined，但是一旦协程被挂起，在恢复执行，会变成线程调度器的设置的线程上面去执行。

* Val job = GlobalScope.launch(Dispatcher.Default,CoroutineStart.LAZY){}

​       Job.start();

​       Job.start:启动协程,除了lazy模式，协程都不需要手动启动。

​       Job.cancel取消一个协程，可以取消，但是不会立马生效，存在一定延迟。

​       Job.join等待协程执行完毕，这是一个耗时操作，需要在协程中使用。

​       Job.cancelAndJoin:等待协程执行完毕然后再取消。

### 2. 委托机制:

* **类委托：**简单来讲就是不需要自己去写具体的实现类，委托给相应的类。

* **属性委托：**帮助我们控制变量的Get，Set的操作。

* **懒委托：**类似单例模式中的懒汉式，现在不需要写静态方法和锁

  val temp:String by lazy{

   Println(“测试变量初始化了”)

   return@lazy “66666”

  }

  调用代码

  Println(“测试开始”)

  Println(“测试第一次输出 ”+temp)

  Println(“测试第二次输出 ”+temp)

  Println(“测试结束)

  日志输出

  System.out: 测试开始

  System.out: 测试变量初始化了

  System.out:第一次输出66666

  System.out:第二次输出 66666

  System.out: 测试结束

  日常开发中可以用来做findViewById private val  viewPager: ViewPager?bylazy { findViewById(R.id.vp_home_pager) }

####  懒委托还提供了几种懒加载模式：

* **LazyThreadSafetyMode.SYNCHRONIZED:**  同步模式，确保只有单个线程可以初始化实例。这种模式下初始化时线程安全的，当by lazy没有指定模式的时候，就是默认用这种模式。

* **LazyThreadSafetyMode.PUBLICATION:**  并发模式，在多线程下允许并发初始化，但是只有第一个返回值作为实例，这种模式下线程安全，和SYNCHRONIZED最大的区别是，这种模式在多线程并发访问下初始化效率是最高的，本质上面试用空间换时间，那个线程执行的快就让那个先返回结果，其他线程执行的结果抛弃。

* **LazyThreadSafetyMode.NONE:** 普通模式，这种模式不会使用锁来现在多线程访问，所以是线程不安全的。

* 懒委托的变量必须声明为val,因为他只能被赋值一次。

### 3. 扩展函数：let, with, run, apply, also

* **let函数：**在数块内可以通过it指定该对象，返回值为函数块的最后一行或者指定renturn表达式。(场景let函数处理需要针对一个可null的对象同意做判空处理) videoPlayer?.let{it.setVideoView(activity.course_video_view) it.setControllerView(activity.course_video_controller_view) it.setCurtainView(activity.course_video_curtain_view) }

* **With函数：**与其他函数不同，不是以扩展的形式存在，它是将某对象作为函数的参数，在函数块内可以通过this指代该对象，返回值为函数块的最后一行或指定return表达式适用于调用同一个类的多个方法时，可以省去类名重复，直接调用类的方法即可，经常用于 Android 中 RecyclerView.onBinderViewHolder 中，数据 model 的属性映射到 UI 上。

  override fun onBindViewHolder(holder: ViewHolder, position: Int){

    val item = getItem(position)?: return

    with(item){

  holder.nameView.text = "姓名：$name"

  ​    holder.ageView.text = "年龄：$age"

    }

  }

* **run函数：**let和with两个函数的结合体，run函数只接受一个lambda函数为参数，以闭包形式返回。如下：没有let的it,也没有with的with直接访问实例。

  override fun onBindViewHolder(holder: ViewHolder, position: Int){

    val item = getItem(position)?: return

    item?.run {

  ​    holder.nameView.text = "姓名：$name"

  ​    holder.ageView.text = "年龄：$age"

    }

  }

* **apply函数: ** 与run函数很像，run函数是以闭包形式返回最后一行代码的值，而apply函数的返回的是传入对象的本身。Apply一般用于对象实例初始化的时候，需要对对象中的属性进行赋值。

  tv_confimRootView = View.inflate(activity, R.layout.example_view, null).apply {

    tv_cancel.paint.isFakeBoldText = true

    tv_confirm.paint.isFakeBoldText = true

    seek_bar.max = 10

    seek_bar.progress = 0

  }

* **also函数:**   与let函数很像，let的返回值是最后一行的返回值而also函数的返回值是返回当前的这个对象。一般可用于多个扩展函数链式调用。

  

##                                                      设计模式篇

创建型模式：工厂模式，单例模式，建造者模式。(用于创建对象，实例化对象)

结构型模式：适配器模式。

行为型模式：策略模式，观察者模式，责任链模式。

### 1.  单例模式：

​	     饿汉式：在使用前创建好实例。(类加载时就初始化，浪费内存，线程安全)

​		 懒汉式:在使用才创建实例，使用volatile 和双重锁检查机制更加安全。

​		*public class Singleton {*  

​		*private volatile static Singleton singleton;*  

  	  *private Singleton (){}*  

  	 *public static Singleton getSingleton() {*  

​      *if (singleton == null) {*  

​      *synchronized (Singleton.class) {*  

​      *if (singleton == null) {*  

​       *singleton = new Singleton(); }  }  }*  

​	  *return singleton;*  

​	  *}*  

​	*}*

​	 静态内部类：这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。这种方式只适用于	 静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。

​	*public class Singleton {*  

​	  *private static class SingletonHolder {*  

​	  *private static final Singleton INSTANCE = new Singleton();*  

​	  *}*  

 	 *private Singleton (){}*  

 	 *public static final Singleton getInstance() {*  

 	 *return SingletonHolder.INSTANCE;*  

 	 *}*  

​	*}*

 	枚举：这种实现方式还没有被广泛采用，但这是实现单例模式的最佳方法。它更简洁，自动支持序列化机制，绝对防止多次实例化。

​     *public enum Singleton { INSTANCE; public void whateverMethod() { } }*

### 2.  工厂模式：

​	主要用于对象的统一创建，统一接口类型的子类可以通过工厂统一创建管理。

public class ShapeFactory {

  //使用 getShape 方法获取形状类型的对象

  public Shape getShape(String shapeType){

   if(shapeType == null){

​     return null;

   }     

   if(shapeType.equalsIgnoreCase("CIRCLE")){

​     return new Circle();

   } else if(shapeType.equalsIgnoreCase("RECTANGLE")){

​     return new Rectangle();

   } else if(shapeType.equalsIgnoreCase("SQUARE")){

​     return new Square();

   }

   return null;

  }

}

### 3.  建造者模式：

  类似函数式编程，通过简洁的代码调用创建复杂的对象。

  例如：new Person.Builder().setName(“张三”).setAge(30).setHeight(188).build();

### 4.  适配器模式：

​	适配器又分为类的适配器，对象的适配器。一个接口中有多个方法需要实现，但是并不是所有的都是需要的，这个时候可以借助一个抽象类，该抽象类实	现了接口，实现了所有的方法，不和原始接口打交道，值和改抽象类取得联系，所以写一个类，继承该抽象类，重写自己需要的方法。

### 5. 策略模式：

​	一个接口有一个方法，多个子类实现该方法，不同的子类实现的方式不一样。外部用户决定使用哪种方式去完成。实际是使用了多态，父类引用指向子类	对象，调用此方法，得到不一样的结果。

### 6. 观察者模式:

   观察者向主题发起订阅，当主题发生改变时，会通知观察者。

   一个观察者通知接口 所有的观察者实现通知接口，订阅类中主要做观察者的添加注册管理(一般使用map缓存订阅者，简单的也可以使用数组)

观察接口：

Public interface Observer{ void notifyNotice();}

观察者：

Pubulic class Observer1 implements Observer{

   notifyNotice(){System.out.println(“1 had receive notices ”);}

}

Public class Observer2 implements Observer{

   notifyNotice(){System.out.println(“2 had receive notices”);}

}

主题：

Public class Subject{

  Private List<Observer> observerList  =  new ArrayList();

  Public void add(Observer observer){ observer.add(observer);}

  Pubiic void notifyAllObserver(){

   For(Observer observer:observerList){

Observer.notifyNotice();

}

}

}

测试类

Public class ObserverTest {

  Public static void main(String[] args){

 Subject subject = new Subject();

 subject .add(new Observer1());

subject .add(new Observer2());

 subject .notifyAllObserver();

}

}

### 7. 责任链模式：

多个对象中，每个对象都持有下一个对象的引用，这就构成了链这种结构，

  一个请求通过链的头部，一直往下传递到链上的每一个结点，知道有某个结点对这个请求做了处理为止。(okhttp中的烂机器就是用的这种责任链)。

 Public abstract class Postma{(快递员抽象类)

​     protected postman nextPostman; //下一个快递

​     Public abstract void handleCourier(String address);//派送快递

}

Public class BeijingPostman ext Postman{

  handleCourier(String address){

 If(address.equals(“Beijing”)){

  System.out.println(“派送到北京”);

   return;

}else{

 nextPostman.handleCourier(address);

}

}

}

 

Public class ShanghaiPostman extends Postman{

handleCourier(String address){

 If(address.equals(“Shanghai”)){

System.out.println(“派送到上海”);

 return;

} else{

 nextPostman.handleCourier(address); 

}

}

}

 

Public class GuangzhouPostman extends Postman{

handleCourier(String address){

 If(address.equals(“Guangzhou”)){

System.out.println(“派送到广州”);

 return;

} else{

If(nextpostman !=null){

nextPostman.handleCourier(address); 

}

 }

}

}

 

测试类

Public void Test(){

Postman beijingPostman = new BeijingPostman();

Postman shanghaiPostman = new ShanghaiPostman();

//创建下一个结点

beijingPostman.nextPostman = shanghaiPostman;

//处理不同区域的快递，都是收结点北京快递员开始

System.out.println(“有一个上海快递需要派送”);

beijingPostman.handleCourier(“Shanghai”);

System.out.println(“有一个广州快递需要派送”);

beijingPostman.handleCourier(“Guangzhou”);

System.out.println(“有一个美国快递需要派送”);

beijingPostman.handleCorier(“Americe”);

}

输出结果：

有一个上海快递需要派送

派送到上海

有一个广州快递需要派送

派送到广州

有一个美国快递需要派送



##                                                       多线程与并发

### 1.  Android中的线程池

ThreadPoolExecutor是线程池的真正实现，android中有4类线程池:FiexdThreadPool,CachedThreadPool,ScheduledThreadPool,SingleThreadExecutor。

1、ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory){

 This(corePoolSize,maxmumPoolSize,KeepAliveTime,unit,workQueue,threadFactory,defaultHandler);

}

**corePoolSize:**  核心线程数，默认一般核心线程会在线程池中一直存活，计时处理空闲状态。但当allowCoreThreadTimeOut设置为true，那么核心线程就会闲置超时时间，闲置超过超时时间就会终止。超时由keppAlive和unit指定。

**MaximumPoolSize:**  最大线程数，当活动线程达到这个数，后续的新任务会被阻塞。

**keepAliveTime:**  非核心线程闲置时的超时时长。非核心线程闲置时间超过此时间会被回收。当allowCoreThreadTimeOut设置为true,keepAliveTime也会作用域核心线程。

**Unit:**   是KeepAliveTime的时间单位，取值是枚举，有TimeUnit.Minutes,TimeUnit.SECONDS..

**workQueue: ** 线程池中的任务队列，通过线程池的execute方法提交的runnable会存在这个队列中。

**ThreadFactory:**   线程工厂，为线程池提供创建新线程的能力。

 

#### ThreadPoolExecutor执行任务的执行规则：

 先从核心线程开始执行，未达到核心线程数，就会启动一个核心线程来执行这个任务。

超过了核心线程数就将任务加入到任务队列中，如果任务无法插入到队列中，未超过最大线程数，就会启动非核心线程执行这个任务，线程数达到最大线程数，那么会拒绝执行这个任务。

* **FiexdThreadPool:**固定的核心线程数量，没有非核心线程，空闲时不会被回收，队列长度无限制。因为不会被回收，所以能快速执行外界请求。

* **CachedThreadPool:**核心线程数0，非核心线程数无限，空闲回收超时时间60s,队列不能插入任务.当所有线程都处于活动状态，就会创建新线程处理任务，否则利用空闲线程处理任务，当整个线程池空闲时所有线程都会被回收，不占用系统资源。因此，适合执行大量耗时较少的任务。

* **ScheduledThreadPool :** 固定核心线程数，不限制非核心线程数，非核心线程闲置回收超时时间是10ms。一般用于执行定时任务，固定周期的重复任务。

* **SingleThreadExecutor：** 仅有一个核心线程，不会回收，可确保所有任务按顺序执行，不用处理线程同步的问题。

  

### 2.  同步锁相关

* **Synchronized：**同步锁主要是保证当前线程执行访问时，防止其他线程访问数据，保证线程执行的顺序,不能保证指令重排序。

* **Volatile:**非原子性，禁止指令重排序，具有可见性。能保证当数据发生变化立马刷新主存数据，达到数据共享的目的，其他线程能同一时间获取到最新数据。

* **指令重排序：**多线程环境下在内存中一段代码块的执行，编译器对代码本身的优化并不一定按照实际的代码一步步执行，能提高性能，此时会出现一种情况后面的代码比前面的代码先执行，到时结果的不同。

* **ConcurrentHashMap：**是一个并发散列映射表的实现，允许完全的并发读取，并且支持给定数量的并发更。相比与线程安全的Hashtable与线程不安全的HashMap使用的是一个全局锁，保证统一时间点只能有一个线程访问，保证多线程间的并发访问，即串行化访问。

* **内存屏障：**是一个cpu指令,保证某些特定操作的执行顺序，保证某些变量的内存可见性。

* **AtomicBoolean AtomicInteger：**等原子变量具有原子特性，能保证只有一个线程修改成功。多线程环境下，无锁的进行原子操作，原子变量的底层使用处理器提供的原子指令。

* **CopyOnWriteArrayList：**如果有多个调用者同时请求相同资源(如内存或者磁盘上的数据存储),他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本给调用者，而其他调用者所见到的最初的资源任然保持不变，**优点:**  是如果调用者没有修改改资源，就不会有副本被建立，因此多个调用者只是读取操作时可以共享同一份资源。底层通过复制数组的方式来实现。**缺点:  **如果经常增删里面的数据，经常要执行add,set，remove的话，比较耗费内存。每次增删都要复制一个数据出来。只能保证数据的最终一致性，不能保证数据的实时一致性。

### 3.  阻塞队列

* **ArrayBlockingQueue：**是一个用数组实现的有界阻塞队列,可配置数组大小。

* **LinkedBlockingQueue：**是一个用链表实现的有界阻塞队列，默认和最大长度位Interger.Max_VALUE。入队列和出队列时使用的不是同一个Lock,这也说明他们之间的操作不存在互斥作用，在多个cpu的情况下，可以做到同一时刻即消费，又生产，能做到并行处理

* **PriorityBlockingQueue：**是一个支持优先级的无界阻塞队列。默认情况下元素采取自然顺序升序排列。

* **DelayQueue：**是一个支持延时获取元素的无界阻塞队列。队列使用priorityBlockingQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。

* **ArrayBlockingQueue与LinkedBlockingQueue区别**

  ArrayBlockingQueue底层是数据结构，创建时指定大小，在完成后在内存中分配指定大小的数组元素，存储通常有限，估其是一个有界的阻塞队列。

  LinkedBlockingQueue底层是链表，可以用用户指定最大存储容量，也可以不用指定。可以看做是一个无界阻塞队列，其结点都是动态创建，并且其结点出队列后便会被GC回收，具有灵活的伸缩性。LinkedBlockingQueue读写使用的是两个lock，因此在多cpu的情况下能保证并发读写操作，吞吐量大于ArrayBlockingQueue。

  

##                                                     数据结构与算法

### 1. 常见数据结构:

* #### 数组(Array)

    优点：按照索引查询元素的速度很快，按照索引遍历数组也很方便。

    缺点：数组的大小在创建后就确定，无法扩容。数组只能存储一种类型的数据。添加，删除元素的操作很耗时间，因为要移动其他元素。

* #### 链表(Linked List)

    链表是一种递归的数据结构，它或者为空，或者是指向一个节点的引用，该节点还有一个元素和一个指向另一个链表的引用。

    链表中的数据按照“链式”的结构存储，因此可以达到内存上非连续的效果，数组必须是连续的一块内存。

    优点：不需要初始化容量，可以添加任意元素，插入和删除的时候只需要更新引用。

    缺点：含有大量的引用，占用的内存空间大，查找元素需要遍历整个链表，耗时。

* #### 栈(Stack)

   按照“后进先出”，“先进后出”的原则来存储数据，先插入的数据被压入栈底，后插入的数据在栈顶，读取数据的时候，从栈顶 开始依次读出。

* #### 队列(Queue)

  队列会对两端进行定义，一端叫队头，一端叫队尾，队头只允许删除操作(出队),队尾只允许插入操作(入队)。栈是先进后出， 队列时先进先出，两者都    是线性表。

* #### 树(Tree)

    树是一种典型的非线性结构，它是由n(n>0)个有限结点组成的一个具有层次关系的集合。

    每个节点都只有有限个子节点或者无子节点。

    没有父节点的节点称为根节点。

    每一个非根节点有且只有一个父节点。

    除了根节点外，每个子节点可以分为多个不相交的子树。

* #### 堆(Heap)

    堆可以被看做是一棵树的数组对象，具有以下特点:

    堆中某个节点的值总是不大于或者小于其父节点的值。

    堆总是一颗完全二叉树,将根节点最大的堆叫做最大堆或者大根堆，根节点最小的堆叫做最小堆或者小根堆。

* #### 哈希表(Hash)

    哈希表，也叫散列表，是一种可以通过关键码值(key-value)直接访问的数据结构，他最大的特点是可以快速实现查找，插入和删除。

    数组的最大特点就是查找容易，插入和删除困难，而链表正好相反，查找困难，而插入和删除容易。哈希表很完美的结合了两者的优点。

    哈希函数在哈希表中起着非常关键的作用，它可以把任意长度的输入变化成固定长度的输出，该输出就是哈希值。哈希函数使得一个数据序列的访问过程变得更加迅速有效，通过哈希函数，数据元素能够很快的进行定位。

  若关键字为K，则其值存放在hash(k)的存储位置上。由此，不需要遍历就可以直接取的K对应的值。

### 2.  排序算法：

* #### 冒泡排序：

两两比较相邻记录的关键字，如果反序则交换，知道没有反序记录为止。

* #### 选择排序:

改进了冒泡排序，将交换次数从N2到N次，比较次数任然是N2。将所有元素扫描一遍，找到最小的，与第0位置交换，此时0位置是有序的，后面依次找到1位置元素。由于交换次数少，所以在交换时间级大于比较的时间级时，选择排序的优势就体现出来了。

* #### 快速排序：

1、把数组或者子数组分割成左边一组和右边一组，2、调用自身对左边的一组进行排序，3、调用自身对右边的一组排序。对子数组的排序采用递归的调用排序算法自身就可以。如果数组中只含有一个数据项，则直接返回，这是这个递归的终止条件。

![img](https://raw.githubusercontent.com/yanbin321/Images/main/img202202111803403.jpg) 

### 3.  查找算法:

* #### 顺序查找:

 从表中的最后一个记录开始，逐个进行记录的关键字与给定的值比较，若某个记录的值与给定的值一致，则查找成功。

* #### 二分查找:

 将中间位置记录的关键字与查找关键字比较，如果两者相同，则查找成功，否则将查找列表分为前，后两片，若中间关键字大于查找的关键字，则从前表开始查找，否则从后表开始查找，直到找到目标关键字。

要求1、必须采用顺序存储结构，2、必须按关键字大小有序排列。

优缺点:折半查找的优点是查找次数少，查找效率高。缺点是：要求待查表是有序表，且插入删除困难，折半查找适用于不经常变动查找频繁且有序列表。

分块查找:

 将n个数据元素分成“m个块”，每个块中的元素不用有序，但是块与块之间需要有序，第一个块中的任何数都要小于第二个块中的数，依次类推。

1、先给每个块添加索引，组成一个块列表通过二分查找，确认元素在哪个块内，

2、再从块内按顺序查找到目标元素。



##                                                       Java基础

Java的集合类主要由两个接口派生而来:Collection 和Map ,Collection 和Map是Java的集合框架根类。

![img](https://raw.githubusercontent.com/yanbin321/Images/main/img202202111803510.jpg)

常用集合类:ArrayList,LinkedList,HashSet,TreeSet,Stack

![img](https://raw.githubusercontent.com/yanbin321/Images/main/img202202111803408.jpg) 

常用集合类：HashMap ,LinkedHashMap,TreeMap。

* **ArrayList:** 以数组实现，节约空间，但是数组有容量限制，超出限制时会增加50%的容量，用System.arrayCopy()复制到新的数组，增删慢，查询快。

* **LinkedList:** 以双向链表实现，链表无容量限制。按下标访问元素需要悲剧的遍历链表将指针移动到位(i>数组大小的一半，会从某位开始移起)。插入和删除只需要修改前后节点的指针即可，但是还是要遍历部分链表的指针才能移动到小角标所指的位置，只有在链表的两头操作add(),addfirst(),removeLast()等能省去指针移动。总的来说增删快，查询慢。

* **HashMap:** 基于数组+链表组成，允许空键/值，非同步，不保证有序，也不保证序随时间变化。

* **LinkedHashMap:** 是Hash表和链表的实现，依靠着双向链表保证数据的插入顺序。

* **TreeMap:**  使用红黑树实现，可以保持key的大小顺序。

* **Java的反射：**运行状态中，对于任何一个类，都能知道这个类的所有属性和方法。对于任意一个对象，都能调用它的方法和属性。

  class1 = Class.forName("com.lvr.reflection.Person");

  Field[] allFields = class1.getDeclaredFields();//获取class对象的所

  // 生成新的对象：用newInstance()方法 

  Object obj = class1.newInstance(); 

  //首先需要获得与该方法对应的Method对象 

  Method method = class1.getDeclaredMethod("setAge", int.class); 

  //调用指定的函数并传递参数 

  method.invoke(obj, 28); 



##                                                       性能优化

### 1. 启动优化:

* 应用的启动分为冷启动和热启动，冷启动需要创建应用进程和Application等一系列流程，热启动因为已经创建了进程，不需要走创建进程和初始话Application的过程，直接启动launcher页面,从速度上讲热启动比冷启动要快。（须知道Activity的启动流程）

#### 启动速度提升：

* 在Application不要做耗时操作，将一些不重要的数据业务放在异步线程执行。

* 应用启动的时候Sp数据读取存放在内存中，读取操作不应该放在主线程，在异步线程中执行。

* 对于MainActivity的第一帧显示，需要对界面进行测量，绘制，等操作，减少界面层级。采用ViewStub做一些延迟加载策略。ViewStub 是一个轻量级的View，它一个看不见的，不占布局位置，占用资源非常小的控件，ViewStub.inflate()的时候，ViewStub所向 的布局就会被Inflate和实例化。

* 优化启动体验:尽量在启动的时候不要做耗时操作，在界面第一帧出来前可以设置theme，在第一帧界面显示之前super.onCreate之前setTheme()。

### 2. 布局优化:

####  setContentView()分析:

 AppCompatDelegateImplV9实现类setContentView() 最终调用LayoutInflater对象的inflate去加载对应的布局---->res.getLayout() ----->loadXmlResourceParse()--->impl.loadXmlResourceParser()加载指定文件的Xml解析器。--->

mAssets.openXmlBlockAsset()  openXmlBlock是个native方法，读取xml文件是通过io流的方式进行的，加载布局流程是一个耗时操作。

分析完getlayout后看inflate(); 首先判断节点是否是merge标签，如果是，则对merge标签进行效验，如果merge节点不是当前布局的父节点，则抛出异常，然通过createViewFromTag方法去根据每一个标签创建对应的View视图。创建view的内部按优先顺序为Factory2和Factory的onCreateView, createView方法进行View的创建，而createView方法内部采用了构造器反射的方式实现。

LayoutInflater.Factory是LayoutInflate中创建View的一个hook，我们利用他在创建View的过程中加入一些日志或者进行其他更高级的定制化处理，如换肤。

我们在基类Activity的onCreate方法中直接使用LayoutInflaterCompat.setFactory2方法的onCreateVew方法进行重写。

LayoutInflateCompat.setFactory2这样我们就实现了利用LayoutinflaterCompat.Factory2全局监控Activity。

![img](https://raw.githubusercontent.com/yanbin321/Images/main/img202202111803148.jpg) 

此hook操作是换肤的基础，能在加载显示view将布局的资源替换。

#### 布局优化常规方案

* 代码动态创建View

* 替换MessageQueue来实现异步创建View.在使用子线程创建视图控件的时候，我们可以吧子线程Looper的MessageQueue替换成主线程的MessageQueue，在创建完需要的视图空间后记得将子线程Looper的MessageQueue恢复为原来的。Awesome-WanAndroid项目中UiUtils的Ui优化工具类中，提供了相应的实现。

![img](https://raw.githubusercontent.com/yanbin321/Images/main/img202202111803558.jpg) 

* 使用AsynclayoutInfalter异步创建View

​       implementation 'com.android.support:asynclayoutinflater:28.0.0'

![img](https://raw.githubusercontent.com/yanbin321/Images/main/img202202111803435.jpeg) 

* 使用ConstraintLayout降低布局嵌套层级，尽可能少用wrap_content,wrap_content会增加布局measure时的计算成本，已知宽高为固定值时，不用wrap_content。

### 3. 内存优化:

  内存存在的问题主要分为几大类:内存抖动(内存波动图形呈锯齿张，GC导致卡顿)，内存泄漏(对象被持有导致无法释放或者不能按照对象正常的生命周期进  行释放)，内存溢出(OOM,程序异常)。

####   内存问题总结:

* 内类时有危险的编码方式

   Activity关闭时，触发onDestory时解除内类和外部的引用关系。

* 普通Handler内部类的问题

  把内类声明成static，来断绝this$0的引用。因为static描述的内类从java编译原理的角度看，“内类”和“外部”相互独立，相互都没有访问对方成员变量的能力。使用weakReference来引用外部类的实例。在外部类销毁的时候使用removeCallbackAndMessages来移除回调和消息。

* WebView的泄漏优化

  * WebView其网络延时，引擎Session管理，CooKies管理，引擎内核线程，HTML5调用系统声音，视频播放组件等产生的引用链条无法及时打断，造成内存问题基本上“无界”来形容。可以将WebView装入另一个进程，对当前Activity设置android:process属性，最后onDestory中退出进程，这样基本上报可以终结WebView造成的泄漏。

  * 不通过xml中创建WebView的实例，WebView内存泄漏的主要原因是引用了Activity/Fragment的Context,加之WebView本身的设计问题，导致Activity/Fragment无法及时释放。既然WebView无法及时释放Context,那就让他引用全局Context。
  * 为了保证界面销毁时，WebView不要在后台继续进行没有意义的加载，避免父控件对WebView的引用导致发生意外泄漏，将webView从父控件中移除，让WebView停止加载页面并释放。

* 在适当的时候对组件进行注销

  我们平常开发过程中经常需要在Activity创建的时候去注册一些组件，如广播，定时器，事件总线等等，这个时候我们应该在合适的时候对组件进行注销，如onPause或者onDestory方法中。

  

####    减少内存占用 

* 使用Android优化后的集合比如SparseArray、ArrayMap来替代HashMap在一些情况下能带来更好的性能提升。(因为它需要为每一个键值对都提供一个对象入口，而SparseArray就避免掉了基本数据类型转换成对象数据类型的时间。)

* bitmap是占用内存的比较多的，anroid提供了优秀的图片处理框架Glide处理图片。

  bitemap内存占用≈像素数据总大小 = 图片宽 **图片高*  * (设备分辨率/资源目录分辨率) ^2 * 每个像素的字节大小。

  从计算公式看可以通过以下方式减少内存占用:

  * 使用底色彩的解析模式，如RGB565，减少单个像素的字节大小。(大约能减少一半的内存开销，Android默认使用ARG888配置来处理色彩，占用4个字节，改用RGB565，将占用2个字节，代价是显示的色彩将相对减少，适用设置丰富度要求不高的场景。)
  * 资源文件合理放置，高分辨率图片可以放到高分辨率目录下。(理论上图片放置的资源目录分辨率越高，占用内存越小，低分辨图片因此会被拉伸，显示上出现失真。)
  * 图片缩小，减少尺寸。(理论上可以减少十几倍的内存适用，基于这个一个事实，原图片尺寸一般都大于目标需要显示的尺寸，因此可以通过缩放的方式，来减少显示时的图片宽高，从而大大减少占用的内存。)可以利用BitmapFactory.Options 设置采样率，inSamlseSize设置缩放比例可以真实的压缩Bitmap占用的内存 inSampleSize比1小的话会被当做1，其他的时候取接近2的幂值。inJustDecodeBounds=tue时BitmapFacory通过decodeResource或者decodeFile解码图片时，将会返回null的Bitmap对象，这样可以避免Bitmap的内存分配，但是它可以返回Bitmap的宽度，高度以及mimeType。
  * 采用内存缓存和磁盘缓存(Lrucache缓存)。

* 避免内存抖动避免在循环体或者频繁调用的函数中创建对象。

  

### 4. 耗电优化

####   JobScheduler

* 在稍后的某个时间点或者当满足某个特定的条件(链接电源，网络状态变化，手机是否空闲时)需要执行一个任务。

####  WorkManager

* 工作管理器提供的接口允许应用程序在一个容器中并发的执行多个工作项

* 应用在退出或者重启后仍然可以允许任务(任务存储在本地)

* 指定时间和任务执行条件完成相应任务。

### 5. APK大小优化

​    将apk文件解压后各个目录占用大小可以一目了然，针对各个目录进行优化。

* lib目录：so编译可以根据自身用户情况而定，一般情况armeabi-v7a使用android4.1以上的机器。*

* 手动lint检查，删除无用资源。
* 使用tinypng等图片压缩工具对图片进行压缩（智能有损压缩，有选择性的减少图像中的颜色数量）。

* 使用gradle 开启shrinkResources移除无用资源文件。
* 使用矢量图svg文件替代图片文件。
* 保留一套高精度图片文件，确保大部分机型都不存在模糊情况。
* 减少classes.dex的大小，默认方法数不超过64k，可以通过multidexing风多个文件，尽量减少第三方库的引用。

* 将大资源文件放到服务端，启动后自动下载。