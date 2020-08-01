

# MySQL

## 认识MySQL

### Windows 里启动服务器程序

#### mysqld

在bin目录下有一个mysqld可执行文件，在命令行输入mysqld，或者双击。

#### 以服务器的方式运行服务器程序

**把某个程序注册为Windows服务**

> 完整路径 -install

**启动MySQL**

> net start MySQL

**关闭服务**

> net stop MySQL



mysql采用TCP作为服务器和客户端之间的网络通信协议。

### 启动Mysql客户端

> mysql -h 主机名 -u 用户名 -p 密码

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714154237026.png" alt="image-20200714154237026" style="zoom:200%;" />

* mysql的各个参数的摆放顺序没有硬性规定；

### 客户端与服务器连接过程

客户端与服务器都是一个个进程，因此客户端向服务器发送请求并得到回复的过程就是一个进程间通信的过程。

MySQL支持下面三种客户端进程与服务器进行间的通信方式。

#### TCP/IP

MySQL服务器启动的时候会默认的申请3306端口号，之后就在此端口号上等待客户端进程

如果3306被占用，在启动服务器时添加-P指定：

> mysqld -P 3307

#### 命名管道和共享内存

* **使用命名管道**

  启动服务器时，加上` --enable-named-pipe`

  启动客户端时，加入` --pipe`或者`--protocol=pipe`参数

* **使用共享内存**

  启动服务器时，加上` --shared-memory`,成功启动之后，共享内存便成了本地客户端程序的默认连接方式

  也可以显示的指定：启动客户端时，加入 ` --protocol=memory`

  该方式必须服务器与客户端在同一台主机上。

#### Unix套接字

如果我们在启动客户端程序的时候指定的主机名为`localhost`，或者指定了`--protocol=socket`的启动参数，那服务器程序和客户端程序之间就可以通过`Unix`域套接字文件来进行通信了.

### 服务器处理客户端请求

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714160150370.png" alt="image-20200714160150370" style="zoom:200%;" />

#### 连接管理

每当有一个客户端进程连接到服务器进程时，服务器进程都会创建一个线程来专门处理与这个客户端的交互，当该客户端退出时会与服务器断开连接，服务器并不会立即把与该客户端交互的线程销毁掉，而是把它缓存起来。

在另一个新的客户端再进行连接时，把这个缓存的线程分配给该新客户端。

如果客户端程序和服务器程序不运行在一台计算机上，我们还可以采用使用了`SSL`（安全套接字）的网络连接进行通信，来保证数据传输的安全性

#### 解析与优化

##### 查询缓存

查询缓存可以在不同客户端之间共享，也就是说如果客户端A刚刚查询了一个语句，而客户端B之后发送了同样的查询请求，那么客户端B的这次查询就可以直接使用查询缓存中的数据。

如果两个查询请求在**任何字符上的不同**（例如：空格、注释、大小写），都会导致缓存**不会命中**。另外，如果查询请求中包含某些系统函数、用户自定义变量和函数、一些系统表，如 `mysql 、information_schema、 performance_schema `数据库中的表**，那这个请求就**不会被缓存。

**从MySQL 5.7.20开始，不推荐使用查询缓存，并在MySQL 8.0中删除。**

##### 语法分析

客户端程序发送过来的请求只是一段文本而已，所以`MySQL`服务器程序首先要对这段文本做分析，判断请求的语法是否正确，然后从文本中将要查询的表、各种查询条件都提取出来放到`MySQL`服务器内部使用的一些数据结构上来。

##### 查询优化

服务器程序获得到了需要的信息，但光有这些是不够的，因为我们写的`MySQL`语句执行起来效率可能并不是很高，`MySQL`的优化程序会对我们的语句做一些优化，如外连接转换为内连接、表达式简化、子查询等。

优化的结果就是生成一个执行计划，这个执行计划表明了应该使用哪些索引进行查询，表之间的连接顺序是啥样的。

### 存储引擎

表是由一行一行的记录组成的，但这只是一个逻辑上的概念，物理上如何表示记录，怎么从表中读取数据，怎么把数据写入具体的物理存储器上，这都是存储引擎负责的事情。

`MySQL`提供了各式各样的存储引擎，不同存储引擎管理的表具体的存储结构可能不同，采用的存取算法也可能不同。

![image-20200714161639485](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714161639485.png)



<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714162154217.png" alt="image-20200714162154217" style="zoom:200%;" />

#### 设置表的存储引擎

储引擎是负责对表中的数据进行提取和写入工作的，我们可以为不同的表设置不同的存储引擎

##### 创建表时指定存储引擎

```mysql
CREATE TABLE 表名(    建表语句; ) ENGINE = 存储引擎名称;
```

##### 修改表的引擎

```sql
ALTER TABLE 表名 ENGINE = 存储引擎名称;
```

## 启动项和系统变量

### 在命令行上使用选项

如果我们在启动服务器程序的时候就禁止各客户端使用`TCP/IP`网络进行通信，可以在启动服务器程序的命令行里添加：

```sql
mysqld --skip-networking
mysqld --skip_networking
```

在命令行中**指定启动选项时需要在选项名前加上`--`前缀**，如果选项名是由多个单词构成的，它们之间可以由短划线`-`连接起来，也可以使用下划线`_`连接起来，也就是说上面含义相同。

如果我们想改变表的默认存储引擎的话，可以这样写启动服务器的命令行：

```sql
mysqld --default-storage-engine=MyISAM
```

使用`mysql --help`可以看到`mysql`程序支持的启动选项，`mysqld_safe --help`可以看到`mysqld_safe`程序支持的启动选项。查看`mysqld`支持的启动选项有些特别，需要使用`mysqld --verbose --help`

### 配置文件中使用选项

在命令行中设置启动选项只对当次启动生效，也就是说如果下一次重启程序的时候我们还想保留这些启动选项的话，还得重复把这些选项写到启动命令行中。

#### 配置文件路径

|                路径                 |                       备注                       |
| :---------------------------------: | :----------------------------------------------: |
| %WINDIR%\my.ini`， `%WINDIR%\my.cnf |                                                  |
|       C:\my.ini`， `C:\my.cnf       | 配置文件可以使用`.ini`的扩展名，也可以使用`.cnf` |
|  BASEDIR\my.ini`， `BASEDIR\my.cnf  |          `BASEDIR`指的是`MySQL`安装目录          |
|         defaults-extra-file         |           命令行指定的额外配置文件路径           |
|    %APPDATA%\MySQL\.mylogin.cnf     |            登录路径选项（仅限客户端）            |

```sql
mysqld --defaults-extra-file=C:\Users\xiaohaizi\my_extra_file.txt
```

就可以额外在`C:\Users\xiaohaizi\my_extra_file.txt`这个路径下查找配置文件

### 系统变量

每个系统变量都有一个默认值，我们可以使用命令行或者配置文件中的选项在启动服务器时改变一些系统变量的值。

大多数的系统变量的值也可以在程序运行过程中修改，而无需停止并重新启动它。

#### 查看系统变量

```sql
SHOW VARIABLES [LIKE 匹配的模式];
```

#### 设置系统变量

* 命令行

```sql
mysqld --default-storage-engine=MyISAM --max-connections=10
```

* 配置文件

```ini
[server]
default-storage-engine=MyISAM
max-connections=10s
```

#### 服务器运行过程中设置

作用范围分为这两种：

* `GLOBAL`：全局变量，影响服务器的整体操作。
* `SESSION`：会话变量，影响某个客户端连接的操作。（注：`SESSION`有个别名叫`LOCAL`）

以`default_storage_engine`举例，在服务器启动时会初始化一个名为`default_storage_engine`，作用范围为`GLOBAL`的系统变量。

之后每当有一个客户端连接到该服务器时，服务器都会单独为该客户端分配一个名为`default_storage_engine`，作用范围为`SESSION`的系统变量，该作用范围为`SESSION`的系统变量值按照当前作用范围为`GLOBAL`的同名系统变量值进行初始化。

通过启动选项设置的系统变量的作用范围都是`GLOBAL`的，也就是对所有客户端都有效的。

想在服务器运行过程中把作用范围为`GLOBAL`的系统变量`default_storage_engine`的值修改为`MyISAM`

```sql
SET GLOBAL default_storage_engine = MyISAM;
SET @@GLOBAL.default_storage_engine = MyISAM;
```

只想对本客户端生效

```sql
SET SESSION default_storage_engine = MyISAM;
SET @@SESSION.default_storage_engine = MyISAM;
SET default_storage_engine = MyISAM;
```

`SHOW VARIABLES`语句默认查看的是`SESSION`作用范围的系统变量.

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714170558424.png" alt="image-20200714170558424"  />

如果某个客户端改变了某个系统变量在`GLOBAL`作用范围的值，并不会影响该系统变量在当前已经连接的客户端作用范围为`SESSION`的值，只会影响后续连入的客户端在作用范围为`SESSION`的值。

* 并不是所有系统变量都具有`GLOBAL`和`SESSION`的作用范围。

  - 有一些系统变量只具有`GLOBAL`作用范围，比方说`max_connections`，表示服务器程序支持同时最多有多少个客户端程序进行连接。
  - 有一些系统变量只具有`SESSION`作用范围，比如`insert_id`，表示在对某个包含`AUTO_INCREMENT`列的表进行插入时，该列初始的值。
  - 有一些系统变量的值既具有`GLOBAL`作用范围，也具有`SESSION`作用范围，比如我们前边用到的`default_storage_engine`，而且其实大部分的系统变量都是这样的，

* 有些系统变量是只读的，并不能设置值。

  比方说`version`，表示当前`MySQL`的版本，我们客户端是不能设置它的值的，只能在`SHOW VARIABLES`语句里查看。

### 状态变量

`状态变量`是用来显示服务器程序运行状况的，所以它们的值只能由服务器程序自己来设置，我们程序员是不能设置的。与`系统变量`类似，`状态变量`也有`GLOBAL`和`SESSION`两个作用范围的。

```sql
SHOW [GLOBAL|SESSION] STATUS [LIKE 匹配的模式];
```

## 字符集和比较规则

### 字符集简介

将一个字符映射成一个二进制数据的过程也叫做编码，将一个二进制数据映射到一个字符的过程叫做解码。

![image-20200714172146807](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714172146807.png)

### 比较规则简介

比方说字符`'a'`的编码为`0x01`，字符`'b'`的编码为`0x02`，所以`'a'`小于`'b'`，这种简单的比较规则也可以被称为二进制比较规则。

但有时候并不符合现实需求，比如在很多场合对于英文字符我们都是不区分大小写的，也就是说`'a'`和`'A'`是相等的，在这种场合下就不能简单粗暴的使用二进制比较规则了。

同一种字符集可以有多种比较规则。

### 一些重要的字符集

* `ASCII`字符集

  共收录128个字符，包括空格、标点符号、数字、大小写字母和一些不可见字符

* `ISO 8859-1`字符集

  共收录256个字符，是在`ASCII`字符集的基础上又扩充了128个西欧常用字符

