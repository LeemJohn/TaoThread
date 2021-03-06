对Qt QThread的简单封装。

Qt的线程有两种场景。

1. 如果是比较独立的任务，不需要交互的，直接QThread::run, 或者QRunable, QtConcurrent都可以。

2. 如果是要交互的，就得QObject moveToThread，或者别的什么事件机制的写法。

有交互的情况下，如果用前面的那几种，得自己管事件循环，不然会把UI卡住。

所以我做了一些封装，使线程不会卡住UI，还能像run一样方便。

--

2019/6/13更新 

前面那种方式删掉了，因为有局限，不方便复用。

改成线程池了，更加好用。

直接调用封装好的 ThreadPool::getInstance()->work() 函数即可，传递两个lambda作为参数。
第一个lambda是线程池里运行的任务。需要返回任务结果为true 或者false。
第二个lambda是任务结束后，在主线程执行的任务。带一个bool参数为任务的执行结果。
```C++
ThreadPool::getInstance()->work(
                [id](){
                    qWarning() << QThread::currentThreadId() << "do" << id;
                    return true;
                },
                [this, id](bool result){
                    qWarning() <<QThread::currentThreadId() << "work result:" << result << id;
                    showId();
                });
```
![](preview.png)

--

2019/9/30 更新
ThreadPool没有取消操作，直接杀线程会对ThreadPool造成不良的影响。
为了支持取消操作，封装了Controller-worker模型。杀QThread是可控的。
```C++
taskId = ThreadController::getInstance()->work(
                [id](){
                    qWarning() << QThread::currentThreadId() << "do" << id;
                    return true;
                },
                [this, id](bool result){
                    qWarning() <<QThread::currentThreadId() << "work result:" << result << id;
                    showId();
                });
```
取消操作是
```C++
ThreadController::getInstance()->cancle(taskId);
```

