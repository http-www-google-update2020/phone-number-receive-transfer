<!--

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

-->

# DDL (数据定义语言)

## 创建存储组

我们可以根据存储模型建立相应的存储组。创建存储组的SQL语句如下所示：

```
IoTDB > set storage group to root.ln
IoTDB > set storage group to root.sgcc
```

根据以上两条SQL语句，我们可以创建出两个存储组。

需要注意的是，当系统中已经存在某个存储组或存储组的父亲节点或者孩子节点被设置为存储组的情况下，用户不可创建存储组。例如在已经有`root.ln`和`root.sgcc`这两个存储组的情况下，创建`root.ln.wf01`存储组是不可行的。系统将给出相应的错误提示，如下所示：

```
IoTDB> set storage group to root.ln.wf01
Msg: org.apache.iotdb.exception.MetadataErrorException: org.apache.iotdb.exception.PathErrorException: The prefix of root.ln.wf01 has been set to the storage group.
```

## 查看存储组

在存储组创建后，我们可以使用[SHOW STORAGE GROUP](../Operation%20Manual/SQL%20Reference.html)语句来查看所有的存储组，SQL语句如下所示：

```
IoTDB> show storage group
```

执行结果为：
<center><img style="width:100%; max-width:800px; max-height:600px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/13203019/51577338-84c70600-1ef4-11e9-9dab-605b32c02836.jpg"></center>

## 创建时间序列

根据建立的数据模型，我们可以分别在两个存储组中创建相应的时间序列。创建时间序列的SQL语句如下所示：

```
IoTDB > create timeseries root.ln.wf01.wt01.status with datatype=BOOLEAN,encoding=PLAIN
IoTDB > create timeseries root.ln.wf01.wt01.temperature with datatype=FLOAT,encoding=RLE
IoTDB > create timeseries root.ln.wf02.wt02.hardware with datatype=TEXT,encoding=PLAIN
IoTDB > create timeseries root.ln.wf02.wt02.status with datatype=BOOLEAN,encoding=PLAIN
IoTDB > create timeseries root.sgcc.wf03.wt01.status with datatype=BOOLEAN,encoding=PLAIN
IoTDB > create timeseries root.sgcc.wf03.wt01.temperature with datatype=FLOAT,encoding=RLE
```

需要注意的是，当创建时间序列时指定的编码方式与数据类型不对应时，系统会给出相应的错误提示，如下所示：
```
IoTDB> create timeseries root.ln.wf02.wt02.status WITH DATATYPE=BOOLEAN, ENCODING=TS_2DIFF
error: encoding TS_2DIFF does not support BOOLEAN
```

详细的数据类型与编码方式的对应列表请参见[编码方式](../Concept/Encoding.html)。

### 标签点管理

我们可以在创建时间序列的时候，为它添加别名和额外的标签和属性信息。
所用到的扩展的创建时间序列的SQL语句如下所示：
```
create timeseries root.turbine.d1.s1(temprature) with datatype=FLOAT, encoding=RLE, compression=SNAPPY tags(tag1=v1, tag2=v2) attributes(attr1=v1, attr2=v2)
```

括号里的`temprature`是`s1`这个传感器的别名。
我们可以在任何用到`s1`的地方，将其用`temprature`代替，这两者是等价的。

> 注意：额外的标签和属性信息总的大小不能超过`tag_attribute_total_size`.

标签和属性的唯一差别在于，我们为标签信息在内存中维护了一个倒排索引，所以可以在`show timeseries`的条件语句中使用标签作为查询条件，你将会在下一节看到具体查询内容。

## 标签点属性更新
创建时间序列后，我们也可以对其原有的标签点属性进行更新，主要有以下五种更新方式：

* 重命名标签或属性
```
ALTER timeseries root.turbine.d1.s1 RENAME tag1 TO newTag1
```
* 重新设置标签或属性的值
```
ALTER timeseries root.turbine.d1.s1 SET tag1=newV1, attr1=newV1
```
* 删除已经存在的标签或属性
```
ALTER timeseries root.turbine.d1.s1 DROP tag1, tag2
```
* 添加新的标签
```
ALTER timeseries root.turbine.d1.s1 ADD TAGS tag3=v3, tag4=v4
```
* 添加新的属性
```
ALTER timeseries root.turbine.d1.s1 ADD ATTRIBUTES attr3=v3, attr4=v4
```
* 更新插入别名，标签和属性
> 如果该别名，标签或属性原来不存在，则插入，否则，用新值更新原来的旧值
```
ALTER timeseries root.turbine.d1.s1 UPSERT ALIAS=newAlias TAGS(tag2=newV2, tag3=v3) ATTRIBUTES(attr3=v3, attr4=v4)
```