* `GB2312`字符集

  收录了汉字以及拉丁字母、希腊字母、日文平假名及片假名字母、俄语西里尔字母。其中收录汉字6763个，其他文字符号682个。又兼容`ASCII`字符集：

  - 如果该字符在`ASCII`字符集中，则采用1字节编码。
  - 否则采用2字节编码。

* `GBK`字符集

  字符集只是在收录字符范围上对`GB2312`字符集作了扩充，编码方式上兼容`GB2312`。

* `utf8`字符集

  收录地球上能想到的所有字符，而且还在不断扩充。这种字符集兼容`ASCII`字符集，采用变长编码方式，编码一个字符需要使用1～4个字节。

### MySQL中支持的字符集和排序规则

#### utf8和utf8mb4

- `utf8mb3`：阉割过的`utf8`字符集，只使用1～3个字节表示字符。
- `utf8mb4`：正宗的`utf8`字符集，使用1～4个字节表示字符。

在`MySQL`中`utf8`是`utf8mb3`的别名，所以之后在`MySQL`中提到`utf8`就意味着使用1~3个字节来表示一个字符，如果大家有使用4字节编码一个字符的情况，比如存储一些emoji表情啥的，那请使用`utf8mb4`。

查看当前`MySQL`中支持的字符集：

```sql
SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
```

#### 比较规则

查看`MySQL`中支持的比较规则的命令如下：

```sql
SHOW COLLATION [LIKE 匹配的模式];
```

`utf8_general_ci`是一种通用的比较规则。

### 字符集和比较规则的应用

#### 各级别的字符集和比较规则

`MySQL`有4个级别的字符集和比较规则，分别是：

- 服务器级别
- 数据库级别
- 表级别
- 列级别

##### 服务器级别

|       系统变量       |         描述         |
| :------------------: | :------------------: |
| character_set_server |  服务器级别的字符集  |
|   collation_server   | 服务器级别的比较规则 |



![image-20200714174307442](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714174307442.png)



![image-20200714174322541](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714174322541.png)

##### 数据库级别

![image-20200714174446945](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714174446945.png)



![image-20200714174550828](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714174550828.png)

 ***character_set_database*** 和 ***collation_database*** 这两个系统变量是只读的，我们不能通过修改这两个变量的值而改变当前数据库的字符集和比较规则。

##### 表级别

![image-20200714174959808](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714174959808.png)

如果创建和修改表的语句中没有指明字符集和比较规则，将使用该表所在数据库的字符集和比较规则作为该表的字符集和比较规则。

##### 列级别

![image-20200714175112455](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714175112455.png)

 如果在创建和修改的语句中没有指明字符集和比较规则，将使用该列所在表的字符集和比较规则作为该列的字符集和比较规则。

- 只修改字符集，则比较规则将变为修改后的字符集默认的比较规则。
- 只修改比较规则，则字符集将变为修改后的比较规则对应的字符集

不论哪个级别的字符集和比较规则，这两条规则都适用。

### MySQL中字符集的转换

从发送请求到接收结果过程中发生的字符集转换：

- 客户端使用操作系统的字符集编码请求字符串，向服务器发送的是经过编码的一个字节串。
- 服务器将客户端发送来的字节串采用`character_set_client`代表的字符集进行解码，将解码后的字符串再按照`character_set_connection`代表的字符集进行编码。
- 如果`character_set_connection`代表的字符集和具体操作的列使用的字符集一致，则直接进行相应操作，否则的话需要将请求中的字符串从`character_set_connection`代表的字符集转换为具体操作的列使用的字符集之后再进行操作。
- 将从某个列获取到的字节串从该列使用的字符集转换为`character_set_results`代表的字符集后发送到客户端。
- 客户端使用操作系统的字符集解析收到的结果集字节串。

|         系统变量         |                             描述                             |
| :----------------------: | :----------------------------------------------------------: |
|   character_set_client   |                 服务器解码请求时使用的字符集                 |
| character_set_connection | 服务器处理请求时会把请求字符串从`character_set_client`转为`character_set_connection` |
|  character_set_results   |             服务器向客户端返回数据时使用的字符集             |

## InnoDB记录结构

`InnoDB`是一个将表中的数据存储到磁盘上的存储引擎。

真正处理数据的过程是发生在内存中的，所以需要把磁盘中的数据加载到内存中，如果是处理写入或修改请求的话，还需要把内存中的内容刷新到磁盘上。

### 页简介

将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB中页的大小一般为 ***16*** KB。

一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中。

### 行格式

我们平时是以记录为单位来向表中插入数据的，这些记录在磁盘上的存放方式也被称为`行格式`或者`记录格式`。

到现在为止设计了Compact`、`Redundant`、`Dynamic`和`Compressed行格式。

![image-20200714185246684](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714185246684.png)



![image-20200714185437323](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714185437323.png)

#### Compact

##### 变长字段长度列表

变长字段中存储多少字节的数据是不固定的，所以我们在存储真实数据的时候需要顺便把这些数据占用的字节数也存起来。

变长字段占用的存储空间分为两部分：

1. 真正的数据内容
2. 占用的字节数

把**所有变长字段**的真实数据占用的**字节长度**都存放在记录的开头部位，从而形成一个**变长字段长度列表**

各变长字段数据占用的字节数按照**列的顺序逆序存放**。

具体用1个还是2个字节来表示真实数据占用的字节数，`InnoDB`有它的一套规则，先声明一下`W`、`M`和`L`的意思：

1. 假设某个字符集中表示一个字符最多需要使用的字节数为`W`
2. 对于变长类型`VARCHAR(M)`来说，这种类型表示能存储最多`M`个字符（注意是字符不是字节），所以这个类型能表示的字符串最多占用的字节数就是`M×W`。
3. 假设它实际存储的字符串占用的字节数是`L`。

* 如果`M×W <= 255`，那么使用1个字节来表示真正字符串占用的字节数。
* 如果`M×W >255`:
  * 如果`L <= 127`，则用1个字节来表示真正字符串占用的字节数。
  * 如果`L > 127`，则用2个字节来表示真正字符串占用的字节数。

变长字段长度列表中只存储值为 ***非NULL*** 的列内容占用的长度，值为 ***NULL*** 的列的长度是不储存的。

##### NULL值列表

`Compact`行格式把这些值为`NULL`的列统一管理起来，存储到`NULL`值列表中。

1. 首先统计表中允许存储***null***的列有哪些。
2. 若表中无允许存储 ***NULL*** 的列，则 *NULL值列表* 也无，否则为每个允许存储***Null***的列对应一个二进制，二进制位按照列的顺序逆序排列。意义如下：
   * 1->该列为***Null***
   * 0->该列不为***Null***
3. `MySQL`规定`NULL值列表`必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补`0`。

![image-20200714193252599](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714193252599.png)

##### 记录头信息

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714193504151.png" alt="image-20200714193504151" style="zoom:200%;" />

|     名称     | 大小（单位：bit） |                             描述                             |
| :----------: | :---------------: | :----------------------------------------------------------: |
|   预留位1    |         1         |                           没有使用                           |
|   预留位2    |         1         |                           没有使用                           |
| delete_mask  |         1         |                     标记该记录是否被删除                     |
| min_rec_mask |         1         |        B+树的每层非叶子节点中的最小记录都会添加该标记        |
|   n_owned    |         4         |                   表示当前记录拥有的记录数                   |
|   heap_no    |        13         |                表示当前记录在记录堆的位置信息                |
| record_type  |         3         | 表示当前记录的类型，`0`表示普通记录，`1`表示B+树非叶子节点记录，`2`表示最小记录，`3`表示最大记录 |
| next_record  |        16         |                   表示下一条记录的相对位置                   |

##### 记录的真实数据

`MySQL`会为每个记录默认的添加一些列（也称为`隐藏列`）

|      列名      | 是否必须 | 占用空间/（字节） |          描述          |
| :------------: | :------: | :---------------: | :--------------------: |
|     row_id     |    否    |         6         | 行ID，唯一标识一条记录 |
| transaction_id |    是    |         6         |         事务ID         |
|  roll_pointer  |    是    |         7         |        回滚指针        |

`InnoDB`表对主键的生成策略：优先使用用户自定义主键作为主键，如果用户没有定义主键，则选取一个`Unique`键作为主键，如果表中连`Unique`键都没有定义的话，则`InnoDB`会为表默认添加一个名为`row_id`的隐藏列作为主键。

##### CHAR（M）列的存储格式

对于 ***CHAR(M)*** 类型的列来说，当列采用的是定长字符集时，该列占用的字节数不会被加到变长字段长度列表，而如果采用变长字符集时，该列占用的字节数也会被加到变长字段长度列表。

对于使用`utf8`字符集的`CHAR(10)`的列来说：

该列存储的数据字节长度的范围是10～30个字节。

即使我们向该列中存储一个空字符串也会占用`10`个字节，这是怕将来更新该列的值的字节长度大于原有值的字节长度而小于10个字节时，可以在该记录处直接更新，而不是在存储空间中重新分配一个新的记录空间，导致原有的记录空间成为所谓的碎片。

#### Redundant行格式

`MySQL5.0`之前用的一种行格式。



![image-20200714195940030](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714195940030.png)

会把该条记录中所有列（包括`隐藏列`）的长度信息都按照逆序存储到`字段长度偏移列表`。

`字段长度偏移列表`实质上是存储每个列中的值占用的空间在`记录的真实数据`处结束的位置。

#### 行溢出数据

##### VARCHAR(M)最多能存储的数据

除了`BLOB`或者`TEXT`类型的列之外，其他所有的列（不包括隐藏列和记录头信息）占用的字节长度加起来不能超过`65535`个字节.

存储一个`VARCHAR(M)`类型的列，其实需要占用3部分存储空间：

- 真实数据
- 真实数据占用字节的长度
- `NULL`值标识，如果该列有`NOT NULL`属性则可以没有这部分存储空间

如果该`VARCHAR`类型的列没有`NOT NULL`属性，那最多只能存储`65532`个字节的数据，因为真实数据的长度可能占用2个字节，`NULL`值标识需要占用1个字节：

```sql
mysql> CREATE TABLE varchar_size_demo(
    ->      c VARCHAR(65532)
    -> ) CHARSET=ascii ROW_FORMAT=Compact;
Query OK, 0 rows affected (0.02 sec)
```

如果`VARCHAR`类型的列有`NOT NULL`属性，那最多只能存储`65533`个字节的数据，因为真实数据的长度可能占用2个字节，不需要`NULL`值标识：

```sql
mysql> CREATE TABLE varchar_size_demo(
    ->      c VARCHAR(65533) NOT NULL
    -> ) CHARSET=ascii ROW_FORMAT=Compact;
Query OK, 0 rows affected (0.02 sec)
```

如果`VARCHAR(M)`类型的列使用的不是`ascii`字符集,那`M`的最大取值取决于该字符集表示一个字符最多需要的字节数。

gbk`字符集表示一个字符最多需要`2`个字节，那在该字符集下，`M`的最大取值就是`32766.

