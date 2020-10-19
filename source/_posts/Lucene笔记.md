---
title: Lucene笔记
date: 2020-02-11 23:55:36
tags:
categories: 搜索引擎
---

# lucene学习笔记

整理自：

<!--more-->

## 基本原理

数据的种类：

- 结构化数据：具有固定格式或固定长度，如数据库、元数据等
- 非结构化数据：不具有固定格式，如邮件,word；又称全文数据

搜索的种类：

- 结构化数据搜索：SQL语句，windows对于文件名、类型、修改时间的搜索
- 非结构化数据搜索：windows搜索文件内容，grep命令，google、百度等

全文检索步骤：

1. 索引创建
2. 搜索索引

![image-20200211235925983](https://tva1.sinaimg.cn/large/0082zybply1gbswl1vqelj31em0emdxc.jpg)

从字符串到文件的映射，称为反向索引

如图，左边是词典，每个字符串都指向包含此字符串的文件的链表，称为倒排表

全文搜索优于顺序扫描的原因：一次索引，多次使用

### 索引创建步骤

1. 需要索引的原文档
2. 将文档传递给分词（Tokenizer）组件：分成一个一个的单词，去除标点符号，去除停用词；经过分词后的结果叫做词元（Token）
3. 将词元传递给语言处理组件（Linguistic Processor）：
   对于英文来说，包括转小写、将单词缩为词根（复数变单数）、将单词转为词根（过去式变原型）；得到的结果叫做词（Term）
4. 将词传递给索引组件（Indexer）:创建词典、排序、合并相同的词组成文档倒排链表（Posting List）

![image-20200212001356895](https://tva1.sinaimg.cn/large/0082zybply1gbsx03yu31j316q0t0kgy.jpg)

## 总体架构

• Lucene的analysis模块主要负责词法分析及语言处理而形成Term。

• Lucene的index模块主要负责索引的创建，里面有IndexWriter。

• Lucene的store模块主要负责索引的读写。

• Lucene的QueryParser主要负责语法分析。

• Lucene的search模块主要负责对索引的搜索。

• Lucene的similarity模块主要负责对相关性打分的实现。

Index->Segment->Document->Field->Term

## 文件格式

### 基本概念

正向信息：索引(Index) –> 段(segment) –> 文档(Document) –> 域(Field) –> 词(Term)

相应文件：

- segments_N保存了此索引包含多少个段，每个段包含多少篇文档。
- XXX.fnm保存了此段包含了多少个域，每个域的名称及索引方式。
- XXX.fdx，XXX.fdt保存了此段包含的所有文档，每篇文档包含了多少域，每个域保存了那些信 息。
- XXX.tvx，XXX.tvd，XXX.tvf保存了此段包含多少文档，每篇文档包含了多少域，每个域包含了 多少词，每个词的字符串，位置等信息。

反向信息：词(Term) –> 文档(Document)

相应文件：

- XXX.tis，XXX.tii保存了词典(Term Dictionary)，也即此段包含的所有的词按字典顺序的排序。
- XXX.frq保存了倒排表，也即包含每个词的文档ID列表。
- XXX.prx保存了倒排表中每个词在包含此词的文档中的位置。

### 文件类型

- Byte：8位
- UInt32：4个Byte
- UInt64：8个Byte
- VInt：
  - 变长整形，包含多个Byte，每个Byte后7位代表数值，最高1位表示是否还有另一个Byte，0代表没有，1代表有
  - 前面的Byte代表低位，后面的Byte代表高位
- Chars：UTF8编码的一系列Byte
- String：一个字符串首先是一个VInt来表示此字符串包含的字符的个数，接着便是UTF-8编码的字符序 列Chars。

### 基本规则

#### Prefix+Suffix规则

要存储的词：term，termagancy，termagant，terminal

普通方式：`[VInt = 4] [t][e][r][m]，[VInt = 10][t][e][r][m][a][g][a][n][c][y]，[VInt = 9][t][e][r][m][a][g][a][n][t]，[VInt= 8][t][e][r][m][i][n][a][l]`，共35个Byte

应用规则：`[VInt = 4] [t][e][r][m]，[VInt = 4 (offset)][VInt = 6][a][g][a][n][c][y]，[VInt = 8 (offset)][VInt = 1][t]，[VInt = 4(offset)][VInt = 4][i][n][a][l]`，共22个Byte

#### Delta规则

要存储如下整数：16386，16387，16388，16389

普通方式：`[(1) 000, 0010][(1) 000, 0000][(0) 000, 0001]，[(1) 000, 0011][(1) 000, 0000][(0) 000, 0001]，[(1) 000, 0100][(1) 000, 0000][(0) 000, 0001]，[(1) 000, 0101][(1) 000, 0000][(0) 000, 0001]`，共12个Byte

应用规则：`[(1) 000, 0010][(1) 000, 0000][(0) 000, 0001]，[(0) 000, 0001]，[(0) 000, 0001]，[(0) 000, 0001]`，共6个Byte

#### 跟随规则（A,B?）

#### 跳跃表规则（Skip List）

- 元素是按顺序排列的，在Lucene中，或是按字典顺序排列，或是按从小到大顺序排列。
- 跳跃是有间隔的(Interval)，也即每次跳跃的元素数，间隔是事先配置好的，如图跳跃表的间隔为3。
- 跳跃表是由层次的(level)，每一层的每隔指定间隔的元素构成上一层，如图跳跃表共有2层。

![image-20200212011032614](https://tva1.sinaimg.cn/large/0082zybply1gbsymzl158j31cw0dkqa0.jpg)

### 具体格式

#### 正向信息

Index –> Segments (segments.gen, segments_N) –> Field(fnm, fdx, fdt) –> Term (tvx, tvd, tvf)

##### segments_N文件格式

![[图]segments_N格式](https://tva1.sinaimg.cn/large/0082zybply1gbtoypoinnj30sk05twfh.jpg)

- Format：索引文件格式的版本号
- Version：索引的版本号，记录了IndexWriter将修改提交到索引文件中的次数。其初始值大多数情况下从索引文件里面读出，仅仅在索引开始创建的时候，被赋予当前的时间，已取得一个唯一值。
- NameCounter：是下一个新段(Segment)的段名。
- SegCount：段(Segment)的个数。
- Seg01的元数据信息：
  - SegName：段名
  - SegSize：此段中包含的文档数。然而此文档数是包括已经删除，又没有optimize的文档的，因为在optimize之前，Lucene的段中包含了所有被索引过的文档，而被删除的文档是保存在.del文件中的，在搜索的过程中，是先从段中读到了被删除的文档，然后再用.del中的标志，将这篇文档过滤掉。
  - DelGen：.del文件的版本号。Lucene中，在optimize之前，删除的文档是保存在.del文件中的。
  - DocStoreOffset
  - DocStoreSegment
  - DocStoreIsCompoundFile：对于域(Stored Field)和词向量(Term Vector)的存储可以有不同的方式，即可以每个段(Segment)单独存储自己的域和词向量信息，也可以多个段共享域和词向量，把它们存储到一个段中去。如果DocStoreOffset为-1，则此段单独存储自己的域和词向量，从存储文件上来看，如果此段段名为XXX，则此段有自己的XXX.fdt，XXX.fdx，XXX.tvf，XXX.tvd，XXX.tvx文件。DocStoreSegment和DocStoreIsCompoundFile在此处不被保存。如果DocStoreOffset不为-1，则DocStoreSegment保存了共享的段的名字，比如为YYY，DocStoreOffset则为此段的域及词向量信息在共享段中的偏移量。则此段没有自己的XXX.fdt，XXX.fdx，XXX.tvf，XXX.tvd，XXX.tvx文件，而是将信息存放在共享段的YYY.fdt，YYY.fdx，YYY.tvf，YYY.tvd，YYY.tvx文件中。
  - HasSingleNormFile：在搜索的过程中，标准化因子(Normalization Factor)会影响文档最后的评分。不同的文档重要性不同，不同的域重要性也不同。因而每个文档的每个域都可以有自己的标准化因子。如果HasSingleNormFile为1，则所有的标准化因子都是存在.nrm文件中的。如果HasSingleNormFile不是1，则每个域都有自己的标准化因子文件.fN
  - NumField：域的数量
  - NormGen：如果每个域有自己的标准化因子文件，则此数组描述了每个标准化因子文件的版本号，也即.fN的N。
  - IsCompoundFile：是否保存为复合文件，也即把同一个段中的文件按照一定格式，保存在一个文件当中，这样可以减少每次打开文件的个数。
  - DeletionCount：记录了此段中删除的文档的数目。
  - HasProx：如果至少有一个段omitTf为false，也即词频(term freqency)需要被保存，则HasProx为1，否则为0。
  - Diagnostics：调试信息。
- User map data：保存了用户从字符串到字符串的映射Map
- CheckSum：此文件segment_N的校验和。

##### 域(Field)的元数据信息(.fnm)

![[图]域的元数据信息](https://tva1.sinaimg.cn/large/0082zybply1gbtpjq8j2gj30gs09zgmq.jpg)

- FNMVersion：是fnm文件的版本号，对于Lucene 2.9为-2
- FieldsCount：域的数目
- Field0：
  - FieldName：域名，如"title"，"modified"，"content"等。
  - FieldBits：一系列标志位，表明对此域的索引方式。最低位：1表示此域被索引，0则不被索引。所谓被索引，也即放到倒排表中去。倒数第二位：1表示保存词向量，0为不保存词向量。倒数第三位：1表示在词向量中保存位置信息。倒数第四位：1表示在词向量中保存偏移量信息。倒数第五位：1表示不保存标准化因子。倒数第六位：是否保存payload



注意：

- 位置(Position)和偏移量(Offset)的区别：位置是基于词Term的，偏移量是基于字母或汉字的。
- 索引域(Indexed)和存储域(Stored)的区别：一个域为什么会被存储(store)而不被索引(Index)呢？在一个文档中的所有信息中，有这样一部分信息，可能不想被索引从而可以搜索到，但是当这个文档由于其他的信息被搜索到时，可以同其他信息一同返回。
- payload的使用：存储每个文档都有的信息：比如有的时候，我们想给每个文档赋一个我们自己的文档号，而不是用Lucene自己的文档号。于是我们可以声明一个特殊的域(Field)"_ID"和特殊的词(Term)"_ID"，使得每篇文档都包含词"_ID"，于是在词"_ID"的倒排表里面对于每篇文档又有一项，每一项都有一个payload，于是我们可以在payload里面保存我们自己的文档号。每当我们得到一个Lucene的文档号的时候，就能从跳跃表中查找到我们自己的文档号。



##### 域(Field)的数据信息(.fdt，.fdx)

![](https://tva1.sinaimg.cn/large/0082zybply1gbtprpy6v0j30gs0buwfu.jpg)

域数据文件(fdt)

- 真正保存存储域(stored field)信息的是fdt文件
- 在一个段(segment)中总共有segment size篇文档，所以fdt文件中共有segment size个项，每一项保存一篇文档的域的信息
- 对于每一篇文档，一开始是一个fieldcount，也即此文档包含的域的数目，接下来是fieldcount个项，每一项保存一个域的信息。
- 对于每一个域，fieldnum是域号，接着是一个8位的byte，最低一位表示此域是否分词(tokenized)，倒数第二位表示此域是保存字符串数据还是二进制数据，倒数第三位表示此域是否被压缩，再接下来就是存储域的值，比如new Field("title", "lucene in action", Field.Store.Yes, …)，则此处存放的就是"lucene in action"这个字符串。

域索引文件(fdx)

- 由域数据文件格式我们知道，每篇文档包含的域的个数，每个存储域的值都是不一样的，因而域数据文件中segment size篇文档，每篇文档占用的大小也是不一样的，那么如何在fdt中辨别每一篇文档的起始地址和终止地址呢，如何能够更快的找到第n篇文档的存储域的信息呢？就是要借助域索引文件。
- 域索引文件也总共有segment size个项，每篇文档都有一个项，每一项都是一个long，大小固定，每一项都是对应的文档在fdt文件中的起始地址的偏移量，这样如果我们想找到第n篇文档的存储域的信息，只要在fdx中找到第n项，然后按照取出的long作为偏移量，就可以在fdt文件中找到对应的存储域的信息。


##### 词向量(Term Vector)的数据信息(.tvx，.tvd，.tvf)

![](https://tva1.sinaimg.cn/large/0082zybply1gbtpuv50y3j30gs0i10v7.jpg)

### 反向信息

##### 词典(tis)及词典索引(tii)信息

![[图]词典及词典索引信息](https://tva1.sinaimg.cn/large/0082zybply1gbtq220lwdj30gs092gmu.jpg)



词典文件(tis)

- TermCount：词典中包含的总的词数
- IndexInterval：为了加快对词的查找速度，也应用类似跳跃表的结构，假设IndexInterval为4，则在词典索引(tii)文件中保存第4个，第8个，第12个词，这样可以加快在词典文件中查找词的速度。
- SkipInterval：倒排表无论是文档号及词频，还是位置信息，都是以跳跃表的结构存在的，SkipInterval是跳跃的步数。
- MaxSkipLevels：跳跃表是多层的，这个值指的是跳跃表的最大层数。
- TermCount个项的数组，每一项代表一个词，对于每一个词，以前缀后缀规则存放词的文本信息(PrefixLength + Suffix)，词属于的域的域号(FieldNum)，有多少篇文档包含此词(DocFreq)，此词的倒排表在frq，prx中的偏移量(FreqDelta, ProxDelta)，此词的倒排表的跳跃表在frq中的偏移量(SkipDelta)，这里之所以用Delta，是应用差值规则。

词典索引文件(tii)

- 词典索引文件是为了加快对词典文件中词的查找速度，保存每隔IndexInterval个词。
- 词典索引文件是会被全部加载到内存中去的。
- IndexTermCount = TermCount / IndexInterval：词典索引文件中包含的词数。
- IndexInterval同词典文件中的IndexInterval。
- SkipInterval同词典文件中的SkipInterval。
- MaxSkipLevels同词典文件中的MaxSkipLevels。
- IndexTermCount个项的数组，每一项代表一个词，每一项包括两部分，第一部分是词本身(TermInfo)，第二部分是在词典文件中的偏移量(IndexDelta)。假设IndexInterval为4，此数组中保存第4个，第8个，第12个词。。。

#### 文档号及词频(frq)信息

![img](https://tva1.sinaimg.cn/large/0082zybply1gbtq9tl15sj30ni0hctc2.jpg)

- 此文件包含TermCount个项，每一个词都有一项，因为每一个词都有自己的倒排表。
- 对于每一个词的倒排表都包括两部分，一部分是倒排表本身，也即一个数组的文档号及词频，另一部分是跳跃表，为了更快的访问和定位倒排表中文档号及词频的位置。



#### 词位置(prx)信息

![[图]词位置信息](https://tva1.sinaimg.cn/large/0082zybply1gbtqf6ylu9j30ni0lnn0z.jpg)

### 其他信息

#### 标准化因子文件(nrm)

![[图]标准化因子文件](https://tva1.sinaimg.cn/large/0082zybply1gbtqhkc1vcj30gs0ajwff.jpg)

- NormsHeader：字符串“NRM”外加Version，依Lucene的版本的不同而不同。
- 接着是一个数组，大小为NumFields，每个Field一项，每一项为一个Norms。
- Norms也是一个数组，大小为SegSize，即此段中文档的数量，每一项为一个Byte，表示一个浮点数，其中0~2为尾数，3~8为指数。

#### 删除文档文件(del)

![[图]删除文档文件](https://tva1.sinaimg.cn/large/0082zybply1gbtqki6a6yj30gs0hggne.jpg)

- Format：在此文件中，Bits和DGaps只能保存其中之一，-1表示保存DGaps，非负值表示保存Bits。
- ByteCount：此段中有多少文档，就有多少个bit被保存，但是以byte形式计数，也即Bits的大小应该是byte的倍数。
- BitCount：Bits中有多少位被至1，表示此文档已经被删除。
- Bits：一个数组的byte，大小为ByteCount，应用时被认为是byte*8个bit。
- DGaps：如果删除的文档数量很小，则Bits大部分位为0，很浪费空间。DGaps采用以下的方式来保存稀疏数组：比如第十，十二，三十二个文档被删除，于是第十，十二，三十二位设为1，DGaps也是以byte为单位的，仅保存不为0的byte，如第1个byte，第4个byte，第1个byte十进制为20，第4个byte十进制为1。于是保存成DGaps，第1个byte，位置1用不定长正整数保存，值为20用二进制保存，第2个byte，位置4用不定长正整数保存，用差值为3，值为1用二进制保存，二进制数据不用差值表示。


