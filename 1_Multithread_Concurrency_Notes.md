# Java Multithread And Concurrency Notes

Ref: [Java中的多线程你只要看这一篇就够了](https://www.cnblogs.com/wxd0108/p/5479442.html)

### 1. 中断线程（InterruptedException）

Ref: [线程中断](https://www.cnblogs.com/lt132024/p/6438897.html)  
[处理 InterruptedException](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html)  
[详细分析Java中断机制](http://www.infoq.com/cn/articles/java-interrupt-mechanism)  

Java中Interrupted线程中断协作机制的用法举例：
```java
public class TestInterrupt {

	public static void main(String[] args) throws InterruptedException {
		Thread t1 = new Thread(() -> {
			while(true) {
				if(Thread.currentThread().isInterrupted()) {
					System.out.println("Interruted !");
					break;
				}
				try {
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					System.out.println("Interruted When Sleep");
					//设置中断状态
					Thread.currentThread().interrupt();
				}
				Thread.yield();
			}
		});
		t1.start();
		Thread.sleep(1000);
		t1.interrupt();
	}

}
```

类似wait()或者sleep()方法由于中断而抛出异常，此时，它会清除中断标记，如果不加处理，那么在下一次循环时，就无法铺货这个中断，**所以这里在异常处理中，在次设置中断标记**。

### 2. 同步加锁机制的缺点：
用synchronized，lock等加锁机制编写代码，容易出错，且在多核CPU上执行所需的成本也比想象的要高，原因：多核CPU的每个处理器内核都有独立的高速缓存。加锁需要这些高速缓存同步运行，然而这又需要在内核间进行较慢的缓存一致性协议通信。

### 3. wait 和 sleep 方法区别：
object.wait()和Thread.sleep()方法都可以让线程等待若干时间。除了wait()可以被唤醒外，另外一个主要区别是wait()方法会释放目标对象的锁，而Thread.sleep()方法不会释放任何资源。（suspend()（@Deprecated）也不会释放对象锁）

### 4. wait和notify必须包含在对应的synchronized语句中调用：
wait和notify必须包含在对应的synchronized语句中调用。wait调用后会进入对应object对象的等待队列，notify时会从队列中随机唤醒一个线程（notifyAll会唤醒等待队列中的所有线程）。
无论是wait或者notify都需要首先获得目标对象的一个监视器（或相应的对象锁），当wait调用后会阻塞在那里**且释放该对象锁**，当线程唤醒后也会首先尝试重新获得该对象锁（或该对象的监视器），若暂时无法获得会继续等待该对象锁，直到获得后才会继续往下执行。


### 5. java命令--jstack 工具
Ref: [java命令--jstack 工具](http://www.cnblogs.com/kongzhongqijing/articles/3630264.html)  
使用jstack命名可以打印系统的线程信息。 可以针对活着的进程做本地的或远程的线程dump;或针对core文件做线程dump。
~$ jps -ml  （**jps可以显示当前系统中所有的Java进程**）
。。。
~$ jstack pid

### 6. 关于Join，wait，notify的一个注意点：

值得注意的一点是：不要在应用程序中，**在Thread对象实例上**使用类似wait()或者notify()等方法，因为这很可能会影响系统API的工作或者被系统API所影响。因为：join()的本质是让调用线程wait()在当前线程对象实例上，当当前线程执行完成后，被等待的线程会在退出前调用notifyAll()通知所有等待在该线程对象实例锁上的线程继续执行。所以使用Thread对象实例上的wait()或notify()方法会与join()方法冲突，因为join()内部就是使用的该Thread对象的内部锁进行的wait的notify。（实战Java高并发程序设计 P50）

### 7. 强烈建议：

强烈建议大家在创建线程和线程组的时候，给它们取一个名字。这样便于以后的调试和查找问题等。

### 8. ThreadLocal相关：

一种编程小技巧： 有时候为了加速垃圾回收，会**特意写出类似obj=null之类的代码**，如果这么做，obj所指向的对象就会更容易地**被垃圾回收器发现，从而加速回收**。

在了解ThreadLocal的内部实现后，自然会引出一个问题。那就是这些变量是维护在Thread类内部的（ThreadLocalMap定义所在类），这也意味着只要线程不退出，对象的引用将一直存在。

当线程退出时，Thread类会进行一些清理工作，其中就包括清理ThreadLocalMap，如一下代码：
```java
/**
 * This method is called by the system to give a Thread
 * a chance to clean up before it actually exits.
 */
private void exit() {
    if (group != null) {
        group.threadTerminated(this);
        group = null;
    }
    /* Aggressively null out all reference fields: see bug 4006245 */
    target = null;
    /* Speed the release of some of these resources */
    threadLocals = null;
    inheritableThreadLocals = null;
    inheritedAccessControlContext = null;
    blocker = null;
    uncaughtExceptionHandler = null;
}
```
但是，如果我们使用线程池，那就意味着当前线程未必会退出（比如固定大小的线程池，线程总是存在）。如果这样，将一些较大的对象设置到ThreadLocal中（他实际保存在线程持有的ThreadLocalMap中），可能会使系统出现内存泄漏的可能（这里作者的意思是：你设置了对象到ThreadLocal中，但是不清理它，在你使用几次后，这个对象也不再有用了，但是它却无法被回收）。

此时，如果你确定不需要这个对象了，希望及时回收对象，最好使用**ThreadLocal.remove()**方法将这个变量从当前线程中移除, 防止内存泄漏。