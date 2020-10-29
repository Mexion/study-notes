## 安装

### 在Docker上安装

首先需要在主机上安装`Docker`和`Docker-compose`，然后创建如下`esAndKibana.yml`文件：

```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.9.2
    container_name: elasticsearch
    environment:
      - "cluster.name=elasticsearch" #设置集群名称为elasticsearch
      - "discovery.type=single-node" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m" #设置使用jvm内存大小
    volumes:
      - /opt/elasticsearch/plugins:/usr/share/elasticsearch/plugins #插件文件挂载
      - /opt/elasticsearch/data:/usr/share/elasticsearch/data #数据文件挂载
    ports:
      - 9200:9200
  kibana:
    image: kibana:7.9.2
    container_name: kibana
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - "elasticsearch.hosts=http://127.0.0.1:9200" #设置访问elasticsearch的地址
      - I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - 5601:5601
```

创建`es`用户：

```bash
# 在root用户下
groupadd es
useradd es -g es
passwd es # 给es用户设置密码

# 给予es用户sudo权限
visudo
```

执行`visudo`命令后会打开`/etc/sudoers`文件，找到如下位置：

```bash
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
es    ALL=(ALL)       ALL	# 增加这行保存退出
```

创建挂载文件夹：

```bash
mkdir -p /opt/elasticsearch/data
mkdir -p /opt/elasticsearch/plugins
```

修改文件夹权限：

```bash
#esAndKibana.yml所在文件夹的权限
# 为了方便，避免出错，直接全部改成777，不然可能出错
chmod -R 777 /opt/files
#插件文件挂载的权限
chmod -R 777  /opt/elasticsearch/plugins/
#数据文件挂载
chmod -R 777  /opt/elasticsearch/data/
#docker-compose的操作权限
chmod -R 777  /usr/local/bin/docker-compose
```

切换到`es`用户：

```bash
su es
```

然后在`esAndKibana.yml`文件夹下运行：

```bash
sudo docker-compose -f esAndKibana.yml up -d
```

验证`es`：

```bash
sudo curl 127.0.0.1:9200
```

验证`kibana`浏览器访问`127.0.0.1:5601`（注意服务器的防火墙设置，比如阿里云的防火墙，不开放端口无法访问）。

## ES核心概念

`Elasticsearch`是面向文档的，它和关系型数据库的概念对比如下：

| **Relational DB**  | **ElasticSearch**                          |
| ------------------ | ------------------------------------------ |
| 数据库（database） | 索引（index）                              |
| 表（table）        | type（逐渐会被废弃），目前只推荐默认的_doc |
| 行（row）          | document                                   |
| 字段（column）     | field                                      |

### 倒排索引
倒排索引也叫反向索引，是一种索引方法，被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射。它是文档检索系统中最常用的数据结构。通过倒排索引，可以根据单词快速获取包含这个单词的文档列表。倒排索引主要由两个部分组成：“单词词典”和“倒排文件”。

倒排索引有两种不同的反向索引形式：

- 一条记录的水平反向索引（或者反向档案索引）包含每个引用单词的文档的列表。

- 一个单词的水平反向索引（或者完全反向索引）又包含每个单词在一个文档中的位置。

后者的形式提供了更多的兼容性（比如短语搜索），但是需要更多的时间和空间来创建。

通过上面的定义可以知道，一个倒排索引包含一个单词词典和一个倒排文件。其中单词词典包含了所有粒度的拆分词；倒排文件则保存了该词对应的所有相关信息。

这里举个例子来解释一下，假如我们要对`3`篇文档（网页）建立索引：

- 文档1：中国古代的精美散文

- 文档2：古代精美散文作者

- 文档3：如何写出精美散文

那么文档中的词典集合为`{中国，古代，精美，散文，作者，如何，写出，的}`，倒排索引见：

| 文档编号 | 文档               |
| -------- | ------------------ |
| 1        | 中国古代的精美散文 |
| 2        | 古代精美散文作者   |
| 3        | 如何写出精美散文   |

| 单词ID | 单词 | 倒排列表 |
| ------ | ---- | -------- |
| 1      | 中国 | 1        |
| 2      | 古代 | 1,2      |
| 3      | 精美 | 1,2,3    |
| 4      | 散文 | 1,2,3    |
| 5      | 作者 | 2        |
| 6      | 如何 | 3        |
| 7      | 写出 | 2        |
| 8      | 的   | 1        |

 建立了索引之后，我们就可以开始查询了。输入“精美散文”，分词得到`{精美，散文}`，查倒排索引，就可以得到结果。 

## IK分词器

### 什么是IK分词器？

所谓分词，就是把一段文本划分成一个个关键字，以方便建立索引和检索。

英文本身是以单词为单位, 单词与单词之间, 句子之间通常是空格、逗号、句号分隔. 因而对于英文, 可以简单的以空格来判断某个字符串是否是一个词, 比如: `I love China`, `love`和`China`很容易被程序处理.

但是中文是以字为单位的, 字与字再组成词, 词再组成句子。中文: `我爱中国`, 电脑不知道“`爱中`”是一个词, 还是“`中国`”是一个词？所以我们需要一定的规则来告诉电脑应该怎么切分, 这就是中文分词器所要解决的问题。

 `IK分词器` 是一个开源的，基于`java`语言开发的轻量级的中文分词工具包，最初，它是以开源项目 `Lucene`为应用主体的，结合词典分词和文法分析算法的中文分词组件。新版本的`IKAnalyzer3.0`则发展为 面向Java的公用分词组件，独立于`Lucene`项目，同时提供了对`Lucene`的默认优化实现。 

### 分词的基本原理？

分词器并不具备人工智能的能力，所以它所做的就是把文本和词典中的词进行比较，按照词典中已有的词进行匹配，匹配完成之后即达到了分词的效果。 将一段文字进行`IK`分词处理一般经过：词典加载、预处理、分词器分词、歧义处理、善后结尾 五个部分。

#### 词典加载

