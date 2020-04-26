---
layout: post
title: "ThreadLocal用法及源码分析"
author: Guangjie
---
>在查看android中的handler通信机制时，看到了Looper使用了一个ThreadLocal的类成员，当时并没有深入理解ThreadLocal的用法，
今天通过查看源码及相关的分析对ThreadLocal的用法及源码进行总结。

#### 前言

- 一开始我在思考handler的消息通信机制时，构想怎么实现线程间通信的过程，我最初的想法是当在当前线程创建一个handler时，需要在此
之前创建一个本地线程相关的Looper对象及一个MessageQueue对象(因为主线程默认创建了这两个对象所以我们只需要创建handler对象即
可),每个线程都有一个对应的Looper和MessageQueue，然后通过对应线程的handler发送Message到MessageQueue即可，对应的线程通
过不断循环获取MessageQueue中的消息，然后根据Message的target找到对应的handler处理消息即可。


- 总的想法没有错，handler传递消息的机制确实也是这样，这种程度的理解可能只是会用handler来实现线程之间的通信，但是想要知道底层
的具体实现还是需要通过解读源码。在这个过程中有个关键问题，在实现handler通信机制的时候，怎么实现每个线程有对应的Looper及相关
联的MessageQueue，这是问题的关键所在。

- 在读源码之前我自己的所想的解决方案是，使用一个Map保存线程与对应的Looper键值对，如果此线程下得Looper还没创建则为null，这样的
话就可以实现在每个线程都有一个对应的Looper，在当前线程下只需要通过Map即可查找到当前线程的Looper了，貌似这种方法还可以实现想要
的结果，而是感觉还不错，但是源码就是这么设计的吗？或者说设计的没这么具体，那思想应该也差不多吧？查看源码才知道并不是完全按照这样
设计的，废话太多了，接下来看看源码的设计吧！

#### ThreadLocal

- 在我一开始查看源码时就看到Looper中使用了一个类变量ThreadLocal，因为不知道ThreadLocal的用法就直接查看ThreadLocal的源码，根
据Looper对ThreadLocal的调用过程不断查看ThreadLocal的具体实现，一开始查看的是一头雾水，但始终坚持多看了下，最初的简单结论就是
ThreadLocal保存了当前线程下的Looper对象到当前线程下的Map下，我一开始始终不能理解为什么要使用Map啊，使用一个变量不就好了吗，因为
只在当前线程保存一个Looper对象啊，难道还有保存许多其他的？？？

- 后面查看ThreadLocal的用法才知道，之所以这样做的原因。ThreadLocal代表一个线程局部变量，因为一个线程下可能有许多个线程局部变量
所以是使用的是一个Map根据ThreadLocal对象作为key存储的键值对，一开始为什么没明白也想得通了，因为我一开始以为ThreadLocal是android
专门为Looper设计的，看来是我太天真了，ThreadLocal是java.lang包下的类，人家是java里面的基础库。

- 对ThreadLocal的进一步了解才知道，ThreadLocal代表这一个线程本地变量，比如说刚才的Looper，每个线程单独都需要自己的Looper而不是为
线程所共享的，所以需要使用ThreadLocal对象来把Looper放入到当前线程下的Map中，而要当前线程需要访问此ThreadLocal所代表的变量的时候，
通过查找Map就能够获取，当不需要此变量的时候也可通过remove来移除此键值对。

- 对ThreadLocal的理解比较抽象，举个简单理解帮助理解下，比如我们每一个人代表一个线程，我们每一人都有自己的公交卡，公交卡代表着我们刚才
说的Looper，当我们坐公交时我们需要拿出我们自己的公交卡，此时假如我们无法直接拿公交卡，而是需要通过一个机构来获取公交卡，这个结构就代表
一个ThreadLocal对象，这样我们每个人坐公交的时候就通过这个机构拿出属于我们自己的公交进行刷卡了，同理在handler通信机制中，我们每个线程
就可以通过ThreadLocal获取当前线程的Looper对象了，这样理解感觉轻松多了。。。

- 其实对于ThreadLocal的很多是用在线程安全访问的提供，需要线程的隔离，如刚才的Looper我们需要每个线程隔离访问，按照文档提示，一般来说
ThreadLocal设置为private static，也就是说多个线程都可以访问同一个ThreadLocal对象，但是根据此对象获取当前线程下的具体的对象，如刚才
的例子，那个机构就是每个人都可以访问的，而每个人都根据结构获取自己所属的公交卡。网上很多解释说用于解决多线程数据共享问题，我的理解是提供
线程的隔离访问某一个对象，反正是要理解了ThreadLocal的本质，具体用来干什么就看需求了。接下我用代码举个例子来看看ThreadLocal的具体用法：

