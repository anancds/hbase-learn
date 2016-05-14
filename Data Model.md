## Data Model

在Hbase，数据是存储在具有rows和colums的tables中。这个概念与关系型数据库(RDBMSs)有所重叠，但是这样的类比帮助不大，相反的，把Hbase的表想象成是一个多维的map更有助于理解。

Hbase数据模型术语

### Table

一个Hbase表由多个rows组成。

### Row

Hbase中的Row包含一个row key，和多个有值的columns，Rows在存储时安装row keys的字母表排列。因此，row key的设计就至关重要。这样设计来存储数据的的目的是为了把相关的数据尽量放在一起。一个典型的row key是一个网站域名，如果你的row key是域名，那么你也许应该相反的方式(org.apache.www, org.apache.mail, org.apache.jira)去存储。这样是为了所有的Apache域名都在一个表中紧邻，而不会因为子域名的首字母而分散在表中存储。

###Column

Hbase中的Column由一个Column family和一个Column qualifier组成，它们用":"隔开。

###Column Family

为了性能上的考虑，Column Family物理上并列的存储一系列的Columns和它们的值。每一个Column family有一系列的存储属性，比如它的值是否应该存储在内存中，怎么压缩它的数据，或者怎么编码row key等等。每一个表中的row都有同样的Column family，即使一个row在Column family没有存储任何数据。

###Column qualifier

Column qualifier添加进Column family是为了提供一批数据的索引。给定一个Column family叫**content**，一个Column qualifier可能叫**content:html**,另外一个Column qualifier可以叫**content:pdf**。

###Cell

一个cell是有row，Column family，Column qualifier，一个值，一个代表值版本的时间戳组成的。