##### 记录中的数据太多产生的溢出

```sql
mysql> CREATE TABLE varchar_size_demo(
    ->       c VARCHAR(65532)
    -> ) CHARSET=ascii ROW_FORMAT=Compact;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO varchar_size_demo(c) VALUES(REPEAT('a', 65532));
Query OK, 1 row affected (0.00 sec)

```

`REPEAT('a', 65532)`是一个生成一个把字符`'a'`重复`65532`次的字符串的函数调用。

`MySQL`是以`页`为基本单位来管理存储空间的，我们的记录都会被分配到某个`页`中存储。

一个页的大小一般是`16KB`，也就是`16384`字节，而一个`VARCHAR(M)`类型的列就最多可以存储`65532`个字节，这样就可能造成一个页存放不了一条记录的尴尬情况。

在`Compact`和`Redundant`行格式中，对于占用存储空间非常大的列，在`记录的真实数据`处只会存储该列的一部分数据，把剩余的数据分散存储在几个其他的页中，然后`记录的真实数据`处用20个字节存储指向这些页的地址（当然这20个字节中还包括这些分散在其他页面中的数据的占用的字节数）

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200714203602412.png" alt="image-20200714203602412" style="zoom:200%;" />

存储超出`768`字节的那些页面也被称为`溢出页`。

##### 行溢出的临界点

`MySQL`中规定一个页中至少存放两行记录。

我们一条记录的某个列中存储的数据占用的字节数非常多时，该列就可能成为`溢出列`。

##### Dynamic和Compressed行格式

这俩行格式和`Compact`行格式挺像

它们不会在记录的真实数据处存储字段真实数据的前`768`个字节，而是把所有的字节都存储到其他页面中，只在记录的真实数据处存储其他页面的地址。

## InnoDB数据页结构

存放表中记录的页称为索引页，表中的记录就是数据，暂时叫数据页。

![image-20200715095724113](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715095724113.png)

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715095558645.png" alt="image-20200715095558645" style="zoom: 33%;" />

### 记录在页中的存储

在一开始生成页的时候，其实并没有`User Records`这个部分，每当我们插入一条记录，都会从`Free Space`部分，也就是尚未使用的存储空间中申请一个记录大小的空间划分到`User Records`部分，当`Free Space`部分的空间全部被`User Records`部分替代掉之后，也就意味着这个页使用完了

![image-20200715100055999](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715100055999.png)

#### 记录头信息

```sql
mysql> CREATE TABLE page_demo(
    ->     c1 INT,
    ->     c2 INT,
    ->     c3 VARCHAR(10000),
    ->     PRIMARY KEY (c1)
    -> ) CHARSET=ascii ROW_FORMAT=Compact;
Query OK, 0 rows affected (0.03 sec)
```

下图为行格式示意图

![image-20200715100542303](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715100542303.png)

```sql
mysql> INSERT INTO page_demo VALUES(1, 100, 'aaaa'), (2, 200, 'bbbb'), (3, 300, 'cccc'), (4, 400, 'dddd');
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0
```

记录头和实际列数据在页中存储：

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715102621858.png" alt="image-20200715102621858" style="zoom:200%;" />

* delete_mask

  这个标志位表示是否删除，如果删除为1，如果未删除为0；

  这些记录不是立即从磁盘上删除，是为了减小删除后重拍带来的性能消耗，垃圾链表组成可重用空间，之后新插入的数据可将其覆盖。

* min_rec_mask

* n_owned

* heap_no

  表示在本页的位置，`InnoDB`自动插入了两条记录为伪记录，分别为最大记录最小记录（比较记录就是通过主键比较，但是无论如何，最大，小的记录都由`InnoDB`创建）

  这两条记录存放在`Infimum + Supremum`

* record_type

  表示当前记录类型。

  `0`表示普通记录，`1`表示B+树非叶节点记录，`2`表示最小记录，`3`表示最大记录。

* next_record

  表示从当前记录的真实数据到下一条记录的真实数据的地址偏移量。

  下一条记录是按照主键大小顺序排列的下一条记录。

  用链表表示就是

  ![image-20200715104047881](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715104047881.png)

当删除数据后，next_record值为0，表示没有下一条记录。

#### Page Directory

如果按照单链表进行查询，每次查询都要遍历链表很麻烦。

因此`InnoDB`制作了目录，制作过程如下：

1. 将所有的记录（包括最大最小，不包括已删除记录）分组。
2. 每个组的最后一条记录中的`n_owned`表示该组中有多少条记录。
3. 将每个组的最后一条记录提取出来放在靠近页的尾部地方，组成`Page Directory`,地址偏移量成为槽。

最小记录所在的分组只有一条记录；

最大记录所在分组只能有1-8条之间；

剩下的分组中记录在4-8条之间

分组步骤：

1. 初始情况只有最大最小记录，因此就两个组；
2. 每插入一条记录，会在`Page Directory`中找到比待插入记录的主键值大且差值最小的槽，然后该槽对应的记录的`n_owned`加1，直到为8.
3. 当一个组的记录数为8时，会拆成两个组，一个4条记录，一个5条记录。并新增一个槽。

```sql
mysql> INSERT INTO page_demo VALUES(5, 500, 'eeee'), (6, 600, 'ffff'), (7, 700, 'gggg'), (8, 800, 'hhhh'), (9, 900, 'iiii'), (10, 1000, 'jjjj'), (11, 1100, 'kkkk'), (12, 1200, 'llll'), (13, 1300, 'mmmm'), (14, 1400, 'nnnn'), (15, 1500, 'oooo'), (16, 1600, 'pppp');
Query OK, 12 rows affected (0.00 sec)
Records: 12  Duplicates: 0  Warnings: 0
```

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715111533069.png" alt="image-20200715111533069" style="zoom:200%;" />

查找的时候是采用二分法查找（例如查主键为6）：

1. high=4,low=0；计算中间槽的位置：`(0+4)/2=2`，槽2对应的主键值为8，8>6，则high=2；
2. high=2,low=0；计算中间槽的位置：`(0+2)/2=1`，槽1对应的主键值为4，4<6，则low=1；
3. high-low=1，则说明在槽2中，因此遍历槽2就好。

#### Page Header

|       名称        | 占用空间/（字节） |                             描述                             |
| :---------------: | :---------------: | :----------------------------------------------------------: |
| PAGE_N_DIR_SLOTS  |         2         |                      在页目录中的槽数量                      |
|   PAGE_HEAP_TOP   |         2         | 还未使用的空间最小地址，也就是说从该地址之后就是`Free Space` |
|    PAGE_N_HEAP    |         2         | 本页中的记录的数量（包括最小和最大记录以及标记为删除的记录） |
|     PAGE_FREE     |         2         | 第一个已经标记为删除的记录地址（各个已删除的记录通过`next_record`也会组成一个单链表，这个单链表中的记录可以被重新利用） |
|   PAGE_GARBAGE    |         2         |                    已删除记录占用的字节数                    |
| PAGE_LAST_INSERT  |         2         |                      最后插入记录的位置                      |
|  PAGE_DIRECTION   |         2         |                        记录插入的方向                        |
| PAGE_N_DIRECTION  |         2         |                  一个方向连续插入的记录数量                  |
|    PAGE_N_RECS    |         2         | 该页中记录的数量（不包括最小和最大记录以及被标记为删除的记录） |
|  PAGE_MAX_TRX_ID  |         8         |        修改当前页的最大事务ID，该值仅在二级索引中定义        |
|    PAGE_LEVEL     |         2         |                   当前页在B+树中所处的层级                   |
|   PAGE_INDEX_ID   |         8         |                索引ID，表示当前页属于哪个索引                |
| PAGE_BTR_SEG_LEAF |        10         |          B+树叶子段的头部信息，仅在B+树的Root页定义          |
| PAGE_BTR_SEG_TOP  |        10         |         B+树非叶子段的头部信息，仅在B+树的Root页定义         |

* PAGE_DIRECTION：新插的记录主键比上一条大，这条记录的插入方向就是右边，反之左边，最后一条记录的插入方向就是PAGE_DIRECTION。
* PAGE_N_DIRECTION：连续几次方向一致，就会记录下来，若最后一次的方向改变，就清零。

#### File Header

Page Header记录数据页的状态信息，File Header针对各种页都通用。

|               名称               | 占用空间大小/（字节） |                             描述                             |
| :------------------------------: | :-------------------: | :----------------------------------------------------------: |
|     FIL_PAGE_SPACE_OR_CHKSUM     |           4           |                   页的校验和（checksum值）                   |
|         FIL_PAGE_OFFSET          |           4           |          页号，类似于身份证号，InnoDB用来定位一个页          |
|          FIL_PAGE_PREV           |           4           |                        上一个页的页号                        |
|          FIL_PAGE_NEXT           |           4           |                        下一个页的页号                        |
|           FIL_PAGE_LSN           |           8           | 页面被最后修改时对应的日志序列位置（英文名是：Log Sequence Number） |
|          FIL_PAGE_TYPE           |           2           |                          该页的类型                          |
|     FIL_PAGE_FILE_FLUSH_LSN      |           8           | 仅在系统表空间的一个页中定义，代表文件至少被刷新到了对应的LSN值 |
| FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID |           4           |                       页属于哪个表空间                       |

校验和：就是对于一个很长很长的字节串来说，我们会通过某种算法来计算一个比较短的值来代表这个很长的字节串，这个比较短的值就称为校验和。这样在比较两个很长的字节串之前先比较这两个长字节串的校验和，如果校验和都不一样两个长字节串肯定是不同的。

* FIL_PAGE_TYPE:页的类型

|        类型名称         | 十六进制 |              描述              |
| :---------------------: | :------: | :----------------------------: |
| FIL_PAGE_TYPE_ALLOCATED |  0x0000  |       最新分配，还没使用       |
|    FIL_PAGE_UNDO_LOG    |  0x0002  |           Undo日志页           |
|     FIL_PAGE_INODE      |  0x0003  |           段信息节点           |
| FIL_PAGE_IBUF_FREE_LIST |  0x0004  |     Insert Buffer空闲列表      |
|  FIL_PAGE_IBUF_BITMAP   |  0x0005  |       Insert Buffer位图        |
|    FIL_PAGE_TYPE_SYS    |  0x0006  |             系统页             |
|  FIL_PAGE_TYPE_TRX_SYS  |  0x0007  |          事务系统数据          |
|  FIL_PAGE_TYPE_FSP_HDR  |  0x0008  |         表空间头部信息         |
|   FIL_PAGE_TYPE_XDES    |  0x0009  |           扩展描述页           |
|   FIL_PAGE_TYPE_BLOB    |  0x000A  |             溢出页             |
|     FIL_PAGE_INDEX      |  0x000B  | 索引页，也就是我们所说的数据页 |

#### File Trailer

在修改后的某个时间需要把数据同步到磁盘中。但是在同步了一半的时候中断电了，为了检测一个页是否完整（也就是在同步的时候有没有发生只同步一半的尴尬情况），设计者在每个**页的尾部**都加了一个`File Trailer`部分，这个部分由`8`个字节组成，可以分成2个小部分：

