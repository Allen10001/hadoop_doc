# Hadoop专题

1. linux文件权限，当前用户可以修改自己用户密码，在修改用户密码的时候会修改root用户的etc文件，当前用户需要什么权限。

2. linux   /tmp 目录下的文件对所有的用户都是可见的， 同时当前用户可以创建文件，root用户也可以创建文件，在删除的时候只有当前用户和root用户可以删除，为啥。

   

## hadoop源码

项目中怎么打开hadoop源码

如何编译hadoop源码

## hadoop中的Configuration对象是什么有什么用？

> Configuration conf=new Configuration（）；
> 创建一个Configuration对象时，其构造方法会默认加载hadoop中的两个配置文件，分别是hdfs-site.xml以及core-site.xml，这两个文件中会有访问hdfs所需的参数值，主要是fs.default.name，指定了hdfs的地址，有了这个地址客户端就可以通过这个地址访问hdfs了。即可理解为configuration就是hadoop中的配置信息。

集群中的hadoop的一些配置文件和mr代码中的Configuration对象存在什么关系？

## crc 文件的作用

校验和文件，占比不到 原文件大小的1%。

HDFS 会对写入的所有数据计算校验和，并在读取数据时验证校验和。

[HADOOP中的CRC数据校验文件](https://blog.csdn.net/weixin_44388193/article/details/102863673)

>Hadoop 系统为了保证数据的一致性，会对文件生成相应的校验文件(.crc文件)，并在读写的时候进行校验，确保数据的准确性。在本地find -name *.crc
>
>hadoop比较适合做离线处理，这个是众所周知的，而且hdfs为了保证数据的一致性，每次写文件时，针对数据的io.bytes.per.checksum字节，都会创建一个单独的校验和。默认值为512字节，因为crc-32校验是4字节，存储开销小于1%。而客户端读取数据时，默认会验证数据的crc校验和。除此之外，每个数据节点还会在后台线程运行一个数据块检测程序，定期检查存储在数据节点上的所有块。当块和对应的crc校验匹配不上，由于hdfs存储着块的副本，它可以复制正确的副本替换出错的副本.
>
>* hadoop为什么要设计crc校验
>
>既然crc校验对hdfs有这么大的性能损耗，那么hadoop还为什么要用crc校验呢，hadoop设计的应用场景就是离线数据的分布式计算，所以这些数据会保存很久，保存一个月，半年，一年，十年等。而数据保存这么久，那么物理存储介质由于中位衰减，会造成数据损坏，这对一个大文件来说，很容易导致一个块由于时间关系，硬盘错位，最终导致整个文件都是错误的，这对离线处理来说是不可以接受的，所以我想，hadoop就是为了离线处理的应用场景，才设计出crc校验。如果hadoop是做实时处理的，crc校验就没有必要了，毕竟数据不会放多久。

## 分块、分片 InputSplit、record

**注意：每个小文件不会占用一个块的。 **

如何更改 map 输入文件的分隔符。

自定义hadoop map/reduce输入文件切割 InputFormat 更改输入 value 的分隔符.

**一个map只处理一个输入分片 split，每个分片被划分成若干个记录，每个记录就是一个 key/value 对**。输入分片和和记录都是逻辑概念，不必将他们对应到文件。

FileInputFormat 是所有使用文件作为其数据源的 InputFormat 实现的基类，提供两个功能，1. 用于指出输入文件的位置，2. 实现输入文件生成分片功能。把分片分割成记录的作业由其子类来完成。

FileInputFormat 只分割大文件，所谓大是指超过HDFS 块的大小，分片通常与HDFS块的大小相同。

分片大小的计算公式：

```mat
max(minimumSize,min(maximumSize,blockSize))
```

默认情况下，

``` math
minimumSize < blockSize < maximumSize
```

小文件多的时候：1. 每个文件一个分片，使map任务数量过多。2. 增加作业寻址的时间。3.浪费 nameNode的内存。

解决方法：

1. CombineFileInputFormat,将多个文件打包到一个分片中。
2. SequenceFile 将这些小文件合并成一个或者多个大文件。

## 编写分区方法

默认的分区：HashPartitioner 



## MapFile、SequenceFile



## 排序

1. job类设置 setSortComparatorClass() 方法；
2. 定义键实现 WritableComparable 接口， 覆盖 compareTo() 方法。

## shuffle 的原理

mr 确保每个 reducer 的输入都是按键排序的。系统执行排序的过程（map的输出作为输入传给 reducer ）成为shuffle。

## combine 的作用

## 分组查询mr程序



## hive中后台执行的mr



## 看视频中几个重要的章节

## 分区、分组

分区是将有关联的键划分给同一个Reducer处理，多一个分区，也就多一个Reducer，同时也提高了分布式运算的效率，同时将有关联的键放入一个分区也便于以后的管理。

分组是将相同键的值迭代成一个列表，就是将相同的键的记录整理成一组记录。

[HDFS中数据块概念及设置大小的学问](https://blog.csdn.net/liu123641191/article/details/80727462)

>为什么不能远大于64MB？
>原因主要从上层的MapReduce框架来寻找。
>
>（1）Map 崩溃问题。系统需要重新启动，启动过程中需要重新加载数据，数据块越大，数据加载时间越长，系统恢复过程越长.
>
>（2）监管时间问题。主节点监管其他节点的情况，每个节点会周期性的与主节点进行汇报通信。倘若某一个节点保持沉默的时间超过一个预设的时间间隔，主节点会记录这个节点状态为死亡，并将该节点的数据转发给别的节点。而这个“预设时间间隔”是从数据块的角度大致估算的。（加入对64MB的数据块，我可以假设你10分钟之内无论如何也能解决完了吧，超过10分钟还没反应，那我就认为你出故障或已经死了。）64MB大小的数据块，其时间尚可较为精准地估计，如果我将数据块大小设为640MB甚至上G，那这个“预设的时间间隔”便不好估算，估长估短对系统都会造成不必要的损失和资源浪费。
>
>（3）问题分解问题。数据量的大小与问题解决的复杂度呈线性关系。对于同一个算法，处理的数据量越大，时间复杂度越高。
>
>（4）约束Map输出。在Map Reduce框架里，Map之后的数据是要经过排序才执行Reduce操作的。这通常涉及到归并排序，而归并排序的算法思想便是“对小文件进行排序，然后将小文件归并成大文件”，因此“小文件”不宜过大。

## Hadoop Journal Node 作用

* [Hadoop Journal Node 作用](https://my.oschina.net/u/189445/blog/661561)

>两个NameNode为了数据同步，会通过一组称作JournalNodes的独立进程进行相互通信。当active状态的NameNode的命名空间有任何修改时，会告知大部分的JournalNodes进程。standby状态的NameNode有能力读取JNs中的变更信息，并且一直监控edit log的变化，把变化应用于自己的命名空间。standby可以确保在集群出错时，命名空间状态已经完全同步了。
>
>在HA集群中，standby状态的NameNode可以完成checkpoint操作，因此没必要配置Secondary NameNode、CheckpointNode、BackupNode。如果真的配置了，还会报错。



## 问题记录

1.[maven在eclipse建立工程,运行出现Server IPC version 9 cannot communicate with client version 4错误](https://blog.csdn.net/vickyrocker1/article/details/47663615?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

### 2. Parquet与ORC：高性能列式存储格式   ？？？



[Parquet与ORC：高性能列式存储格式](https://blog.csdn.net/yu616568/article/details/51868447)

># Parquet文件格式
>
>## 文件结构
>
>Parquet 文件是以二进制方式存储的，是不可以直接读取和修改的，Parquet 文件是自解析的，文件中包括该文件的数据和元数据。在HDFS文件系统和Parquet文件中存在如下几个概念：
>
>- HDFS块(Block)：它是HDFS上的最小的副本单位，HDFS会把一个Block存储在本地的一个文件并且维护分散在不同的机器上的多个副本，通常情况下一个Block的大小为256M、512M等。
>- HDFS文件(File)：一个HDFS的文件，包括数据和元数据，数据分散存储在多个Block中。
>- **行组(Row Group)：按照行将数据物理上划分为多个单元，每一个行组包含一定的行数，在一个HDFS文件中至少存储一个行组，Parquet读写的时候会将整个行组缓存在内存中，所以如果每一个行组的大小是由内存大的小决定的。**
>- **列块(Column Chunk)：在一个行组中每一列保存在一个列块中，行组中的所有列连续的存储在这个行组文件中。不同的列块可能使用不同的算法进行压缩。**
>- **页(Page)：每一个列块划分为多个页，一个页是最小的编码的单位，在同一个列块的不同页可能使用不同的编码方式。**
>
>通常情况下，在存储Parquet数据的时候会按照HDFS的Block大小设置行组的大小，由于一般情况下每一个Mapper任务处理数据的最小单位是一个Block，这样可以把每一个行组由一个Mapper任务处理，增大任务执行并行度。Parquet文件的格式如下图所示。
>
>![image-20201009163438408](/Users/allen/big_data_learning/hadoop/hadoop_note_img/image-20201009163438408.png)
>
>## 数据访问
>
>说到列式存储的优势，Project下推是无疑最突出的，它**意味着在获取表中原始数据时只需要扫描查询中需要的列，由于每一列的所有值都是连续存储的，避免扫描整个表文件内容。**
>
>在Parquet中原生就支持Project下推，执行查询的时候可以通过Configuration传递需要读取的列的信息，这些列必须是Schema的子集，**Parquet每次会扫描一个Row Group的数据，然后一次性得将该Row Group里所有需要的列的Cloumn Chunk都读取到内存中，每次读取一个Row Group的数据能够大大降低随机读的次数，除此之外，Parquet在读取的时候会考虑列是否连续，如果某些需要的列是存储位置是连续的，那么一次读操作就可以把多个列的数据读取到内存。**
>
>在数据访问的过程中，Parquet还可以利用每一个row group生成的统计信息进行谓词下推，这部分信息包括该Column Chunk的最大值、最小值和空值个数。通过这些统计值和该列的过滤条件可以判断该 Row Group 是否需要扫描。另外Parquet未来还会增加诸如Bloom Filter和Index等优化数据，更加有效的完成谓词下推。
>
># ORC文件格式
>
>ORC文件格式是一种Hadoop生态圈中的列式存储格式，和Parquet类似，它并不是一个单纯的列式存储格式，仍然是首先根据行组分割整个表，在每一个行组内进行按列存储。
>
>## 数据模型
>
>和Parquet不同，ORC原生是不支持嵌套数据格式的，而是通过对复杂数据类型特殊处理的方式实现嵌套格式的支持，例如对于如下的hive表：
>
>```sql
>CREATE TABLE `orcStructTable`(
>  `name` string,
>  `course` struct<course:string,score:int>,
>  `score` map<string,int>,
>  `work_locations` array<string>)
>```
>
>## 文件结构(理解不深)
>
>和Parquet类似，ORC文件也是以二进制方式存储的，所以是不可以直接读取，ORC文件也是自解析的，它包含许多的元数据，这些元数据都是同构ProtoBuffer进行序列化的。
>
>* ORC文件：保存在文件系统上的普通二进制文件，一个ORC文件中可以包含多个stripe，每一个stripe包含多条记录，这些记录按照列进行独立存储，对应到Parquet中的row group的概念。
>
>* 文件级元数据：包括文件的描述信息PostScript、文件meta信息（包括整个文件的统计信息）、所有stripe的信息和文件schema信息。
>
>stripe：一组行形成一个stripe，每次读取文件是以行组为单位的，一般为HDFS的块大小，保存了每一列的索引和数据。
>
>stripe元数据：保存stripe的位置、每一个列的在该stripe的统计信息以及所有的stream类型和位置。
>
>row group：索引的最小单位，一个stripe中包含多个row group，默认为10000个值组成。
>
>stream：一个stream表示文件中一段有效的数据，包括索引和数据两类。索引stream保存每一个row group的位置和统计信息，数据stream包括多种类型的数据，具体需要哪几种是由该列类型和编码方式决定。
>
>![image-20201009172322708](/Users/allen/big_data_learning/hadoop/hadoop_note_img/image-20201009172322708.png)
>
>## 数据访问
>
>读取ORC文件是从尾部开始的，第一次读取16KB的大小，尽可能的将Postscript和Footer数据都读入内存。文件的最后一个字节保存着PostScript的长度，它的长度不会超过256字节，PostScript中保存着整个文件的元数据信息，它包括文件的压缩格式、文件内部每一个压缩块的最大长度(每次分配内存的大小)、Footer长度，以及一些版本信息。在Postscript和Footer之间存储着整个文件的统计信息(上图中未画出)，这部分的统计信息包括每一个stripe中每一列的信息，主要统计成员数、最大值、最小值、是否有空值等。
>
>接下来读取文件的Footer信息，它包含了每一个stripe的长度和偏移量，该文件的schema信息(将schema树按照schema中的编号保存在数组中)、整个文件的统计信息以及每一个row group的行数。
>
>由于ORC中使用了更加精确的索引信息，使得在读取数据时可以指定从任意一行开始读取，更细粒度的统计信息使得读取ORC文件跳过整个row group，**ORC默认会对任何一块数据和索引信息使用ZLIB压缩，因此ORC文件占用的存储空间也更小**，这点在后面的测试对比中也有所印证。
>
>## 性能测试结果分析
>
>从上述测试结果来看，星状模型对于数据分析场景并不是很合适，多个表的join会大大拖慢查询速度，并且不能很好的利用列式存储带来的性能提升，在使用宽表的情况下，列式存储的性能提升明显，ORC文件格式在存储空间上要远优于Text格式，较之于PARQUET格式有一倍的存储空间提升，在导数据（insert into table select 这样的方式）方面ORC格式也要优于PARQUET，在最终的查询性能上可以看到，无论是无嵌套的扁平式宽表，或是一层嵌套表，还是多层嵌套的宽表，两者的查询性能相差不多，较之于Text格式有2到3倍左右的提升。
>
>另外，通过对比场景二和场景三的测试结果，可以发现扁平式的表结构要比嵌套式结构的查询性能有所提升，所以如果选择使用大宽表，则设计宽表的时候尽可能的将表设计的扁平化，减少嵌套数据。
>
>通过这三种文件存储格式的测试对比，ORC文件存储格式无论是在空间存储、导数据速度还是查询速度上表现的都较好一些，并且ORC可以一定程度上支持ACID操作，社区的发展目前也是Hive中比较提倡使用的一种列式存储格式，另外，本次测试主要针对的是Hive引擎，所以不排除存在Hive与ORC的敏感度比PARQUET要高的可能性。







## 公司hadoop集群

206 作为入口机，集群中有认证，只能在打包在集群的206上跑。



