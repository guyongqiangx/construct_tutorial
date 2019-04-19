# Compression and checksuming

## Compression

**例1**

{% code-tabs %}
{% code-tabs-item title="compression.py" %}
```text
from construct import *
from construct.lib.hex import hexdump

d = Prefixed(VarInt, Compressed(GreedyBytes, "zlib"))

data = d.build(bytes(100))
print(hexdump(data, 16))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

执行结果：

```text
$ python3 compression.py
hexundump("""
0000   0C 78 9C 63 60 A0 3D 00 00 00 64 00 01            .x.c`.=...d..
""")
```

## Checksum

**例1**

{% code-tabs %}
{% code-tabs-item title="checksum.py" %}
```python
from construct import *
from construct.lib.hex import hexdump

import hashlib

x = Struct(
    "fields" / RawCopy(Struct(
        "a" / Byte,
        "b" / Byte,
    )),
    "checksum" / Checksum(Bytes(16), lambda data: hashlib.md5(data).digest(), this.fields.data),
)

y = hashlib.md5(b"\x01\x02").digest()

data = x.build(dict(fields=dict(data=b"\x01\x02")))
print(hexdump(data, 16))

print(x.parse(b'\x01\x02' + y))

```
{% endcode-tabs-item %}
{% endcode-tabs %}

执行结果:

```text
$ python3 checksum.py
hexundump("""
0000   01 02 0C B9 88 D0 42 A7 F2 8D D5 FE 2B 55 B3 F5   ......B.....+U..
0010   AC 7A                                             .z
""")

Container: 
    fields = Container: 
        value = Container: 
            a = 1
            b = 2
        length = 2
        offset1 = 0
        offset2 = 2
        data = b'\x01\x02' (total 2)
    checksum = b'\x0c\xb9\x88\xd0B\xa7\xf2\x8d\xd5\xfe+U\xb3\xf5\xacz' (total 16)
```

