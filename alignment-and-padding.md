# Alignment and padding

### 1. Padding

从代码的实现来看，Padding是对Padded类的封装:

{% code-tabs %}
{% code-tabs-item title="core.py" %}
```python
def Padding(length, pattern=b"\x00"):
    ...
    macro = Padded(length, Pass, pattern=pattern)
    ...
    return macro
```
{% endcode-tabs-item %}
{% endcode-tabs %}

因此Padding的调用方式为`Padding(length, pattern=b"\x00")`

**例1**

{% code-tabs %}
{% code-tabs-item title="simple-padding.py" %}
```python
from construct import *
from construct.lib.hex import hexdump

st = Struct(
    "signature" / Const(b"BMP"),
    Padding(16)
)

data = st.build({})
print(hexdump(data, 16))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

执行结果：

```text
$ python3 simple-padding.py
hexundump("""
0000   42 4D 50 00 00 00 00 00 00 00 00 00 00 00 00 00   BMP.............
0010   00 00 00                                          ...
""")
```

从输出结果可见，这里是直接在3个字节的"BMP"后面直接以0x00填充了16个字节。注意这里是单独填充了16个字节，这16字节中并不包含3字节的"BMP"数据。

**例2**

{% code-tabs %}
{% code-tabs-item title="padding.py" %}
```python
from construct import *
from construct.lib.hex import hexdump

import hashlib

d = Struct(
    "fields" / RawCopy(Struct(
        Padding(128, b'\x30'),
    )),
    "checksum" / Checksum(Bytes(64),
        lambda data: hashlib.sha512(data).digest(),
        this.fields.data),
)

data = d.build(dict(fields=dict(value={})))
print(hexdump(data, 16))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

这个例子中定义了一个数据结构d，有用两个成员：

* "fields": 通过Padding\(256, b'\x30'\)操作生成
* "checksum": 对fields数据计算sha512得到

> 这个例子包含的知识点比较多，有：  
> 1. Struct的嵌套  
> 2. RawCopy访问原始数据  
> 3. Padding操作生成数据  
> 4. Checksum操作生成数据  
> 5. Checksum中使用lamda传递数据

执行结果为：

```bash
$ python3 padding.py
hexundump("""
0000   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0010   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0020   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0030   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0040   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0050   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0060   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0070   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0080   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0090   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
00A0   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
00B0   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
00C0   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
00D0   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
00E0   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
00F0   30 30 30 30 30 30 30 30 30 30 30 30 30 30 30 30   0000000000000000
0100   35 0C F3 E6 D7 1A 8D 70 69 D4 DC 94 98 33 30 AD   5......pi....30.
0110   B6 AA 13 4E 01 A0 05 A7 70 9A 51 F5 1F 16 01 DC   ...N....p.Q.....
0120   5D 37 B9 9E 88 B6 14 C5 1F BA 29 B3 E9 40 C1 58   ]7........)..@.X
0130   70 F4 B3 4A 06 73 13 14 7A C6 A8 8C 78 0B CE 8D   p..J.s..z...x...
""")
```

**例3**

基于例2的一个更加复杂的例子：

{% code-tabs %}
{% code-tabs-item title="padding2.py" %}
```text
from construct import *
from construct.lib.hex import hexdump

import hashlib

d = Struct(
    "offset" / Tell,
    "checksum" / Padding(32),
    "fields" / RawCopy(Struct(
        Padding(128, b'\x20'),
    )),
    "checksum" / Pointer(this.offset, Checksum(Bytes(20),
        lambda data: hashlib.sha1(data).digest(),
        this.fields.data)),
)

data = d.build(dict(fields=dict(value={})))
print(hexdump(data, 16))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

> 相比前面的例1，这里例2包含的知识点更多，有：  
> 1. Tell操作  
> 2. Padding操作\(两处Padding\)  
> 3. Struct嵌套  
> 4. RawCopy访问原始数据  
> 5. Checksum操作生成数据  
> 6. Checksum中使用lamda传递数据  
> 7. Pointer指向引用数据

运行结果：

```text
$ python3 padding2.py
hexundump("""
0000   FF B5 E5 D9 6E 19 71 4F FE F6 0A C8 74 9E CA EF   ....n.qO....t...
0010   BE C9 D2 95 00 00 00 00 00 00 00 00 00 00 00 00   ................
0020   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
0030   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
0040   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
0050   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
0060   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
0070   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
0080   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
0090   20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
""")
```

### 2. Padded



### 3. Aligned



### 4. AlignedStruct

