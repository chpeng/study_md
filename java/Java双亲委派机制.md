## Java类加载器

1. 启动类加载器（Bootstrap ClassLoader）: 加载 jre/lib/rt.jar ,resource.jar,sun.boot.class.path下的jar,只会加载java,javax,sun包下的类
2. 扩展类加载器(Extension ClassLoader): 加载 jre/lib/ext中的class
3. 系统类加载器（Applicaltion ClassLoader）
4. 自定义加载器



## 双亲委派

![image-20200411205145630](Java%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6.assets/image-20200411205145630.png)

## 双亲委派优势

![image-20200411210537918](Java%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6.assets/image-20200411210537918.png)