## 查看时间序列

* SHOW LATEST? TIMESERIES prefixPath? showWhereClause? limitClause?

  SHOW TIMESERIES 中可以有四种可选的子句，查询结果为这些时间序列的所有信息

时间序列信息具体包括：时间序列路径名，存储组，Measurement别名，数据类型，编码方式，压缩方式，属性和标签。

示例：

* SHOW TIMESERIES

  展示系统中所有的时间序列信息

* SHOW TIMESERIES <`Path`>

  返回给定路径的下的所有时间序列信息。其中 `Path` 需要为一个前缀路径、带星路径或时间序列路径。例如，分别查看`root`路径和`root.ln`路径下的时间序列，SQL语句如下所示：

```
IoTDB> show timeseries root
IoTDB> show timeseries root.ln
```

执行结果分别为：

<center><img style="width:100%; max-width:800px; max-height:600px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/13203019/51577347-8db7d780-1ef4-11e9-91d6-764e58c10e94.jpg"></center>
<center><img style="width:100%; max-width:800px; max-height:600px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/13203019/51577359-97413f80-1ef4-11e9-8c10-53b291fc10a5.jpg"></center>

* SHOW TIMESERIES (<`PrefixPath`>)? WhereClause 
  
  返回给定路径的下的所有满足条件的时间序列信息，SQL语句如下所示：

```
show timeseries root.ln where unit=c
show timeseries root.ln where description contains 'test1'
```

执行结果分别为：
<center><img style="width:100%; max-width:800px; max-height:600px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/16079446/79682385-61544d80-8254-11ea-8c23-9e93e7152fda.png"></center>

> 注意，现在我们只支持一个查询条件，要么是等值条件查询，要么是包含条件查询。当然where子句中涉及的必须是标签值，而不能是属性值。

* SHOW TIMESERIES LIMIT INT OFFSET INT

  只返回从指定下标开始的结果，最大返回条数被 LIMIT 限制，用于分页查询

* SHOW LATEST TIMESERIES

  表示查询出的时间序列需要按照最近插入时间戳降序排列
  
需要注意的是，当查询路径不存在时，系统会返回0条时间序列。


## 查看子路径

```
SHOW CHILD PATHS prefixPath
```

可以查看此前缀路径的下一层的所有路径，前缀路径允许使用 * 通配符。

示例：

* 查询 root.ln 的下一层：show child paths root.ln

```
+------------+
| child paths|
+------------+
|root.ln.wf01|
|root.ln.wf02|
+------------+
```

* 查询形如 root.xx.xx.xx 的路径：show child paths root.\*.\*

```
+---------------+
|    child paths|
+---------------+
|root.ln.wf01.s1|
|root.ln.wf02.s2|
+---------------+
```

## 统计时间序列总数

IoTDB支持使用`COUNT TIMESERIES<Path>`来统计一条路径中的时间序列个数。SQL语句如下所示：
```
IoTDB > COUNT TIMESERIES root
IoTDB > COUNT TIMESERIES root.ln
IoTDB > COUNT TIMESERIES root.ln.*.*.status
IoTDB > COUNT TIMESERIES root.ln.wf01.wt01.status
```

除此之外，还可以通过定义`LEVEL`来统计指定层级下的时间序列个数。这条语句可以用来统计每一个设备下的传感器数量，语法为：`COUNT TIMESERIES <Path> GROUP BY LEVEL=<INTEGER>`。

例如有如下时间序列（可以使用`show timeseries`展示所有时间序列）：

<center><img style="width:100%; max-width:800px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/19167280/69792072-cdc8a480-1200-11ea-8cec-321fef618a12.png"></center>

那么Metadata Tree如下所示：

<center><img style="width:100%; max-width:600px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/19167280/69792176-1718f400-1201-11ea-861a-1a83c07ca144.jpg"></center>

可以看到，`root`被定义为`LEVEL=0`。那么当你输入如下语句时：

