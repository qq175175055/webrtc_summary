# rtc_base

## 1. memory

### 1.1 aligned_malloc

* GetRightAlign()：返回ptr后面对其alignment的指针
* AlignedMalloc()/AlignedFree()：分配/释放内存并返回对其alignment的指针

### 1.2 AlignedArray

行对齐alignment的数组，row/col形成一个矩阵

## 2. numerics

### 2.1 ExpFilter

指数级滤波，通常用于平滑带宽预测和丢包预测

* Apply(float exp, float sample)：y(k) = min(alpha_^ exp * y(k-1) + (1 - alpha_^ exp) * sample, max_)

### 2.2 HistogramPercentileCounter

统计数据的百分位

* Add()：添加统计数据
* GetPercentile()：获取参数为百分位以上的数据

### 2.3 MovingMaxCounter

在window_length_ms统计周期内，存储当前周期内的sample的值。在队列samples_中，存储的特点是front的元素最大，back的元素最小。

### 2.4 MovingMedianFilter

window_size统计周期内，返回50%百分位的元素

### 2.5 safe_compare

定义了6种常量表达式：
//   rtc::SafeEq  // ==
//   rtc::SafeNe  // !=
//   rtc::SafeLt  // <
//   rtc::SafeLe  // <=
//   rtc::SafeGt  // >
//   rtc::SafeGe  // >=

### 2.6 SampleCounter/RollingAccumulator

用于统计常见的如：(max最大./avg平均./variance方差)

### 2.7 sequence_number_util

用于rtp包的序号的常见操作，如比较两个序号的前后关系，将序号变成不回绕的值等

## 3. strings

### 3.1 SimpleStringBuilder

用于构造char* 字符串，通常用于打印日志用，也可以用于构造一个格式化的字符串（char*）

### 3.2 stringencode

用于处理类似字符转8进制等

### 3.3 stringutils

字符串辅助类

## 4. synchronization

### 4.1 rw_lock_wrapper

读写锁，通常RWLockWrapper，ReadLockScoped/WriteLockScoped配合使用  
receive_crit_(RWLockWrapper::CreateRWLock());
ReadLockScoped read_lock(*receive_crit_);

### 4.2 CriticalSection/CritScope/TryCritScope

锁

## 5. system

### 5.1 FileWrapper

文件操作的封装类

* 有好几种创建对象的办法：Create(), Open(), FileWrapper构造函数
* 打开关闭文件：OpenFile/OpenFromFileHandle/CloseFile
* 读写：Read/Write

## 6. third_party

### 6.1 base64

base64的常用操作

### 6.2 sigslot

Signal/Slot机制，当signal产生时，调用绑定的slot函数

## 7. arraysize

宏，返回数组大小

## 8. AsyncInvoker/FireAndForgetAsyncClosure/AsyncClosure/Location

函数的异步执行，基本用法：  

AsyncInvoker invoker_;
invoker_.AsyncInvoke<void>(RTC_FROM_HERE, thread, Bind(&MyClass::AnotherAsyncTask, this));  

在Location中定义RTC_FROM_HERE宏

## 9. AsyncPacketSocket/Socket/AsyncSocket/AsyncTCPSocket/AsyncUDPSocket/FirewallSocketServer

socket相关操作类

## 10. AtomicOps

原子操作相关类

## 11. Bind()

将对象的函数转换为函数对象（functor），基础用法如下：  
// Example usage:
//   struct Foo {
//     int Test1() { return 42; }
//     int Test2() const { return 52; }
//     int Test3(int x) { return x*x; }
//     float Test4(int x, float y) { return x + y; }
//   };
//
//   int main() {
//     Foo foo;
//     cout << rtc::Bind(&Foo::Test1, &foo)() << endl;
//     cout << rtc::Bind(&Foo::Test2, &foo)() << endl;
//     cout << rtc::Bind(&Foo::Test3, &foo, 3)() << endl;
//     cout << rtc::Bind(&Foo::Test4, &foo, 7, 8.5f)() << endl;
//   }

## 12. BitBuffer/ByteBuffer/ByteBufferReader/ByteBufferWriter

用来操作bits或者bytes的数据类

## 13. Buffer/BufferQueue/CopyOnWriteBuffer

用来表示uint8_t类型的buffer，以及Buffer的队列BufferQueue  
CopyOnWriteBuffer表示带拷贝的buffer

## 14. Callback0/Callback1/Callback2/Callback3/Callback4

