# Struct

**Struct**对应于C语言中的结构体，完全可以按照C语言中结构体的方式来理解这里的Struct。

Struct是一个包含各种域\(Field\)的集合，这些域按顺序存放，通常每个域都有一个名字。

> 什么是域\(Field\)？  
> 简单来说，域\(Field\)就是一个Construct类的实例，多个域\(Field\)堆叠组成一个复杂的Struct。  
> Struct本身也可以作为另外一个Struct的域。  
> 更多关于域\(Field\)的说明，请参考 [Field](field.md) 一节

Struct的域按顺序定义，并且通常情况下每个域都有一个单独的名字。

**为什么说域是按顺序定义的呢？**

因为解析二进制数据时，是按照字节读取的顺序进行的。二进制数据的每一个字节都会被读取并解析，然后依次存放到结构体相应位置的域\(Field\)中。

为什么说域需要名字呢？

主要有以下两点：  
1. 当进行parse解析操作时，解析的结果是以字典的格式返回，其键值key对应于域\(Field\)的名字。  
    当进行build构建操作时，每个域\(Field\)都会以字典的方式通过key取其对应的value来构建二进制数据。  
2. 各种域parse和build操作的结果会按照其名字依次存放到表示结构的上下文字典中。

**例1：**

{% code-tabs %}
{% code-tabs-item title="struct1.py" %}
```python
#!/usr/bin/env python3

from construct import *

format = Struct(
    "signature" / Const(b"BMP"),
    "width" / Int8ub,
    "height" / Int8ub,
    "pixels" / Array(this.width * this.height, Byte),
)

bmp_data = format.build(dict(width=3, height=2, pixels=[7, 8, 9, 11, 12, 13]))
print(type(bmp_data))
print(bmp_data)

print(10 * '-')

bmp_struct = format.parse(bmp_data)
print(type(bmp_struct))
print(bmp_struct)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

上面定义了一个包含4个域的Struct，并将其存放到format变量中，其中：

* 名为signature的域，是一个内容为b"BMP"的字节常量\(Const\)，占用3个Byte
* 名为width和height的域，分别是一个字节组成的无符号整型数据\(Int8ub\)
* 名为pixels的域，是由字节\(Byte\)构成的数组\(Array\)，其大小通过另外两个域width和height计算得到

上面例子的运行结果如下：

```bash
$ python3 struct1.py 
<class 'bytes'>
b'BMP\x03\x02\x07\x08\t\x0b\x0c\r'
----------
<class 'construct.lib.containers.Container'>
Container: 
    signature = b'BMP' (total 3)
    width = 3
    height = 2
    pixels = ListContainer: 
        7
        8
        9
        11
        12
        13
```