```
package org.wangguangjie.learn;

/**
 * Created by wangguangjie on 2018/3/18.
 */
public class ThreadLocalTest {

    private static ThreadLocal<Long> mThreadLocal=new ThreadLocal<>();

    public static void main(String[] args){
        GetMyId getMyId=new GetMyId();
        new Thread(getMyId).start();
        try{
            Thread.sleep(2);
            new Thread(getMyId).start();
        }catch (InterruptedException ie){
            ie.printStackTrace();
        }

    }

    static  class GetMyId implements Runnable{

        @Override
        public void run() {
            for(int i=0;i<10;i++) {
                if (mThreadLocal.get() == null) {
                    System.out.println(Thread.currentThread().getName() + " 没有设置Id");
                    //设置当前的ID,对其他线程无影响，从而提供了隔离访问;
                    mThreadLocal.set(Thread.currentThread().getId());
                    System.out.println(Thread.currentThread().getName() + " ID已设置完毕");
                } else {
                    System.out.println(Thread.currentThread().getName() + " ID=" + mThreadLocal.get());
                }
            }
        }

    }
}



```


#### 源码解析
- 首先查看Looper，因为在实现消息通信机制时，首先要创建Looper，在主线程中使用时默认启动已经创建了Looper对象了。那我们来具体
看看Looper的源码

构造器：
```
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
 ```
 可以看到是private修饰的，也就是说无法通过构造器创建，那通过什么创建呢，可以看到源码解释:
 ```
    /**
     * Initialize the current thread as a looper, marking it as an
     * application's main looper. The main looper for your application
     * is created by the Android environment, so you should never need
     * to call this function yourself.  See also: {@link #prepare()}
     */
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }

 ```
 也就是说主线程通过这个方法创建，而其他线程呢？可以看如下方法：
 ```
     /** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

 ```
可以看到其他线程可以通过如下两个方法创建Looper对象，这是方法的重构，也就是说调用者可以输入一个Boolean参数来设置此Looper是否
可以停止，默认是可以可以停止的。通过上面三个方法可以看到，无论是主线程还是子线程创建Looper都不是直接通过构造函数的，而是通过静
态函数都是先判断当前线程是否存在一个Looper对象，如果不存在才创建，如果存在就抛出异常，这就是常见的单例模式，这样做的原因是一个
线程只能有一个Looper用于对消息的处理，只要此线程存在就保证只有一个Looper存在。接下来其实就简单了，因为上面我们已经讲解了如何
使用ThreadLocal了，因为在这里Looper就使用了ThreadLocal的功能，通过创建一个静态ThreadLocal变量，用来使得不同线程隔离访问
Looper对象，通俗讲就是每个线程都有一个Looper了，通过这个静态ThreadLocal变量来访问当前线程下得Looper。Looper重要的方法还有
loop()，看起来方法很复杂，但其实就是通过循环不断取出MessageQueue的Message，然后查看Message是否有回调，如果有回调则直接执行
，如果没有回调则交给Message的target执行，代码如下：

```
    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
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

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
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

- 当Looper创建时，也会创建MessageQueue，MessageQueue代码比较简单，简单来说就是一个存储Message的队列，具体
的代码有时间再慢慢分析.
- Looper和MessageQueue都创建后，想要实现handler消息通信的话，就需要是用目的线程的handler来向它发生消息了，此时需要
创建handler对象，handler对象的创建比较简单，可以选择重写handleMessage方法，也可以选择不重写，而使用一个回调接口来处理接收
到的消息；handler重载了很多构造器，当总的来说在创建handler时，它会关联一个当前线程下得Looper对象，这个Looper对象可以通过
构造器传参实现，如果没有通过构造器传入的话则它会自动调用Looper.myLooper()获取当前线程下的Looper，所以在创建Handler之前需要先
创建Looper对象，否则将会使得该handler无法关联到Looper对象。

- 现在对整个handler实现机制有了全部的了解，那我们在细看看ThreadLocal的源代码吧，虽然前面基本已经把ThreadLocal的实现讲解了，
先看构造器：

```
    /**
     * Creates a thread local variable.
     * @see #withInitial(java.util.function.Supplier)
     */
    public ThreadLocal() {
    }

```
构造器什么也没有，并没有创建Map，所以只是创建了一个线程局部变量；在来看看重要的几个函数：

```
    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }


```

这个函数是获取此局部变量所指代的具体对象，首先是获取当前线程对象，然后再获取当先线程下的ThreadLocalMap, 这个Map对象保存
该线程下得全部局部变量；当Map==null时就创建一个Map，并进行初始化，具体待会儿看setInitialValue()函数；而如果Map不为空
则通过ThreadLocal对象获取相对于的对象，如果不为空则返回，为空则调用setInitialValue()函数；接下我们继续查看setInitialValue()
函数：

```
    /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

