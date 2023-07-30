# Java

## 集合类

### List

#### ArrayList

**概念**

1. 底层使用的**动态数组**实现，能够快速根据下标定位，复杂度为O(1)
2. 如果需要频繁删除数据或者在中间插入数据的话，需要移动数据，复杂度为O(N)，代价较高
3.  该类适合数据不频繁更新的应用场景。

**源码**

- 默认构造函数是不创建数组的，只有当第一次插入的时候才会创建数组，默认为10
- 每次插入前会先判断数组是否满了，如果满了就需要扩容，扩容为当前的1.5倍，然后将数据拷贝到新数组
- List和数组转化：
  `List list = Arrays.asList(arr);` 修改数组**会改变**list，直接引用`return new ArrayList<>(a);`
  `String[] strArr = list.toArrays();`  修改list**不会改变**数组，拷贝了一份新的对象`System.arraycopy(elementData, 0, a, 0, size);`

#### LinkedList

**概念**

1. 底层使用的是双向链表实现，插入删除效率高，复杂度为O(1)
2. 默认是头插法，如果需要指定下标插入、查找或者指定下标插入复杂度为O（n）
3. 该类型适合频繁插入删除头尾节点的情况。

**源码**

- 实现了List和Deque接口，除了具备List功能外，还支持双端队列功能。
- 默认构造器什么都不做，内部成员变量保存头尾指针
- remove()移除头结点，add()是头插，add(object)实现了队列接口方法为尾插

#### ArrayList和LinkedList的区别

1. 底层数据结构：ArryList使用动态数组，LinkedList使用的是双向链表
2. **效率**：
   根据下标查询：ArrayList按照下标查询复杂度为O(1)效率高，LinkedList根据下标查询是O(n)
   根据值查找：两者都需要遍历，时间复杂度为O(n)
   新增和删除：ArrarList尾部插入删除为O(1)，其他位置需要移动元素为O(n); LinkedList头尾插入删除为O(1)，其他位置需要遍历链表为O(n)
3. 内存空间占用：ArrayList底层是数组，内存连续，节省内存。LinkedList是双向链表，除了存储数据外还需要存储前后两个指针更占用空间。
4. 线程是否安全：都不是线程安全的。解决办法：在方法内使用，局部变量是线程安全的；加锁或者在外部包裹锁Collections.synchroinzedList();


### Set

### Map

#### HashMp

##### 实现原理

1. 底层实现是数组+链表+红黑树（JDK8）
2. 当添加数据时，首先会根据key的hashcode来定位数组下标。
   如果不冲突则直接插入，如果哈希冲突，则放入链表或者红黑树中
3. 链表转化为红黑树的要求是：**链表长度大于8，数组长度大于64** 。如果链表节点小于等于6个，会退化为链表

##### put()具体流程

1. 首先，计算key的hash值
2. 第一次插入，需要先初始化数组，默认长度为16
3. 第二次插入，如果插入相同的key，则会直接覆盖
4. 如果是不同的key，则会进行hash定位，如果没有哈希冲突，直接插入；
5. 如果有hash冲突，则判断是否为红黑树，如果是，则在红黑树中插入；
6. 如果是链表，则遍历链表判断是否存在key，如果存在则替换；如果不存在则插入，插入后检查是否需要转化为红黑树。
7. 插入成功后判断是否超过了阈值（数组长度*0.75*）如果超过需要扩容 ，扩大为当前容量的2倍

![image-20230605193134804](img\HashMap_put流程.png)

##### 扩容机制

首先，判断旧数组容量是否为0，如果是0，进行初始化，默认容量为16，扩容阈值为12。

如果旧数组长度不为0，则扩大容量为原来的2倍，然后将旧数组里的元素移动到新数组中，共分为三种情况：

- 如果当前hash值位置只有一个元素，则重新hash后直接添加到新数组中
- 如果有多个元素，先判断下是否为红黑树，如果是的话就添加到红黑树中；
- 如果是链表，遍历链表，将元素的hash值与oldCap做&操作来判断是放在原来的位置还是新扩展对应的位置newTab[j+oldCap]（ 我理解的这里是为了分散同一个hashcode值下挂载的元素数）

![image-20230605215553763](img\HashMap扩容机制.png)

##### 寻址算法

使用的算法是：二次哈希法

代码有两处优化：

1. 求hash值：hash值的高16位和低16位进行异或运算`(h = key.hashCode()) ^ (h >>> 16)`，使得hash过后同时具备高16位和低16位的特征
2. 定位数组索引：寻址没有用取模运算而是用按位与操作，是因为位运算效率更高。两者作用是等价的，但是位运算效率更改。但是只能用在被除数是2的倍数上。`tab[i = (n - 1) & hash]`

##### 为什么hashmap长度一定是2的次幂

1. 计算索引时：可以使用位运算效率更高
2. 扩容时：重新计算索引时效率更高。每一个旧元素都需要重新计算位置，位运算效率更高。

## 多线程

### 线程基础

#### 进程和线程

1. 关系：进程是正在运行的实例，一个进程包含很多线程，每一个线程执行不同的任务
2. 内存占用：不同线程使用不同的内存空间，在同一个进程中的线程共享内存空间，线程切换上下文成本低
3. 对于JAVA来说：JVM属于进程级别，在JVM内部包含许多线程

#### 并行和并发

1. 线程之间轮流执行称为并发，多个线程同时运行称为并行。
2. 单核CPU只能并发，多核CPU可以并发也可以并行，如果在某一时间点有多个CPU在执行线程称为并行。

#### 创建线程

1. 继承Thread类
2. 实现Runner接口
3. 实现Callable接口，可以获取到线程执行结果
4. 使用线程池创建

#### 线程状态

哪些状态：新建，就绪，阻塞，等待，超时等待，终止。

线程转化：

- 创建完线程后状态是新建(NEW)
- 调用start()方法后变为就绪状态(Runnable)
- 获取到CPU后变成执行状态和就绪态公用同一个状态（Runnable）
- 如果执行完时间片还没结束则返回就绪状态，如果需要拿锁则进入阻塞态（Blocked)，如果调object.wait(xxx)或者Thread.sleep(xxx)进入限期等待状态，如果调用object.wait()或者thread.join()则进入无限期等待状态。
- 运行结束将变为终止状态

![image-20230606224733363](img\线程状态.png)

#### 如何顺序执行三个线程

- 使用join()：join方法会让出CPU给join的线程，等新加入的线程执行完毕后再继续获取CPU执行下面的代码。实现：t1线程正常运行; t2在线程体先`t1.join()`; t3在线程体内先`t2.join`
- 使用condition：创建三个condition分别在三个线程内调用singal()和await()方法来控制执行顺序。

#### Wait和Sleep区别

共同点：都暂时放弃CPU使用权，进入阻塞状态

不同点：

- sleep是Thread的静态方法，wait方法是object的成员方法，每个对象都有
- sleep执行固定的时间后唤醒，wait方法可以被notify唤醒，也可以指定固定时间后唤醒
- wait调用前必须先获取到锁，否则会报错，调用后释放锁；sleep调用后仍然持有锁，其他线程拿不到锁继续等待

#### 如何停止线程

1. 在线程中使用循环访问一个标志位，在线程外部通过改变标志位来控制线程
2. 调用interrupt方法中断线程：打断阻塞的线程，会抛出InterruptedException异常`t1.interrupt()`；如果是非阻塞的状态，类似第一种加flag `if(current.isInterrupted())break;`
3. 使用stop()方法停止，但是有可能损失数据，不推荐使用

### 并发安全

#### synchronized关键字

1. 是一个互斥的对象锁，在同一时刻只有一个线程持有对象锁。
2. 作用在成员方法上，锁的是对象实例this；作用在静态方法上，锁的是类；作用在某个对象上，锁的是该对象。
3. 底层实现是由monitor实现，有三个属性：<font color="red">owner（只有一个），entryList（阻塞的线程），waitSet（等待的线程）</font>

#### 锁升级

synchronized有三种形式：

1. 偏向锁：一段很长的时间内都只有一个线程使用锁，可以使用偏向锁。对象头MarkWord存储的是偏向线程的ID
2. 轻量级锁：线程在加锁的时间是错开的，也就是没用竞争，不同线程交替持有锁，可以使用轻量级锁来优化。对象头存储线程栈中的LockRecord指针
3. 重量级锁：多线程竞争锁，底层使用Monitor是像，涉及到用户态和内核态的切换、成本较高，性能比较低。对象头存储指向堆中的monitor对象的指针。

一旦锁发生了竞争，都会升级为重量级锁。

#### JMM