```
IoTDB > COUNT TIMESERIES root GROUP BY LEVEL=1
IoTDB > COUNT TIMESERIES root.ln GROUP BY LEVEL=2
IoTDB > COUNT TIMESERIES root.ln.wf01 GROUP BY LEVEL=2
```

你将得到以下结果：
<center><img style="width:100%; max-width:800px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/19167280/69792071-cb664a80-1200-11ea-8386-02dd12046c4b.png"></center>

> 注意：时间序列的路径只是过滤条件，与level的定义无关。

## 统计节点数

IoTDB支持使用`COUNT NODES <Path> LEVEL=<INTEGER>`来统计当前Metadata树下指定层级的节点个数，这条语句可以用来统计设备数。例如：

```
IoTDB > COUNT NODES root LEVEL=2
IoTDB > COUNT NODES root.ln LEVEL=2
IoTDB > COUNT NODES root.ln.wf01 LEVEL=3
```

对于上面提到的例子和Metadata Tree，你可以获得如下结果：
<center><img style="width:100%; max-width:800px; margin-left:auto; margin-right:auto; display:block;" src="https://user-images.githubusercontent.com/19167280/69792060-c73a2d00-1200-11ea-8ec4-be7145fd6c8c.png"></center>

> 注意：时间序列的路径只是过滤条件，与level的定义无关。

## 删除时间序列

我们可以使用`DELETE TimeSeries <PrefixPath>`语句来删除我们之前创建的时间序列。SQL语句如下所示：
```
IoTDB> delete timeseries root.ln.wf01.wt01.status
IoTDB> delete timeseries root.ln.wf01.wt01.temperature, root.ln.wf02.wt02.hardware
IoTDB> delete timeseries root.ln.wf02.*
```

## 查看设备

与 `Show Timeseries` 相似，IoTDB 目前也支持两种方式查看设备。
* `SHOW DEVICES` 语句显示当前所有的设备信息，等价于 `SHOW DEVICES root`。
* `SHOW DEVICES <PrefixPath>` 语句规定了 `PrefixPath`，返回在给定的前缀路径下的设备信息。

SQL语句如下所示：
```
IoTDB> show devices
IoTDB> show devices root.ln
```

# TTL

IoTDB支持对存储组级别设置数据存活时间（TTL），这使得IoTDB可以定期、自动地删除一定时间之前的数据。合理使用TTL
可以帮助您控制IoTDB占用的总磁盘空间以避免出现磁盘写满等异常。并且，随着文件数量的增多，查询性能往往随之下降,
内存占用也会有所提高。及时地删除一些较老的文件有助于使查询性能维持在一个较高的水平和减少内存资源的占用。

## 设置 TTL

设置TTL的SQL语句如下所示：
```
IoTDB> set ttl to root.ln 3600000
```
这个例子表示在`root.ln`存储组中，只有最近一个小时的数据将会保存，旧数据会被移除或不可见。

## 取消 TTL

取消TTL的SQL语句如下所示：

```
IoTDB> unset ttl to root.ln
```

取消设置TTL后，存储组`root.ln`中所有的数据都会被保存。

## FLUSH

将指定存储组的内存缓存区Memory Table的数据持久化到磁盘上，并将数据文件封口。

```
IoTDB> FLUSH 
IoTDB> FLUSH root.ln
IoTDB> FLUSH root.sg1,root.sg2
```

## MERGE

合并顺序和乱序数据。当前IoTDB支持使用如下两种SQL手动触发数据文件的合并：

* `MERGE` 仅重写重复的Chunk，整理速度快，但是最终磁盘会存在多余数据。
* `FULL MERGE` 将需要合并的顺序和乱序文件的所有数据都重新写一份，整理速度慢，最终磁盘将不存在无用的数据。

```
IoTDB> MERGE
IoTDB> FULL MERGE
```

## CLEAR CACHE

手动清除chunk, chunk metadata和timeseries metadata的缓存，在内存资源紧张时，可以通过此命令，释放查询时缓存所占的内存空间。
```
IoTDB> CLEAR CACHE
```

## 为 SCHEMA 创建快照

为了加快 IoTDB 重启速度，用户可以手动触发创建 schema 的快照，从而避免服务器从 mlog 文件中恢复。
```
IoTDB> CREATE SNAPSHOT FOR SCHEMA
```
