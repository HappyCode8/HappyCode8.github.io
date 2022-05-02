# JUC

```java
public enum State {
    /**
     *线程未开始的状态.
     */
    NEW,

    /**
     正在Java虚拟机中执行但是可能会等待来自操作系统的资源比如处理器
     */
    RUNNABLE,

    /**
    看不懂
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     */
    BLOCKED,

    /**
     * 处于waiting状态是由于调用了如下方法之一:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     *
   处于waiting态的线程在等待其它线程的某些操作，比如调了Object.wait()，那就正等待另一个线程调
     Object.notify()，调了Thread.join()那就等另一个线程结束
     */
    WAITING,

    /**
     一个线程处于time_waiting态是由于调用了有超时时间的如下方法之一:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     */
    TERMINATED;
}
```