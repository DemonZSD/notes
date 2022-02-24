# mysql 

本章节内容主要是针对InnoDB存储引擎进行介绍，其他存储引擎，例如MyISAM等不在此章节讨论范围内。

## 缓冲池 vs 数据页 vs 表空间 vs 段 vs 区 vs 数据行

### 缓冲池（Buffer pool）
内存


### 表空间 vs 段 vs 区 vs 页 vs 数据行

表空间、段、区、页、数据行均是占用的磁盘空间，如下图所示，说明了表空间 vs 段 vs 区 vs 页 之间的关系。

![表空间、段、区、页、数据行](./4183291211-3b3d40e37b3967ab_fix732.png)


**数据行**：从上图可见，

**行格式 vs 文件格式**

MySQL 5.1 以后的版本，支持2中文件格式`Antelope`（羚羊）、`Barracuda`（梭鱼）。4种行格式：`REDUNDANT`、`COMPACT`、`DYNAMIC`、`COMPRESSED`。

一个表可以指定一个行是以什么样的格式进行存储，例如如下代码：

```sql
CREATE TABLE customer (
name VARCHAR(10) NOT NULL, address VARCHAR(20),
gender CHAR(1),
job VARCHAR(30),
school VARCHAR(50)
) ROW_FORMAT=COMPACT;
```

每种文件格式又支持一种或者多种行格式，如下表所示：

|行格式	|紧凑的存储特性	|增强的可变长度列存储	|大索引键前缀支持|	压缩支持|	支持的表空间类型|	所需文件格式|
|-----|-----|-----|-----|-----|-----|-----|
|REDUNDANT|	不|	不|	不|	不|	系统，每表文件，一般|	羚羊或梭鱼|
|COMPACT|	是的|	不|	不	|不|	系统，每表文件，一般	|羚羊或梭鱼|
|DYNAMIC|	是的|	是的|	是的|	不|	系统，每表文件，一般|	梭鱼|
|COMPRESSED|	是的|	是的|	是的|	是的|	每表文件，一般|	梭鱼|

参考文章: [《数据行结构和行溢出机制》](https://alsritter.icu/posts/d2ad62f9/)

**数据页**：数据库的最小单位是数据页，但数据页里不是一行一行的数据，其实一个数据页包含了下面的部分：文件头，数据页头，最大最小记录，多个数据行和空闲区域，最后是数据页目录和文件尾部。



**表空间**：为磁盘空间一组逻辑上的空间区域。 
