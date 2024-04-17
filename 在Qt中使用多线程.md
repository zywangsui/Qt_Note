# 1.使用方法
创建用于运行任务的类
创建线程，并在类中添加线程
启动
1. 创建任务类
继承QObject，声明Q_OBJECT。定义一个槽函数，用于线程的执行
code:
class threadhid : public QObject
{
    Q_OBJECT
public:
    explicit threadhid(QObject *parent = nullptr);
    ~threadhid();
  void stopRun(void);   //用于退出
 
public:
   bool m_Run ;      //退出线程的标志位
 
public slots:
  void Receive();    //用于线程循环
 
};
函数实现：
threadhid::threadhid(QObject *parent ): QObject(parent)
{
   
}
threadhid::~threadhid()
{
 
}
void threadhid::stopRun(void)
{
    m_bRun = false;
}
void threadhid::Receive()
{
     
      m_Run = true;
      while(true) {
 
 
          if(!m_Run)
          {
             qDebug()<<"退出";
             break;
          }
  
           QThread::sleep(1);
           qDebug()<<"doing...";
              
      }
 
}

2.1 添加线程启动(计时器启动)
QThread* thread_1 = new QThread;			// 创建类线程
threadhid* Task_Handle = new threadhid();	// 创建任务类
Task_Handle->moveToThread(thread_1);		// 添加到线程中
connect(thread_1, &QThread::finished, Task_Handle, &QObject::deleteLater);	//用信号槽释放线程
 需要注意的是，当使用 QObject::deleteLater() 槽函数时，
 要确保所要删除的对象是在其所属线程的事件循环中被创建的，
 否则 deleteLater() 函数可能不会生效。
thread_1.start();						// 线程启动
{										//创建单次定时器启动线程任务

	timerHid = new QTimer(this);
	connect(timerHid, &QTimer::timeout, Task_Handle, &threadhid::Receive);
	timerHid->setSingleShot(true);
	timerHid->start(1000);
}
2.2 添加线程启动(立即启动)
connect(thread_1, &QThread::started, Task_Handle, &threadhid::Receive); // 连接线程槽
thread_1->start();       //启动
这种属于是立即执行线程的工作函数，但我认为使用计时器更加可靠；建议使用2.1的单次计时器执行线程工作

3. 线程类的基本接口和使用
void quit();		// 良好的退出函数，在事件循环中停止运行
void start();
void terminate();	// 谨慎使用，会导致未处理的异常和内存泄露等情况

isRunning();		// 用于检查线程是否在运行中；
setPriority()		// 设置线程的优先级
 *Qt 支持以下几种优先级：IdlePriority、LowestPriority、LowPriority、NormalPriority、HighPriority、HighestPriority、TimeCriticalPriority。
yieldCurrentThread()// 用于将 CPU 时间让给其他线程。
finished() 信号：当线程结束时会发出该信号，可以通过该信号来实现线程的清理工作。

4. 线程释放
a.释放类内资源，并退出死循环
b.退出事件循环
c.等待线程退出
d.释放内存
    Task_Handle->stopRun(); //先退出死循环 m_Run = false；
    thread_1.quit(); //退出事件循环
    thread_1.wait(); //等待线程退出
   // QThread::terminate()//也可以，但是上面的方式更加温柔
 
    delete  thread_1;
    delete Task_Handle ;

# 2.QThreadPool 概念

1.线程复用：
当任务提交给线程池时，如果有空闲的线程，那么任务会立即在该线程上执行；
如果没有空闲线程，但线程池中的线程数未达到最大限制；那么线程池会创建一个新线程来执行任务，避免频繁开辟线程和销毁线程
2.全局线程池
Qt提供的全局QtThreadpool，通过直接使用创建自己的线程池；
3.任务管理
通过使用qtp::start()方法提交一个QRunnable对象到线程池。
注意QRunnable是一个抽象类，我们需要创建一个类继承自他，重写run()方法来定义任务的具体执行内容
4.线程数控制
通过setmaxthreadcount()设置最大线程数
5.异步执行
不会阻塞主线程，也就是不会阻塞ui界面
6.信号与槽
信号和槽可以和QThreadPool和QRunnable实现线程间的通信和事件处理