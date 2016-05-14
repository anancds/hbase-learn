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

Column qualifier添加进Column family是为了提供一批数据的索引。给定一个Column family叫**content**，一个Column qualifier可能叫**content:html**,另外一个Column qualifier可以叫**content:pdf**。虽然Column family在表创建时就已经确定了，但是不同的row之间的Column qualifier会相差非常大。

###Cell

一个cell是有row，Column family，Column qualifier，一个值，一个代表值版本的时间戳组成的。

###Timestamp

伴随着每个值都有一个时间戳，是为了标记值的不同版本。默认情况下，时间戳是数据写入RegionServer的时间，但是当你把数据放入cell时，你可以指定一个不同的时间戳。

##Conceptual View

你可以在Jim R. Wilson的博客[Understanding HBase and BigTable](http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf)读到关于Hbase 数据模型更易于理解的解释。另外一个PDF文档是Amandeep Khurana写的[Introduction to Basic Schema Design](http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf)。

上述两个文档的信息量和以下几节内容差不多，却在理解Hbase schema设计上提供了不一样的视角。

下面是对BigTable论文的第二页的例子的轻微修改。有一个表叫**webtable**，包含了两行(**com.cnn.www**和**com.example.www**)，三个Column family叫**contents**，**anchor**和**people**。在这个例子中，第一行(**com.cnn.www**)中的 **anchor** 包含了两列(**anchor:cssnsi.com**, **anchor:my.look.ca**)，**contents**包含了一列(**contents:html**)。**com.cnn.www**这个row key包含了5个row的5个版本，**com.example.www**包含了row的一个版本。**contents：html**这个Column qualifier包含了一个网页的所有内容。**anchor**的每个qualifier包含了连接到row这个网站的外部链接，伴随着这个锚链接的文本。列族 **people**表示与这个网站相关的人。

>column Names
>
>根据约定，列的名字是Column family和Column qualifier的组合，中间用":"隔开。


|Row Key	|Time Stamp	|ColumnFamily contents	|ColumnFamily anchor|ColumnFamily people|
|---|---|--|--|
|"com.cnn.www"|t9||anchor:cnnsi.com = "CNN"||
|"com.cnn.www"|t8||anchor:my.look.ca = "CNN.com"||
|"com.cnn.www"|t6|contents:html = "<html>…​"|||
|"com.cnn.www"|t5|contents:html = "<html>…​"|||
|"com.cnn.www"|t3|contents:html = "<html>…​"|||

这个表中的空的单元格，并不占用空间，或者说在hbase中根本不存在，这样的设计让Hbase可以很稀疏。表格式的观察Hbase中的数据并不是唯一的方式或者说最精确的方式。下面的例子用多维map来表示表中的信息，这是一种表现数据的模型，并不是很精确。

    {
      "com.cnn.www": {
    	contents: {
      		t6: contents:html: "<html>..."
      		t5: contents:html: "<html>..."
     	 	t3: contents:html: "<html>..."
    }
    anchor: {
      t9: anchor:cnnsi.com = "CNN"
      t8: anchor:my.look.ca = "CNN.com"
    }
    people: {}
      }
      "com.example.www": {
    	contents: {
      		t5: contents:html: "<html>..."
    }
    anchor: {}
    people: {
      t5: people:author: "John Doe"
    }
      }
    }

##Physical View

虽然在概念试图中一个表是由rows表示的稀疏集合，但是在物理试图上，一个表是根据Column family来存储的。一个新的Column qualifier可以在任何时候添加到Column family中。

省略表格。。。

在概念试图中的空的单元格其实根本就不存储，因为请求在列 **contents:html** 的 **t8** 时间戳上的数据将会返回空。相似的，请求在列**anchor:my.look.ca**的**t9**时间戳上的数据将会返回空。然而，如果没有提供时间戳，那么列上最近的数据将会被返回。对于多version的数据，最近的数据也就是第一条数据，因为时间戳的存储是倒序。因此如果没有指定时间戳，那么在row **com.cnn.www**查询 **contents.html**的值将会返回在 **t6**上的值。 **anchor:cnnsi.com**会返回 **t9**上的值， **anchor:my.look.ca**会返回 **t8**上的值。

关于Apache Hbase更多的存储数据的信息，请访问：[regions.arch](https://hbase.apache.org/book.html#regions.arch)。