主程序需要加载词典。`IK`分词的词典主要包括 `main2012.dic(主词典)`、`quantifier.dic(量词)`、`stopword.dic(停用词)`、`ext.dic(扩展词,可选)`四个字典。字典的结构使用字典树进行存储。关于字典树的具体介绍可以参考： [字典树](http://www.cnblogs.com/rush/archive/2012/12/30/2839996.html)。

#### 预处理

预处理是在加载文档的时候做的，其实预处理部分只是做两件事情：

- 识别出每个字符的字符类型
- 字符转化：全角转半角，大写转小写

字符类型是在分词的时候需要，不同的分词器会根据不同的字符类型做特别处理。而字符转化则是要求文本字符与词典中的字符想匹配，比如对于`12288`、`32`这两个字符，分别是汉字和英文的空格，如果一篇文本中包含了汉字包含了这两个字符，那么分词器在分词的时候需要判断两个都是空格。同理在英文大小写上也存在类似的问题。预处理主要在`org.wltea.analyzer.core.CharacterUtil.java`类中实现。

#### 分词器分词

`IK`分词包括三个分词器：`LetterSegmenter（字符分词器）`,`CN_QuantifierSegmenter（中文数量词分词器）`,`CJKSegmenter（中日韩文分词器）`。分词器分词有两种模式，即：`smart`模式和非`smart`模式，非`smart`模式可以认为是最小力度的分词，`smart`模式则反之。 具体的实例：

```
张三说的确实在理

smart模式的下分词结果为：  
张三 | 说的 | 确实 | 在理

而非smart模式下的分词结果为：
张三 | 三 | 说的 | 的确 | 的 | 确实 | 实在 | 在理
```

`IK`分词使用了”正向迭代最细粒度切分算法“，简单说来就是：
`Segmenter`会逐字识别词元，设输入”中华人民共和国“并且”中“单个字也是字典里的一个词，那么过程是这样的：”中“是词元也是前缀（因为有各种中开头的词），加入词元”中“；继续下一个词”华“，由于中是前缀，那么可以识别出”中华“，同时”中华“也是前缀因此加入”中华“词元，并把其作为前缀继续；接下来继续发现“华人”是词元，“中华人”是前缀，以此类推……
关于`IK`分词的三个分词器，基本上逻辑是一样的，如果有兴趣了解的话，重点看一下`CJKSegmenter`就好了。

#### 歧义处理

非`smart`模式所做的就是将能够分出来的词全部输出；`smart`模式下，`IK`分词器则会根据内在方法输出一个认为最合理的分词结果，这就涉及到了歧义判断。 以“张三说的确实在理”为依据，分词的结果为：

```
张三 | 三 | 说的 | 的确 | 的 | 确实 | 实在 | 在理 
```

在判断歧义的时候首先要看词与词之间是否用冲突，“张三”与“三”相冲突，“说的”，“的确”，“的”，“确实”，“实在”，“在理”相冲突，所以分成两部分进行歧义判断。我们以后一种为例：“说的”，“的确”，“的”，“确实”，“实在”， 按照默认的生成的一组结果为“说的|确实|在理”，与这个结果相冲突的词是“的确”、“的”、“实在”。回滚生成的结果，将这三个词倒叙逐一放入到生成的结果中，生成可选分词方案是:

```
"实在"放入后，分词方案为："说的|实在|";
"的"放入后，生成的分词方案为："的|确实|在理";
"的确"放入后，生成的分词方案为："的确|实在";
```

这样加上原有的一共四种可选的分词方案，从中选取最优的方案。选取的原则顺序如下：

- 1、比较有效文本长度
- 2、比较词元个数，越少越好
- 3、路径跨度越大越好
- 4、最后一个词元的位置越靠后约好（根据统计学结论，逆向切分概率高于正向切分，因此位置越靠后的优先）
- 5、词长越平均越好
- 6、词元位置权重比较

> 总体说来就是： 主要使用的就是贪心算法获取局部最优解，然后继续处理来获取最终的结果。

#### 善后结尾

处理遗漏中文字符，遍历输入文本，把词之间每个中文字符也作为词输出，虽然词典中没有这样的词。 处理数量词，如果中文量词刚好出现在中文数词的后面，或者中文量词刚好出现在阿拉伯数字的后面，则把数词和量词合并。比如"九寸""十二亩"。

### 安装

首先我们查看容器`ID`：

```bash
sudo docker ps
```

拿到容器`ID`后进入容器：

```bash
sudo docker exec -it 容器id /bin/bash
```

进入容器后在线下载并安装（要和`es`版本对应）：

```bash
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.2/elasticsearch-analysis-ik-7.9.2.zip
```
等待完成后运行：
```bash
./bin/elasticsearch-plugin list
# 输出列表中有analysis-ik则成功
analysis-ik
```

然后退出并重启容器：

```bash
sudo docker restart 容器id
```

测试下分词效果，我们访问服务器的`5601`端口，打开`kibana`，然后在开发者工具的控制台输入：

```json
GET _analyze
{
    "analyzer": "ik_smart",
    "text": "中华人民共和国人民大会堂"
}
```

可以看到访问了`_anlyze`这个`api`，发送了一段`json`，`analyzer`表示分词器名称（`ik_smart`会做最粗粒度拆分），`text`表示要分词的文本，点击运行，会返回：

```json
{
  "tokens" : [
    {
      "token" : "中华人民共和国",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "人民大会堂",
      "start_offset" : 7,
      "end_offset" : 12,
      "type" : "CN_WORD",
      "position" : 1
    }
  ]
}
```

我们也可以选择`ik_max_word（最细粒度拆分）`：

```json
GET _analyze
{
    "analyzer": "ik_max_word",
    "text": "中华人民共和国人民大会堂"
}
```

得到：

```json
{
  "tokens" : [
    {
      "token" : "中华人民共和国",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "中华人民",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "中华",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "华人",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "人民共和国",
      "start_offset" : 2,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "人民",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 5
    },
    {
      "token" : "共和国",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "共和",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "国人",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 8
    },
    {
      "token" : "人民大会堂",
      "start_offset" : 7,
      "end_offset" : 12,
      "type" : "CN_WORD",
      "position" : 9
    },
    {
      "token" : "人民大会",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "CN_WORD",
      "position" : 10
    },
    {
      "token" : "人民",
      "start_offset" : 7,
      "end_offset" : 9,
      "type" : "CN_WORD",
      "position" : 11
    },
    {
      "token" : "大会堂",
      "start_offset" : 9,
      "end_offset" : 12,
      "type" : "CN_WORD",
      "position" : 12
    },
    {
      "token" : "大会",
      "start_offset" : 9,
      "end_offset" : 11,
      "type" : "CN_WORD",
      "position" : 13
    },
    {
      "token" : "会堂",
      "start_offset" : 10,
      "end_offset" : 12,
      "type" : "CN_WORD",
      "position" : 14
    }
  ]
}
```

 两种分词器使用的最佳实践是：**索引时用`ik_max_word`，在搜索时用`ik_smart`** 。

### 自定义词典

`ik`自身词典可能无法满足需求，提供了自定义词典功能。

1. 新建扩展名为`dic`文本文件，写入要增加的词条，每词单独一行，如`my.dic(编码utf-8`)，

