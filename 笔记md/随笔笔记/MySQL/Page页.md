参考地址：https://mp.weixin.qq.com/s/A7VJSZhkQhIuwI97UaalMg

### 一、InnoDB中的存储结构

<img src="https://gitee.com/liuzihao169/pic/raw/master/image/image-20210415094415699.png" alt="image-20210415094415699" style="zoom:90%; float:left" />

#### 1.表空间（**table space**）

> 表空间（Tablespace）是一个逻辑容器，表空间存储的对象是段，在一个表空间中可以有一个或多个段，但是一个段只能属于一个表空间

#### 2. 段（**segment**）

> 段（Segment）由一个或多个区组成，区在文件系统是一个连续分配的空间（在 InnoDB 中是连续的 64 个页），不过在段中不要求区与区之间是相邻的。**段是数据库中的分配单位**

#### 3.区（**extent**）

> 在 InnoDB 存储引擎中，一个区会分配 64 个连续的页。因为 InnoDB 中的页大小默认是 16KB，所以一个区的大小是 64*16KB=1MB。在任何情况下每个区大小都为1MB.
>
> 默认情况下，InnoDB存储引擎的页大小为16KB，即一个区中有64个连续的页

#### 4.页（**Page**）

> 页是InnoDB存储引擎磁盘管理的最小单位，每个页默认是16KB;
>
> \1. 数据页（B-tree Node)
>
> \2. undo页（undo Log Page）
>
> \3. 系统页 （System Page）
>
> \4. 事物数据页 （Transaction System Page）
>
> \5. 插入缓冲位图页（Insert Buffer Bitmap）
>
> \6. 插入缓冲空闲列表页（Insert Buffer Free List）
>
> \7. 未压缩的二进制大对象页（Uncompressed BLOB Page）
>
> \8. 压缩的二进制大对象页 （compressed BLOB Page）

page的结构如下：

<img src="https://gitee.com/liuzihao169/pic/raw/master/image/image-20210415095309150.png" alt="image-20210415095309150" style="zoom:70%;float:left" />

页目录索引：作用，用来实现二分查找，提高检索效率。实现过程：

a .将所有的记录分成几个组，记录包括最小、最大记录

b.页目录存储每一组的最大记录，这种记录称为“slot”槽

查找时利用二分查找，**先定位槽，再槽内数据定位**



#### 5.行（row）

> InnoDB存储引擎是按行进行存放的，每个页存放的行记录也是有硬性定义的，最多允许存放16KB/2-200，即7992行记录。

#### 

### 二、B+树索引查找数据的过程

命中索引，根据索引逐层检索，找到叶子节点，确定数据页，将数据加载到内存中，页目录中的槽（slot）采用二分查找的方式先找到一个粗略的记录分组，然后再在分组中通过链表遍历的方式查找记录。