1. JMM(JAVA Memory Model）JAVA内存模型主要是为了解决多线程数据交互的问题。
2. JMM内存分为两块，一块是线程私有的工作内存，一块是主内存。
3. 线程的是有工作内存是相互隔离的，但可以使用主内存进行交互。

#### CAS

1. 全称Compare And Swap(比较再交换)，属于乐观锁，在无锁的情况下保证线程安全。
2. 有三个参数：V（当前内存值），E（预期的值），N（新值）；当V=E时，更新为N；当不等于时，会更新当前的值
3. 用处比较多：AQS框架，juc.atomic.AtomicXXX类，synchronize
4. 在操作共享变量时使用自旋锁效率更高
5. 可能出现ABA问题，即一个线程将A改成B再改成A，但是在另外一个线程看来是没用变化。可以用版本号来解决这个问题

#### 乐观锁和悲观锁

乐观锁：最乐观的估计，认为没用别的线程来共享资源，即使使用了也可以使用其他方法来做线程安全，例如CAS。

悲观锁：最悲观的估计，防止其他线程共享变量，在任何时间都对资源上锁，例如synchronize重量级锁

#### Volatile

1. 可见性：volatile变量的值在各个线程中是一致的，当一个线程修改了变量，其他线程可以立刻得知。然而普通变量却不行，因为普通变量在线程之间传递需要通过主内存来完成。
2. 禁止指令重排：修饰的共享变量会在读写变量时加入屏障，阻止其他读写操作越过屏障，防止重排序。

#### AQS

- Abstract Queued Synchroniezer，抽象队列同步器，是一种锁机制，例如像ReentrantLock，Semaphore都是基于AQS实现的。
- AQS内部维护了一个FIFO的双端队列，队列汇总存储的是排队的线程
- AQS内部有一个state属性来判断当前是否有线程获取到资源，默认是0
- 如果多个线程来争抢state资源，使用CAS来保证原子性。
- 该实现类有公平锁和非公平锁两种，非公平锁指的是新加入的线程会和队列中的线程来争抢资源。公平锁则将新加入的线程放入队列尾部。

#### ReentrantLock

1. 是一种可重入锁
2. 底层使用AQS+CAS实现
3. 支持公平锁和非公平锁两种形式

#### ReentrantLock与synchronize的区别

1. 语法层面：synchronized属于java关键字，不能显示调用，自动加锁释放锁；Lock属于API，可以直接使用，手动加锁释放锁；
2. 功能方面：两者都属于悲观锁，都具备互斥、同步和重入功能；但Lock有更多其他功能，例如：可打断，可超时，公平锁，多条件变量等。
3. 性能方面：在没用竞争时，synchronize做了很多优化例如偏向锁，轻量级锁等，性能较好；但是竞争激烈时，Lock性能更好；参考JVM第13章线程安全的事项

#### 死锁

产生死锁的四个条件：资源有限，请求并保持，不可抢占，循环等待

如何诊断死锁：阔以使用jdk自带工具：jps和jstack；也可以使用可视化工具jconsole, visualVM查看。

#### ConcurrentHashMap

1. 底层结构：1.7使用的是分段的数组+链表；1.8使用的是数组+链表+红黑树，和HashMap结构相同；
2. 加锁方式：1.7采用分段加锁，底层使用的是Reentrantlock，锁住每一个分段；1.8使用的是CAS+synchronized， 添加新节点采用CAS，hash冲突后添加会用synchronized锁住头结点后添加到链表或红黑树。相对于segement力度更细，性能更好。

#### 导致并发出现问题原因

1. 原子性：synchronize，lock
2. 可见性：volatile，synchronize，lock
3. 有序性：volatile

### 线程池

#### 核心参数

线程池七大核心参数：核心线程、最大线程、存活时间、时间单位、阻塞队列、线程工厂（起名字）、四种拒绝策略

执行顺序

1. 提交任务后，参看核心线程数是否已满，如果没有慢，直接创建核心线程执行任务
2. 如果核心线程数满了，则放入阻塞队列中
3. 如果阻塞队列满了，则创建临时线程执行任务
4. 如果临时线程+核心线程数大于最大线程数，则执行拒绝策略

注意：

- 如果执行的核心线程和临时线程完成任务后会检查阻塞队列中是否有等待的任务；如果有，则弹出继续执行。
- 如果一个线程空闲时间超过指定的存活时间，则判断当前线程池中的线程数是否大于核心线程数，如果大于则会被停掉。

#### 线程池中的阻塞队列

1. ArrayBlockingQueue：数组阻塞队列，底层是用数组实现的，所以在创建实例时必须指定队列容量；只有一把锁控制插入和弹出操作
2. LinkedBlockingQueue：底层使用的是链表，可以不指定容量，默认为Interger最大值，也可以指定容量。插入弹出操作由两把锁控制，一把控制插入，另一把控制弹出，效率较高，推荐使用。

#### 如何确定核心线程数

- 如果计算量大，线程切换少的任务，线程可以少一点，避免线程切换，核心线程数可以设置为N(N是CPU核心数)
- 如果计算量小，大部分都是IO操作，线程可以多一些，因为大部分时间CPU都不起作用，例如远程调用，IO等耗时操作，可以设置为2N(N是线程数)

#### 线程池的种类

在Executors中提供了四种快捷创建线程池的静态方法

1. FixThreadPool：核心线程数和最大线程数相同的线程池，没有临时线程，阻塞队列是Interger的Max_value
2. SingleThreadPool：只有固定的一个线程的线程池，没有临时线程，阻塞队列仍然是Interger的Max_Value.
3. CachededThreadPool：没有使用核心线程，都是临时线程，可以无限创建线程，阻塞队列SynchronousQueue不存储任何元素。
4. ScheduledThreadPool：可以设置核心线程数，最大线程数为Max_Value，可以延迟执行任务。

#### 为什么不建议使用Executors创建线程池

根据阿里巴巴代码手册，不能使用Executors创建线程池只要有以下两个问题

![image-20230610002512338](D:\LearningNote\笔记\img\线程池.png)

### 使用场景

#### CountDownLatch

CountDownLatch主要有两个方法：

- cd.countDown()，cd数量减1
- cd.await()，调用的线程被阻塞，当cd减少到0时，会唤醒当前线程，继续执行。

在新建一个CountDownLatch对象时，需要指定减少的数量。当减少到0时，唤醒所有的await线程。

```java
public static void main(String[] args) throws InterruptedException {
    CountDownLatch cd = new CountDownLatch(2);
    new Thread(() -> {
        try {Thread.sleep(2000);} catch (InterruptedException e) {e.printStackTrace();}
        cd.countDown();
    }, "t1").start();
    new Thread(() -> {
        try {Thread.sleep(3000);} catch (InterruptedException e) {e.printStackTrace();}
        cd.countDown();
    }, "t2").start();
    cd.await();
    System.out.println("main thread complete");
}
```

#### Semaohore

Semaphore[ˈseməfɔːr]（谐音：四毛佛）信号量，可以用来控制当前执行线程数。

- 初始化时需要指定信号量的值
- 调用s.acquire()时请求一个信号量，如果当前信号量大于0，信号量-1，如果小于等于0时，将会被阻塞。
- 调用s.release()时释放一个信号量，信号量+1

#### ThreadLocal

ThreadLocal又叫做线程本地存储，目的是为了实现不同线程之间数据隔离的功能。

底层使用的是threadLocalMap， 然而对于每一个线程内部都包含了一个ThreadLocalMap变量，当用户设置值时，会将threadLocal作为key，用户数据作为value放入线程中ThradLocalMap中。该类有三个主要的方法： set(), get(). remove()分别对应赋值，取值和移除。

ThreadLocal有可能会产生内存泄漏问题，因为ThreadLocalMap的key是弱引用，但是value是弱引用，key有可能会被GC线程清除但是value不会，因此在不使用的时候及时清除。

#### 场景1：大量数据导入

例如对于需要在有限的资源内处理千万量级的数据，可以采用数据分页+线程池的方式来解决。

#### 场景2：数据汇总

一个任务需要调用各个系统下并且没用依赖关系，可以使用线程池 + Future来实现

伪代码如下

```java
public void static main(){
    Future<Object> future1 = new Thread(()->restTemplate.getObject(...));
    Future<Object> future2 = new Thread(()->restTemplate.getObject(...));
    Future<Object> future3 = new Thread(()->restTemplate.getObject(...));
    future1.get();
    future2.get();
    future3.get();
}
```

#### 场景3：异步调用

例如数据搜索外，需要保存搜索记录，此时可以另外一个开启线程来异步保存。

```java
void searchAndInsert(){
    search();
    insert();
}

// 异步保存
@Async
void insert(){}
```

# Spring Farmowork

## Spring

### IOC

#### Bean线程安全吗

Spring容器中的Bean对象可以是单例或者多例。一般来说容器中如果注入的是无状态的对象，是线程安全的，但是如果是有状态的或者定义可修改的成员变量时，则不上线程安全的，需要考虑加锁。

例如在Spring容器中注入一个Bean，该Bean中定义一个ArrayList成员变量，并且在Bean中的成员方法中涉及到list变量的修改查询操作，这种情况不是线程安全的，需要加锁。

例如下面的例子，在1000个并发请求下，会出现线程安全的问题：可能出现插入相同size的值或者抛出并发修改异常（ConcurrentModificationException）。

```java
@Component
public class XXXService{
    private final List<String> list = new ArrayList<>();

    @GetMapping("/test")
    public String add() {
        list.add(String.valueOf(list.size()));
        return list.toString();
    }
}
```

#### 底层接口

##### BeanFactory

Spring的顶层接口，也是Spring的核心容器，表面上只是getBean，但是控制控制反转、依赖注入，包括Bean的生命周期都是由它的实现类提供的

##### ApplicationContext

![image-20230613214905621](img\ApplicationContext.png)

由BeanFactory派生出来的，扩展了spring容器的功能，底层是实现下面的接口

- ApplicationEventPublisher: 事件发布功能
- BeanFactory: spring容器
- EnvironmentCapable: 环境信息，系统变量，properties文件，yml文件等配置的值
- MessageSource: 国际化
- ResourcecLoader:解析资源文件

##### BeanFactoryPostProcessor

增强beanDefination

- `ConfigurationClassPostProcessor`:能够解析@ComponentScan、@Bean、@Import、@ImportResource注解
- `MapperScannerConfigurer`：解析@MapperScan注解

##### BeanPostProcessor

增强bean

- `AutowiredAnnotationBeanPostProcessor`能够解析`@Autowired`
- `CommonAnnotationBeanPostProcessor`解析`@Resource`、`@PostConstruct`、`@PreDestroy`
- `ConfigurationPropertiesBindingPostProcessor`解析`@ConfigurationProperties`注解

#### Bean生命周期

```java
AbstractAutowireCapableBeanFactory{
    createBean(){
        //1.解析BeanDefiniation
        beanDefinition = new RootBeanDefinition(mbd);
        doCreateBean(){
            // 2.根据BeanDefiniition实例化对象
            instance = createBeanInstance(beanName, beanDefinition, args);
            // 3.填充bean对象的属性
            populateBean(beanName, BeanDefinition, instance);
            initializeBean(){
                // 4.调用各种Aware，注入相关容器信息
                invokeAwareMethods();
                // 5.调用后置处理器-前置处理
                applyBeanPostProcessorsBeforeInitialization();
                // 6.调用初始化方法：InitializingBean#afterPropertiesSet()和自定义的初始化方法<bean id="xxx" initMethodName=“xxx”/>
                invokeInitMethods();
                // 7.调用后置处理器-后置处理
                applyBeanPostProcessorsAfterInitialization();
            }
        }
     }
}
```

****

解析BeanDefination -> 构造函数(new) -> 依赖注入 -> Aware接口 -> BeanPostProcessor#before  -> 初始化方法 -> BeanPostProcessor#after -> 使用 -> 销毁

![image-20230612233535181](img\bean生命周期.png)

##### BeanDefinition

将XML文件中定义的bean解析为BeanDefinition对象，根据这个这个类来创建对象。

##### 构造函数

实例化对象

##### 依赖注入

三种方式：XML注入；注解@Value注入；@Configuration手动注入

##### Aware接口

底层实现： **invokeAwareMethods**

可以用来注入一些与容器相关的信息，例如BeanNameAware获取bean的名字， BeanFactoryAware获取bean工厂，ApplicationContextAware获取应用上下文

#### Spring循环引用

##### **什么是循环引用**

当对象A引用对象B，并且对象B引用对象A，或多个对象之间的引用形成了闭环时，成为循环引用。

##### **循环引用有哪几种形式**

属性注入，Setter注入，构造器注入。Spring内部只能自动解决前两种，构造器注入形式无法解决。为什么不能解决呢？是因为spring内部解决依赖冲突时在bean生命周期的构造函数和依赖注入中间阶段来解决的，如果在实例化时就发生了循环依赖，就无法解决。但是可以手动在构造器中添加@Lazy注解来解决，等真正使用的时候再初始化。

```java
public class A{
    public A(@Lazy B){}
}
```

##### **如何解决的**

参考博客：https://blog.csdn.net/lzb348110175/article/details/125086262

引入三级缓存，一级缓存存储已经创建好的对象，二级缓存存储已经实例化但是没用初始化的对象，三级缓存存储实例化后的对象的ObjectFactory对象

**如果A被AOP代理，那么通过ObjectFactory获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象。**

1. 首先A对象实例化，然后依赖注入B对象前，会先把A对象对应的ObjectFactory对象放入到三级缓存中
2. 开始实例化B对象，然后创建B对象对应的ObjectFactory放入到三级缓存中
3. 当B对象需要注入A对象时，B对象会从三级缓存中获取到A对象的ObjectFactory，创建A对象后放入二级缓存。
4. 当A对象注入到B对象后，B对象创建完，然后将B对象放入一级缓存并注入到A对象中。
5. 最后完成创建A对象，清除二级缓存中的A对象，并放入一级缓存。

##### 二级缓存能解决吗？

如果只是创建普通对象，不是代理对象，只需要二级缓存就可以，如果是代理对象，当B需要注入A对象时，需要从三级缓存获取到ObjectFactory对象然后调用getObject获取A的代理对象完成注入。

如果没用三级缓存，有两种解决方式：

第一种：每次A对象在实例化之后都完成AOP代理，放入到二级缓存。

第二种：每次都在需要注入A对象时判断是否需要对A进行代理，如果需要代理，从二级缓存中取出A对象完成代理。如果同时有很多引用A的代理对象，需要每次都调用ObjectFactory.getObject()获取方法，**但是每次获取到的A的代理对象是不同的**，导致不同的对象引用了不同的A代理对象，很明显和A对象是单例的像冲突。

![image-20230613234924323](img\三级缓存-2.png)

![image-20230613221807096](img\循环引用流程.png)

![image-20230613222436185](img\三级缓存.png)

### AOP

#### 什么是AOP

面向切面编程，指的是在现有的代码逻辑前后加入前置增强、后置增强或者环绕增强代码，实现增强代码逻辑和功能代码逻辑的解耦。例如，日志，事务等场景。

#### 项目中如何使用的

在项目中，有一项任务需要通过restful方式调用另外一个部门的API接口，在调用数据接口之前需要登录。因此在所有的调用数据的请求前加入后置处理，分为三步：

1. 调用取数据代码，如果正常返回数据，则结束。
2. 如果返回的http状态码是401，则会调用登录的api来取cookie，然后加入到httpclient中，接着会重新调用取数据的api代码后然后返回。

#### 事务是如何实现的

底层使用AOP实现，对标有声明式事务注解@Transaction的方法添加环绕增强，执行方法前开启事务，执行完毕后根据执行情况来判断是提交还是回滚事务。

#### 事务失效场景

##### 异常捕获

在添加@Transaction的方法上手动捕获异常，并没有把异常抛出。解决办法是在catch代码块中添加`throw new Excetion(e)`抛出异常。

```java
@Transaction
public void test1(){
    try{
        queryDB();
        int i= 1/0;
        insertDB();        
    }catch(Exception e){
        throw new Runtime(e);
    }
}
```

##### 抛出检查时异常

Spring事务默认只检查运行时异常，不检查其他异常。解决方法是更改tranaction回滚的异常类型。

```java
@Tranaction(rollbackFor=Exception.class)
public void test2() throw FileNotFoundException{
    query();
    new FileInputStream("");
    insert();
}
```

##### 非public方法

```java
@Tranaction(rollbackFor=Exception.class)
void test2() throw FileNotFoundException{
    query();
    new FileInputStream("");
    insert();
}
```

## SpringMVC

### 执行流程

#### 简约流程

1. 请求发送给DispatchServlet后，调用handleMapping获取到handleExecutionChain对象
2. DispatchServlet通过处理器获取到处理适配器，并调用处理适配器处理请求产生mv对象。如果返回的mv对象是空，直接结束。
3. DispatchServlet调用视图解析器获取到视图
4. DispatchServlet进行视图渲染

#### 视图流程

1. 用户发送请求到前端控制器DispatchServelet
2. DispatchServelt收到请求后调用处理映射器HandleMapping，
3. HandleMapping找到具体的处理器，生成HandleExecutionChain返回给DispatchServlet，包含拦截器和处理器
4. DispatchServlet调用HandleAdapter
5. HandleAdapter调用具体的处理器
6. 处理完成后返回modelAndView对象给HandleAdapter
7. HandleAdapter将结果ModelAndView返回给DispatchServelte，如果是空直接结束，不进行试图渲染。
   如果加了@responseBody注解，将会把处理后的诗句利用messageConvert转化后存入到response中
8. DispatchServlet将mv传给视图解析器
9. 视图解析器处理完成后返回View对象给DispatchServlet
10. DispatchServlet渲染视图

核心代码

```java
public class DispatcherServlet extends HttpServlet{
    doDispatch(){
    	HandlerExecutionChain mappedHandler = getHandler(processedRequest);
        HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
        ModelAndView mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
        if(mv != null){
            render(mv, request, response){
                View view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
        		view.render(mv.getModelInternal(), request, response);}
        }
	}
}
```

![image-20230614201119348](img\SpringMvc执行流程.png)

### 组件

#### DispatchServlet

作用：接受请求，响应结果。相当于中央处理器。

#### HandlerMapping

根据请求的URL找到对应的handle，支持多种方式，例如配置文件，实现接口，注解方式等。

#### HandlerAdapter

查看博客：https://blog.csdn.net/liuhaibo_ljf/article/details/106000158

处理适配器，在流程中根据处理器来获取到对应的处理适配器，该接口有下面几个实现类

- SimpleControllerHandlerAdapter： 当处理器实现了``Controller`接口，使用该处理适配器
- SimpleServletHandlerAdapter：当处理器`Servlet`接口，使用该处理适配器
- HttpRequestHandlerAdapter：当处理器实现`HttpRequestHandler`接口，使用该处理适配器
- **RequestMappingHandlerAdapter**: 当处理器使用@RequestingMapping注解时进行URL和方法映射时，使用该处理适配器

#### ViewResolver

视图解析器，根据逻辑视图名解析为真正的视图。

## SpringBoot

### 自动配置原理

在Springboot项目中的引导类上有一个注解@SpringBootApplication，该注解包含了三个注解

1. `@SpringBootConfiguration`：和`@Configuration`注解作用相同，表示当前引导为为配置类
2. `@ComponentScan`：扫描引导类所在包并注入到Spring容器中
3. `@EnableAutoConfiguration`: 自动配置注解，通过@Import注解导入配置选择器。内部实现是通过解析META-INF/Spring.factories文件中配置类，并结合@conditional注解判断是否导入到Spring容器中

## 常用注解

### Spring

![image-20230618145803030](img\Spring常用注解.png)

### Spring MVC

![image-20230618145833118](img\SpringMVC常用注解.png)

### SpringBoot

![image-20230618150025500](img\SpringBoot常用注解.png)

# Spring Cloud

## Consul

### 概念

是一款用于实现分布式系统服务注册与发现的开源工具。主要有五个特点：**服务注册与发现**、**健康检查**、**KV存储**、**安全通信**和**多数据中心**。

底层架构使用Client-Server模式，server一般3台或者五台，client是server的代理，可以扩展很多台。

Server一般是3台或者5台，因为当存活节点大于全部节点的一半时，集群才能正常工作，太少不满足可靠性，太多机器的话需要花费大量时间做数据同步。

每一个数据中心都是相对独立的一部分，在每一个数据中心内部选取单个leader，来负责处理该数据中心的所有查询和事务。

### 优点

数据一致性：当leader节点宕机时会停止所有数据节点，然后选举新的Leader后重启集群，在重启过程中，会拒绝服务。保证了数据的一致性。

多数据中心：Consul可以搭建多数据中心，每一个数据中心都是一个独立的集群，默认情况下，每一个集群里的数据是不同步的，也就是不相互影响。但是也可以支持多数据中心的数据同步，官方也给出响应的解决方案。

### 整合

第一步，添加spring-cloud-consul 依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
```

第二步，配置服务注册与发现的地址，同时也可以配置健康检查路径

```properties
spring.cloud.consul.host=wpc.vm
spring.cloud.consul.port=8500
spring.cloud.consul.discovery.hostname=wpc.vm
spring.cloud.consul.discovery.heartbeat.enabled=true
spring.cloud.consul.discovery.health-check-path=
```

第三步，在主启动类上添加@EnableDiscoveryClient注解，开启自动配置

## APIGateway

### 概念

参考博客：https://blog.csdn.net/crazymakercircle/article/details/125057567

该项目是是基于SpringBoot和Reactor开发的网关，

目的是

1. **提供一种简单有效的API路由管理方式**
2. 安全校验：例如是否是在黑名单中，是否是合法用户，是否具备访问资源的权限
3. 监控请求
4. 限流

SpringCloud-Gateway是基于WebFlux框架来实现的，底层使用了高性能的Reactor模式通信的Netty框架。

### 处理流程

1. 客户端发起请求到Gateway
2. Gateawy在Handle Mapping中找到与请求匹配的路由，然后发给WebHandler
3. WebHandler通过各种过滤器的preHandle后，发送到具体的服务上
4. 服务处理完成之后的响应信息会再走一遍过滤器的postHandle方法

![Spring Cloud Gateway Diagram](img\SpringCloudGateway处理流程.png)

### 组件

路由Route：由ID，目标URI，一组断言和一组过滤器组成

断言Predocate：路由转发条件，可以通过对HTTP请求进行匹配，例如请求头，请求参数，请求时间，请求路径等

过滤器Filter：对请求进行拦截，处理和修改

#### 断言

SpringCloud-Gateway对于断言和过滤器的编写提供了两种方式：配置文件（一般是yml形式）和编码。

另外，路由匹配的前提是满足一系列的断言规则。

SrpingCloud内置了很多断言规则可以使用。例如根据路径，时间，Cookie，请求头，查询参数，请求主机，请求方法等。可以参考官网文档part5: https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories

当然，也可以自定义断言，需要实现RoutePredicateFactory接口（也可以继承AbstractRoutePredicateFactory抽象类）并注入到Spring容器中。

#### 过滤器

可以对传入的HTTP请求和响应进行修改。包含路由过滤器和全局过滤器两种。

- 路由过滤器：作用于特定的路由。官方内置了一些过滤器：对路径加前缀，改写路径，设置请求头等。也可以自定义路由过滤器，需要实现`GatewayFilterFactory`接口（也可以继承AbstractGatewayFilterFactory抽象类）并注入到容器中
- 全局过滤器：作用于全部的路由，也可以自定义全局过滤器，实现`GlobalFilter`接口。

## Sleuth

### 概念

`SpringCloud-Sleuth`提供了一种分布式链路追踪的解决方案，也集成了Zipkin。分布式链路追踪就是将一次分布式请求还原成调用链路进行日志分析，比如各个节点的耗时，处理机器、处理状态等。

内部实现

1. Span：基本的工作单元，使用SpanId来标记。除了记录了在当前服务组件的开始、处理和结束时间外，还记录了时间的名称，请求信息等元数据。
2. Trace：表示一条请求链路，通常为树状结构，由一组Span串联而成。一条链路的TraceId是相同的。
3. Annotation/Event：用来记录一段时间内的时间，例如Client Send / Server Received / Server Send / Client Received

### 实现原理

1. 当一个请求在分布式系统中流转时，始终保持传递这一个标识TraceId。
2. 对于每一个服务组件，创建对应的Span，并使用SpanId来标识。此外，还会将上一个服务组件的SpanId标识为parentSpanId，这样一条链路便串联起来。
3. 具体实现是通过在传递的消息头上添加响应的TraceID和SpanId等信息，例如在HTTP头信息，JMS的头信息等消息上。

### 抽样收集

为了节约资源，Sleuth还支持抽样收集，同时消息是否被收集也是通过消息头上的Sampled来决定

Sleuth内置了一些抽样策略，例如通过百分比的方式收集，同时还支持自定义的抽样策略，需要实现Sample接口。

### 整合ELK

Sleuth在整合ELK中，主要负责日志收集，配合Logstash完成数据对接。由于SpringBoot和LogStash都支持logback方式记录收集日志，因此，只需要定义日志转化的格式即可。

例如Sleuth安装指定格式收集日志到JSON文件中，然后ELK可以负责展示管理。

### 整合Zipkin

Zipkin是推特下的一个开源项目，主要有以下功能

1. 收集请求链路上的各个服务器上的跟踪数据
2. 提供API接口来查询跟踪数据，从而找到系统性能瓶颈
3. 提供UI组件更直观地搜索跟踪信息和分析请求链路明细。
4. 默认情况下，Zipkin将跟踪信息存储在内存中，也可以配置Mysql持久化存储功能。

## Contract

### 概念

**契约测试**：简单理解为提前模拟服务端的数据给调用者。

具体操作：

1. 服务端提前定义一个已知的结果，也叫契约，可以使用Groovy或者YML文件定义，并生成stub包提供给消费者
2. 当消费者调用的时候直接返回该结果，该结果并不是调用的真实服务，而是从stub包中获取到的结果
3. 根据提供的结果来判断消费者程序是否正确。

常用于测试方法中，介于单元测试和集成测试之间，既脱离了服务端的真实服务依赖，又使用了服务端的相对真实的数据。

### 底层实现

当服务端代码编写完毕进行打包时，除了打出来服务的jar包，还有一个stubs的jar包，在该jar包内包含了基本测试类，编写的契约文件和生成的Json文件，里面定义了测试的具体内容，例如调用的接口和应该返回的结果。

消费者进行测试时，当启动服务时，不会连接真实的服务端，因为Contract stub会启动mock服务器，并使用之前定义好的契约来返回数据。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ConsumerApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
//初始化测试配置,测试controller需要
@AutoConfigureMockMvc
@AutoConfigureJsonTesters
@AutoConfigureStubRunner(ids = {"com.xiaobai:producer:+:stubs:10001"},stubsMode = StubRunnerProperties.StubsMode.LOCAL)
@Slf4j
public class ConsumerTest {
    @Autowired
    private RestTemplate restTemplate;

    @Test
    public void testMethod() throws Exception {
        ResponseEntity<JSONObject> response = restTemplate.exchange(
                "http://localhost:10001/user/1",
                HttpMethod.GET,
                null,
                JSONObject.class
        );
        log.info("测试数据:"+ response.getBody().toJSONString());
    }
}

```



# 中间件

## Redis

### 概念

Redis是一种开源的数据存储系统，可以用作**数据库、缓存、消息中间件**。

- 数据结构：字符串String，列表List，集合Set，哈希表Hash，有序集合Zset等
- 内置功能：复制，LUA脚本，LRU驱动时间，事务和不同类型的持久化策略（RDB和AOF）
- 高可用：通过哨兵模式和自动分区提供高可用

### 持久化

#### RDB

全称：Redis Database Backup file

一定时间间隔内把内存中的数据写入磁盘，相当于打个快照，恢复时也是直接读文件到内存中。

该方式是默认开启的，在持久化过程中，会fork一个子进程。先写入临时文件，然后持久化结束后再把这个文件替换上次持久化的文件。

缺点是最后一次持久化操作后的数据可能丢失。

##### 保存策略

- save 900 1 900秒内有1个key变化就保存
- save 300 10 300秒内有10个key变化就保存
- save 60 10000 60秒内有10000个key变化就保存
- save "" 禁用RDB模式

##### 优点

文件类型：持久化文件是二进制文件，同时也记录了时间戳，可以快速定位数据版本

性能方面：在持久化过程中，只需要fork子进程就可以，持久化工作是由子进程来实现，不影响Redis主线程的正常运行

数据恢复：二进制文件数据恢复和传输比较方便

##### 缺点

数据完整性：无法保证数据的完整性，在最后一次持久化后的数据将会丢失。

性能：如果Redis整体数据特别大，每次持久化会很耗时，持久化过程将占用大量系统资源，导致主线程无法及时响应

#### AOF

全称Append Only File

AOP是以日志文件保存每一次的增删改操作。当Redis重启时，会重新执行这些命令来恢复数据。

默认不开启，需要手动设置。

##### 保存策略

- appendfsync always: 每当有新的修改数据命令时都会执行保存操作：效率低，但是安全
- appendfsync everysec: 每秒保存一次，如果断电，可能会丢失一秒的数据。
- appendfsync no: 从不保存，将数据交给操作系统来处理。

##### 优点

1. 相对于RDB更安全
2. 以日志文件格式保存，包含的是各种redis命令，容易解读

##### 缺点

1. 比RDB更占用磁盘空间
2. 恢复速度慢
3. 每次读写都要同步的话，有一定的性能压力

### 事务

参考博客：https://blog.csdn.net/shark_chili3007/article/details/120884205

Redis中的事务只是一个单独的隔离操作，主要作用是串联多个命令防止其他命令插队。

Redis中的事务时基于乐观锁来实现的，底层实现是用CAS

![image-20230624185437191](img\Redis事务.png)

![image-20230624190746615](img\Redis事务案例.png)

常用命令：**Multi**开启事务，**Exec**执行事务，**Discard**取消事务

事务的错误和回滚：

- 如果语法层面上有报错，会遗弃所有的命令。
- 如果是运行中报错，则会执行所有的正确指令。

### 消息订阅

Redis是基于发布订阅的方式实现消息通信，即：发送者发送消息，订阅者接受消息。

- Subscribe [channel...] 订阅频道
- Publish [channel] [message] 发布消息到指定的频道

### 主从复制

当配置多态服务器时，主机和从机的身份是分开的，主机以写为主，从机以读为主。

当从机联通后，会向主机发送sync指令来同步数据。当主机接收到写操作时，也会发送给从机。

从机是从头开始复制主机的信息，是完全负责。另外从机不能写，只能读

### 哨兵模式

在主从模式中，系统不具备自动修复功能，当主节点宕机时，需要手动切换服务器，安全性得不到保障。

Redis官方推荐一种高可用方案：哨兵模式。当主机发生故障时，会自动进行故障转移。

![Redis哨兵模式](img/哨兵模式.png)

1. 主观下线：适用于主机和从机。在规定的时间内没用收到服务器的有效回复时，会被判定为主观下线。
2. 客观下线：只适用于主机。当哨兵发现主机出现故障时，会询问其他哨兵对主机的判断，如果超过半数以上的哨兵认为主机宕机了，就会被标记为客观下线。
3. 投票选举：所有的哨兵会通过投票机制，选举一个哨兵做故障转移操作。被选举的哨兵按照一定的规则从从服务中选举一个最优的服务器作为主服务器，并通过发布订阅方式通知其余从节点更改配置，跟随新上任的主服务器。至此完成了主从切换。

### 缓存问题

参考博客：http://c.biancheng.net/redis/cache.html

#### 缓存穿透

数据在缓存中没有，在数据库中也没有

解决方案：缓存空对象和布隆过滤器（将所有存在的数据都哈希到一个足够到的map中，如果这个map里没有，则一定没有）

#### 缓存击穿

数据在缓存中没有，但在数据库中存在。

一般出现在热点数据过期场景，大量并发访问该数据将会直接访问数据库。

解决方案：改变过期时间和分布式锁。

#### 缓存雪崩

大量的key同时过期，导致大量请求同时访问数据库。

解决方案：对key设置随机不同的过期时间

#### 缓存预热

在系统上线时，阿静热点数据加载到缓存系统中，这样当用户请求时，就可以直接查询缓存了。

## Nginx

### 概念

1. 是一款高性能**反向代理**服务器。
2. **占用内存少**，**并发能力强**。
3. 支持**动静分离**，即静态志愿和动态资源来自于不同的服务器，加快解析速度。也可以作为静态页面的web服务器。
4. 具备**负载均衡**能力

Tips：正向代理是代理客户端，对服务器隐藏真实信息，如VPN。反向代理是代理服务器，隐藏真实访问的服务器。

### 配置文件

主要由三部分构成

1. 全局块：主要设置一些整体的配置，例如：用户组、日志文件路径、配置文件等
2. event块：主要是配置Nginx服务器与用户的网络连接。例如序列化，网络连接数，驱动模型等。
3. http块：包含http全局块和server块。
   每个http块可以包含多个server块，每一个server块相当于一台虚拟主机。
   每个server块可以包含多个location块，代表不同规则。

```nginx
worker_processes  1;

#定义负载均衡组，upstream可以放在全局中，也可以放在http块中
upstream backend {
    #ip_hash 使用hash算法库避免登录信息丢失
    #weight=3 添加服务器比重
    server 192.168.56.128:8080 max_conns=2 fail_timeout=15s weight=3;
    server 192.168.56.129:8080 weight=7;
    #fair; 按照响应时间来分派
}

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

#         location后面也有一个参数: 等号=是精准匹配；波浪号~是正则匹配；波浪号加星~*是忽略大小写正则匹配；^~表示正常匹配后不再正则匹配
        location / {
            root   /home/ruoyi/projects/ruoyi-ui;
            try_files $uri $uri/ /index.html;
            index  index.html index.htm;
        }

        location /prod-api/ {
#             $http_host代表本机
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header REMOTE-HOST $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://ruoyi-gateway:8080/;
        }

        # 避免actuator暴露
        if ($request_uri ~ "/actuator") {
            return 403;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## Tomcat

### 总体架构

Tomcat服务器主要包含两部分，连接器Connector和容器Container，分别使用Coyote和Catalina实现的。

![img](img\Tomcat顶层架构图.png)

### Connector(Coyote)

链接器主要作用和流程是是

1. 监听：EndPoint接口类监听服务器端口，读取客户端请求
2. 解析：Processor将请求根据指定协议进行解析，例如HTTP/AJP
3. 映射：通过映射表并根据请求地址，来匹配正确的容器进行处理。底层使用了适配器模式解耦。
4. 响应：将响应返回给客户端

![img](img\Tomcat-Coyote流程.png)

支持的协议：HTTP和AJP

支持的IO方式，BIO(8.5版本移除)，NIO，NIO2，APR

Tomcat针对不同的协议和IO方式，提供了不同的实现，他们都实现了ProtocolHandler接口,这里使用了**桥接模式**，因为处理器的种类有两个变化维度，在AbstractProtocol中包含了AbstractEndpoint来定义注入不同IP方式的实现。

![img](img\Tomcat-链接器原理.png)



### Container(Catalina)

Catalina是Servlet容器的具体实现。具体工作：

1. 解析：使用Digester解析XML文件
2. 创建容器：根据配置文件创建容器对象，在tomcat容器中，都实现了Container接口。
3. 初始化：初始化创建的对象
4. 启动容器：容器类都实现了Lifecycle接口，实现了例如init,start,stop等方法。

在容器的层级关系中，大概如下图的结构

1. 一个serve实例包含很多service，一个service包含多个Connector链接器和Engine引擎
2. 引擎内部也是一对多的关系：分别是Host虚拟主机，Context上下文，Wapper具体的Servlet包装类。

![img](img\Tomcat容器结构.png)

## Kafka

### 概念

1. Kafka是一个**分布式**的基于**发布订阅模式**的**消息队列**，主要应用于大数据实时处理领域。
2. 发布订阅：分为多种类型，订阅者根据需求选择性的订阅
3. 优点：解耦，削峰，异步

4. 两种模式：

   1. 点对点模式：一个生产者一个消费者，一个topic，消费完后删除数据。

   2. 发布订阅模式：多个生产者和多个消费者相互独立，多个topic消费完不会删除数据
5. 架构
   1. 生产者：
   2. broker集群
      1. broker服务器
      2. topic主题
      3. partition分区
      4. replication副本
      5. leader&follower主从
   3. 消费者
      1. 消费者之间相互独立
      2. 消费者组（某个分区只能有一个消费者消费）
   4. zookeeper
      1. 在线的broker(broker.ids): /kafka/brokers/ids
      2. 分区leader: /kafka/brokers/topics/first/partitions/0/state

### 生产者

![image-20230718001229514](img\kafka-procedure-send-flow.png)

#### 分区

分区的好处：

1. 合理的利用资源，如果数据量很大，把数据放在一个很大的区域内，不方便操作
2. 提高并行度：生产者可以以分区为单位发送数据，消费者可以以分区为单位消费数据。

默认分区规则：

- 如果指定分区，则按照分区规则执行
- 如果没有指定分区，但是设置的有key，则根据key的哈希值与分区数取模定位分区
- 如果没有指定分区也没有key，使用粘性策略，第一次随机挑选，到达一定时间，重新随机

自定义分区规则：

- 定义类实现Partitioner接口，
- 重写partition
- 在配置中引用该自定义分区规则

#### 提高吞吐量

1. 修改批次大小(batch.size)：16k -> 32k
2. 修改等待时间(linger.ms) ：0-5ms
3. 压缩
4. 缓冲区大小：32MB ->64MB

#### 可靠性

acks:

- 0: 不需要等待服务器响应，可靠性差，但效率高，会丢失数据
- 1：需要leader应答后继续发，也可能会丢失数据，适合传输普通日志
- -1：需要leader和ISR队列中的所有的folloer应答，可靠性高，但是效率差
- 完全可靠：ack =-1 & 副本大于2 & isr >=2

#### 数据重复

幂等

- <pid,分区号，序列号> 默认打开
- 幂等性只能保证单分区会话不重复

事务

- 底层基于幂等
- 5个API

#### 数据有序

单分区内有序：

多分区有序

乱序：1.x版本

### Broker集群

#### zookeeper

![image-20230718001058985](img\kakfa-broker-zookeeper.png)

1. 当前所有的节点broker.ids
2. leader
3. 辅助选举controller

#### 副本

- 作用：提高数据可靠性，默认是一个，生产环境一般配置两个
- 分区统称为AR。AR=ISR(活跃的Follow集合)+OSR
- controller：前提是ISR上存活，第一抢到的broker设置controler leader，如果leader挂了轮询选一个 
- leader挂了（选举一个leader然后同步follower）和follower挂了（同步最高水位线HW后的数据）
- 副本分配，默认是均匀的分配到每个机器上，防止过度集中和依赖某个机器。也可以手动配置副本的分配。

#### 存储机制

1. Topic是逻辑概念，partition是物理概念，每个分区对应一个log文件。
2. procedure生产出来的数据会被不断追加到log文件末端
3. kafka采用分片和索引机制，将每个partition分为多个segment，每个segment包含索引文件，数据文件和时间戳。
4. segement默认是1GB，index默认是4kb。所以采用的是稀疏索引。每当写入4KB的数据时，插入一个索引。
5. 数据默认保存时间为7天，默认直接删除，页可以选择压缩保存

#### 高效读写

kafka为什么运行那么快，为什么可以高效读写

- kafka本身是分布式集群，可以采用分区技术，并行度高
- 读数据采用稀疏索引，可以快速定位消费的数据
- 顺序写磁盘
- 页缓存和零拷贝技术

### 消费者

![image-20230724222956040](img\Kafka消费者初始化.png)

![image-20230724223057797](img\Kafka消费者流程.png)

#### 消费方式

kafka默认是消费者**主动拉取数据**的策略。因为如果是推送模式的话，如果每个消费者有不同的消费速率，很难决定推送的速度。

拉模式也有确定，如果broker集群没有数据，将一直空转。

#### 消费者组

1. 由多个消费者组成，具备相同的groupid
2. 消费者组内每个消费者负责不同分区的数据，一个分区只能由一个消费者消费
3. 消费者组可以理解为一个独立的消费者，消费者组之间互不影响
4. 消费者组里消费者应该消费哪个分区的数据？主流的分区分配策略是：RoundRobin，Range，Sticky（粘性）CooperativeSticky（合作者粘性）。可以同时使用多个分区分配策略，默认是Range+CooperativeSticky

#### Offset

在0.9前存储在zookeeper中，0.9版本后为了建设过多的通信，放入到kafka内置的__consumer_offset主题中

- 默认存储在topic中
- 自动提交 5S
- 手动提交，同步或者异步
- 也可以按照offset或者时间消费
- 自动提交可能会出现，漏消费或者重复消费，可以采用事务来解决

#### 数据积压

消费者如何提高吞吐量？

1. 可以适当增加分区数和消费者组中的消费者
2. 提高每批次拉取的数量



## Tibco EMS

### JMS

全称Java message Service，Java消息服务，定义应用之间消息传递的Java框架规范。是接口规范，而不是具体的实现。

JMS组成：生产者，服务器和消费者。其中生产者和消费者统称为client

JMS模型：点对点和发布订阅

### TIBCO

TIBCO公司的产品 TIBCO Enterprise Message Service也是JMS标准规范的实现，除了作为消息的中介，还提供了企业级的特性，如容错、消息路由等，同时还支持和TIBCO Rendezvous等产品组进行消息通信。

Tibco EMS创建消息时的增强包括：

- 错误恢复：备份消息到磁盘防止服务器宕机是丢失信息
- 权限控制：在队列或者主题上定义权限
- 消息路由：消息可以路由到其他服务器，也就是说服务器可以作为一个消费者接受其他server的消息
- 加密连接：client和服务器间可以设置SSL加密连接
- 容错、负载均衡、流量控制、高扩展和高可靠性

## Gemfire

### 是什么

Geode/Gemfire 是Pivotal公司开发的一款开源的、分布式NoSql内存数据库，可用来进行完成分布式缓存，数据持久化，分布式事务、动态扩展等功能。

参考博客：https://www.php.cn/faq/562818.html

### 术语

- 数据网格：是一个高扩展、分布式的内存缓存系统。
- 区域：网格中的区域，是数据管理中心。在gemfire中，每个区域对应一个缓存，每个缓存都提供了一个独立的数据区域，可以访问、查询和更新数据。区域可以视为多个缓存的组合。
- 节点：网格中一个实际运行的服务和集群中的一个成员。节点提供了可靠的通信机制，支持在整个网格中进行数据访问和存储。它还包含了一些核心的可配置的参数，例如IP地址和端口。

### 优势

- 高效、可扩展、分布式、支持多种数据源
- 基于事件的通讯机制，及时响应变化
- 提供很多开箱即用的工具和库，方便和其他系统集成 

# Mysql

## 索引

Mysql索引的分类：B+树、Hash索引、全文索引

### InnoDB/MyISAM

两者都是使用B+树来实现索引的。

两者区别：

- InnnoDB是聚簇索引，MyISAM是非聚簇索引。聚集索引是指，索引结构和内容是存储在一起的，比如说字典。非聚集索引是索引和内容是分开的，比如目录。
- MyISAM索引文件和数据文件是分离的，当索引查到具体的值时，索引值保存的是指向内容的指针。

B+树

当一个表没有创建索引时，也会创建B+树用来存储数据

1. 如果有主键会创建聚簇索引
2. 如果没有主键会生成rowid作为隐式主键

### B树和B+树

Mysql底层使用页来存储数据，每个页的大小为16KB。

B树每个节点/页都会存储数据，

B+树只有叶子节点存储数据，非叶子节点只会存储索引。

一个B+树大概**存储多少索引**？大概存储千万级别

1. 假设一条记录的主键是int类型，则占据4个字节；指针为6个字节，共占据4+6=10个字节，一页可以存储16kb/10 = 1600条索引
2. 假设一条记录的大小为1kb，一页为16kb，则一页可以存储16条数据。
3. 对于三层索引来看，根节点（一级索引）可以存储160条二级索引，二级索引可以存储1600*1600=2,560,000条三级索引，三级索引对应的是叶子节点，一页存储16条，索引三阶索引结构可以存储1600 * 1600 * 16 = 40,960,000

### 记录信息

每一个记录包含了以下内容

- record_type：记录的类型，0是普通数据，1是非叶子节点，2是最小记录，3是最大记录，一般来说，非叶子节点的页的记录链表为：2->1->1....->3；叶子节点的记录链表为2->0->0->.....->3
- next_record下一条记录的相对位置
- 列值：各个列的值
- 其他信息

### 聚簇索引

#### 查找逻辑

1. 根据根节点的页的节点内的链表中的值来判断在那个子页
2. 依次深度查找直到叶子页
3. 在叶子节点查找对应的记录和记录的值

#### 复杂度

查找复杂度： O(NLogn)，N表示B+树的高度，n表示一页的个数

CRUD复杂度和查找相同。

#### 特点

- 索引和数据在同一个B+树中
- 页和页之间是双向链表
- 页内技术是按照主键的顺序排成的单向链表
- 非叶子节点存储的是记录的主键+页号
- 叶子节点存储的是完整的用户记录

![image-20230728223940210](img\Mysql-聚簇索引.png)

#### 优点

- 数据访问快
- 对于主键查找和**范围查找**速度非常快
- 由于数据紧密相连，减少IO操作

#### 缺点

- 插入速度依赖主键排序规则。因此，一般innoDB表都会定义一个自增的id列为主键。
- 更新主键时，需要移动数据。因此，一般定义为主键不可更新。

#### 限制

- 只有InnoDB支持，MyISAM不支持
- 为了充分利用聚簇特性，需要保证主键有序

### 非聚簇索引

聚簇索引存储的是**主键**，当条件为主键时才会发挥作用。

非聚簇索引存储的是非主键，当用非主键列查询时有用。



#### 区别

非聚簇索引的B+树存储的是非主键列，非叶子节点存储的是索引，叶子节点存储的不是完整的数据，而是非主键列的值和主键的值。当通过非聚簇索引查找时，

1. 在**非聚簇索引**中查找到非主键列对应的记录
2. 记录内包含了非主键列的值和主键的值
3. 使用主键的值在**聚簇索引**中查找对应的数据。

#### 比较

1. 如果存储的是数字，直接比较
2. 如果存储的是字符串，将比较ascii码来比较大小
3. 如果存储的是文本/联合索引，也会比较ascii码
4. 最好使用后置模糊匹配，前置模糊匹配会导致索引失效全局扫描

### 自适应哈希索引

是InnoDB的一个特殊功能，当某些索引值被频繁使用时，会在B树上在创建一个哈希索引，更快速的获取获取数据。这完全是系统自动行为，用户无法控制。

```SQL
-- 查看INSERT BUFFER AND ADAPTIVE HASH INDEX
SHOW ENGINE INNODB STATUS;
```



# Shell

# Mybatis

## 基础

### 概念

是一个半自动ORM框架，封装了JDBC的操作，专注于SQL语句的编写，提高开发效率

### 优缺点

**优点**

- 基于SQL语句编程，开发相对灵活
- 封装了各种JDBC操作，和原生JDBC相比，减少了一半的代码量
- 兼容各种数据库
- 和Spring很好的兼容
- 提供很多灵活的标签，支持动态SQL

**缺点**

- 需要手动写SQL，需要一定的SQL基础
- SQL语句依赖具体的数据库，移植性较差

### #{}和${}

#{}是预编译，会调用PreparedStatement的set方法赋值

```java
// 原生SQL
PreparedStatement ps = connection.prepareStatement("select * from user where id=?");  
ps.setInt(1, 123);  
ResultSet rs = ps.executeQuery();  
```

${}是直接替换，有SQL注入的风险

```java
// 原生SQL
int sid=123;
Statement stmt = conn.createStatement();  
ResultSet rs = stmt.executeQuery("select * from user where id="+sid);  
```

### 字段名不同

1. 通过SQL别名解决

2. 通过resultMap设置字段和属性的映射关系
   ```xml
   <resultMap type=”me.gacl.domain.order” id=”orderresultmap”>
    <!–用 id 属性来映射主键字段–>
     <id property=”id” column=”order_id”>
    <!–用 result 属性来映射非主键字段，property 为实体类属性名，column
    为数据表中的属性–>
     <result property = “orderno” column =”order_no”/>
     <result property=”price” column=”order_price” />
   </reslutMap>
   ```

### 插入获取主键

获取到insert插入后的主键id，可以在insert标签中添加useGeneratedKeys参数即可

```xml
<insert id="insertAuthor" useGeneratedKeys="true" keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

## 进阶

### 设计模式

缓存模块：装饰器模式

日志模块：适配器模式

SqlSessionFactory: 工厂模式

SqlSessionFactoryBuilder：建造者模式

Mapper接口：代理接口，JDK动态代理

### 执行流程

1. 读取Mybatis配置文件，加载运行环境和Mapper映射文件
2. 构建会话工厂SqlSessionFactory
3. 会话工厂创建SqlSession对象，包含了SQL语句增删改查的所有方法
4. 操作Executor执行器，同时负责缓存的维护
5. Executor接口方法中有一个MappedStatement类型的参数，封装了映射的信息
6. Executor执行前，进行输入参数映射，执行SQL
7. 输出结果映射

![image-20230618160816700](img\Mybatis执行流程.png)

### 执行器

1. SimpleExecutor: 简单的执行器，每次执行操作都会开启一个新的Statement对象，用完就会立刻关闭。
2. ReuseExecutor:重复使用执行器，实现了对Statement对象的复用
3. BatchExecutor:批处理任务执行器。

所有的执行器都和SqlSession生命周期保持一致

### 延迟加载

当遇到1对多的情况下，可以手动配置延迟加载：即当用到的时候才加载，默认是false，立即加载。

延迟加载的底层是使用CGLIB代理来实现的

具体做法是在<collection>中设置`fetchType="lazy"`或者在全局配置中设置`lazyLoadingEnabled=true`

```java
public class User{
    private int id;
    private String name;
    private List<Order> orderList;
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.mapper.StudentMapper">
    <resultMap id="studentMap" type="org.mybatis.example.bean.Student" autoMapping="true">
        <id property="id" column="id" />
        <result property="name" column="name"/>
        <!-- 懒加载，只有当用到blogList的时候才会加载-->
        <collection property="blogList" ofType="blog" column="id"
                    select="org.mybatis.example.mapper.BlogMapper.selectBlogBySid" fetchType="lazy"/>
    </resultMap>
    <select id="selectStudent" resultMap="studentMap">
        select * from student where id = #{id}
    </select>
</mapper>
```

### 约定

1. Mapper接口和XML映射文件名称一致
2. Mapper接口的方法名要和XML映射文件中的id一致
3. Mapper接口的方法返回类型和XML定义的标签返回类型一致
4. Mapper接口的全类名和XML全路径一致

## 高级

### SqlSession线程安全问题

SqlSession是线程不安全的，我们在工作中不会单独使用DefaultSqlSession，而是整合了Spring框架来使用。

Spring框架整合Mybatis是如何解决线程安全问题的?

首先，线程不安全的原因是多个线程同时操作同一个成员变量，如果将该成员编程编程局部变量，将不存在线程安全的问题。



Spring框架的做法是创建一个SqlSessionTemplate模板对象，定义了数据库操作的相关方法，本质上是通过代理对象来获取DefaultSqlSession对象后来执行的，而且吧DefaultSqlSession对象声明在了局部变量中，从而解决了线程不安全的问题。

```java
  // MybatisAutoConfiguration，注入SqlSessionTemplate
  @Bean
  @ConditionalOnMissingBean
  public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
    ExecutorType executorType = this.properties.getExecutorType();
    if (executorType != null) {
      return new SqlSessionTemplate(sqlSessionFactory, executorType);
    } else {
      return new SqlSessionTemplate(sqlSessionFactory);
    }
  }
```

```java
// org.mybatis.spring.SqlSessionTemplate，构造器	
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType,
      PersistenceExceptionTranslator exceptionTranslator) {
    // 创建代理对象
    this.sqlSessionProxy = (SqlSession) newProxyInstance(SqlSessionFactory.class.getClassLoader(),
        new Class[] { SqlSession.class }, new SqlSessionInterceptor());
}
```

![image-20230619235536546](img\SqlSession线程安全问题)

### 缓存

mybatis的一级缓存和二级缓存，默认开启一级缓存，关闭二级缓存，如果需要开启二级缓存，只需要再映射文件中添加标签<cache />即可

缓存更新机制：当某一个作用于进行增删改操作后，默认会清空改作用于下的所有查询的缓存。

一级缓存：基于本地的HashMap本地缓存，其作用域是session，当session进行刷新或者关闭后，缓存就会被清空。

```java
public class TransactionalCacheManager {
	Map<Cache, TransactionalCache> transactionalCaches = new HashMap<>();
}
```



![image-20230619115340312](img\Mybatis一级缓存.png)

二级缓存：作用域是namespace和mapper的作用于，不依赖session。当某一个作用域发生了增删改操作后，会清空二级缓存数据。

![image-20230619120157911](img\Mybatis二级缓存.png)

# 数据结构

## 散列表

1. 概念：又名哈希表，是一种根据给定的key直接方法内存中的值的数据结构，由数组演化而来，利用数组根据下标可以快速定位的特性
2. 底层实现：是数组加链表或者数组加红黑树
3. 复杂度：平均情况下O(1)，如果发生哈希冲突，则查询复杂度：链表O(n)，红黑树O(logN)
4. hash函数：如果两个对象值相等，则hashcode一定相等；反之不成立
5. 发生哈希冲突后，可以使用拉链法（链表）

## 树

### 二叉搜索树

Binary Search Tree, BST

1. 概念：左边的值 < 中间的值<  右边的值
2. 复杂度：查找，插入和删除的时间复杂度为O(logn)
3. 极坏情况下，例如有序递增数组，生成的二叉搜索树会退化为链表

### 平衡二叉树（AVL)

1. 概念：左右子树的**高度差**的绝对值不超过1
2. 复杂度：查找时间复杂度最坏和平均都是O(nlogn)
3. 添加删除后需要判断是否失衡，如果失衡则需要进行调整，总共分为四种

### B树

B-tree，B代表balance，又叫平衡多路查找树。

一颗m阶的B树应该满足：

1. 每个节点最多只有m个子节点。
2. 每个非叶子节点（除了根）具有至少⌈ m/2⌉子节点。
3. 如果根不是叶节点，则根至少有两个子节点。
4. **具有k个子节点的非叶节点包含k -1个键。**
5. 所有叶子都出现在同一水平，没有任何信息（高度一致）。

### B+树

- 有m个子树的中间节点包含有m个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引；
- 所有的叶子结点中包含了全部关键字的信息，及指向含有这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大的顺序链接。 (而B 树的叶子节点并没有包括全部需要查找的信息)；
- 所有的非终端结点可以看成是索引部分，结点中仅含有其子树根结点中最大（或最小）关键字。 (而B 树的非终节点也包含需要查找的有效信息)；

![在这里插入图片描述](img\数据结构-B+树.png)

### B/B+树区别

<table>
    <thead>
        <tr>
            <th align="left"><strong>B树</strong></th>
            <th align="left"><strong>B&#43;树</strong></th>
        </tr>
	</thead>
    <tbody>
        <tr><td align="left">B树的每个节点&#xff0c;有m个key&#xff0c;m&#43;1个指针&#xff0c;每个指针分别是区间&#xff0c;代表大于前面的key&#xff0c;小于后面的key</td><td align="left">B&#43;树的每个节点&#xff0c;有m&#43;1个key&#xff0c;m&#43;1个指针&#xff0c;每个指针与一个key对应&#xff0c;代表子节点中的数全部大于当前key。同时&#xff0c;因此每个节点的key值更多&#xff0c;所以整个树的高度更低。</td></tr><tr><td align="left">B树中每个节点的每个key都有数据信息</td><td align="left">B&#43;树中只有叶子节点有数据信息&#xff0c;非叶子节点没有。所以B&#43;树的非叶子节点大小更小</td></tr><tr><td align="left">B树的所有数据是一个整体&#xff1a;非叶子节点也是数据的一部分&#xff0c;可能还没到叶子节点就已经找到&#xff0c;返回了。B树的所有节点都是数据</td><td align="left">B&#43;树的非叶子节点就是单纯的索引&#xff0c;所有实际的数据都存储在叶子节点中&#xff0c;所以每次查询&#xff0c;都必须查询到叶子节点&#xff0c;所以每次查询的速度就十分的稳定</td></tr><tr><td align="left">B树不可以进行叶子节点间的顺序查找&#xff0c;同时若是可以也没意义&#xff0c;因为是中序遍历</td><td align="left">B&#43;树的叶子节点有指针连着&#xff0c;可以范围查找&#xff0c;即循着范围起点的叶子节点进行顺序遍历</td></tr></tbody></table>

### 红黑树

#### 性质

1. 节点是红色或者黑色
2. 根节点是黑色
3. 叶子节点都是黑色的空节点
4. 红色节点的子节点都是黑色
5. 从任意节点到叶子节点的所有路径包含相同数目的黑色节点

#### 概念

- 概念：红黑树是一种自平衡二叉查找树，需要满足五个性质来保证平衡
- 复杂度：查找、添加和删除都是O(logN)
- 添加删除后需要进行旋转调整操作，但是复杂度是O(1)，因此对于添加和删除的整体复杂度仍然是O(logN)

### 算法

#### 遍历方式

先序遍历：根 - 左 - 右

中序遍历：左 - 根 - 右

后序遍历：左 - 右 - 根

#### 层次遍历

队列版

```java
private void levelTraverse(Queue<TreeNode> queue, List<Integer> res) {
    TreeNode root = queue.poll();
    if (root == null)
        return;
    res.add(root.val);
    if (root.left != null)
        queue.add(root.left);
    if (root.right != null)
        queue.add(root.right);
    levelTraverse(queue, res);
}
```

列表版

```java
 public void levelOrder(TreeNode root, List<List<Integer>> list, int level){
    if(root==null)
        return;
    if(list.size()<=level)
        list.add(new ArrayList<>());
    list.get(level).add(root.val);
    levelOrder(root.left,list,level+1);
    levelOrder(root.right,list,level+1);
}
```



# 设计模式

## 23种设计模式

- 创建型：**单例模式**，**工厂模式**，抽象工厂模式，**建造者模式**，原型模式
- 结构型：**适配器模式**，桥接模式，**组合模式**，**装饰模式**，外观模式，享元模式，**代理模式**
- 行为型：**责任链模式**，**命令者模式**，解释器模式，迭代器模式，中介者模式，备忘录模式，状态模式，**策略模式**，**模板方法模式**，**观察者模式**，访问者模式

## 项目中使用到的设计模式

模式：策略模式+责任链模式+命令者模式+适配器

1. 工厂模式：工厂会根据不同的key创建不同的commander
2. 策略模式：根据不同的应用场景，采用不同的策略，可以使用if/else来实现（**只要代码中有冗长的if-else或者switch判断都可以使用策略模式优化**），但是不利于维护，可以考虑使用策略模式。在本项目中，针对不同的业务流，采用不同的命令链模型，底层是使用Map实现的，key是一个字符串表示是什么样的业务流，例如新建，更新，更换交易对手方，行权等业务流，映射的value是一个命令链，都实现了apache下的commons-chains的Chain方法，然后执行与之对应的命令链。key的具体值是通过flowKeyHelper来确定，大致流程是JMS接受到消息后，根据消息内容来判断key是什么。
3. 责任链模式：为了避免请求发送者与多个请求耦合在一起，将所有请求的处理着根据前一个对象记住下一个对象的引用而形成一条链路，当请求发生时，可以沿着这条链传递，直至处理结束。例如，**报销流程审批系统，拦截器，过滤器**。首先由策略模式确定了使用哪个命令链CommandChain，然后对于命令链的实现首先是责任链，在某一个命令链中一般有几个命令对象组成：例如**存储数据命令，格式转化命令和发布命令**等。这些命令都实现了Command接口，重写execute方法，如果返回Procecssing则继续执行下一个命令，如果返回Complete则结束命令链路，后面的命令不再执行。
4. 命令者模式：命令发起人将命令封装给一个对象发给接受方执行的一种模式，同时也将两者解耦。接收方可以从命令中获取到命令的主要执行内容和需要的数据。在项目中，主要是定义了很多命令类，都实现了Command接口并加入到上述的命令链中。Command接口中的execute方法有一个上下文Context参数，可以获取到之前传入的各种数据，可能会在命令代码中用到。
5. 适配器模式：将一个类的接口通过适配器转化为另外一种期待的接口，从而使得两个系统都连在一起正常工作。在项目中，有两个个适配器的模块，他的主要功能就是将外部的系统发过来的数据转化为内部系统可以识别的一种数据格式。具体实现是创建一个Mapper接口，该接口有两个泛型分别是源数据类型和目标数据类型，有一个map方法，传入的是源数据和上下文，返回值是目标数据。在该接口的具体实现类中经常遇到层级调用情况，对于每一个实现类都可以认为是一种适配器。

# 算法

## 排序算法

参考博客：https://blog.csdn.net/weixin_50886514/article/details/119045154

![img](img\排序算法.png)

### 插入排序

![在这里插入图片描述](img\插入排序演示.gif)

```java
private static void insertSort(int[] arr, int n) {
    for (int i = 1; i < n; i++) {
        int temp = arr[i];
        for (int j = i - 1; j >= 0; j--) {
            if (temp > arr[j]) break;
            arr[j + 1] = arr[j];
            arr[j] = temp;
        }
    }
}
```

### 选择排序

![在这里插入图片描述](img\选择排序.png)

```java
private static void selectSort(int[] arr, int n) {
    for (int i = 0; i < n; i++) {
        int minIndex = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIndex])
                minIndex = j;
        }
        // swap
        if (minIndex != i) {
            int temp = arr[minIndex];
            arr[minIndex] = arr[i];
            arr[i] = temp;
        }
    }
}
```

### 冒泡排序

![在这里插入图片描述](img\冒泡排序.png)

```java
private static void bubbleSort(int[] arr, int n) {
    int temp;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            // swap
            if (arr[j] > arr[j + 1]) {
                temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

### 快速排序

![在这里插入图片描述](img\快速排序.png)

```java
private static void quickSort(int[] arr, int left, int right) {
    if (left >= right)
        return;
    int leftTemp = left;
    int rightTemp = right;
    int key = arr[left];
    while (left < right) {
        while (left < right && arr[right] >= key) {
            right--;
        }
        while (left < right && arr[left] <= key) {
            left++;
        }
        if (left < right) {
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
        }
    }
    if (left == right) {
        arr[leftTemp] = arr[left];
        arr[left] = key;
    }
    quickSort(arr, leftTemp, left - 1);
    quickSort(arr, right + 1, rightTemp);
}
```

### 希尔排序

![在这里插入图片描述](img\希尔排序.png)

```java
private static void shellSort(int[] arr, int n) {
    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = 0; i < gap; i++) {
            for (int j = i + gap; j < n; j += gap) {
                if (arr[j - gap] > arr[j]) {
                    int temp = arr[j - gap];
                    arr[j - gap] = arr[j];
                    arr[j] = temp;
                }
            }
        }
    }
}
```

### 归并排序

分解 - 排序 -合并

![img](img\归并排序.gif)

```java
private static int[] mergeSort(int[] arr, int left, int right) {
    if (left >= right) 
        return new int[]{arr[left]};
    // 分解
    int mid = (left + right) / 2;
    // 排序
    int[] leftArr = mergeSort(arr, left, mid);
    int[] rightArr = mergeSort(arr, mid + 1, right);
    // 合并
    int[] newArr = new int[leftArr.length + rightArr.length];
    int l = 0, r = 0, m=0;
    while (l < leftArr.length && r < rightArr.length) {
        newArr[m++] = leftArr[l] < rightArr[r] ? leftArr[l++] : rightArr[r++];
    }
    while (l < leftArr.length) {
        newArr[m++] = leftArr[l++];
    }
    while (r < rightArr.length) {
        newArr[m++] = rightArr[r++];
    }
    return newArr;
}
```

### 堆排序

总体流程：

1. 构建大顶堆
2. 最后一个节点和第一个节点交换
3. 重新堆化
4. 循环2,3步骤

![在这里插入图片描述](img\堆排序.gif)

1. 先构建一个堆，升序排序构建大顶堆，降序构建小顶堆
   1. 当前节点的左子节点下标为：2i+1，右子节点为2i+2
   2. 当前节点的父节点下标为：(i-1)/2
2. 比较左右子节点和根点，如果发生交换，则需要向下递归根节点，对子树堆化
3. 交换，砍树，重新堆化

```java

private static void heapSort(int[] arr, int n) {
    buildMaxHeap(arr, n);
    for (int i = n - 1; i >= 0; i--) {
        swap(arr, i, 0);
        // 注意最后一个参数不是数组长度，而是当前索引
        // 因为要把排序过的数字排除出去
        heapify(arr, 0, i);
    }
}

// 创建大顶堆
private static void buildMaxHeap(int[] arr, int length) {
    int last = length - 1;
    int lastParent = (last - 1) / 2;
    // 从最后一个一个节点的父节点开始创建
    for (int parent = lastParent; parent >= 0; parent--) {
        heapify(arr, parent, length);
    }
}

// 堆化
// 可以以一个简单的三个节点的二叉树作为例子开始
// 在最后排序完成后再对子节点进行堆化
private static void heapify(int[] arr, int currentIndex, int length) {
    int left = currentIndex * 2 + 1;
    int right = currentIndex * 2 + 2;
    int maxIndex = currentIndex;
    if (left < length && arr[left] > arr[maxIndex])
        maxIndex = left;
    if (right < length && arr[right] > arr[maxIndex])
        maxIndex = right;
    if (maxIndex != currentIndex) {
        swap(arr, currentIndex, maxIndex);
        //对从上面传下来的节点，重新堆化操作
        heapify(arr, maxIndex, length);
    }
}
private static void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

### 桶排序/计数排序

一般用于差距不是特别大的数组排序，否则会出现大量的空间浪费的情况

```java
public static void bucketSort(int[] arr) {
    int max = Integer.MIN_VALUE;
    int min = Integer.MAX_VALUE;
    for (int i = 0; i < arr.length; i++) {
        max = Math.max(max, arr[i]);
        min = Math.min(min, arr[i]);
    }
    //创建桶
    int bucketNum = (max - min) / arr.length + 1;
    ArrayList<ArrayList<Integer>> bucketArr = new ArrayList<>(bucketNum);
    for (int i = 0; i < bucketNum; i++) {
        bucketArr.add(new ArrayList<Integer>());
    }
    //将每个元素放入桶
    for (int i = 0; i < arr.length; i++) {
        int num = (arr[i] - min) / (arr.length);
        bucketArr.get(num).add(arr[i]);
    }
    //对每个桶进行排序
    for (int i = 0; i < bucketArr.size(); i++) {
        Collections.sort(bucketArr.get(i));
    }
    // 赋值回数组
    int num = 0;
    for (ArrayList<Integer> integers : bucketArr) {
        for (Integer integer : integers) {
            arr[num++] = integer;
        }
    }
}
```

### 基数排序

1. 首先取最大值，和最大值的位数
2. 根据个位的数字排序
3. 再根据十位，百位...排序
4. 最后合并所有位数的list

```java
public void sort(int[] arr, int n){
    // 最大值
    int maxNum = theMaxNum();
    // 最大位数
    int maxTime = theMaxTime();
    // 创建十个list
    //建立 10 个队列;
    List<ArrayList> queue=newArrayList<ArrayList>();
    for(int i=0;i<10;i++){
        queue.add(new ArrayList<Integer>());
    }
}
```

## 常用方法

### Map常用方法

1. computeIfAbsent(K, Function<key, new Value>): 当不存在时，执行function获取到value并插入到map中
2. computeIfPresent(K, BiFunction<key, oldValue,newValue>): 当存在时，执行BiFunction获取到新值并替换旧值
3. computeIfPresent(K, BiFunction<key, oldValue,newValue>) ：存在则替换，不存在插入
4. getOrDefault(key,defaultValue)：如果有则取。如果没用则用默认值

### 数组常用方法

- 字符串转化为数字数组

  ```java
  String s = "123456789"; 
  int[] array = s.chars().map(i -> i - 48).toArray();
  ```

- 基本类型数组转化为链表

  ```java
  List<Integer> collect = Arrays.stream(arr).boxed().collect(Collectors.toList());
  ```

- 基本类型链表转数组

  ```java
  list.stream().mapToInt(Integer::intValue).toArray();
  ```

- 求整数数组的最小值	

  ```java
  int min = Arrays.stream(num).min().orElse(0);
  ```

- xx

### 链表常用方法

- 基本类型链表转数组

```java
list.stream().mapToInt(Integer::intValue).toArray();
```

## 剑指Offer

### 3数组中重复的数字

```java
Set<Integer> set = new HashSet<>();

public int findRepeatNumber(int[] nums) {
    int n = nums.length;
    for(int i =0;i<n;i++) {
        if(set.contains(nums[i])) 
            return nums[i];
        set.add(nums[i]);
    }
    return 0;
}
```

### 4.二维数组查找

从左下到右上，当前值小则向右，当前值大向上

```java
public boolean findNumberIn2DArray(int[][] matrix, int target) {
   int m = matrix.length;
    if (m == 0) return false;
    int n = matrix[0].length;
    if (n == 0) return false;
    int i = m - 1;
    int j = 0;
    while (i >= 0 && j < n) {
        if (matrix[i][j] > target) i--;
        else if (matrix[i][j] < target) j++;
        else return true;
    }
    return false;
}
```

### 5.替换空格

```java
public String replaceSpace(String s) {
    return s.replaceAll(" ","%20");
}
```

### 6.反向打印链表

递归版

```java
public int[] reversePrint(ListNode head) {
    List<Integer> list = new ArrayList<>();
    reversePrint(head, list);
    return list.stream().mapToInt(Integer::intValue).toArray();
}

private void reversePrint(ListNode head, List<Integer> list) {
    if (head == null)
        return;
    int val = head.val;
    reversePrint(head.next, list);
    list.add(val);
}
```

链表版

```java
public int[] reversePrint(ListNode head) {
    ListNode temp = null;
    int n=0;
    while(head!=null){
        ListNode linked = new ListNode();
        linked.val=head.val;
        linked.next=temp;
        temp=linked;
        head = head.next;
        n++;
    }
    int[] res = new int[n];
    for(int i=0;i<n;i++){
        res[i]=temp.val;
        temp=temp.next;
    }
    return res;
}
```

### 7.重建二叉树

力扣：https://leetcode.cn/problems/zhong-jian-er-cha-shu-lcof/

```java
Map<Integer, Integer> map = new HashMap<>();

public TreeNode buildTree(int[] preorder, int[] inorder) {
    Map<Integer, Integer> map = new HashMap<>();
    if (preorder.length == 0) return null;
    for (int i = 0; i < preorder.length; i++) {
        map.put(inorder[i], i);
    }
    return buildTree(preorder, 0, preorder.length - 1, map);
}

private TreeNode buildTree(int[] preorder, int root, int right, Map<Integer, Integer> map) {
    int val = preorder[root];
    TreeNode treeNode = new TreeNode(val);
    Integer inorderIndex = map.get(val);
    treeNode.left = root + 1 > inorderIndex ? null : buildTree(preorder, root + 1, inorderIndex - 1, map);
    treeNode.right = inorderIndex + 1 > right ? null : buildTree(preorder, inorderIndex + 1, right, map);
    return treeNode;
}
```



### 9.两个栈实现队列

```java
class CQueue {

    private final Stack<Integer> stack1 = new Stack<>();
    private final Stack<Integer> stack2 = new Stack<>();

    public CQueue() {
    }

    public void appendTail(int value) {
        stack1.push(value);
    }

    public int deleteHead() {
        if (stack2.isEmpty()) {
            if (stack1.isEmpty())
                return -1;
            else {
                while (!stack1.isEmpty()) {
                    stack2.push(stack1.pop());
                }
            }
        }
        return stack2.pop();
    }
}
```

### 10斐波那契数列和青蛙跳台阶

```java
.// 斐波那契数列
public int fib(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    int[] res = new int[n + 1];
    res[0] = 0;
    res[1] = 1;
    for (int i = 2; i <= n; i++) {
        int resNum = res[i - 1] + res[i - 2];
        res[i] = resNum > max ? resNum % max : resNum;
    }
    return res[n];
}
// 青蛙跳台阶
public int numWays(int n) {
    if(n==0) return 1;
    if(n==1) return 1;
    int[] res=new int[n+1];
    res[0]=1;
    res[1]=1;
    for(int i=2;i<=n;i++){
        res[i]=(res[i-1]+res[i-2])%1000000007;
    }
    return res[n];
}
```

### 11最小数字

力扣：https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/

```java
public int minArray(int[] numbers) {
    return Arrays.stream(numbers).min().orElse(0);
}
```

### 12矩阵中的数列

力扣：https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof/

```java
```



## 动态规划

两步：

第一步：列出转移表达式

第二步骤：列出初始条件

### 斐波那序列

```java
private static int fib(int n) {
    if (n <= 0)
        return 0;
    if (n == 1 || n == 2)
        return 1;
    return fib(n - 1) + fib(n - 2);
}
//改进为记忆化搜索
private static int fib2(int n) {
    Map<Integer, Integer> map = new HashMap<>();
    map.put(1, 1);map.put(2, 1);
    for (int i = 3; i <= n; i++) {
        map.put(i, map.get(i - 1) + map.get(i - 2));
    }
    return map.get(n);
}
```

### 爬楼梯

力扣70题：https://leetcode.cn/problems/climbing-stairs/

```java
private static int climbStairs(int n) {
    if (n == 1)
        return 1;
    if (n == 2)
        return 2;
    return climbStairs(n - 1) + climbStairs(n - 2);
}

private static int climbStairs2(int n) {
    if (n == 1)
        return 1;
    if (n == 2)
        return 2;
    Map<Integer, Integer> map = new HashMap<>();
    map.put(1, 1);
    map.put(2, 2);
    for (int i = 3; i <= n; i++) {
        map.put(i, map.get(i - 1) + map.get(i - 2));
    }
    return map.get(n);
}
```

### 三角型最小路径

力扣120：https://leetcode.cn/problems/triangle/

```java
public static int minimumTotal(List<List<Integer>> triangle) {
    for (int i = 1; i < triangle.size(); i++) {
        List<Integer> list = triangle.get(i);
        for (int j = 0; j < list.size(); j++) {
            list.set(j, list.get(j) + min(triangle, i, j));
        }
    }
    return Collections.min(triangle.get(triangle.size() - 1));
}

private static int min(List<List<Integer>> triangle, int i, int j) {
    List<Integer> lastRowList = triangle.get(i - 1);
    // 最左侧的，直接取第一个
    if (j == 0) return lastRowList.get(0);
    // 最右侧的，直接取最后一个
    if (j == lastRowList.size()) return lastRowList.get(lastRowList.size() - 1);
    return Math.min(lastRowList.get(j - 1), lastRowList.get(j));
}
```

### 最小路径和

力扣64: https://leetcode.cn/problems/minimum-path-sum/

```java
public int minPathSum(int[][] grid) {
    for (int i = 0; i < grid.length; i++) {
        int[] row = grid[i];
        for (int j = 0; j < row.length; j++) {
            grid[i][j] = grid[i][j] + min(grid, i, j);
        }
    }
    return grid[grid.length - 1][grid[0].length - 1];
}

public int min(int[][] grid, int row, int column) {
    if (row == 0 && column == 0) return 0;
    if (row == 0) return grid[row][column - 1];
    if (column == 0) return grid[row - 1][column];
    return Math.min(grid[row - 1][column], grid[row][column - 1]);
}
```

### 整数拆分

力扣343:https://leetcode.cn/problems/integer-break/

```JAVA
   // 取F(i)*F(n-i)的最大值，
   // 例如F(63)=Max(F(2)*F(62),F(3)*F(61).....F(31)*F(32))
    public int integerBreak(int n) {
        if (n == 1) return 1;
        if (n == 2) return 1;
        if (n == 3) return 2;
        Map<Integer, Integer> map = new HashMap<>();
        map.put(1, 1);
        map.put(2, 2);
        map.put(3, 3);
        for (int i = 4; i <= n; i++) {
            int max = 0;
            // 这里循环的最大值是i，不是n
            for (int j = 2; j <= i / 2; j++) {
                max = Math.max(max, map.get(j) * map.get(i - j));
            }
            map.put(i, max);
        }
        return map.get(n);
    }
```

### 完全平方数

```java
public int numSquares(int n) {
    int[] arr = new int[n + 1];
    arr[1] = 1;
    for (int i = 2; i <= n; i++) {
        int min = Integer.MAX_VALUE;
        // 例如F(12) = 1 + Min(F(12-1), F(12-4).F(12-9))
        for (int j = 1; j <= Math.sqrt(i); j++) {
            min = Math.min(min, arr[i - j * j]);
        }
        arr[i] = 1 + min;
    }
    return arr[n];
}
```

### 解码方法

力扣91：https://leetcode.cn/problems/decode-ways/

```java
public int numDecodings(String s) {
    if (s == null || s.length() == 0) return 0;
    // 将字符串转化为数字数组
    int[] array = s.chars().map(i -> i - 48).toArray();
    int n = array.length;
    return numDecodings(array, n);
}

private int numDecodings(int[] array, int n) {
    int[] res = new int[n];
    res[0] = array[0] == 0 ? 0 : 1;
    for (int i = 1; i < n; i++) {
        int temp = array[i - 1] * 10 + array[i];
        if (temp == 0 || (temp % 10 == 0) && temp >= 30)
            return 0;
        else if (temp == 10 || temp == 20)
            res[i] = tempValue(i, res);
        else
            res[i] = res[i - 1] + (temp >= 11 && temp <= 26 ? tempValue(i, res) : 0);
    }
    return res[n - 1];
}

private int tempValue(int i, int[] res) {
    if (i - 2 < 0) return 1;
    return res[i - 2];
}
```



### 不同路径

与最小路径相似

```java
public int uniquePaths(int m, int n) {
    int[][] arr = new int[m][n];
    for (int i = 0; i < m; i++) {
        arr[i][0] = 1;
    }
    for (int j = 0; j < n; j++) {
        arr[0][j] = 1;
    }

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            arr[i][j] = arr[i - 1][j] + arr[i][j - 1];
        }
    }
    return arr[m - 1][n - 1];
}
```

### 打家劫舍

力扣198：https://leetcode.cn/problems/house-robber/

```java
public int rob(int[] nums) {
    int n = nums.length;
    int[] res = new int[n + 1];
    res[1] = nums[0];
    for (int i = 2; i <= n; i++) {
        // 如果新加第n户，那么有两种选择
        //      第一种是选择前n-2户的最大值加上第n户的值；
        //      第二种是选择前n-1户的最大值
        res[i] = Math.max(res[i - 2] + nums[i - 1], res[i - 1]);
    }
    return res[n];
}
```

### 打家劫舍II

力扣213：https://leetcode.cn/problems/house-robber-ii/description/

```java
// 将0-n分为两部分，一部分是arr[0...n-1]另外一部分是arr[1...n]
// 求两个数组的最大值
public int rob(int[] nums) {
    int n = nums.length;
    if (n == 1)
        return nums[0];
    int[] res = new int[n];
    res[0] = nums[0];
    res[1] = Math.max(nums[0], nums[1]);
    for (int i = 2; i < n - 1; i++) {
        res[i] = Math.max(res[i - 1], res[i - 2] + nums[i]);
    }

    int[] res2 = new int[n];
    res2[1] = nums[1];
    for (int i = 2; i < n; i++) {
        res2[i] = Math.max(res2[i - 1], res2[i - 2] + nums[i]);
    }
    return Math.max(res[n - 2], res2[n - 1]);
}
```



### 打家劫舍III

力扣337：https://leetcode.cn/problems/house-robber-iii/

递归版本，超出时间限制

```java
public int rob(TreeNode root) {
    return calc(root, true);
}