通常用于定义回调函数的类型，基本用法如下：  
// Example:
//   int sqr(int x) { return x * x; }
//   struct AddK {
//     int k;
//     int operator()(int x) const { return x + k; }
//   } add_k = {5};
//
//   Callback1<int, int> my_callback;
//   cout << my_callback.empty() << endl;  // true
//
//   my_callback = Callback1<int, int>(&sqr);
//   cout << my_callback.empty() << endl;  // false
//   cout << my_callback(3) << endl;  // 9
//
//   my_callback = Callback1<int, int>(add_k);
//   cout << my_callback(10) << endl;  // 15
//
//   my_callback = Callback1<int, int>();
//   cout << my_callback.empty() << endl;  // true

## 15. CryptString

通常用于类似密码字符串的应用

## 16. DataRateLimiter/RateLimiter/RateStatistics/RateTracker

用于统计周期时间内，是否达到预定的max值，是否还有空间。例如统计可用带宽等。

RateStatistics统计窗口内的rate

RateTracker根据sample计算rate的值

## 17. Event/OneTimeEvent

事件的处理类

## 18. File/UnixFilesystem/Win32Filesystem

文件处理类

## 19. FileStream/FileRotatingStream/DirectoryIterator

循环日志

## 20. helpers/Random

一些函数，用于帮助更方便的编写代码。  

* InitRandom()初始化随机数种子
* CreateRandomString()返回随机字符串，base64编码
* CreateRandomData()返回随机data字符串
* CreateRandomUuid()返回随机uuid
* CreateRandomId()返回32位无符号整数
* CreateRandomId64()返回64位无符号整数
* CreateRandomNonZeroId()返回大于0的32位无符号整数
* CreateRandomDouble()返回[0.0,1.0)区间的数
* GetNextMovingAverage()返回下一个移动平均数

Random类实现了一个随机类，可以用于获得随机数/高斯分布等随机值

## 21. HttpBase/HttpListenServer

实现基于http的组件的一个状态机模型

## 22. IfAddrsConverter/InterfaceAddress

原生态address接口和内部ip address类之间的转换

## 23. json

json的辅助函数

## 24. logging/FileRotatingLogSink

日志类

## 25. memory_usage

GetProcessResidentSizeBytes()：返回当前进程使用内存

## 26. MessageHandler/FunctorMessageHandler/MessageQueueManager/Message/DelayedMessage/MessageQueue

消息队列

## 27. NATInternalSocketFactory/NATServer/NAT

处理nat

## 28. OpenSSLAdapter/OpenSSLCertificate/OpenSSLDigest/OpenSSLIdentity/OpenSSLStreamAdapter

处理openssl协议

## 29. Pathname

处理文件路径

## 30. PhysicalSocketServer/PhysicalSocket

封装底层os的socket

## 31. platform_file/platform_thread

与底层操作系统相关的文件或者线程处理

## 32. RefCountInterface/RefCountedObject

引用计数器的实现

## 33. scoped_refptr

智能指针，基本用法：  
//   class MyFoo : public RefCounted<MyFoo> {
//    ...
//   };
//
//   void some_function() {
//     scoped_refptr<MyFoo> foo = new MyFoo();
//     foo->Method(param);
//     // |foo| is released when this function returns
//   }
//
//   void some_other_function() {
//     scoped_refptr<MyFoo> foo = new MyFoo();
//     ...
//     foo = nullptr;  // explicitly releases |foo|
//     ...
//     if (foo)
//       foo->Method(param);
//   }  

//   {
//     scoped_refptr<MyFoo> a = new MyFoo();
//     scoped_refptr<MyFoo> b;
//
//     b.swap(a);
//     // now, |b| references the MyFoo object, and |a| references null.
//   }  

//   {
//     scoped_refptr<MyFoo> a = new MyFoo();
//     scoped_refptr<MyFoo> b;
//
//     b = a;
//     // now, |a| and |b| each own a reference to the same MyFoo object.
//   }  

## 34. SequencedTaskChecker/SequencedTaskCheckerImpl

保证某个类的方法在同一个线程中被调用

## 35. SignalThread/Worker/Thread/ThreadManager/Runnable

线程处理类

## 36. StreamInterface/StreamAdapterInterface/FileStream/MemoryStreamBase/MemoryStream/FifoBuffer

流式处理相关类

## 37. StringToNumber

字符串转数字的辅助类

## 38. SwapQueue

producer/consumer可以在不同线程访问的队列

## 39. QueuedTask/ClosureTask/ClosureTaskWithCleanup/TaskQueue

task相关处理类

## 40. timeutils

时间处理辅助类

## 41. WeakReference/WeakReferenceOwner/WeakPtrBase/WeakPtr

实现一个类似c++11的weak_ptr

## 42. win32辅助类/ExplicitZeroMemory

比如获取win的版本号，socket初始化等

ExplicitZeroMemory()：0填充内存

