# BitStruct

默认情况下Construct库里面的数据是按照字节解析和构建的。很多时候，在字节层面还不够，需要到Bit进行解析，此时需要用到BitStruct类。

在使用BitStruct进行解析时，数据需要首先被转换为比特流，然后按照BitStruct的定义对比特流顺序进行拆分并生成BitStruct对象。因此BitStruct包含的子结构都是基于比特\(bit\)而不是字节\(byte\)。

> BitStruct跟C语言中的结构体类似，但是二者定义的成员顺序是不一样的。  
> 例如：  
> C语言中，定一个基于字节的结构体：  
> BitStruct定义的结构体是按照比特流的顺序进行的：

**例**

解析一个32 bit的结构体，其定义如下：

* Bit 0~7，表示某种意义的Bond选项
* Bit 8~13，表示版本的字符
* Bit 14~19，表示产品名称的第二个字符
* Bit 20~25，表示产品名称的第一个字符

在C语言中我们是如何定义这个结构体的呢？

```c

```

使用Construct库需要这样的顺序来定义：

```text
part = BitStruct(
    BitsInteger(6),
    "first" / BitsInteger(6),
    "second" / BitsInteger(6),
    "revision" / BitsInteger(6),
    "bond" / BitsInteger(8),
)
```

示例代码如下：

{% code-tabs %}
{% code-tabs-item title="decode-32bit-data.py" %}
```python
from construct import *

"""
bit[25:20] = First Letter    (0x1 = A, 0x2 = B …)
bit[19:14] = Second Letter   (0x1 = A, 0x2 = B …)
bit[13:08] = Revision Letter (0x1 = A, 0x2 = B, ...)
bit[07:00] = Bond option (0x1 = 1, 0x2 = 2, …)
"""

letter = Enum(BitsInteger(6),
              A=1,
              B=2,
              C=3,
              D=4,
              E=5,
              F=6,
              G=7,
              H=8,
              I=9,
              J=10,
              K=11,
              L=12,
              M=13,
              N=14,
              O=15,
              P=16,
              Q=17,
              R=18,
              S=19,
              T=20,
              U=21,
              V=22,
              W=23,
              X=24,
              Y=25,
              Z=26
              )

part = BitStruct(
    BitsInteger(6),
    "first" / letter,
    "second" / letter,
    "revision" / letter,
    "bond" / BitsInteger(8),
)

d = (0x15A08203).to_bytes(4, byteorder='big')
print(d)
x = part.parse(d)
print(x)

x = part.parse(b'\x15\xA0\x82\x03')
print(x)

print("Part:{}{}, Rev:{}, Bond:{}".format(x.first, x.second, x.revision, x.bond))

```
{% endcode-tabs-item %}
{% endcode-tabs %}

执行结果如下：

```bash
$ python3 decode-32bit-data.py
b'\x15\xa0\x82\x03'
Container: 
    first = (enum) Z 26
    second = (enum) B 2
    revision = (enum) B 2
    bond = 3
Container: 
    first = (enum) Z 26
    second = (enum) B 2
    revision = (enum) B 2
    bond = 3
Part:ZB, Rev:B, Bond:3
```