public int calc(TreeNode root, boolean flag) {
    if (root == null) return 0;
    int tempFalse = calc(root.left, true) + calc(root.right, true);
    int tempTrue = root.val + calc(root.left, false) + calc(root.right, false);
    return flag ? Math.max(tempTrue, tempFalse) : tempFalse;
}
```

循环版本

```java
// f存储当前节点被选中的最大值
private final Map<TreeNode, Integer> f = new HashMap<>();
// g存储当前节点未选中的值
private final Map<TreeNode, Integer> g = new HashMap<>();

public int rob(TreeNode root) {
    dfs(root);
    return Math.max(f.get(root), g.get(root));
}

public void dfs(TreeNode root) {
    if (root == null) return;
    dfs(root.left);
    dfs(root.right);
    f.put(root, root.val + g.getOrDefault(root.left, 0) + g.getOrDefault(root.right, 0));
    g.put(root, Math.max(f.getOrDefault(root.left, 0), g.getOrDefault(root.left, 0)) +
            Math.max(f.getOrDefault(root.right, 0), g.getOrDefault(root.right, 0)));
}
```

### 最佳买卖股票时机含冷冻期

力扣：https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/description/

```java
public int maxProfit(int[] prices) {
    int n = prices.length;
    // 持有股票的最大收益
    int[] f = new int[n];
    // 持有股票冷冻期的最大收益
    int[] g = new int[n];
    // 持有股票非冷冻期的最大收益
    int[] h = new int[n];
    f[0] = -prices[0];
    for (int i = 1; i < n; i++) {
        f[i] = Math.max(f[i - 1], h[i - 1] - prices[i]);
        g[i] = f[i - 1] + prices[i];
        h[i] = Math.max(h[i - 1], g[i-1]);
    }
    return Math.max(g[n - 1], h[n - 1]);
}
```

### 0-1背包问题

递归实现

```java
private static int maxValue(int[] weightArr, int[] valueArr, int[] flag, int num, int weight) {
    if (num < 0) return 0;
    // 不偷第n个东西，f(n,c) = f(n-1,c)
    int noStealNumIndexObj = maxValue(weightArr, valueArr, flag, num - 1, weight);
    if (weight < weightArr[num] || flag[num] == 1)
        return noStealNumIndexObj;
    flag[num] = 1;
    // 偷第n个东西，f(n,c) = f(n,c-w[n])+v[n]
    int stealNumIndexObj = 
        maxValue(weightArr, valueArr, flag, num, weight - weightArr[num]) + valueArr[num];
    flag[num] = 0;
    return Math.max(noStealNumIndexObj, stealNumIndexObj);
}
```

### 最长递增子序列

力扣300：https://leetcode.cn/problems/longest-increasing-subsequence/

```java
public int lengthOfLIS(int[] nums) {
    if (nums == null || nums.length == 0) return 0;
    int n = nums.length;
    int[] res = new int[n];
    res[0] = 1;
    for (int i = 1; i < n; i++) {
        int max = 0;
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i])
                max = Math.max(max, res[j]);
        }
        res[i] = max + 1;
    }
    return IntStream.of(res).boxed().max(Integer::compareTo).get();
}
```

https://leetcode-cn.com/problems/wiggle-subsequence/)

### 最长公共子序列

leetcode.1143 [最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

校验

```java
public int longestCommonSubsequence(String text1, String text2) {
    char[] s1 = text1.toCharArray();
    char[] s2 = text2.toCharArray();
    int m = s1.length;
    int n = s2.length;
    if (m == 0 || n == 0)
        return 0;
    return f1recursion(s1, m - 1, s2, n - 1);
    return f2Loop(s1, m, s2, n);
}
```

递归版：超出时间限制

```java
private int f1recursion(char[] s1, int m, char[] s2, int n) {
    if (m < 0 || n < 0)
        return 0;
    if (s1[m] == s2[n])
        return 1 + f(s1, m - 1, s2, n - 1);
    return Math.max(f(s1, m - 1, s2, n), f(s1, m, s2, n - 1));
}
```

循环版

```java
    public int longestCommonSubsequence(String text1, String text2) {
        char[] s1 = text1.toCharArray();
        char[] s2 = text2.toCharArray();
        int m = s1.length;
        int n = s2.length;
        if (m == 0 || n == 0)
            return 0;
        return f2Loop(s1, m, s2, n);
    }
```
