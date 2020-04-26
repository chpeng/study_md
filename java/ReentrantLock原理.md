## ReentrantLock

### 原理

1. 通过cas 对一个 volatile 修饰int类型state进行修改值，如果修改成功，则表示修改成功的线程获取到锁了
2. 如果修改失败，则修改失败的线程会进入队列，并将线程pack(LockSuport.pack()),等待获取锁的线程处理完成后，唤醒
3. 当获取锁的线程逻辑处理完成后，则会修改则个volatile修饰的状态 -1 ，如果是0，则是当前没有人获取到锁，并从队列中取出一个任务，将其唤醒
4. 被唤醒的线程，继续通过cas修改状态，修改成功，则表示抢到锁，继续执行线程的业务逻辑

### 代码实现

#### 公平锁实现

##### 1. 获取锁的过程tryAcquire(arg)

1. 获取锁的状态变量，

2. 如果为0表示当前是无锁状态，判断是否需要进入等待队列

   + 如果不需要进入等待队列，则通过cas对锁状态进行修改，修改成功，则表示获取到锁，并将AQS的持有持有当前线程的属性设置成当前线程,继续执行锁以后的业务逻辑
   + 如果等待队列中已经有任务在等待，则将改任务加入到队列末尾

3.  如果不为0，则判断当前线程是否为持有锁的线程，如果是则是重入锁，否则返回false后续进入等待队列，并挂起

4. 判断是否需要进入等待队列源码如下：

   在AQS

   ``` java
    public final boolean hasQueuedPredecessors() {
   /**
   head属性，保存的是队列的头部元素
   tail属性，保存的是队列的尾部元素
   如果是第一次进入hasQueuedPredecessors 方法，则tail和head都是null，只有在任务需要进入队列后才会初始化
   1. 如果是第一次进入hasQueuedPredecessors 
   2. h != t 表示队列中的头部node和队列中的tail节点不相同，说明队列中至少有一个元素
   3. 如果 h.next == null 说明队列中已经没有要排队的额节点了
   4.  s.thread != Thread.currentThread()，是为了防止被线程唤醒的时候，再次进入队列等待
   	原因：
   		当被唤醒的线程执行hasQueuedPredecessors方法的时候，这时候的head还是原来持有锁的对象
   		队列中对一个等待的线程即h.next就是被唤醒线程封装的node
   		如果没有这个判断s.thread != Thread.currentThread()，则唤醒的线程有会进入等待方法
   		则程序有陷入了死循环了
   */
           Node t = tail;
           Node h = head;
           Node s;
           return h != t &&
               ((s = h.next) == null || s.thread != Thread.currentThread());
       }
   ```

   

##### 2. 获取失败加入等待队列并挂起

1. 在获取锁失败的时候，则tryAcquire会返回false

2. 这时候，程序会执行`addWaiter(Node.EXCLUSIVE)`方法，这个方法就是将任务放入队尾

   1. 先创建一个线程的封装对象Node

   2. 如果当前AQS的tail元素不为空，则将新创建的node的prev属性指向，这个tail

   3. 并通过cas将tail设置成新创建的node,然后将原来tail的next属性指向新创建的node，并返回新创建的node

   4. 如果原来‘的tail为空，则会执行enq(node)

   5. 源码如下：

      ```java
      private Node addWaiter(Node mode) {
              Node node = new Node(Thread.currentThread(), mode);
              Node pred = tail;
              if (pred != null) {
                  node.prev = pred;
                  if (compareAndSetTail(pred, node)) {
                      pred.next = node;
                      return node;
                  }
              }
              enq(node);
              return node;
          }
      
      ```

      

3. enq(步骤)，这里面是一个for死循环

   1. 如果当前队列还没有创建，即head和tail都为空，则会创建一个空的节点node,并将tail和head都设置成空节点，在次进入循环
   2. 在次进入循环的时候，这个时候，head和tail都不为空，将封装线程的node放到队尾（head的next=node,node.prev=head）

   源码如下：

   ```java
   private Node enq(final Node node) {
           for (;;) {
               Node t = tail;
               if (t == null) { // Must initialize
                   if (compareAndSetHead(new Node()))
                       tail = head;
               } else {
                   node.prev = t;
                   if (compareAndSetTail(t, node)) {
                       t.next = node;
                       return t;
                   }
               }
           }
       }
   ```

4. 当执行完`addWaiter`后，封装新线程的队列就进入队尾了

5. 然后执行acquireQueued（）方法

   1. 这个方法类似一个自旋操作
   2. 先判断当前node的前一个元素是否为head,如果为head,则是说明下一次要唤醒的是自己，就再次执行一次tryAcquire,如果成功，则表示head已经释放锁了，如果失败，则需要执行shouldParkAfterFailedAcquire方法有三个作用：
      1.  若pred.waitStatus状态位大于0，说明这个节点已经取消了获取锁的操作，doWhile循环会递归删除掉这些放弃获取锁的节点。
      2. 若状态位不为Node.SIGNAL,且没有取消操作，则会尝试将状态位修改为Node.SIGNAL。
      3. 状态位是Node.SIGNAL，表明线程是否已经准备好被阻塞并等待唤醒。
   3. 线程会阻塞在parkAndCheckInterrupt())方法中，等待被唤醒

##### 3. 获取锁成功，执行完成业务逻辑后，释放锁

1. 获取锁的线程在执行完业务逻辑和，调用lock.unlock()
2. 会调用release()方法
   1. 首先调用tryRelease()方法，将多状态设置成state -1 (如果不是重入锁，则state设置后为0)
   2. 调用唤醒线程的方法unparkSuccessor
   3. 取出head的next的node,并将node封装的线程唤醒
3. 在队列中的线程等待在在parkAndCheckInterrupt防止中，被唤醒后，返回线程是否为中断状态

唤醒的线程继续执行循环，再次尝试获取锁，成功后将head设置成抢到锁的线程，并返回，后续继续执行业务逻辑

## ReentrantLoc Condition

1. 通过lock.newCondition()方法创建一个新的ConditionObject()对象

2. ConditionObject对象有两个属性，用于保存同一把锁的第一个条件和最后一个唤醒条件

   ```java
   
   private transient Node firstWaiter;
   private transient Node lastWaiter;
   ```

3. 当conditionObject调用await()方法后，会创建一个node对象，并放到队尾









