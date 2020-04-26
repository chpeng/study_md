

## Mysql 索引

1. 是数据结构
2. 排好序的快速查找的数据结构
3. 目的是提高查询效率



### 适合建索引



![image-20200409211800676](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409211800676.png)



### 不适合建索引

![image-20200409211850063](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409211850063.png)

### explain

![image-20200409213847732](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409213847732.png)

![image-20200409214524193](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409214524193.png)



#### id

* id相同，自上而下的执行
* id不同，id值越大，执行优先级越高

#### select_type 查询类型

![image-20200409215242362](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409215242362.png)

#### table 查询的表

#### type 访问类型排序

**system > const > eq_ref > ref > range > index > all**

##### ![image-20200409221355111](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409221355111.png)

![image-20200409221441003](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409221441003.png)

![image-20200409221533967](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409221533967.png)

![image-20200409221950311](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409221950311.png)

#### possible_keys 访问类型排

显示出查询这个字段可能用到的索引，但不一定被实际使用

#### keys 访问类型排

查询时候，实际使用的索引

#### ref

显示索引的那一列被使用了

#### rows

要查询的行数

#### Extra

![image-20200409230200759](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409230200759.png)

![image-20200409225128716](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409225128716.png)

![image-20200409225637952](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409225637952.png)

![image-20200409230350058](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409230350058.png)

示例

![image-20200409230447814](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409230447814.png)

### 索引优化

* 小表驱动大表
* 被驱动的表要查询的字段加索引

### 索引失效

![image-20200409234306019](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200409234306019.png)

![image-20200410092457610](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200410092457610.png)

![image-20200410093552729](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200410093552729.png)

![image-20200410095015146](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200410095015146.png) 

![image-20200410095035105](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200410095035105.png)



![image-20200410095133126](Mysql%E7%B4%A2%E5%BC%95.assets/image-20200410095133126.png)