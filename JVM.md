# JVM

## 字节码

* java编译后，针对变量，会在当前操作栈中，按顺序生成一个局部变量表，用于记录变量的位置，在后对变量进行操作时候，就是对变量的位置进行操作，位置就代表变量

  ``` java
  public class JavaPTest {
  
      public static void main(String[] args) {
          //【i】 在第一个位置，【k】在局部变量表中第【10】位，【t】在局部变量表中第【19】位
          int i = 10,a,b,c,d,e,f,j,h,k=0,l,m,n,o,p,q,r,s,t;
          i = i + 1;
          t = 1;
          t++;
          k++;
          System.out.println(t);
      }
  
  }
  ```

* javap -c 编译出来的字节码

  ```
  Compiled from "JavaPTest.java"
  public class cn.chpeng.study.jvm.JavaPTest {
    public cn.chpeng.study.jvm.JavaPTest();
      Code:
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
  
    public static void main(java.lang.String[]);
      Code:
         0: bipush        10		//将一个8位带符号整数压入栈
         2: istore_1				//将int类型的值存入局部变量1
         3: iconst_0				//将int类型0压入栈
         4: istore        10		//将int类型存入局部变量10
         6: iload_1				//从局部变量1中装载int数据
         7: iconst_1				//将int类型1压入栈
         8: iadd					//执行整型加法操作
         9: istore_1				//将整型数据存入局部变量1
        10: iconst_1				//将整型1压入栈
        11: istore        19		//将整型存入局部变量19
        13: iinc          19, 1	//把一个常量值加到一个int类型的局部变量上（把局部变量位置是19的整型数据+1）
        16: iinc          10, 1	//把一个常量值加到一个int类型的局部变量上（把局部变量位置是10的整型数据+1）
        19: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;    // 从类中获取静态字段
        22: iload         19		//从局部变量19中装载int数据
        24: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V  //调度对象的实例方法：invokevirtual
        27: return
  }
  
  ```

## JVM优化策略
* 尽量减少Full GC 的次数
  根据业务需要，进行分析，了解对象的特定，进行合理的设置，在java虚拟机中，年轻代和老年代的大小默认比是 1:2 ，eden : survivor0 : survivor1 = 8 : 1 : 1
+ 这个策略就是尽量减少GC的次数,就是说垃圾对象，尽量在MinGC中清除掉，也就是说，尽量减少垃圾对象进入老年代的存储空间
  + 对象进入老年代的的场景
  
  - 长期存活的对象(`-XX:MaxTenuringThreshold`)， 晋升年龄最大阈值，默认15。在新生代中对象存活次数(经过YGC的次数)后仍然存活，就会晋升到老年代。
  
  - 动态对象年龄判定（`-XX:TargetSurvivorRatio`）,对同一GC年龄的内存大小约survivor的比例 默认一半，即survivor区对象目标使用率为50%。
  
  - 对象大于`-XX:PretenureSizeThreshold` 配置的值，会直接进入老年代 
  + 所以优化策略就是
    根据老年到的场景，动态年龄判断对象，和大对象，是朝生夕死的对象，那就应该尽量不免进入老年代，因为进入老年代，需要FullGC才能被回收，尽量做到在MinGC的时候，进行回收，这样就可以减少Full Gc的次数从而提高系统性能，方式是 可以提高退年轻代的大小，从而减少这种对象进入老年代 
* 尽量减少GC时间
* 选择合适的垃圾处理器并发回收

## JVM 垃圾回收器

+ serial 收集器 这个是单线程的年轻代垃圾回收器，使用的是复制算法
+ preNew 收集器 这个是多线程的 年轻代 垃圾回收器，使用复制算法
+ serialOld 老年代 使用的是标记整理算法
+ prallel 老年代，使用的是标记整理算法
+ CMS 使用的是标记清除算法
+ G1分代收集，将java堆划分为大小相等的多个区域（region）,划分出回收优先级





