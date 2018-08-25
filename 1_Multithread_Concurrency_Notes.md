# Java Multithread And Concurrency Notes

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
object.wait()和Thread.sleep()方法都可以让线程等待若干时间。除了wait()可以被唤醒外，另外一个主要区别是wait()方法会释放目标对象的锁，而Thread.sleep()方法不会释放任何资源。

### 4. wait和notify必须包含在对应的synchronized语句中调用：
wait和notify必须包含在对应的synchronized语句中调用。wait调用后会进入对应object对象的等待队列，notify时会从队列中随机唤醒一个线程（notifyAll会唤醒等待队列中的所有线程）。
无论是wait或者notify都需要首先获得目标对象的一个监视器（或相应的对象锁），当wait调用后会阻塞在那里且释放该对象锁，当线程唤醒后也会首先尝试重新获得该对象锁（或该对象的监视器），若暂时无法获得会继续等待该对象锁，直到获得后才会继续往下执行。


### 5. java命令--jstack 工具
Ref: [java命令--jstack 工具](http://www.cnblogs.com/kongzhongqijing/articles/3630264.html)  
使用jstack命名可以打印系统的线程信息。 可以针对活着的进程做本地的或远程的线程dump;或针对core文件做线程dump。
~$ jps -ml
。。。
~$ jstack pid

### 6. 