- 前4个字节代表页的校验和

  这个部分是和`File Header`中的校验和相对应的。

  每当一个页面在内存中修改了，在同步之前就要把它的校验和算出来，因为`File Header`在页面的前边，所以校验和会被首先同步到磁盘。

  当完全写完时，校验和也会被写到页的尾部，如果完全同步成功，则页的首部和尾部的校验和应该是一致的。

  如果写了一半儿断电了，那么在`File Header`中的校验和就代表着已经修改过的页，而在`File Trailer`中的校验和代表着原先的页，二者不同则意味着同步中间出了错。

- 后4个字节代表页面被最后修改时对应的日志序列位置（LSN）

  这个部分也是为了校验页的完整性的。

## B+树索引

### 没有索引查找

#### 在一个页中的查找

* 以主键为搜索条件，通过页目录中使用二分法找槽，再进行遍历
* 以其他列为搜索条件，就无法通过二分搜索了。

#### 在很多页中查找

在没有索引的情况下，不论是主键列还是其他列都得先定义到记录所在的页，因此就得先遍历所有页，之后再对记录进行查找。

### 索引

因为各个页中的记录并没有规律，我们并不知道搜索条件匹配哪些页中的记录，因此需要遍历所有数据页。

```sql
mysql> CREATE TABLE index_demo(
    ->     c1 INT,
    ->     c2 INT,
    ->     c3 CHAR(1),
    ->     PRIMARY KEY(c1)
    -> ) ROW_FORMAT = Compact;
Query OK, 0 rows affected (0.03 sec)
```

因此，尝试建立目录：

* 下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值。

  约定：一个页最多存储3条记录（假设的，实际很多）；

  插入数据：

  ```sql
  mysql> INSERT INTO index_demo VALUES(1, 4, 'u'), (3, 9, 'd'), (5, 3, 'y');
  Query OK, 3 rows affected (0.01 sec)
  Records: 3  Duplicates: 0  Warnings: 0
  ```

  页结构为：

  <img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715131216565.png" alt="image-20200715131216565" style="zoom: 67%;" />

  新分配的数据页在存储空间可能并不是挨着的：

  当再插入一条数据时：

  ```sql
  mysql> INSERT INTO index_demo VALUES(4, 4, 'a');
  Query OK, 1 row affected (0.00 sec)
  ```

  <img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715131451087.png" alt="image-20200715131451087" style="zoom:67%;" />

  下面得进行页分裂：

  <img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715131549586.png" alt="image-20200715131549586" style="zoom:67%;" />

* 给所有的页建立目录

  <img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715131812939.png" alt="image-20200715131812939" style="zoom:67%;" />

  每个页做一个目录项：

  * 页中记录的最小主键作为key；
  * 页号作为page_no

  ![image-20200715132352339](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715132352339.png)



此目录页称为索引。

后来把目录项作为记录进行存储，其中record_type的值为1的时候为说明是一个目录项记录。

![image-20200715140420651](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715140420651.png)

但是一个页只能有16KB的大小，因此，如果表中数据太多，那么就会分成多一个的目录项记录：

![image-20200715142021466](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715142021466.png)

上图是假设，日志记录页只能存四条日志项记录。

因此查询的步骤：

1. 确定日志项记录的页；
2. 在日志项记录页查询出用户记录真实存在的页；
3. 在真实存在的页查找到记录。

但是第一部中日志项记录页的定位就成了问题，因此增加更高一级的目录：

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715142413227.png" alt="image-20200715142413227" style="zoom:200%;" />

一般用到的B＋树的高度不会超过四层。

##### 聚簇索引

B＋树有以下特点；

1. 使用记录主键值的大小进行记录和页的排序:
   * 页内的记录按照主键大小顺序排列成一个单链表
   * 各个用户记录页根据主键大小顺序排成一个双链表
   * 同一层的目录项记录也是根据页中目录项记录的主键大小排成双链表
2. B+树的叶子节点都是存放的都是用户记录.

具有上述两种特性的B+树称为聚簇索引;

聚簇索引并不需要我们手动创建,在InnoDB中,聚簇索引就是数据的存储方式.

##### 二级索引

上面的聚簇索引只有当搜索条件是主键时才能发挥作用;

当搜索条件不是主键的时候,需要多建几棵树,不同的B+树的排序规则不同。

以C2列的大小作为排序规则：

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715145424083.png" alt="image-20200715145424083" style="zoom:200%;" />

使用C2列为排序规则：

* 页面内的记录按照C2列的顺序大小排成一个单链表
* 用户记录的页根据页中记录的C2列的大小顺序排成一个双向链表
* 同一层次目录项记录页按照C2列的大小顺序排成一个双向链表

叶子节点中存储的是主键及C2列的的值；

目录记录中是C2+页号；

以查找C2列为4的记录：

1. 确定目录项记录，根据根页面，定位出页42；
2. 通过目录页查找真实页，但是2<4≤ 4,所以用户记录页在34和35页中；
3. 根据所在页定位到真实记录；
4. 根据主键值在聚簇索引中再查找一遍用户记录（回表）。

不将所有的记录存放在叶子节点是因为太占地方了，因为，没建立一次树就会存放一次数据，性能消耗太大。

上述的树也称为二级索引，也称辅助索引。

#### 联合索引

同时以多个列作为排序规则建立索引。

例如想按照C2和C3列的大小进行排序：

* 先把各个记录和页按照C2进行排序；
* C2列相同时，按照C3进行排序。

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715152552404.png" alt="image-20200715152552404" style="zoom:200%;" />

#### InnoDB的B+索引注意事项

##### 根页面不动

* 每当为一个表在创建B+索引的时候，会首先创建一个根页面，表中无数据的时候根页面页无数据。
* 向表中添加数据的时候，先将用户数据插入到该页面中。
* 之后，当该页面的可用空间没有了，还继续插入记录，此时就会将根页面复制到一个新页面a，然后新页面进行页分裂，分裂成页面a，页面b。之后将这个记录根据排序规则分配到页面a或页面b；
* 根页面升级为目录项存储页。

##### 内节点中目录项记录的唯一性

对于***索引列+页号***的搭配对于二级索引来说不严谨。

当表中的数据是：

![image-20200715154955160](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715154955160.png)

此时只用页号和索引列的值：

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715154850058.png" alt="image-20200715154850058" style="zoom:67%;" />

当插入（9，1，`‘c’`）的时候，没有办法知道是去页面4还是页面5。

所以对于二级索引是由三部分：

* 索引列值
* 主键值
* 页号

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715155303291.png" alt="image-20200715155303291" style="zoom: 80%;" />

为了B+树的构建，保证每个页面最少有两条存储记录，因为如果只有一条那分层的树形效果不明显。

## B+树索引的使用

```sql
CREATE TABLE person_info(
    id INT NOT NULL auto_increment,
    name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    phone_number CHAR(11) NOT NULL,
    country varchar(100) NOT NULL,
    PRIMARY KEY (id),
    KEY idx_name_birthday_phone_number (name, birthday, phone_number)
);
```

该表会为聚簇索引和上述的联合索引建立两棵B+树

下图为联合索引的B+树，内节点中保留有`name`、`birthday`、`phone_number`、`id`这四个列的真实数据值：

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200715162416966.png" alt="image-20200715162416966" style="zoom:200%;" />

* 会先按照name进行排序；
* name相同按照birthday进行排序；
* birthday相同就按照phone_number进行排序。

### B+树索引适用条件

#### 全值匹配

如果搜索条件与索引列一致，那么就称为全值匹配，例如下方的语句：

```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1990-09-27' AND phone_number = '15123983239';
```

#### 匹配最左边的列

其实在搜索时，不一定需要包含全部的联合索引的列，只包含左边的就行：

```sql
SELECT * FROM person_info WHERE name = 'Ashburn';
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday = '1990-09-27';

```

向下面的就用不到索引：

```sql
SELECT * FROM person_info WHERE birthday = '1990-09-27';
```

因为是在name相同的情况下才对birthday进行排序；

同理：

```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND phone_number = '15123983239';
```

只会用到name索引。

#### 匹配列前缀

比如在对name列进行排序的时候，是逐个比较字符的大小，比如第一个字符相同，就比较第二个字符；一直比到字符不相同的时候就是有比较的结果了。

也就是这些前缀部分是排好的，因此，我们可以通过前缀快速定位记录:

```sql
SELECT * FROM person_info WHERE name LIKE 'As%';
```

但是，如果给出后缀和中间的是无法快速定位的，只能通过全表扫描：

```sql
SELECT * FROM person_info WHERE name LIKE '%As%';
SELECT * FROM person_info WHERE name LIKE '%As';
```

如果非得查后缀，必须得先将表进行逆序，然后通过查前缀的方式进行：

```tex
想查表中url字段以com为后缀的记录，url是索引列
+----------------+
| url            |
+----------------+
| www.baidu.com  |
| www.google.com |
| www.gov.cn     |
| ...            |
| www.wto.org    |
+----------------+
WHERE url LIKE '%com'（会扫描全表）

+----------------+
| url            |
+----------------+
| moc.udiab.www  |
| moc.elgoog.www |
| nc.vog.www     |
| ...            |
| gro.otw.www    |
+----------------+
WHERE url LIKE 'moc%'（会使用索引）
```

#### 匹配范围值

```sql
SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow';
```

查询过程：

* 找到name为‘Asa’的记录
* 找到name为‘Barlow’的记录
* 记录之间单链表，数据页之间双链表，取出中间的记录的主键
* 回表取出完整记录

多个列进行范围查找的时候，只有最左边的列会用到B+索引

```sql
SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow' AND birthday > '1980-01-01';
```

查询过程：

1. 通过`name > 'Asa' AND name < 'Barlow'`查到结果；
2. 再通过`birthday > '1980-01-01'`进行条件过滤。

但是上述的第2步中，用不到索引，因为只有name相同的时候才会使用`birthday`进行排序。但是上述的结果集中的name并不是相同的。所以`birthday`不会使用到索引。

#### 精确匹配某一列并范围匹配到另外一列

当最左列是精确匹配的时候，右边的列可以使用范围匹配：

```sql
SELECT * FROM person_info WHERE name = 'Ashburn' AND birthday > '1980-01-01' AND birthday < '2000-12-31' AND phone_number > '15100000000';
```

查询过程：

1. name精确匹配，会使用到索引；
2. `birthday > '1980-01-01'AND birthday < '2000-12-31'`因为name是精确匹配，name值得到的结果是相同的，因此birthday会使用到索引。
3. `phone_number > '15100000000'`不会使用到索引，因为通过birthday得到的结果中birthday值不相同。

#### 用于排序

在写查询语句的时候需要对查询出来的记录通过order by进行排序；

一般是先把记录加载到内存中，再通过排序算法进行排序；

但是当结果集很大的时候，需要借助磁盘进行存放中间结果；