2. 在容器内部 `config/analysis-ik` 下新建子目录。如：`/config/analysis-ik/mydic/my.dic` 。

3. 修改`ik`配置文件`IKAnalyzer.cfg.xml`（位于`config/analysis-ik`目录下），在配置文件中修改如下条目：

```xml
<entry key="ext_dict">mydict/my.dic</entry>
```

4. 重启容器使生效。

同时，我们还可以配置停止词词库（即本来是一个词，但是不让它成为关键词，比如”结果“是一个词，在停止词词库中加入这个词之后就不会认为它是一个词了），远程词库，远程停止词词库。

```xml
<properties> 
    <comment>IK Analyzer 扩展配置</comment> 
    <!--用户可以在这里配置自己的扩展字典 --> 
    <entry key="ext_dict">custom/mydict.dic;custom/single_word_low_freq.dic</entry>
    <!--用户可以在这里配置自己的扩展停止词字典--> 
    <entry key="ext_stopwords">custom/ext_stopword.dic</entry>  
    <!--用户可以在这里配置远程扩展字典 --> 
    <entry key="remote_ext_dict">http://192.168.90.57:8080/test/mydic.dic</entry>  
    <!--用户可以在这里配置远程扩展停止词字典--> 
    <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties> 
```



## ES Rest CRUD

| method | url地址                       | 描述                   |
| ------ | ----------------------------- | ---------------------- |
| PUT    | /索引名/类型名/文档id         | 创建文档（指定文档id） |
| POST   | /索引名/类型名称              | 创建文档（随机文档id） |
| POST   | /索引名/类型名/文档id/_update | 修改文档               |
| DELETE | /索引名/类型名/文档id         | 删除文档               |
| GET    | /索引名/类型名/文档id         | 查询指定文档           |
| POST   | /索引名/类型名/_search        | 查询所有数据           |

### 创建一个索引

索引受文件系统的限制。仅可能为小写字母，不能下划线开头。同时需遵守下列规则：

- 不能包括 , /, *, ?, ", <, >, |, 空格, 逗号, #
- 7.0版本之前可以使用冒号:,但不建议使用并在7.0版本之后不再支持
- 不能以这些字符 -, _, + 开头
- 不能包括 . 或 …
- 长度不能超过 255 个字符

#### 自动创建

当我们创建一个文档时，指定索引名称，当没有这个索引时会自动创建索引：

```json
PUT /test1/type1/1
{
  "title": "叹庭前甘菊花",
  "author": "杜甫",
  "content": "庭前甘菊移时晚，青蕊重阳不堪摘。明日萧条醉尽醒，残花烂熳开何益？篱边野外多众芳，采撷细琐升中堂。念兹空长大枝叶，结根失所缠风霜。"
}
```

可得：