```
通过函数我们可以看到，首先对value进行初始化值，其实就是null(不贴代码啦),然后判断map是否创建了，创建了直接赋初值，没创建
就先创建map，同时也把初始传过去，其实就是null啦！！！这也就是ThreadLocal的get()方法的整个过程啦，总的来说就是通过此
ThreadLocal对象来在Map中查找对于的value，如果找打返回，没找到看Map创建没，没创建就创建，但最终都返回null。下面我们再看看
set()方法，其实也很简单，直接看代码比较好理解：
```
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

```
就是把当前的要存的对象和相关联的ThreadLocal对象存储map中，如果map不存在则创建map，最后在看看ThreadLocal中重要的一个方法：
```
        /**
         * Remove the entry for key.
         */
        private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();
                    expungeStaleEntry(i);
                    return;
                }
            }
        }

```
这个方法作用是把Map中的关于此ThreadLocal对象的键值对移除，为什么要移除呢，主要是因为这个键值对保存着对value对象的强引用，这样
垃圾收集器就不回收此对象，很容易造成内存泄露。但值得注意的ThreadLocalMap存储的键值对是以Entery对象存储的，而Entry对象存储的
ThreadLocal是弱引用，也就是说只要垃圾收集器随时可能回收ThreadLocal的内存，这样同样也会防止内存泄露，但是呢有一个问题，假如
ThreadLocal对象被回收了，这样value同样保存着对其关联的对象的强引用，这样如果该线程一直不结束，会不会同样也会造成内存泄露呢，
当然在设计ThreadLocal及ThreadLocalMap的时候设计者也是考虑过的了，我们可以查看源码看看是怎么处理的吧：
```
 /**
         * Set the value associated with key.
         *
         * @param key the thread local object
         * @param value the value to be set
         */
        private void set(ThreadLocal<?> key, Object value) {

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }

```
通过源码我们看到，当key为null，也就是上面我们说的情况，如果key为空，那value还保持着引用啊，那具体的处理我们再看replaceStaleEntry
函数：

```

    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }


        /**
         * Get the entry associated with key.  This method
         * itself handles only the fast path: a direct hit of existing
         * key. It otherwise relays to getEntryAfterMiss.  This is
         * designed to maximize performance for direct hits, in part
         * by making this method readily inlinable.
         *
         * @param  key the thread local object
         * @return the entry associated with key, or null if no such
         */
        private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }


        /**
         * Version of getEntry method for use when key is not found in
         * its direct hash slot.
         *
         * @param  key the thread local object
         * @param  i the table index for key's hash code
         * @param  e the entry at table[i]
         * @return the entry associated with key, or null if no such
         */
        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                if (k == null)
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }




                /**
         * Expunge a stale entry by rehashing any possibly colliding entries
         * lying between staleSlot and the next null slot.  This also expunges
         * any other stale entries encountered before the trailing null.  See
         * Knuth, Section 6.4
         *
         * @param staleSlot index of slot known to have null key
         * @return the index of the next null slot after staleSlot
         * (all between staleSlot and this slot will have been checked
         * for expunging).
         */
        private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // expunge entry at staleSlot
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // Rehash until we encounter null
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }


```
通过上面的源码我们可以看到，当从map中获取一个Entry时，如果找到键值对不为空并且key相等，则直接放回value，但如果不满足
条件的话，则调用getEntryAfterMiss()函数,对key为空的键值对进行擦除，并且不断循环查找，如果键值对为空则继续擦除，最终
返回值为空。

- 本篇博客主要是理顺handler消息机制的原理，并且通过源码查看hander怎样通过Looper、MessageQueue实现线程间的通信，并且
重点分析了ThreadLocal的用法和ThreadLocal的底层实现，深入掌握ThreadLocal的用法与原理。（总的感觉写的不是特别好，等有
时间了再好好修改修改）


下面是我的个人连接，有兴趣的的可以关注我
> - [CSDN](http://blog.csdn.net/wgj13718925364)
 - [博客园](http://www.cnblogs.com/wangguangjie/)
 - [office365:xm649@silently.party](https://www.office.com/1/?auth=2&home=1&from=ShellLogo)
 - [github](https://github.com/wangguangjie)
 - [QQ]()
 <img src="http://ovy9gem9a.bkt.clouddn.com/HIT/QQ.png" class="qq-picture" width="138" align="center">
 - [微信]()
  <img src="http://ovy9gem9a.bkt.clouddn.com/HIT/weixin.png" class="weixin-picture" width="138" align="center">
----------------------