这种排序统称为文件排序。

因此，就会很慢，但是如果order by字句中的条件是索引列，就可以省去在内存中或文件中排序的步骤：

```sql
SELECT * FROM person_info ORDER BY name, birthday, phone_number LIMIT 10;
```

直接从索引中提取数据，然后回表，就可以了。

#### 不可以使用索引进行排序的几种情况

##### ASC、DESC混用

* ORDER BY name, birthday LIMIT 10

  这种情况就是从最左边往右读10条记录

* ORDER BY name DESC, birthday DESC LIMIT 10

  这种情况就是从最右边往左读10条记录

但是如果先按照`name`列进行升序排列，再按照`birthday`列进行降序排列

```sql
SELECT * FROM person_info ORDER BY name, birthday DESC LIMIT 10;
```

* 先从索引的最左边确定name列的最小值，然后找到name列等于该值的所有记录（就是找到name所有的最小值），然后从这个地方往右找10个。
* 如果最小值记录不足10，就从第二小接着找，直到够10条数据。

##### 排序列包含非同一个索引的列

```sql
SELECT * FROM person_info ORDER BY name, country LIMIT 10;
```

country 与name并不属于一个联合索引中的列。

##### 排序列使用了复杂的表达式

使用索引进行排序操作，必须保证索引列以单独列的形式出现，而不是修饰过的形式，

```sql
SELECT * FROM person_info ORDER BY UPPER(name) LIMIT 10;
```

### 用于分组

```sql
SELECT name, birthday, phone_number, COUNT(*) FROM person_info GROUP BY name, birthday, phone_number;
```

1. 所有name值相同的记录划为一组。
2. 在name值相同的前提下，将birthday值相同的记录划为一组。
3. 再在上述两个前提下，将phone_number值相同的记录划分为一个小组。

### 回表的代价

```sql
SELECT * FROM person_info WHERE name > 'Asa' AND name < 'Barlow';
```

因为索引会首先按照name进行排序，因此Asa-Barlow之间的记录是连续的，我们可以很快地从磁盘中读出来。这种读取方式称为**顺序I/O**。

根据第一步获取的主键值不一定在聚簇索引上是连续的，从不连续的id值到聚簇索引中访问完整的用户记录可能分布在不同页中，这样的读取方式称为**随机I/O**。

顺序I/O的效率比随机I/O高。

因此，需要回表的记录越多，使用二级索引的效率越低。

为了规避回表带来的性能消耗，最好在查询列表里只包含索引列：

```sql
SELECT name, birthday, phone_number FROM person_info WHERE name > 'Asa' AND name < 'Barlow'
```

通过第一步查找之后，就不需要去聚簇索引进行里进行回表操作。

这种只需要用到索引的查询方式称为索引覆盖。

### 如何挑选索引

#### 只为用于搜索、排序或分组的列创建索引

只为出现在`where`,`group by`,`order by`的子句中的列创建索引。

出现在查询列表中的列就没必要创建索引了。

```sql
SELECT birthday, country FROM person_name WHERE name = 'Ashburn';
```

birthday和country字段没必要创建索引，只需要在name列创建索引即可。

#### 考虑列的基数

列的基数指的是某一列中不重复数据的个数，比如2，5，8，5，8，2，2，5，8基数为3。

假设某个列的基数为1，那么所有记录在该列的值都一样。

也就是列的基数越大，该列中的值越分散。基数越小，该列中的值越集中。

因此，尽量为基数大的列建立索引，为基数小的列建立索引效果不好。

#### 索引列的类型尽量小

在表示的整数范围允许的情况下，尽量让索引列使用较小的类型。

* 数据类型越小，查询时进行的比较操作越快
* 数据类型越小，索引占用的存储空间越小，一个数据业内可以放下更多的记录，减少磁盘I/O的性能损耗，可以放更多的数据在内存中，加快读写效率。
* 对于主键列更应该选用小的数据类型，因为不光会建立聚簇索引，还有二级索引的节点也会存储主键列。

#### 索引字符串值的前缀

为字符串列建立索引时的问题：

* B+树索引中的记录需要把该列的完整字符串存储起来，字符串越长，在索引中占用的存储空间越大。
* B+树索引中索引列存储的字符串越长，在字符串比较时月耗时。

解决方案：

只对字符串的前几个字符进行索引。

在二级索引的记录中只保留字符串前几个字符，在查找是虽然不能精确的定位记录的位置，但是可以找到前缀所在的位置。根据前缀相同的记录的主键值回表查询完整的字符串值，再对比就好了。

```sql
CREATE TABLE person_info(
    name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    phone_number CHAR(11) NOT NULL,
    country varchar(100) NOT NULL,
    KEY idx_name_birthday_phone_number (name(10), birthday, phone_number)
);    
```

name(10)表示只保留记录的前10个字符。

索引前缀不可精确匹配也会有不适用的情况：

```sql
SELECT * FROM person_info ORDER BY name LIMIT 10;
```

因为无法展示完全name的值，因此上面的查询方法会对钱十个字符相同，后边字符不相同的字符进行排序。

也就是索引列前缀的方式无法支持使用索引排序。

#### 让索引列在比较表达式中单独出现

* WHERE my_col * 2 < 4
* WHERE my_col < 4/2

上面的子句，语义一样，但是效率不同：

* 第一个会依次遍历所有的记录，计算这个表达式的值是不是小于4。这种情况下使用不到为my_col列建立的B+树索引。
* 第二个子句中可以直接使用B+树索引。

#### 主键插入顺序

当没有显示创建索引时，表的数据存储在聚簇索引的叶子节点上。

而记录又是存储在数据页上，数据页和记录是按照主键值从小到大的顺序进行排列。

因此，如果插入的记录的主键值依次增大，没插满一个数据页就换到下一个数据页。

如果忽大忽小，就会把进行页面分裂，记录移位等操作，就会造成性能损耗。

#### 冗余和重复索引

```sql
CREATE TABLE person_info(
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    birthday DATE NOT NULL,
    phone_number CHAR(11) NOT NULL,
    country varchar(100) NOT NULL,
    PRIMARY KEY (id),
    KEY idx_name_birthday_phone_number (name(10), birthday, phone_number),
    KEY idx_name (name(10))
); 

```

```sql
CREATE TABLE repeat_index_demo (
    c1 INT PRIMARY KEY,
    c2 INT,
    UNIQUE uidx_c1 (c1),
    INDEX idx_c1 (c1)
);  
```

## MySQL的数据目录

MySQL服务器程序在启动时会到文件系统的某个目录下加载一些文件，之后在运行过程中产生的数据也都会存储到这个目录下的某些文件中，这个目录就称为***数据目录***

![image-20200716094902591](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716094902591.png)

### 数据目录结构

#### 数据库在文件系统中的表示

```sql
CREATE DATABASE 数据库名
```

`MySQL`会帮我们做这两件事儿：

1. 在***数据目录***下创建一个和数据库名同名的子目录（或者说是文件夹）。
2. 在该与数据库名同名的子目录下创建一个名为`db.opt`的文件，这个文件中包含了该数据库的各种属性，比方说该数据库的字符集和比较规则是个啥。

数据目录下的内容：

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716095212752.png" alt="image-20200716095212752" style="zoom: 50%;" />

#### 表在文件系统中的表示

每个表的信息其实可以分为两种：1.表结构定义；2. 表中数据

1. 表结构的定义：该表的名称是啥，表里边有多少列，每个列的数据类型是啥，有啥约束条件和索引，用的是啥字符集和比较等各种信息。（表名.frm）

   <img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716095621472.png" alt="image-20200716095621472" style="zoom:50%;" />

   至于表数据在不同的存储引擎就不同。

##### InnoDB是如何存储表数据

***表空间***或者***文件空间***（英文名：`table space`或者`file space`）的概念，这个表空间是一个抽象的概念，它可以对应文件系统上一个或多个真实文件（不同表空间对应的文件数量可能不同）。

每一个***表空间***可以被划分为很多很多很多个页，我们的表数据就存放在某个表空间下的某些页里。

###### 系统表空间（system tablespace）

对应文件系统上一个或多个实际的文件。

在数据目录下个名为`ibdata1`。

这个文件是所谓的自扩展文件，也就是当不够用的时候它会自己增加文件大小。

想让系统表空间对应文件系统上多个实际文件，或者仅仅觉得原来的`ibdata1`这个文件名难听，那可以在`MySQL`启动时配置对应的文件路径以及它们的大小，比如我们这样修改一下配置文件：

