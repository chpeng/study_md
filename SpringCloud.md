![image-20200310012224707](E:\develop\study_md\SpringCloud.assets\image-20200310012224707.png)

![image-20200310012250091](E:\develop\study_md\SpringCloud.assets\image-20200310012250091.png)

![image-20200310123450621](E:\develop\study_md\SpringCloud.assets\image-20200310123450621.png)

![image-20200311022642417](E:\develop\study_md\SpringCloud.assets\image-20200311022642417.png)

![image-20200311183540719](E:\develop\study_md\SpringCloud.assets\image-20200311183540719.png)

![image-20200311183556330](E:\develop\study_md\SpringCloud.assets\image-20200311183556330.png)

# Eurake

* Eureka server 是SpringCoud的注册中心，当服务提供者启动的时候，会通过Eureka client 向Eureka server 发送注册信息
* 服务消费者 通过Eurake Client 向注册中心拉取服务信息，并保存在本地，在调用服务的时候，选择一个服务地址进行调用
* 服务提供者，每隔30s向注册中心发送心跳

# Ribbon

##  负载均衡策略

* 轮询  
* 随机
* 权重：计算每隔服务器的权重，权重越大，调用的可能性越大
* 可用过滤

## 重试机制

* 设置连接超时时间，如果没有返回，则会请求另外一台服务器

# ZUUL网关

![image-20200311184441556](E:\develop\study_md\SpringCloud.assets\image-20200311184441556.png)

![image-20200311184836038](E:\develop\study_md\SpringCloud.assets\image-20200311184836038.png)

![image-20200311184814509](E:\develop\study_md\SpringCloud.assets\image-20200311184814509.png)



 ![image-20200312124458811](E:\develop\study_md\SpringCloud.assets\image-20200312124458811.png)

![image-20200312124527205](E:\develop\study_md\SpringCloud.assets\image-20200312124527205.png)

![image-20200312130219716](E:\develop\study_md\SpringCloud.assets\image-20200312130219716.png)

