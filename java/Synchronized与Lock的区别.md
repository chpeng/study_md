## synchronize和Lock的区别

1. synchronize 是jdk关键字，是通过字节monitorentor和monitorexit两个字节指令实现的
   lock是jdk5以后的普通类
2. synchronize 不需要手动释放锁，lock需要手动释放锁 unlock
3. synchronize 在执行的过程中，不能被中断
   lock 可以通过tryLock(timeout) 和lockinterruptituy() 实现中断
4. synchronize是非公平锁，lock可以实现公平锁
5. lock可以绑定多个条件condition，实现精确唤醒