```ini
[server]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

`MySQL`启动之后就会创建这两个512M大小的文件作为***系统表空间***，其中的`autoextend`表明这两个文件如果不够用会自动扩展`data2`文件的大小

###### 独立表空间(file-per-table tablespace)

在MySQL5.6.6以及之后的版本中，`InnoDB`并不会默认的把各个表的数据存储到系统表空间中，而是为每一个表建立一个独立表空间，也就是说我们创建了多少个表，就有多少个独立表空间。

使用***独立表空间***来存储表数据的话，会在该表所属数据库对应的子目录下创建一个表示该***独立表空间***的文件，文件名和表名相同，只不过添加了一个`.ibd`的扩展名.

我们也可以自己指定使用***系统表空间***还是***独立表空间***来存储数据，这个功能由启动参数`innodb_file_per_table`控制。

我们想刻意将表数据都存储到***系统表空间***时

```ini
[server]
innodb_file_per_table=0
```

当`innodb_file_per_table`的值为`1`时，代表使用独立表空间。不过`innodb_file_per_table`参数只对新建的表起作用，对于已经分配了表空间的表并不起作用。

##### MyISAM是如何存储表数据的

在`MyISAM`中的索引全部都是二级索引，该存储引擎的数据和索引是分开存放的.

所以在文件系统中也是使用不同的文件来存储数据文件和索引文件。

表数据都存放到对应的数据库子目录下.

假如`person`表使用`MyISAM`存储引擎的话，那么在它所在数据库对应的`test`目录下会为`person`表创建这三个文件：

```tex
person.frm
person.MYD-----表的数据文件，也就是我们插入的用户记录
person.MYI-----表的索引文件，我们为该表创建的索引都会放到这个文件中
```

#### 视图在文件系统中的表示

我们知道`MySQL`中的视图其实是虚拟的表，也就是某个查询语句的一个别名而已，所以在存储视图的时候是不需要存储真实的数据的，只需要把它的结构存储起来就行了。

和表一样，描述视图结构的文件也会被存储到所属数据库对应的子目录下边，只会存储一个***视图名.frm***的文件。

### 文件系统对数据库的影响

* 数据库名称和表名称不得超过文件系统所允许的最大长度。
* `MySQL`会把数据库名和表名中所有除数字和拉丁字母以外的所有字符在文件名里都映射成 `@+编码值`的形式作为文件名。
* 文件长度受文件系统最大长度限制

## InnoDB的表空间

* 表空间中的每一个页都对应着一个页号，也就是`FileHeader`中的`FIL_PAGE_OFFSET`，这个页号由4个字节组成，也就是32个比特位，所以一个表空间最多可以拥有2³²个页，如果按照页的默认大小16KB来算，一个表空间最多支持64TB的数据。
* 某些类型的页可以组成链表，链表中的页可以不按照物理顺序存储，而是根据`FileHeader`中的`FIL_PAGE_PREV`和`FIL_PAGE_NEXT`来存储上一个页和下一个页的页号。
* 每个页的类型由`FIL_PAGE_TYPE`表示

### 独立表空间结构

#### 区（extent）概念

对于16KB的页来说，连续的64个页就是一个**区**，也就是说一个区默认占用1MB空间大小。

系统表空间还是独立表空间，都可以看成是由若干个区组成的，每256个区被划分成一组。

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716105429667.png" alt="image-20200716105429667"  />

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716105608831.png" alt="image-20200716105608831" style="zoom:200%;" />

- 第一个组最开始的3个页面的类型是固定的，也就是说`extent 0`这个区最开始的3个页面的类型是固定的，分别是：
  - `FSP_HDR`类型：这个类型的页面是用来登记整个表空间的一些整体属性以及本组所有的区，也就是`extent 0` ~ `extent 255`这256个区的属性，稍后详细唠叨。需要注意的一点是，整个表空间只有一个`FSP_HDR`类型的页面。
  - `IBUF_BITMAP`类型：这个类型的页面是存储本组所有的区的所有页面关于`INSERT BUFFER`的信息。
  - `INODE`类型：这个类型的页面存储了许多称为`INODE`的数据结构。
- 其余各组最开始的2个页面的类型是固定的，也就是说`extent 256`、`extent 512`这些区最开始的2个页面的类型是固定的，分别是：
  - `XDES`类型：全称是`extent descriptor`，用来登记本组256个区的属性，也就是说对于在`extent 256`区中的该类型页面存储的就是`extent 256` ~ `extent 511`这些区的属性，对于在`extent 512`区中的该类型页面存储的就是`extent 512` ~ `extent 767`这些区的属性。上边介绍的`FSP_HDR`类型的页面其实和`XDES`类型的页面的作用类似，只不过`FSP_HDR`类型的页面还会额外存储一些表空间的属性。
  - `IBUF_BITMAP`类型：上边介绍过了。

#### 段（segment）的概念

**区概念的引出**

* 我们每向表中插入一条记录，本质上就是向该表的聚簇索引以及所有二级索引代表的`B+`树的节点中插入数据。
* 而`B+`树的每一层中的页都会形成一个双向链表，如果是以页为单位来分配存储空间的话，双向链表相邻的两个页之间的物理位置可能离得非常远。
* 如果链表中相邻的两个页物理位置离得非常远，就是所谓的随机***I/O***。
* 应该尽量让链表中相邻的页的物理位置也相邻，这样进行范围查询的时候才可以使用所谓的***顺序I/O***。
* 所以才引入了***区（`extent`）***的概念，一个区就是在物理位置上连续的64个页。

**段概念的引出**

* 范围查询是对`B+`树叶子节点中的记录进行顺序扫描，而如果不区分叶子节点和非叶子节点，统统把节点代表的页面放到申请到的区中的话，进行范围扫描的效果就大打折扣了。
* 对`B+`树的叶子节点和非叶子节点进行了区别对待，也就是说叶子节点有自己独有的`区`，非叶子节点也有自己独有的`区`。
* 存放叶子节点的区的集合就算是一个`段`（`segment`），存放非叶子节点的区的集合也算是一个`段`。

**碎片区：**

* 一个索引会生成2个段，而段是以区为单位申请存储空间的，一个区默认占用1M存储空间，所以默认情况下一个只存了几条记录的小表也需要2M的存储空间么？
* 在一个碎片区中，并不是所有的页都是为了存储同一个段的数据而存在的，而是碎片区中的页可以用于不同的目的，比如有些页用于段A，有些页用于段B，有些页甚至哪个段都不属于。
* 碎片区直属于表空间，并不属于任何一个段。

**为某个段分配存储空间的策略是这样的：**

* 在刚开始向表中插入数据的时候，段是从某个碎片区以单个页面为单位来分配存储空间的。
* 当某个段已经占用了32个碎片区页面之后，就会以完整的区为单位来分配存储空间。

所以现在段不能仅定义为是某些区的集合，更精确的应该是某些零散的页面以及一些完整的区的集合。

##### 区的分类

- 空闲的区：现在还没有用到这个区中的任何页面。（Free）
- 有剩余空间的碎片区：表示碎片区中还有可用的页面。(Free_FRAG)
- 没有剩余空间的碎片区：表示碎片区中的所有页面都被使用，没有空闲页面。(FULL_FRAG)
- 附属于某个段的区。每一个索引都可以分为叶子节点段和非叶子节点段，除此之外InnoDB还会另外定义一些特殊作用的段，在这些段中的数据量很大时将使用区来作为基本的分配单位。(FSEG)

处于`FREE`、`FREE_FRAG`以及`FULL_FRAG`这三种状态的区都是独立的算是直属于表空间.

而处于`FSEG`状态的区是附属于某个段的.

为了方便管理区：

![image-20200716113555519](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716113555519.png)

- 

- `Segment ID`（8字节）

  每一个段都有一个唯一的编号，用ID表示，此处的`Segment ID`字段表示就是该区所在的段。当然前提是该区已经被分配给某个段了，不然的话该字段的值没啥意义。

- `List Node`（12字节）

  这个部分可以将若干个`XDES Entry`结构串联成一个链表。

- `State`（4字节）

  这个字段表明区的状态。可选的值就是我们前边说过的那4个，分别是：`FREE`、`FREE_FRAG`、`FULL_FRAG`和`FSEG`。

- `Page State Bitmap`（16字节）

  16个字节，也就是128个比特位。一个区默认有64个页，这128个比特位被划分为64个部分，每个部分2个比特位，对应区中的一个页。

##### XDES Entry链表

向某个段中插入数据的过程：

* 当段中数据较少的时候，首先会查看表空间中是否有状态为`FREE_FRAG`的区。

  * 如果找到了，那么从该区中取一些零散的页把数据插进去；
  * 否则到表空间下申请一个状态为`FREE`的区，也就是空闲的区，把该区的状态变为`FREE_FRAG`，然后从该新申请的区中取一些零散的页把数据插进去。
  * 之后不同的段使用零散页的时候都会从该区中取，直到该区中没有空闲空间，然后该区的状态就变成了`FULL_FRAG`。

  * 把状态为`FREE`的区对应的`XDES Entry`结构通过`List Node`来连接成一个链表；

    把状态为`FREE_FRAG`的区对应的`XDES Entry`结构通过`List Node`来连接成一个链表；

    把状态为`FULL_FRAG`的区对应的`XDES Entry`结构通过`List Node`来连接成一个链表。

## 单表访问方法

```sql
CREATE TABLE single_table (
    id INT NOT NULL AUTO_INCREMENT,
    key1 VARCHAR(100),
    key2 INT,
    key3 VARCHAR(100),
    key_part1 VARCHAR(100),
    key_part2 VARCHAR(100),
    key_part3 VARCHAR(100),
    common_field VARCHAR(100),
    PRIMARY KEY (id),
    KEY idx_key1 (key1),
    UNIQUE KEY idx_key2 (key2),
    KEY idx_key3 (key3),
    KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;
```

### 访问方法（access method）的概念

查询的执行方式大致分为下边两种：

* 使用全表扫描进行查询

* 使用索引进行查询

  * 针对主键或唯一二级索引的等值查询

  * 针对普通二级索引的等值查询
  * 针对索引列的范围查询
  * 直接扫描整个索引

MySQL执行查询语句的方式称之为***访问方法***或者***访问类型***。

#### const

* 针对主键或唯一二级索引的等值查询

```sql
SELECT * FROM single_table WHERE id = 1438;
```



<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716143620151.png" alt="image-20200716143620151" style="zoom: 50%;" />

```sql
SELECT * FROM single_table WHERE key2 = 3841;
```

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716143730283.png" alt="image-20200716143730283" style="zoom:67%;" />

把这种通过***主键或者唯一二级索引列来定位一条记录***的访问方法定义为：`const`，意思是常数级别的，代价是可以忽略不计的。

如果主键或者唯一二级索引是由多个列构成的话，索引中的每一个列都需要与常数进行等值比较，这个`const`访问方法才有效（这是因为只有该索引中全部列都采用等值比较才可以定位唯一的一条记录）。

```sql
SELECT * FROM single_table WHERE key2 IS NULL;
```

查询该列为`NULL`值的情况比较特殊,因为唯一二级索引列并不限制 NULL 值的数量，所以上述语句可能访问到多条记录，也就是说 上边这个语句不可以使用`const`访问方法来执行.

#### ref

```sql
SELECT * FROM single_table WHERE key1 = 'abc';
```

可以选择全表扫描来逐一对比搜索条件是否满足要求;

也可以先使用二级索引找到对应记录的`id`值，然后再回表到聚簇索引中查找完整的用户记录。

由于普通二级索引并不限制索引列值的唯一性，所以可能找到多条对应的记录，也就是说使用二级索引来执行查询的代价取决于等值匹配到的二级索引记录条数。

这种搜索条件为***二级索引列与常数等值比较***，采用二级索引来执行查询的访问方法称为：`ref`。

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716145220750.png" alt="image-20200716145220750" style="zoom: 67%;" />

* 二级索引列值为`NULL`的情况

不论是普通的二级索引，还是唯一二级索引，它们的索引列对包含`NULL`值的数量并不限制，所以我们采用`key IS NULL`这种形式的搜索条件最多只能使用`ref`的访问方法，而不是`const`的访问方法。

* 对于某个包含多个索引列的二级索引来说，只要是最左边的连续索引列是与常数的等值比较就可能采用`ref`的访问方法，比方说下边这几个查询：

```sql
SELECT * FROM single_table WHERE key_part1 = 'god like';

SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 = 'legendary';

SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 = 'legendary' AND key_part3 = 'penta kill';
```

如果最左边的连续索引列并不全部是等值比较的话，它的访问方法就不能称为`ref`了

```sql
SELECT * FROM single_table WHERE key_part1 = 'god like' AND key_part2 > 'legendary';
```

#### ref_or_null

有时候我们不仅想找出某个二级索引列的值等于某个常数的记录，还想把该列的值为`NULL`的记录也找出来.

```sql
SELECT * FROM single_table WHERE key1 = 'abc' OR key1 IS NULL;
```

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716145824408.png" alt="image-20200716145824408" style="zoom:67%;" />

#### range

```sql
SELECT * FROM single_table WHERE key2 IN (1438, 6328) OR (key2 >= 38 AND key2 <= 79);
```

搜索条件就不只是要求索引列与常数的等值匹配了，而是索引列需要匹配某个或某些范围的值.

在本查询中`key2`列的值只要匹配下列3个范围中的任何一个就算是匹配成功了：

- `key2`的值是`1438`
- `key2`的值是`6328`
- `key2`的值在`38`和`79`之间。

这种利用索引进行范围匹配的访问方法称之为：`range`。

3个范围也就对应着3个区间：

- 范围1：`key2 = 1438`
- 范围2：`key2 = 6328`
- 范围3：`key2 ∈ [38, 79]`，注意这里是闭区间。

`范围1`和`范围2`都可以被称为单点区间，像`范围3`这种的我们可以称为连续范围区间。

#### index

`key_part2`并不是联合索引`idx_key_part`最左索引列，所以我们无法使用`ref`或者`range`访问方法来执行这个语句.

我们可以直接通过遍历`idx_key_part`索引的叶子节点的记录来比较`key_part2 = 'abc'`这个条件是否成立，把匹配成功的二级索引记录的`key_part1`, `key_part2`, `key_part3`列的值直接加到结果集中就行了。

采用遍历二级索引记录的执行方式称之为：`index`。采用遍历二级索引记录的执行方式称之为：`index`。

#### all

使用全表扫描执行查询的方式称之为：`all`。

### 注意事项

#### 明确range访问方法使用的范围区间

对于`B+`树索引来说，只要索引列和常数使用`=`、`<=>`、`IN`、`NOT IN`、`IS NULL`、`IS NOT NULL`、`>`、`<`、`>=`、`<=`、`BETWEEN`、`!=`（不等于也可以写成`<>`）或者`LIKE`操作符连接起来，就可以产生一个所谓的`区间`。

##### 所有搜索条件都可以使用某个索引的情况

```sql
SELECT * FROM single_table WHERE key2 > 100 AND key2 > 200;
```

搜索条件都可以使用到`key2`

搜索条件使用`AND`连接起来

要取两个范围区间的交集

##### 有的搜索条件无法使用索引的情况

```sql
SELECT * FROM single_table WHERE key2 > 100 AND common_field = 'abc';
```

查询语句中能利用的索引只有`idx_key2`一个，而`idx_key2`这个二级索引的记录中又不包含`common_field`这个字段.

`common_field = 'abc'`这个条件是在回表获取了完整的用户记录后才使用的

在确定*范围区间*的时候不需要考虑`common_field = 'abc'`这个条件，我们在为某个索引确定范围区间的时候只需要把用不到相关索引的搜索条件替换为`TRUE`就好了。

也就是说最上边那个查询使用`idx_key2`的范围区间就是：`(100, +∞)`。

使用`OR`的情况:

```sql
SELECT * FROM single_table WHERE key2 > 100 OR common_field = 'abc';
简化：
SELECT * FROM single_table WHERE key2 > 100 OR TRUE;
接着简化：
SELECT * FROM single_table WHERE TRUE;
```

也就是说一个使用到索引的搜索条件和没有使用该索引的搜索条件使用`OR`连接起来后是无法使用该索引的。

##### 复杂搜索条件下找出范围匹配的区间

```sql
SELECT * FROM single_table WHERE 
        (key1 > 'xyz' AND key2 = 748 ) OR
        (key1 < 'abc' AND key1 > 'lmn') OR
        (key1 LIKE '%suf' AND key1 > 'zzz' AND (key2 < 8000 OR common_field = 'abc')) ;
```

* 首先查看`WHERE`子句中的搜索条件都涉及到了哪些列，哪些列可能使用到索引

  * 涉及到了`key1`、`key2`、`common_field`这3个列
  * key1`列有普通的二级索引`idx_key1
  * `key2`列有唯一二级索引`idx_key2`。

* 对于那些可能用到的索引，分析它们的范围区间。

  * 假设我们使用`idx_key1`执行查询

    * 我们需要把那些用不到该索引的搜索条件暂时移除掉，移除方法也简单，直接把它们替换为`TRUE`就好了。

    ```sql
    (key1 > 'xyz') OR
    (key1 < 'abc' AND key1 > 'lmn') OR
    (key1 > 'zzz')
    ```

    * 替换掉永远为`TRUE`或`FALSE`的条件

    ```sql
    (key1 > 'xyz') OR (key1 > 'zzz')
    ```

    * 继续化简区间

    ```te
    key1 > 'xyz'和key1 > 'zzz'之间使用OR操作符连接起来的，意味着要取并集，所以最终的结果化简的到的区间就是：key1 > xyz。
    ```

  * 假设我们使用`idx_key2`执行查询

    * 用不到该索引的搜索条件暂时使用`TRUE`条件替换掉

    ```sql
    (TRUE AND key2 = 748 ) OR
    (TRUE AND TRUE) OR
    (TRUE AND TRUE AND (key2 < 8000 OR TRUE))
    ```

    * 这个结果也就意味着如果我们要使用`idx_key2`索引执行查询语句的话，需要扫描`idx_key2`二级索引的所有记录，然后再回表，这不是得不偿失么，所以这种情况下不会使用`idx_key2`索引的。

#### 索引合并

`MySQL`在一般情况下执行一个查询时最多只会用到单个二级索引;

在这些特殊情况下也可能在一个查询中使用到多个二级索引.

这种使用到多个索引来完成一次查询的执行方法称之为：`index merge`.

##### Intersection合并

某个查询可以使用多个二级索引，将从多个二级索引中查询到的结果取交集:

```sql
SELECT * FROM single_table WHERE key1 = 'a' AND key3 = 'b';
```

使用`Intersection`合并的方式执行的话，那这个过程就是这样的：

- 从`idx_key1`二级索引对应的`B+`树中取出`key1 = 'a'`的相关记录。
- 从`idx_key3`二级索引对应的`B+`树中取出`key3 = 'b'`的相关记录。
- 二级索引的记录都是由`索引列 + 主键`构成的，所以我们可以计算出这两个结果集中`id`值的交集。
- 按照上一步生成的`id`值列表进行回表操作，也就是从聚簇索引中把指定`id`值的完整用户记录取出来，返回给用户。

读取二级索引的操作是***顺序I/O***，而回表操作是***随机I/O***，所以如果只读取一个二级索引时需要回表的记录数特别多，而读取多个二级索引之后取交集的记录数非常少，当节省的因为**回表**而造成的性能损耗比访问多个二级索引带来的性能损耗更高时，读取多个二级索引后取交集比只读取一个二级索引的成本更低。

使用到`Intersection`索引合并情况:

* 二级索引列是等值匹配的情况，对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现只匹配部分列的情况。

  ```sql
  SELECT * FROM single_table WHERE key1 = 'a' AND key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c';
  ```

  不能进行`Intersection`索引合并:

  ```sql
  SELECT * FROM single_table WHERE key1 > 'a' AND key_part1 = 'a' AND key_part2 = 'b' AND key_part3 = 'c';
  
  SELECT * FROM single_table WHERE key1 = 'a' AND key_part1 = 'a';
  ```

* 主键列可以是范围匹配

  ```sql
  SELECT * FROM single_table WHERE id > 100 AND key1 = 'a';
  ```

##### Union合并

`Union`是并集的意思，适用于使用不同索引的搜索条件之间使用`OR`连接起来的情况。

`Union`索引合并：

* 级索引列是等值匹配的情况，对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能出现只出现匹配部分列的情况。
* 主键列可以是范围匹配
* 使用`Intersection`索引合并的搜索条件

##### Sort-Union合并

* 

## 连接的原理

### 连接简介

#### 连接的本质

本质就是把各个连接表中的记录都取出来依次匹配的组合加入结果集并返回给用户。

```sql
mysql> SELECT * FROM t1;
+------+------+
| m1   | n1   |
+------+------+
|    1 | a    |
|    2 | b    |
|    3 | c    |
+------+------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM t2;
+------+------+
| m2   | n2   |
+------+------+
|    2 | b    |
|    3 | c    |
|    4 | d    |
+------+------+
3 rows in set (0.00 sec)
```



<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716170603002.png" alt="image-20200716170603002" style="zoom:67%;" />

过程看起来就是把`t1`表的记录和`t2`的记录连起来组成新的更大的记录，所以这个查询过程称之为连接查询。



## MySQL基于成本的优化

### 什么是成本

* `I/O`成本

  存储引擎都是将数据和索引都存储到磁盘上的。

  想查询表中的记录时，需要先把数据或者索引加载到内存中然后再操作。

* `CPU`成本

  读取以及检测记录是否满足对应的搜索条件、对结果集进行排序等这些操作损耗的时间称之为`CPU`成本。

### 单表查询成本

```sql
CREATE TABLE single_table (
    id INT NOT NULL AUTO_INCREMENT,
    key1 VARCHAR(100),
    key2 INT,
    key3 VARCHAR(100),
    key_part1 VARCHAR(100),
    key_part2 VARCHAR(100),
    key_part3 VARCHAR(100),
    common_field VARCHAR(100),
    PRIMARY KEY (id),
    KEY idx_key1 (key1),
    UNIQUE KEY idx_key2 (key2),
    KEY idx_key3 (key3),
    KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;
```

#### 基于成本优化步骤

MySQL的查询优化器会找出执行该语句所有可能使用的方案，对比之后找出成本最低的方案，这个成本最低的方案就是所谓的执行计划

1. 根据搜索条件，找出所有可能使用的索引
2. 计算全表扫描的代价
3. 计算使用不同索引执行查询的代价
4. 对比各种执行方案的代价，找出成本最低的那一个

```sql
SELECT * FROM single_table WHERE 
    key1 IN ('a', 'b', 'c') AND 
    key2 > 10 AND key2 < 1000 AND 
    key3 > key2 AND 
    key_part1 LIKE '%hello%' AND
    common_field = '123';
```



## InnoDB统计数据是如何收集的

## 基于规则优化



## Explain详解（上）

一条查询语句在经过优化之后，会生成一个执行计划。

这个执行计划会展示接下来具体执行查询的方式。

MySQL提供了EXPLAIN语句帮助我们查看执行计划。

例如:

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716133812864.png" alt="image-20200716133812864" style="zoom:200%;" />

```sql
CREATE TABLE single_table (
    id INT NOT NULL AUTO_INCREMENT,
    key1 VARCHAR(100),
    key2 INT,
    key3 VARCHAR(100),
    key_part1 VARCHAR(100),
    key_part2 VARCHAR(100),
    key_part3 VARCHAR(100),
    common_field VARCHAR(100),
    PRIMARY KEY (id),
    KEY idx_key1 (key1),
    UNIQUE KEY idx_key2 (key2),
    KEY idx_key3 (key3),
    KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;
```

有两表s1，s2与single_table结构相同。

### id

```sql
mysql> EXPLAIN SELECT * FROM s1 WHERE key1 = 'a';
```

|      id      |    1     |
| :----------: | :------: |
| select_type  |  SIMPLE  |
|    table     |    s1    |
|  partitions  |   NULL   |
|     type     |   ref    |
| possible_key | idx_key1 |
|     key      | idx_key1 |
|   key_len    |   303    |
|     ref      |  const   |
|     rows     |    8     |
|   filtered   |  100.00  |
|    Extra     |   NULL   |

只有一个select关键字，explain的结果中也只有一条id列为1的记录；

```sql
 EXPLAIN SELECT * FROM s1 INNER JOIN s2;
```

|      id      |   1    |                   1                   |
| :----------: | :----: | :-----------------------------------: |
| select_type  | SIMPLE |                SIMPLE                 |
|    table     |   s1   |                  s2                   |
|  partitions  |  NULL  |                 NULL                  |
|     type     |  All   |                  ALL                  |
| possible_key |  NULL  |                 NULL                  |
|     key      |  NULL  |                 NULL                  |
|   key_len    |  NULL  |                 NULL                  |
|     ref      |  NULL  |                 NULL                  |
|     rows     |  9688  |                 9954                  |
|   filtered   | 100.00 |                100.00                 |
|    Extra     |  NULL  | Using join buffer (Block Nested Loop) |

一个select关键字后边的From可以跟多个表，所以连接查询计划中，每个表都会对应一条记录，但是这些记录的id值都是相同的。

在连接查询的执行计划中，每个表都会对应一条记录，这些记录的id列的值是相同的，出现在前边的表表示驱动表（s1），出现在后边的表表示被驱动表(s2)。

```sql
EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2) OR key3 = 'a';
```

|      id      |      1      |      2      |
| :----------: | :---------: | :---------: |
| select_type  |   PRIMARY   |  SUBQUERY   |
|    table     |     s1      |     s2      |
|  partitions  |    NULL     |    NULL     |
|     type     |     All     |    index    |
| possible_key |  idx_key3   |  idx_key1   |
|     key      |    NULL     |  idx_key1   |
|   key_len    |    NULL     |     303     |
|     ref      |    NULL     |    NULL     |
|     rows     |    9688     |    9954     |
|   filtered   |   100.00    |   100.00    |
|    Extra     | Using where | Using index |

`s1`表在外层查询中，外层查询有一个独立的`SELECT`关键字，所以第一条记录的`id`值就是`1`，`s2`表在子查询中，子查询有一个独立的`SELECT`关键字，所以第二条记录的`id`值就是`2`。

```sql
 EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key3 FROM s2 WHERE common_field = 'a');
```

|      id      |              1              |       2       |
| :----------: | :-------------------------: | :-----------: |
| select_type  |           SIMPLE            |    SIMPLE     |
|    table     |             s1              |      s2       |
|  partitions  |            NULL             |     NULL      |
|     type     |             All             |      ref      |
| possible_key |          idx_key3           |   idx_key1    |
|     key      |            NULL             |   idx_key1    |
|   key_len    |            NULL             |      303      |
|     ref      |            NULL             | text.s2.key3  |
|     rows     |            9954             |       1       |
|   filtered   |            10.00            |    100.00     |
|    Extra     | Using where;Start temporary | End temporary |

虽然我们的查询语句是一个子查询，但是执行计划中`s1`和`s2`表对应的记录的`id`值全部是`1`，这就表明了查询优化器将子查询转换为了连接查询.

## 事务简介

### 事务的起源

#### 原子性（Atomicity）

要么全做，要么全不做的规则称之为`原子性`

#### 隔离性（Isolation）

不仅要保证这些操作以`原子性`的方式执行完成，而且要保证其它的状态转换不会影响到本次状态转换，这个规则被称之为`隔离性`。

#### 一致性（Consistency）

如果数据库中的数据全部符合现实世界中的约束（all defined rules），我们说这些数据就是一致的，或者说符合`一致性`的。

#### 持久性（Durability）

当现实世界的一个状态转换完成后，这个转换的结果将永久的保留称为`持久性`。

### 事务的概念

把需要保证`原子性`、`隔离性`、`一致性`和`持久性`的一个或多个数据库操作称之为一个`事务`（英文名是：`transaction`）。

`事务`大致上划分成了这么几个状态：

- 活动的（active）

  事务对应的数据库操作正在执行过程中时，我们就说该事务处在`活动的`状态。

- 失败的（failed）

  当事务处在`活动的`或者`部分提交的`状态时，可能遇到了某些错误（数据库自身的错误、操作系统错误或者直接断电等）而无法继续执行，或者人为的停止当前事务的执行，我们就说该事务处在`失败的`状态。

- 中止的（aborted）

  当`回滚`操作执行完毕时，也就是数据库恢复到了执行事务之前的状态，我们就说该事务处在了`中止的`状态。

- 提交的（committed）

  当一个处在`部分提交的`状态的事务将修改过的数据都同步到磁盘上之后，我们就可以说该事务处在了`提交的`状态。

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716175820337.png" alt="image-20200716175820337" style="zoom: 67%;" />

只有当事务处于提交的或者中止的状态时，一个事务的生命周期才算是结束了。

### MySQL中事务的语法

#### 开启事务

- `BEGIN [WORK];`

  `BEGIN`语句代表开启一个事务，后边的单词`WORK`可有可无。开启事务后，就可以继续写若干条语句，这些语句都属于刚刚开启的这个事务。

  ```
  mysql> BEGIN;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> 加入事务的语句...
  ```

- `START TRANSACTION;`

  `START TRANSACTION`语句和`BEGIN`语句有着相同的功效，都标志着开启一个事务，比如这样：

  ```
  mysql> START TRANSACTION;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> 加入事务的语句...
  ```
  不过比`BEGIN`语句牛逼一点儿的是，可以在`START TRANSACTION`语句后边跟随几个`修饰符`，就是它们几个：

  - `READ ONLY`：标识当前事务是一个只读事务，也就是属于该事务的数据库操作只能读取数据，而不能修改数据。
  - `READ WRITE`：标识当前事务是一个读写事务，也就是属于该事务的数据库操作既可以读取数据，也可以修改数据。
  - `WITH CONSISTENT SNAPSHOT`：启动一致性读

  不能同时把`READ ONLY`和`READ WRITE`放到`START TRANSACTION`语句后边。另外，如果我们不显式指定事务的访问模式，那么该事务的访问模式就是`读写`模式。

#### 提交事务

```sql
COMMIT [WORK]
```

当最后一条语句写完了之后，我们就可以提交该事务了.

#### 手动中止事务

写了几条语句之后发现上边的某条语句写错了，我们可以手动的使用下边这个语句来将数据库恢复到事务执行之前的样子：

```sql
ROLLBACK [WORK]
```

`ROLLBACK`语句就代表中止并回滚一个事务.

`ROLLBACK`语句是我们程序员手动的去回滚事务时才去使用的，如果事务在执行过程中遇到了某些错误而无法继续执行的话，事务自身会自动的回滚。

#### 支持事务的存储引擎

`MySQL`中并不是所有存储引擎都支持事务的功能,`InnoDB`和`NDB`存储引擎支持.

如果某个事务中包含了修改使用不支持事务的存储引擎的表，那么对该使用不支持事务的存储引擎的表所做的修改将**无法进行回滚**。

#### 自动提交

`MySQL`中有一个系统变量`autocommit`：

如果我们不显式的使用`START TRANSACTION`或者`BEGIN`语句开启一个事务，那么每一条语句都算是一个独立的事务。

想关闭这种`自动提交`的功能，可以使用下边两种方法之一：

- 显式的的使用`START TRANSACTION`或者`BEGIN`语句开启一个事务。

  这样在本次事务提交或者回滚前会暂时关闭掉自动提交的功能。

- 把系统变量`autocommit`的值设置为`OFF`，就像这样：

  ```
  SET autocommit = OFF;
  ```

  这样的话，我们写入的多条语句就算是属于同一个事务了，直到我们显式的写出`COMMIT`语句来把这个事务提交掉，或者显式的写出`ROLLBACK`语句来把这个事务回滚掉。

#### 隐式提交

当我们使用`START TRANSACTION`或者`BEGIN`语句开启了一个事务，或者把系统变量`autocommit`的值设置为`OFF`时，事务就不会进行`自动提交`，但是如果我们输入了某些语句之后就会`悄悄的`提交掉，就像我们输入了`COMMIT`语句了一样。

* 定义或修改数据库对象的数据定义语言(Data definition language，缩写为：`DDL`)

  当我们使用`CREATE`、`ALTER`、`DROP`等语句去修改这些所谓的数据库对象时，就会隐式的提交前边语句所属于的事务。

* 隐式使用或修改`mysql`数据库中的表

  当我们使用`ALTER USER`、`CREATE USER`、`DROP USER`、`GRANT`、`RENAME USER`、`REVOKE`、`SET PASSWORD`等语句时也会隐式的提交前边语句所属于的事务。

* 事务控制或关于锁定的语句

  当我们在一个事务还没提交或者回滚时就又使用`START TRANSACTION`或者`BEGIN`语句开启了另一个事务时，会隐式的提交上一个事务。

* 加载数据的语句

  比如我们使用`LOAD DATA`语句来批量往数据库中导入数据时，也会隐式的提交前边语句所属的事务。

* 关于`MySQL`复制的一些语句

  使用`START SLAVE`、`STOP SLAVE`、`RESET SLAVE`、`CHANGE MASTER TO`等语句时也会隐式的提交前边语句所属的事务。

* 使用`ANALYZE TABLE`、`CACHE INDEX`、`CHECK TABLE`、`FLUSH`、 `LOAD INDEX INTO CACHE`、`OPTIMIZE TABLE`、`REPAIR TABLE`、`RESET`等语句也会隐式的提交前边语句所属的事务。

#### 保存点

在事务对应的数据库语句中打几个点，我们在调用`ROLLBACK`语句时可以指定会滚到哪个点，而不是回到最初的原点。

```sql
SAVEPOINT 保存点名称;
```

回滚到某个保存点:

```sql
ROLLBACK [WORK] TO [SAVEPOINT] 保存点名称;
```

删除某个保存点:

```sql
RELEASE SAVEPOINT 保存点名称;
```

## redo日志

 如果我们只在内存的`Buffer Pool`中修改了页面，假设在事务提交后突然发生了某个故障，导致内存中的数据都失效了，那么这个已经提交了的事务对数据库中所做的更改也就跟着丢失了。

在事务提交完成之前把该事务所修改的所有页面都刷新到磁盘，但是这个简单粗暴的做法有些问题：

* 刷新一个完整的数据页太浪费了

  有时候我们仅仅修改了某个页面中的一个字节，但是我们知道在`InnoDB`中是以页为单位来进行磁盘IO的，也就是说我们在该事务提交时不得不将一个完整的页面从内存中刷新到磁盘，我们又知道一个页面默认是16KB大小，只修改一个字节就要刷新16KB的数据到磁盘上显然是太浪费了。

* 随机IO刷起来比较慢

只需要把修改了哪些东西记录一下就好。

把上述内容刷新到磁盘中，即使之后系统崩溃了，重启之后只要按照上述内容所记录的步骤重新更新一下数据页，那么该事务对数据库中所做的修改又可以被恢复出来，也就意味着满足`持久性`的要求。

`redo`日志本质上只是记录了一下事务对数据库做了哪些修改。

<img src="C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200716190344479.png" alt="image-20200716190344479" style="zoom: 50%;" />