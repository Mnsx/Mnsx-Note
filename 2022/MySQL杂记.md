# MySQL下text与blob区别

* 相同

  * 在text与blob列的存储或检索过程中，不存在大小写转换，分配一个超过该列类型最大长度的值，值将会截取以保证合适，如果字符不为空格，将会产生一个警告，如果使用严格SQL模式，会产生错误，并且将值直接拒绝
  * blob和text不能有默认值
  * 当保存或检索blob和text列的值时不删除尾部空格
  * 对于blob和text的索引，必须指定索引前缀的长度

* 不同

  **text**

  * text值时大小写不敏感
  * text被视为非二进制字符串
  * text有一个字符集，并且根据字符集的，校对规则对值进行排序和比较
  * 可以将text列是为varchar列
  * blob可以存储图片，text不行，text只能存储纯文本文件，4个text类型tinytext、text、mediumtext和longtext对应4个blob类型，并且有同样的最大长度和存储需求

  **blob**

  * blob值的排序和比较以大小写敏感方式执行
  * blob被视为二进制字符串
  * blob列没有字符集，并且排序和比较基于列值字节的数值值
  * 大多数方面，可以将blob列视为足够大的varbinary列
  * 一个blob是一个能保存可变数量的数据的二进制的大对象，4个blob类型tinyblob、blob、mediumblob、longblob仅仅在他们能保存值的最大长度方面有所不同

  