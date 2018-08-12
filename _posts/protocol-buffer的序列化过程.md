---
title: protocol buffer的序列化过程
date: 2017-12-06 20:14:18
tags:
categories: Java
---

网上看到一篇很详细的文章，因为涉及到很多图片，所以直接发地址把

<!--more-->

[文章](http://blog.csdn.net/carson_ho/article/details/70568606)

protocol buffer序列化过的对象，最终转成二进制流，它的形式是T-L-V的格式，

#### 1. Tag

- 定义：经过 `Protocol Buffer`采用`Varint` & `Zigzag`编码后 的消息字段 标识号 & 数据类型 的值

- 作用：标识 字段

  > 1. 存储了字段的标识号（`field_number`）和 数据类型（`wire_type`），即`Tag` = 字段数据类型（`wire_type`） + 标识号（`field_number`）
  > 2. 占用 一个字节 的长度（如果标识号超过了16，则占用多一个字节的位置）
  > 3. 解包时，`Protocol Buffer`根据 `Tag` 将 `Value` 对应于消息中的 字段

- 具体使用

```Java
// Tag 的具体表达式如下
 Tag  = (field_number << 3) | wire_type
// 参数说明：
// field_number：对应于 .proto文件中消息字段的标识号，表示这是消息里的第几个字段
// field_number << 3：表示 field_number = 将 Tag的二进制表示 右移三位 后的值 
// field_num左移3位不会导致数据丢失，因为表示范围还是足够大地去表示消息里的字段数目

//  wire_type：表示 字段 的数据类型
//  wire_type = Tag的二进制表示 的最低三位值
//   wire_type的取值
 enum WireType { 
      WIRETYPE_VARINT = 0, 
      WIRETYPE_FIXED64 = 1, 
      WIRETYPE_LENGTH_DELIMITED = 2, 
      WIRETYPE_START_GROUP = 3, 
      WIRETYPE_END_GROUP = 4, 
      WIRETYPE_FIXED32 = 5
   };
// 从上面可以看出，`wire_type`最多占用 3位 的内存空间（因为 3位 足以表示 0-5 的二进制）
//其中，3，4已经弃用，对于0，1，5类型，L位可以省略掉，也就是只需要T-V表示即可，对于2类型，才需要TLV表示
```

![wire_type对应数据类型](http://upload-images.jianshu.io/upload_images/944365-ecd00f3d1fd8bbf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 实例说明

```
// 消息对象
 message person
 { 
    required int32     id = 1;  
    // wire type = 0，field_number =1 
    required string    name = 2;  
    // wire type = 2，field_number =2 
  }

//  如果一个Tag的二进制 = 0001 0010
标识号 = field_number = field_number  << 3 =右移3位 =  0000 0010 = 2
数据类型 = wire_type = 最低三位表示 = 010 = 2
```



### 2. Value

经过 `Protocol Buffer`采用`Varint` & `Zigzag`编码后 的消息字段的值

#### 实例说明

下面通过一个实例进行整个编码过程的说明：

- 消息说明

```
message Test
{

required int32 id1 = 1；

required int32 id2 = 2；
}

// 在代码中给id1 附上1个字段值：296
// 在代码中给id2 附上1个字段值：296
Test.setId1（300）；
Test.setId2（296）；

// 编码结果为：二进制字节流 = [8，-84，2，16, -88, 2]
//8和16是编码后的tag的值，其余字节表示value编码后的值
```

对于String类型，value则直接是其UTF8的字节表示，L则是其长度

对于嵌套类型，则表示有可能是TL-TLV-TLV

还有一种情况，则是`packed`修饰的 `repeat` 字段，可以存储为TL-VVVVV的形式



比较核心和有意思的则是它编码的方法,从效果来看很像huffman coding，思想都是把高频的数据进行压缩，比如

低于256的int值，我们不需要4字节去存储，可以去掉高位的三字节0位，压缩为1字节去存储

`Varint`编码:

```Java
private void writeVarint32(int n) {   
  int idx = 0;  
  while (true) {  
    if ((n & ~0x7F) == 0) {  
      //n & ~0x7F表示和1000 0000相与，判断条件则表示最高位是否为1，即最后七位
      //或者不足七bit位的时候，直接赋值n,表示头部补0，并且退出循环
      i32buf[idx++] = (byte)n;  
      break;  
    } else {  
        // 取出字节串末7位然后高位补1，存到buf里面取
      i32buf[idx++] = (byte)((n & 0x7F) | 0x80);
      //将n字节串整体往右移7位，继续从字节串的末尾选取7位，高位补1存入，直到取完为止。
      n >>>= 7;  
    }  
  }  
  trans_.write(i32buf, 0, idx); 
      //将上述形成的每个字节 按序拼接 成一个字节串 即该字节串就是经过Varint编码后的字节
}  
```

采用Varint编码，对于高位为0，不足4字节的正数，值越小编码后生成的byte值就越小，但对于负数，由于计算机里面有符号数负数表示为高位1，那么采用Varint编码的时候，则会把4byte的数据转成5byte，因此在数据中有负数的时候，需要先用 `Zigzag`编码转一下

#### `Zigzag`编码方式详解

#### i. 简介

- 定义：一种变长的编码方式

- 原理：使用 无符号数 来表示 有符号数字；

- 作用：使得绝对值小的数字都可以采用较少 字节 来表示； 

  ​

  >   特别是对 表示负数的数据 能更好地进行数据压缩

#### b. 原理

- 源码分析

```
public int int_to_zigzag(int n)
{
        return (n <<1) ^ (n >>31);   
        // 对于sint 32 数据类型，使用Zigzag编码过程如下：
        // 1. 将二进制表示数 左移1位（左移 = 整个二进制左移，低位补0）
        // 2. 将二进制表示数 右移31位 
              // 对于右移：
              // 首位是1的二进制（有符号数），是算数右移，即右移后左边补1
              // 首位是0的二进制（无符号数），是逻辑左移，即右移后左边补0
        // 3. 将上述二者进行异或

        // 对于sint 64 数据类型 则为： return  (n << 1> ^ (n >> 63) ；
}


// 附：将Zigzag值解码为整型值
public int zigzag_to_int(int n) 
{
        return(n >>> 1) ^ -(n & 1);
// 右移时，需要用不带符号的移动，否则如果第一位数据位是1的话，就会补1
}

```

比如-2，2表示为0000 0010，-2则是1111 1110，按照公式，-2左移一位，得到1111 1100，-2右移31位，得到1111 1111，两者异或，得到0000 0011，十进制是3，这样子，-2就转成了3



### 对比于`XML` 的序列化 & 反序列化过程

XML的反序列化过程如下： 

1. 从文件中读取出字符串 
2. 将字符串转换为 `XML` 文档对象结构模型 
3. 从 `XML` 文档对象结构模型中读取指定节点的字符串 
4. 将该字符串转换成指定类型的变量

上述过程非常复杂，其中，将 `XML` 文件转换为文档对象结构模型的过程通常需要完成词法文法分析等大量消耗 CPU 的复杂计算。

因为序列化 & 反序列化过程简单，所以序列化 & 反序列化过程速度非常快，这也是 `Protocol Buffer`效率高的原因



# 总结

- `Protocol Buffer`的序列化 & 反序列化简单 & 速度快的原因是： 
  a.  编码 / 解码 方式简单（只需要简单的数学运算 = 位移等等） 
  b. 采用 **Protocol Buffer 自身的框架代码 和 编译器** 共同完成
- `Protocol Buffer`的数据压缩效果好（即序列化后的数据量体积小）的原因是： 
  a. 采用了独特的编码方式，如`Varint`、`Zigzag`编码方式等等 
  b. 采用`T - L - V` 的数据存储方式：减少了分隔符的使用 & 数据存储得紧凑