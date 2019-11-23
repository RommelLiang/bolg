### 概述
Handler通常用来做子线程和主线程之间的通信，可以毫不客气的说：**有子线程和主线程通讯的地方，就一定有Handler**。
完整的消息处理机制包含四个要素：

1. Message(消息)
2. MessageQueue(消息队列)
3. Looper(消息循环)
4. Handler(发送和处理消息)

Handler的简单用法如下：
```
Handler handler = new Handler(){
    @Override
    public void handleMessage(@NonNull Message msg) {                     
    
        super.handleMessage(msg);
    }
};
Message message = new Message();
handler.sendMessage(message);
```
其工作流程如下图所示：

![](https://user-gold-cdn.xitu.io/2019/11/24/16e990877ca7be67?w=670&h=417&f=png&s=125180)
从发送消息到接收消息的流程概括如下：

1. handler发送消息
2. 消息进入消息队列
3. Looper不断的从消息队列里取出消息
4. 消息发送给handler处理

### 源码分析
我们从UI主线程里的消息机制最开始创建的地方开始分析！也就是`ActivityThread`，我们先看一下`ActivityThread`的`main`方法：
```
 public static void main(String[] args) {
        ...
        //去掉无关代码
        Looper.prepareMainLooper();

        ...
        Looper.loop();

}
```
如果我们在自己的线程是使用Handler，其代码如下：
```
new Thread(){
    @Override
    public void run() {
        Looper.prepare();
        ...
        Looper.loop();
    }
};
```
### Lopper
这里首先调用了Looper的prepareMainLooper静态方法，我们进去追踪一下：
```
public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
        //主线程内只能勋在一个looper，并且由系统创建，永远不要手动调用
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
}

 private static void prepare(boolean quitAllowed) {
        //在当前线程初始化一个looper
        if (sThreadLocal.get() != null) {
            //确保当前线程内只有一个Looper
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
}

private Looper(boolean quitAllowed) {
        //创建消息队列
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
}

static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```
这里补充一下注释里没提到的东西：
`prepareMainLooper`是用来给主线程创建Looper使用的，其本质还是使用`prepare`方法实现。
我们可以通过`getMainLooper`在任何地方获取主线程的looper。
Looper是通过ThreadLocal创建的，它保证了只有在指定线程下才能访问数据，其他线程无法获取其中的数据。Looper做到了线程内唯一且只能由当前线程访问。
在Lopper的构造方法里，创建了一个MessageQueueb，也就是说每一个looper都持有一个消息队列。
在创建完looper需要调用loop方法去启动它。
接下来看一下loop方法：
```
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        // Allow overriding a threshold with a system prop. e.g.
        // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
        final int thresholdOverride =
                SystemProperties.getInt("log.looper."
                        + Process.myUid() + "."
                        + Thread.currentThread().getName()
                        + ".slow", 0);

        boolean slowDeliveryDetected = false;

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            // Make sure the observer won't change while processing a transaction.
            final Observer observer = sObserver;

            final long traceTag = me.mTraceTag;
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            Object token = null;
            if (observer != null) {
                token = observer.messageDispatchStarting();
            }
            long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
            try {
                msg.target.dispatchMessage(msg);
                if (observer != null) {
                    observer.messageDispatched(token, msg);
                }
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } catch (Exception exception) {
                if (observer != null) {
                    observer.dispatchingThrewException(token, msg, exception);
                }
                throw exception;
            } finally {
                ThreadLocalWorkSource.restore(origWorkSource);
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (logSlowDelivery) {
                if (slowDeliveryDetected) {
                    if ((dispatchStart - msg.when) <= 10) {
                        Slog.w(TAG, "Drained");
                        slowDeliveryDetected = false;
                    }
                } else {
                    if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                            msg)) {
                        // Once we write a slow delivery log, suppress until the queue drains.
                        slowDeliveryDetected = true;
                    }
                }
            }
            if (logSlowDispatch) {
                showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```
这个方法有点长，但是核心只有两个，我们重点看它的核心也就是里面的for循环。

* loop里有一个死循环，它不断检查消息队列是否有消息，消息队列里只有在MessageQueue的next方法返回为空的时候才会跳出循环。这里先插个疑问，`如果消息队列里没了消息，会返回null导致loop结束,如果是的话难道每次向空的消息队列插入消息之后都要启动一下吗？`
* 
* 如果消息不为空，则会通过`msg.target.dispatchMessage(msg);`处理分发消息，其中msg就是Message消息，target我们稍后再讲。


接下来我们lopper的其他方法：
退出方法：
```
  //非安全退出，调用直接退出
 public void quit() {
    mQueue.quit(false);
}

//安全退出，等消息队列里的消息执行完毕后退出，调用后消息队列不再接受新的消息
public void quitSafely() {
    mQueue.quit(true);
}
```
mQueue.quit方法我们稍后再讲

### MessageQueue
还记得looper的两个核心吗？我们直接看它的消息队列的next方法：
```
 Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }

```
我嘞个去，又是一个循环。
