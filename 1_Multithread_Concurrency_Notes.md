# Java Multithread And Concurrency Notes

### 1. 中断线程（InterruptedException）

Ref: [线程中断](https://www.cnblogs.com/lt132024/p/6438897.html)
[处理 InterruptedException](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html)
[详细分析Java中断机制](http://www.infoq.com/cn/articles/java-interrupt-mechanism)

### 2. 同步加锁机制的缺点：
用synchronized，lock等加锁机制编写代码，容易出错，且在多核CPU上执行所需的成本也比想象的要高，原因：多核CPU的每个处理器内核都有独立的高速缓存。加锁需要这些高速缓存同步运行，然而这又需要在内核间进行较慢的缓存一致性协议通信。

### 3. 