```json
#! Deprecation: [types removal] Specifying types in document index requests is deprecated, use the typeless endpoints instead (/{index}/_doc/{id}, /{index}/_doc, or /{index}/_create/{id}).
{
  "_index" : "test1",
  "_type" : "type1",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

我们可以看到在最上方有醒目的提醒，说**`type`已经被废弃，不建议在文档索引请求中指定类型** 。

#### 手动创建

我们也可以手动创建索引：

```json
PUT /test2
```

这会创建一个索引，`7`版本以上，默认情况下， 创建的索引分片数量是 `1` 个，副本数量是 `1` 个。 

也可以通过如下参数来制定分片和副本数量：

```json
PUT /test3
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 2
    }
}
```

#### 创建索引规则（mapping）

`mapping`是类似于数据库中的表结构定义，主要作用如下：

- 定义`index`下的字段名
- 定义字段类型，比如数值型、浮点型、布尔型等
- 定义倒排索引相关的设置，比如是否索引、是否记录`position`等

我们可以通过以下`API`开查看某个索引的`mapping`：

```json
GET /test1/_mapping
```

在创建索引时，我们也可以通过`mappings`属性来设置`mapping`：

```json
PUT test4
{
    "mappings": {
        "properties": {
            "title": {
                "type": "keyword"
            },
            "author": {
                "type": "keyword",
                "index": false	//不让字段被索引，这通常用在有关安全隐私方面的信息
            },
            "content": {
                "type": "text"
            }
        }
    }
}
```

##### 对字段指定分词器

可以对字段指定分词器：

```json
PUT test4
{
    "mappings": {
        "properties": {
            "title": {
                "type": "keyword"
            },
            "author": {
                "type": "keyword"
            },
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",	// 指定content字段分词器为ik_max_word
                "search_analyzer": "ik_smart" // 如果不指定，默认会使用analyzer指定的分词器进行搜索分词
            }
        }
    }
}
```



##### 基本数据类型

| 类型       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| text       | 文本类型，用于全文索引。该类型的字段将通过分词器进行分词，最终用于构建索引。 |
| keyword    | 关键词，不分词。                                             |
| long       | 长整型，有符号64位。                                         |
| integer    | 整形，有符号32位。                                           |
| short      | 短整型，有符号16位。                                         |
| byte       | 字节型，有符号8位。                                          |
| double     | 双精度浮点型，64位IEEE 754 浮点数。                          |
| float      | 单精度浮点型，32位IEEE 754 浮点数。                          |
| half_float | 半精度浮点型，16位IEEE 754 浮点数。                          |
| boolean    | 布尔型。                                                     |
| date       | 日期。                                                       |
| binary     | 二进制。                                                     |

1. 日期类型除了`type`外，通常会额外包含一个`format`字段，用于帮助`ES`完成日期和字符间的转换，如下：

```json
"createDate": {
    "type": "date",
    "format": "yyyy-MM-dd"
}
```

如果想同时支持多种日期格式，可以使用`||`：

```json
"createDate": {
    "type": "date",
    "format": "yyyy-MM-dd || yyyy-MM-dd HH:mm:ss"
}
```

2. 如果日期格式未提供`format`属性，`ES`将使用默认的`format`属性：`strict_date_optional_time||epoch_millis`。
   `strict_date_optional_time`的含义是 `yyyy-MM-dd’T’HH:mm:ss.SSSZ or yyyy-MM-dd`。
   `epoch_millis`的含义是 `Unix`时间戳。

3. `ES 5`版本后，将原来的`string`类型拆分成了两个类型，即`text`和`keyword`。这两个字段的区别在于，`text`会自动分词，而`keyword`不会自动分词。这可能就是你在精确匹配某个`text`类型字段时无法匹配到的原因，就是因为，原来的`text`字段已经被分词了。

4. `text`和`keyword`的另一个区别是`keyword`支持聚合，而`text`不支持聚合。通常情况下，大段的文本我们会设置为`text`属性，如文章内容，商品描述等，而较简短的属性我们设置为`keyword`，如文章标签，商品品牌等。

5. `binary`类型可以接受`Base64`编码的字符串，该字段默认不存储，也不可搜索。事实上，我们一般不用该类型。但在某些特殊情况下可能会被使用，比如说文件查重等。

##### 复杂数据类型

| 属性名称 | 说明                     |
| -------- | ------------------------ |
| 空域     | 不会被索引               |
| 数组     | 一组相同类型的数组       |
| 对象     | 一个包含若干子属性的对象 |

说明：

1. 空域不会被索引，自然也无法搜索，如下形式都会被es认为是空域，如

```javascript
null
[]
[null]
```

2. 数组，`es`称之为多值域对象，包含一系列具有相同类型的值。当通过索引数组创建新域时，`es`会默认使用数组中的第一个值的类型作为这个域的类型。

3. 对象事实上就是属性的嵌套。需要注意的是，`es`对这种嵌套的处理方式是扁平化，这是由于`lucence`不支持内部对象。如：

```json
{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}
```

在`es`中的表示形式是下面这样的：

```json
{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
```

4. `es`针对对象的扁平化处理带来一个问题，如下：

```json
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
```

在`es`中被描述成了这样：

```
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
```

这导致，无法精确找到那个`26`岁，叫`Alex`的人。

5. 我们可以把一个原本的`object`类型修改为`nested`类型，这样，`es`此时会把一个`nested对象`索引为一个单独的隐藏文档，就能解决扁平化的问题。如下：

```json
// 文档1
  {
    "followers.age":    [ 19 ],
    "followers.name": [ mary, white ],
  }
  // 文档2
  {
    "followers.age":    [ 26],
    "followers.name": [ alex, jones ],
  }
  // 文档3
  {
    "followers.age":    [ 35],
    "followers.name": [ lisa, smith ],
  }
```

##### 范围类型

| 属性名称      | 说明       |
| ------------- | ---------- |
| long_range    | 长整形范围 |
| integer_range | 整型范围   |
| float_range   | 单精度范围 |
| double_range  | 双精度范围 |
| date_range    | 日期范围   |
| ip_range      | ip范围     |

说明：

1. 范围类型用于某个范围的描述。比如说，某聚会只允许`18`到`24`的年轻`MM`参与，那`mapping`可能是：

```json
{
  "title": {
    "type":"keyword"
  },
  "age_range": {
    "type": "integer_range"
  }
}
```

会议记录可能是：

```json
{
  "title"："单身男——女女女女相亲会",
  "age_range": {
    "lte": 18,
    "gte": 24
  }
}
// 这时候，如果来一个38的阿姨想要参与，那也自然是搜索不到了。
```

2. 对于`ip`来说，`ip`白名单可以这样设置：

```json
{
    "ip_whitelist" : "192.168.0.0/16"
}
```

##### 特殊数据类型

| 属性名称    | 说明     |
| ----------- | -------- |
| geo_point   | 地理点   |
| geo_shape   | 地理形状 |
| ip          | ip类型   |
| token_count | 计数     |

1. 地理点用来存放地图上某个点的经纬度，我们可以通过以下形式插入记录：

```json
{
    "text": "这圪塔",
    "location": {
        "lat": 23.11, "lon": 113.33     // 纬度: latitude, 经度: longitude
    }
}
// 或
{
  "text": "那圪塔",
  "location": "23.11, 113.33"           // 纬度, 经度
}
// 或
{
  "text": "全是圪塔",
  "location": [ 113.33, 23.11 ]         // 经度, 纬度
}
```

查询时，可能通过盒子模型来查询：

```json
{
    "query": {
        "geo_bounding_box": {
            "location": {
                "top_left": { "lat": 24, "lon": 113 },      // 地理盒子模型的上-左边
                "bottom_right": { "lat": 22, "lon": 114 }   // 地理盒子模型的下-右边
            }
        }
    }
}
```

2. 存储一个`IP`的地址。可以在某个段使用`/`来描述范围，如下：

```json
"term": { "ip_addr": "192.168.0.0/16"}
```

`192.168.0.12`可以被匹配到。

3. 计数用于统计一个字符串中包含的单词的个数。如下定义：

```json
"name": {
    "type": "text",
    "fields": {
        "length": {
            "type": "token_count",
            "analyzer": "standard"
        }
    }
}
```

如果传入的`name`是 `Mary White`，`name.length`就会是2。

4. `get_shape`，即地理形状，用得较少，但有可能也会用到。它通过一系列的地理点来描绘地图上的一个多边型区域，如：

```json
{
    "location" : {
        "type" : "polygon",
        "coordinates" : [
            [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0] ]
        ]
    }
}
```

##### index 参数

 `index`参数作用是控制当前字段是否被索引，默认为`true`，`false`表示不记录，即不可被搜索。

```json
PUT test5
{
    "mappings": {
        "doc": {
            "properties": {
                "cookie": {
                    "type": "text",
                    "index": false
                },
                "content": {
                    "type": "text",
                    "index": true
                }
            }
        }
    }
}
```

> 这个`index`有两个字段，其中cookie设定为不可被搜索

 **当在es中存储了一些不想要被检索的字段如身份证、手机等，这是对于这些字段就可以使用index设置为false，这样有一定的安全性还可以节省空间**。

##### 自动识别类型

在写入文档的时候如果`index`不存在的话`es`会自动创建这个索引。但是`es`是如何确定`index`字段的类型的呢？`es`可以自动识别文档字段的类型，这样可以降低用户的使用成本。

 `es`是依靠`json`文档的字段类型来实现自动识别字段类型的： 

| json 类型 | es 类型                                                      |
| --------- | ------------------------------------------------------------ |
| null      | 忽略                                                         |
| boolean   | boolean                                                      |
| 浮点      | float                                                        |
| 整数      | long                                                         |
| object    | object                                                       |
| array     | 由第一个非null值的类型决定                                   |
| string    | 匹配为日期则设为date类型，匹配为数字设为float或long（默认关闭），设置为Text，并且增加keyword子字段。 |

##### 自定义mapping 的建议

1. 写一条文档到`es`的临时索引中，获取`es`自动生成的`mapping`
2. 修改第一步得到的`mapping`，自定义相关配置
3. 使用第`2`步的`mapping`创建自定义索引

### 删除索引

```bash
DELETE /<index>
```

比如：

```bash
DELETE /test2
```

如果要删除所有索引：

```bash
DELETE /*
```



### _cat查看ES状态

使用`_cat`可以查看支持的命令：

```bash
GET /_cat
```

```bash
=^.^=
/_cat/allocation	// 显示每个节点分片数量、占用空间
/_cat/shards		// 显示索引分片信息
/_cat/shards/{index}
/_cat/master	// 显示master节点信息
/_cat/nodes		// 显示node节点信息
/_cat/tasks
/_cat/indices	// 查看索引信息
/_cat/indices/{index}
/_cat/segments		// 显示分片中的分段信息
/_cat/segments/{index}
/_cat/count			// 显示索引文档数量
/_cat/count/{index}
/_cat/recovery		// 显示正在进行和先前完成的索引碎片恢复的视图
/_cat/recovery/{index}
/_cat/health		// 查看集群健康状况
/_cat/pending_tasks		//显示正在等待的任务
/_cat/aliases		// 显示别名，过滤器，路由信息
/_cat/aliases/{alias}
/_cat/thread_pool		// 显示线程池信息
/_cat/thread_pool/{thread_pools}
/_cat/plugins	//显示节点上的插件
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs		// 显示node节点属性
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates		// 显示模板信息
/_cat/ml/anomaly_detectors
/_cat/ml/anomaly_detectors/{job_id}
/_cat/ml/trained_models
/_cat/ml/trained_models/{model_id}
/_cat/ml/datafeeds
/_cat/ml/datafeeds/{datafeed_id}
/_cat/ml/data_frame/analytics
/_cat/ml/data_frame/analytics/{id}
/_cat/transforms
/_cat/transforms/{transform_id}
```

每个命令都可以使用`?v`参数，来显示详细的信息：

```bash
GET /_cat/indices?v
```

### 新增一个文档

```bash
PUT /<index>/_doc/<_id>
```

比如：

```json
PUT /test1/_doc/1
{
  "title": "（唐）叹庭前甘菊花",
  "author": "杜甫",
  "content": "庭前甘菊移时晚，青蕊重阳不堪摘。明日萧条醉尽醒，残花烂熳开何益？篱边野外多众芳，采撷细琐升中堂。念兹空长大枝叶，结根失所缠风霜。"
}
```

`ES`分配`_id`：

```json
POST /ems/emp // 注意，使用的是POST
{
    "title": "叹庭前甘菊花",
    "author": "杜甫",
    "content": "庭前甘菊移时晚，青蕊重阳不堪摘。明日萧条醉尽醒，残花烂熳开何益？篱边野外多众芳，采撷细琐升中堂。念兹空长大枝叶，结根失所缠风霜。"
}
```

### 修改一个文档

```bash
POST /<index>/_update/<_id> 
```

比如：

```json
POST /test1/_update/1
{
    "doc": {
        "title": "叹庭前甘菊花",
        "author": "杜甫",
        "content": "庭前甘菊移时晚，青蕊重阳不堪摘。明日萧条醉尽醒，残花烂熳开何益？篱边野外多众芳，采撷细琐升中堂。念兹空长大枝叶，结根失所缠风霜。"
    }
}
```

表示修改索引`test1`下的`id`为`1`的文档。

### 获取一个文档

```bash
GET /<index>/_doc/<_id>
```

比如：

```bash
GET /test1/_doc/1
```



### 删除文档

```bash
DELETE /<index>/_doc/<_id>
```

比如：

```bash
DELETE /test1/_doc/1
```

### 搜索

#### URL简单搜索

`URL搜索`顾名思义就是通过`URL querystring`传递参数进行检索，比如要查询`title`包含`菊花`的文档：

```bash
GET /test1/_search?q=title:菊花
```

可得：

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "test1",
        "_type" : "type1",
        "_id" : "1",
        "_score" : 0.5753642,
        "_source" : {
          "title" : "叹庭前甘菊花",
          "author" : "杜甫",
          "content" : "庭前甘菊移时晚，青蕊重阳不堪摘。明日萧条醉尽醒，残花烂熳开何益？篱边野外多众芳，采撷细琐升中堂。念兹空长大枝叶，结根失所缠风霜。"
        }
      }
    ]
  }
}
```

要查询所有文档：

```bash
GET /<index>/_search?q=*
```

查询文档排序：

```bash
GET /<index>/_search?q=查询条件&sort=排序字段
```

默认是升序，如果想要降序：

```bash
GET /<index>/_search?q=查询条件&sort=排序字段:desc
```

默认只会返回`10`条数据，要自定义数量可以指定`size`：

```bash
GET /<index>/_search?q=查询条件&size=数量
```

要指定偏移量可以使用`from`：

```bash
GET /<index>/_search?q=查询条件&size=数量&from=偏移量（从0开始）
```

要指定`_source`返回的字段，可以使用：

```bash
GET /<index>/_search?q=查询条件&_source=字段1,字段2,...
```

更多字段可以查看[Search API](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/search-search.html)。了解即可，一般不推荐`URL`方式查询。

#### DSL复杂搜索

`DSL`通过传递`JSON`作为请求体(`Request body`)进行检索，`API`同样是`GET /<index>/_search`。

#### 查询所有

```json
GET /<index>/_search
{
    // 首先要写query字段
    "query": {
        "match_all": {}		// match_all字段表示匹配所有文档，后面必须带空对象
    }
}
```

#### 查询排序

要对查询的结果进行排序：

```json
GET /<index>/_search
{
    "query": {
        "match_all": {}
    },
    // 和query同级写sort
    "sort": [
        {
            // 可以进行多字段排序，但是分词的字段不能排序，比如Text类型
        	"age": {
                "order": "desc"
            },
            "address": {	// keyword类型可以排序
                "order": "desc"
            }
    	}
    ]
}
```

#### 查询分页

要实现分页：

```json
GET /test2/_search
{
  "query": {
    "match_all": {}
  },
	// query同级 书写size和from
  "size": 2,
  "from": 1
}
```

#### 指定返回字段

指定返回的字段：

```json
GET /test2/_search
{
  "query": {
    "match_all": {}
  },
	// query同级
	"_source": "name" // 只返回一个字段
}


GET /test2/_search
{
  "query": {
    "match_all": {}
  },
	// query同级
	"_source": ["name", "age"] // 返回多个字段，数组格式
}
```

#### term关键词查询

基于**关键词**查询：

```json
GET /test2/_search
{
    "query": {
        "term": {	// query内部写 term，但是避免对text字段进行term查询，因为索引中汉字会进行分词，默认分词会拆成单字，如果直接使用一个词进行查询是查询不到结果的
            "age": {
                "value": 12		// 搜索 age字段中的 12，搜索时要转换为小写
        }
    }
}
```

#### match关键词查询

```json
GET /test1/_search
{
    "query": {
        "match": {		// 使用match字段，如果搜索的字段分词那么它也会进行分词后再搜索，比如默认分词器会将 “风霜”分成 “风”、“霜”，只要内容中包含其中一个，都会返回。
            "content": "风霜"
        }
    }
}
```

综上可知，`term`和`match`的区别就是`term`不会对搜索词进行分词处理，所以必须包含整个搜索词的关键词才会被击中，比如“我爱你”，使用`term`搜索，必须整个关键词中包含“我爱你”三个字才会被击中。但是由于默认的分词器在生成索引时会将语句分词成一个一个字，比如“我爱你中国”，会被分成五个字，所以三个字不可能击中一个字的关键词，自然无法匹配。

但是`match`会根据要搜索的字段进行判断，如果要搜索的字段是可以分词的字段（`text`），那么搜索词也进行分词，所以“我爱你”使用默认分词器会被分成三个字，只要有一个字匹配就会匹配，那么自然就可以匹配上。如果要搜索的字段本身就不分词，那么它也不会对字段进行分词，直接使用搜索词进行匹配，大概等于`term`。

`match`可以在搜索时指定搜索词分词器：

```json
GET /test1/_search
{
    "query": {
        "match": {
            "content": {
                "query": "风霜",
                "analyzer": "ik_smart"	// 指定分词器
            }
        }
    }
}
```



#### 范围查询

范围查询：

```json
GET /test2/_search
{
    "query": {
        "range": {	// query中写age
            "age": {
                "gte": 15,	// 大于等于15 ，大于则用gt
                "lte": 20	// 小于等于20， 小于则用lt
            }
        }
    }
}
```

#### 前缀查询

前缀查询（基于关键词的前缀查询，如果查询的是“Text”类型，并且使用默认的分词器，那么由于是单字分词，多个字的前缀并不会被查询到，比如：“王大牛” 会被分成“王”，“大”， “牛”，如果搜“王大”，是搜索不到结果的，但是搜“牛”是可以搜索到结果的，因为是基于每个关键词的前缀来进行检索的）：

```json
GET /test2/_search
{
    "query": {
        "prefix": {		// query中写prefix
            "name": {
                "value": "王"
            }
        }
    }
}
```

#### 通配符查询

通配符查询（`?`匹配一个任意字符，`*`匹配任意多个（包括`0`）任意字符，不管是`?`还是`*`都只能放在后面，而不能放在开头）：

```json
GET /test2/_search
{
    "query": {
        "wildcard": {		// query里写wildcard
            "name": {
                "value": "李?"
            }
        }
    }
}
```

#### 多个id查询

多个`id`查询：

```json
GET /test2/_search
{
    "query": {
        "ids": {	// query中写ids
            "values": ["1", "2", "3"]	//values 写数组，里面放多个id
        }
    }
}
```

#### 多字段查询

可以对一个搜索词对多个字段进行匹配，比如要搜索`redis`，那么想要查询`title`中或者`content`中存在`redis`关键词的，可以使用`multi_match`：

```json
GET /test2/_search
{
    "query": {
        "multi_match": {
            "query": "redis",
            "fileds": ["title", "content"]	// redis 会在title和content中进行搜索
        }
    }
}
```

`multi_match`和`match`一样，会根据搜索字段分不分词来决定搜索词分不分词，如果搜索字段分词，那么搜索词分词，否则不分词。

可以指定分词器：

```json
GET test2/_search
{
  "query": {
   "multi_match" : {
      "query":      "Jon",
      "type":       "cross_fields",
      "analyzer":   "standard", 		// 指定分词器
      "fields":     [ "first", "last", "*.edge" ]
    }
  }
}
```





#### 多字段分词查询

多字段分词查询`query_string`可以进行分词查询：

```json
GET /test2/_search
{
    "query": {
        "query_string": {
            "default_field": "content",		// 默认字段
            "query": "redis is an open source db"	// 会分词后再匹配
        }
    }
}

GET /test2/_search
{
    "query": {
        "query_string": {
            "query": "redis is an open source db",	// 会分词后再匹配
            "fields": ["name", "content"]		// 不加default_filed,加fields可以实现多字段查询
        }
    }
}
```

同样，可以指定分词器：

```json
GET /test2/_search
{
    "query": {
        "query_string": {
            "query": "中华人民共和国",
            "fields": ["name", "content"],
            "analyzer": "ik_smart" 	//指定分词器
        }
    }
}
```



#### 模糊查询

```json
GET /test2/_search
{
    "query": {
        "fuzzy": {
            "name": "elasticsearch"
        }
    }
}
// 模糊查询是基于关键词的，这里当搜索elasticSearch时如果完全匹配一个关键词，那么可以搜索到， 如果搜索elasticseorch，错误了一个字母，那么也同样可以搜索到，可以允许错误的字数是有要求的，当整个关键词长度为0-2时，不允许错误，当长度为3-5时，允许一个字错误，当大于5时允许两个字错误，最大可以错误两个字。
```

#### 正则查询

```json
GET /test2/_search
{
    "query": {
        "regexp": {
            "mobile": "139[0-9]{8}"		// 以 139开头的手机号
        }
    }
}
```

注意： `prefix`、`fuzzy`、`wildcard`、`regexp`查询效率都比较低，要求效率比较高时，避免使用。

#### 布尔查询

布尔查询可以组合多个条件实现复杂查询。

- `must`: 相当于 `&&` 同时成立
- `should`： 相当于`||`成立一个就行
- `must_not`： 不能满足任何一个

```json
GET /test2/_search
{
    "query": {
        "bool": {
            "must": [	//这里面的条件必须同时成立
                {
                    "term": {
                        "age": {	// 年龄必须是20
                            "value": 20
                        }
                    }
                }
            ]
        }
    }
}

GET /test2/_search
{
    "query": {
        "bool": {
            "must": [	//这里面的条件必须同时成立
                {
                    "term": {
                        "age": {	// 年龄必须是20
                            "value": 20
                        }
                    }
                }, {
                    "term": {
                        "address": {
                            "value": "北京" // 并且地址必须是北京(address必须是keyword)
                        }
                    }
                }
            ]
        }
    }
}

GET /test2/_search
{
    "query": {
        "bool": {
            "should": [	//里面的条件满足其一即可
                {
                    "term": {
                        "age": {	// 年龄是20
                            "value": 20
                        }
                    }
                }, {
                    "term": {
                        "address": {
                            "value": "北京" // 并且地址是北京(address必须是keyword)
                        }
                    }
                }
            ]
        }
    }
}

GET /test2/_search
{
    "query": {
        "bool": {
            "must_not": [	//里面的条件全不满足
                {
                    "term": {
                        "age": {	// 年龄是20
                            "value": 20
                        }
                    }
                }, {
                    "term": {
                        "address": {
                            "value": "北京" // 并且地址是北京(address必须是keyword)
                        }
                    }
                }
            ]
        }
    }
}

// 当然，布尔查询可以结合所有查询，不一定是term
GET /test2/_search
{
    "query": {
        "bool": {
            "must_not": [	//里面的条件全不满足
                {
                    "term": {
                        "age": {	// 年龄是20
                            "value": 20
                        }
                    }
                }, {
                    "range": {	// 这里换成了范围查询
                        "age": {
                            "gte": 0,
                            "lte": 10
                        }
                    }
                }
            ]
        }
    }
}
```

#### 高亮查询

高亮查询其实并不算一个查询，只是对查询结果进行二次渲染高亮：

```json
// 首先插入一条数据

PUT /test3/_doc/1
{
  "title": "redis introduction",
  "content": "Redis is an open source (BSD licensed), in-memory data structure store, used as a database."
}

// 高亮所有字段
GET /test3/_search
{
    "query": {
        "term": {
            "content": {
                "value": "redis"
            }
        }
    },
    "highlight": {
        "require_field_match": false, // 默认只能高亮匹配字段，要高亮非匹配字段，必须设置为false
        "fields": {		// 对哪些字段进行高亮，比如查询的是content，可能title也有redis关键词，可以同时实现高亮
            "*": {}		// * 表示对所有字段高亮
        }
    }
}

// 只高亮content字段
GET /test3/_search
{
    "query": {
        "term": {
            "content": {
                "value": "redis"
            }
        }
    },
    "highlight": {
        "fields": {		// 对哪些字段进行高亮，比如查询的是content，可能title也有redis关键词，可以同时实现高亮
            "content": {}		// 只对content高亮
        }
    }
}
```

返回：

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "test3",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "title" : "redis introduction",
          "content" : "Redis is an open source (BSD licensed), in-memory data structure store, used as a database."
        },
        "highlight" : {
          "title" : [
            "<em>redis</em> introduction"
          ],
          "content" : [
            "<em>Redis</em> is an open source (BSD licensed), in-memory data structure store, used as a database."
          ]
        }
      }
    ]
  }
}

// 可以看到结果不是直接修改_source，而是增加了highlight字段，在字段中对高亮结果进行了返回，高亮其实就是在查询的关键词外部嵌套了HTML标签，默认为em标签
```

要修改默认的`em`标签，可以这样查询：

```json
GET /test3/_search
{
    "query": {
        "term": {
            "title": {
                "value": "redis"
            }
        }
    },
    "highlight": {
      "require_field_match": false,
        "fields": {
          "*": {
            "pre_tags": ["<span>"],		// 设置起始标签
            "post_tags": ["</span>"]	//设置结束标签
          }
        }
    }
}
```

可以指定高亮数据返回多少个字符：

```json
GET /test3/_search
{
    "query": {
        "term": {
            "title": {
                "value": "redis"
            }
        }
    },
    "highlight": {
      "require_field_match": false,
        "fields": {
          "*": {
            "pre_tags": ["<span>"],		
            "post_tags": ["</span>"],	
             "fragment_size": 20	// 高亮数据返回20个字符，包括高亮词，一个词算一个字符，一个符号算一个字符，包括标签	
          }
        }
    }
}
```



#### 返回结果分析

```json
GET /test2/_search
{
    "query": {
        "match_all": {}
    },
    "size": 2
}
```

得到：

```json
{
  "took" : 0,		//返回结果使用的时间（毫秒），这里表示0毫秒，啪一下就返回了，很快啊！
  "timed_out" : false,	// 是否超时
  "_shards" : {		// 分片信息
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {	// 查询击中的结果对象描述
    "total" : {
      "value" : 4,	// 结果条数
      "relation" : "eq"
    },
    "max_score" : 1.0,	//结果中最大得分
    "hits" : [		//击中结果数据列表
      {
        "_index" : "test2",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,		// 分数， 这里都是1.0是因为查询所有，不计算得分
        "_source" : {		// 数据的原始信息
          "name" : "王大牛",
          "age" : 10
        }
      },
      {
        "_index" : "test2",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "李二狗",
          "age" : 12
        }
      }
    ]
  }
}
```

### 过滤搜索
`ES`中的搜索分为两种，一种是之前提到的`搜索(query)`，另外一种是`过滤(filter)`。`query`会计算每个返回文档的得分，然后根据得分排序。而`filter`则只会筛选出符合的文档，不计算得分，并且它可以缓存文档，所以从性能上来说，`filter`更快。
`filter`适合大范围筛选数据，查询则适合精确匹配数据。一般应用时，应先使用过滤操作过滤数据，然后使用查询匹配数据。

在执行`filter`和`query`时，会先执行`filter`再执行`query`，也就是先过滤掉了一些结果，在进行查询，同时`ES`会对经常使用的`filter`结果进行缓存。

```json
GET /test2/_search
{
    "query": {
        "bool": {		// 因为是两种查询方式，一次查询使用多种查询方式必须使用布尔查询
            "must": [
                {"match_all": {}}
            ],
            "filter": {		// 注意，和must同级，也就是说bool里可以写must，should，must_not，filter
                "range": {
                    "age": {
                        "gte": 10
                    }
                }
            }
        }
    }
}
```

常见的过滤器类型还有`term`，`terms`，`exists`， `ids`：

```json
GET /test2/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "name": {
                            "value": "二狗"
                        }
                    }
                }
            ],
            "filter": {
                "term": {		// 使用了term对关键词进行过滤
                    "content": "redis"	// 这里就会先过滤出content有redis的结果
                }
            }
        }
    }
}
```

```json
GET /test2/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "name": {
                            "value": "二狗"
                        }
                    }
                }
            ],
            "filter": {
                "terms": {		// 使用了terms
                    "content": ["架构", "redis"]	// 会过滤出有这两种结果其一的
                }
            }
        }
    }
}
```

`exists`过滤存在指定字段，获取字段不为空的记录：

```json
GET /test2/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "name": {
                            "value": "二狗"
                        }
                    }
                }
            ],
            "filter": {
                "exists": {
                    "field": "content"		// 过滤content字段不为空的
                }
            }
        }
    }
}
```

`ids`过滤含有指定`id`的索引记录：

```json
GET /test2/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "name": {
                            "value": "二狗"
                        }
                    }
                }
            ],
            "filter": {
                "ids": {
                    "values": ["1", "2", "3"]		// 过滤id为1,2,3的
                }
            }
        }
    }
}
```

### 搜索删除

根据搜索结果删除文档：

```json
POST /test2/_delete_by_query
{
    "query": {
        "range": {
            "age": {
                "lt": 10	// 删除age小于10的文档
            }
        }
    }
}
```

### boosting 搜索

`boosting`查询可以帮助我们影响搜索后的分数。

`boosting`提供了三个字段参数：

- `positive`： 只有匹配上`positive`查询的内容，才会放到返回的结果集中。
- `negative`： 如果匹配上了`positive`并且也匹配上了`negative`，就可以降低这样的文档的`score`。
- `negative_boosting`： 为`negative`匹配上的文档，也就是要降低分数的文档，指定系数，必须小于`1`。

```json
GET /test2/_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "text": "apple"
        }
      },
      "negative": {
        "term": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

### 聚合查询

`ES`的聚合查询和`MySQL`的聚合查询类似，比如`MySQL`有`count`、`Max`等，`ES`的聚合查询提供的统计数据的方式很多。

-  Metric(指标):   指标分析类型，如计算最大值、最小值、平均值等等 （对桶内的文档进行聚合分析的操作） 
- Bucket(桶):     分桶类型，类似SQL中的GROUP BY语法 （满足特定条件的文档的集合） 
- Pipeline(管道): 管道分析类型，基于上一级的聚合分析结果进行在分析 
- Matrix(矩阵):   矩阵分析类型（聚合是一种面向数值型的聚合，用于计算一组文档字段中的统计信息） 

在查询请求体中以`aggregations（可以简写为aggs）`节点按如下语法定义聚合分析：

```makefile
"aggregations" : {
    "<aggregation_name>" : {       //聚合的名字，可以自己随便取，比如max_price
        "<aggregation_type>" : {   //聚合的类型
            <aggregation_body>     //聚合体：对哪些字段进行聚合
        }
        [,"meta" : {  [<meta_data_body>] } ]?        //元
        [,"aggregations" : { [<sub_aggregation>]+ } ]?  //在聚合里面在定义子聚合
    }
    [,"<aggregation_name_2>" : { ... } ]*               //聚合的名字
}
```

虽然`Elasticsearch`有四种聚合方式，但在一般实际开发中，用到的比较多的就是`Metric`和`Bucket`。

- 桶（bucket）

1. 简单来说桶就是满足特定条件的文档的集合。
2. 当聚合开始被执行，每个文档里面的值通过计算来决定符合哪个桶的条件，如果匹配到，文档将放入相应的桶并接着开始聚合操作。
3. 桶也可以被嵌套在其他桶里面。

- 指标（metric）

1. 桶能让我们划分文档到有意义的集合，但是最终我们需要的是对这些桶内的文档进行一些指标的计算。分桶是一种达到目的地的手段：它提供了一种给文档分组的方法来让我们可以计算感兴趣的指标。
2. 大多数指标是简单的数学运算（如：最小值、平均值、最大值、汇总），这些是通过文档的值来计算的。

#### 去重记数查询

去重计数，`cardinality` 先将返回的文档中的一个指定的`field`进行去重，统计一共有多少条：

```json
// 去重计数 查询 province
GET /test/_search
{
  "aggs": {
    "provinceCount": {
      "cardinality": {
        "field": "province"
      }
    }
  }
}
```

结果：

```json
{
   ...
   "hits": { ... },
   "aggregations": {
      "provinceCount": {		// 与你查询时起的名字相同
         "value": 4				// 去重后得到4个
      }
   }
}
```

#### 范围记数查询

统计一定范围内出现的文档个数，比如，针对某一个`field` 的值在`0~100`,`100~200`,`200~300` 之间文档出现的个数分别是多少。范围统计 可以针对 普通的数值，针对时间类型，针对ip类型都可以响应。
- 数值： `range`
- 时间：  `date_range`
- `ip`：   `ip_range`

```json
//针对数值方式的范围统计  from 带等于效果 ，to 不带等于效果
GET /test/_search
{
  "aggs": {
    "agg": {
      "range": {
        "field": "fee",
        "ranges": [
          {
            "to": 30
          },
           {
            "from": 30,
            "to": 60
          },
          {
            "from": 60
          }
        ]
      }
    }
  }
}


//时间方式统计
GET /test/_search
{
  "aggs": {
    "agg": {
      "date_range": {
        "field": "sendDate",
        "format": "yyyy", 
        "ranges": [
          {
            "to": "2000"
          },{
            "from": "2000"
          }
        ]
      }
    }
  }
}


//ip 方式 范围统计
GET /test/_search
{
  "aggs": {
    "agg": {
      "ip_range": {
        "field": "ipAddr",
        "ranges": [
          {
            "to": "127.0.0.8"
          },
          {
            "from": "127.0.0.8"
          }
        ]
      }
    }
  }
}
```

#### 统计聚合查询

统计聚合查询可以直接查询多种聚合结果，比如指定`field`的最大值，最小值，平均值，平方和...
统计聚合查询使用参数 `extended_stats`：
```json
// 统计聚合查询 extended_stats
POST /test/_search
{
  "aggs": {
    "agg": {
      "extended_stats": {
        "field": "fee"		// 使用很简单，直接写字段即可
      }
    }
  }
}
```

返回结果：
```json
{
   ...
   "hits": { ... },
   "aggregations": {
      "agg": {		// 与你查询时起的名字相同
         "count": 20,
         "min": 4.0,
         "max": 8.0,
         "avg": 5.0,
         "sum": 116.0,
         "sum_of_squares": 716.0,
         "variance": 2.160000000024,
         "std_deviation": 1.4696938456699076,
         "std_deviation_bounds": {
         	"upper": 8.739387691339815,
         	"lower": 2.8606123086601847
         }
      }
   }
}
```
更多聚合查询可以查看[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/search-aggregations.html)。

