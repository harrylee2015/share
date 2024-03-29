# 以太坊RLP编码

RLP（Recursive Length Prefix）编码是以太坊中数据序列化的一个主要编码方式，可以将任意的嵌套二进制数据进行序列化。
以太坊中针对RLP的编码规则定义如下：

* 1. 如果是一个单字节并且其值在[0x00,0x7f]范围内，RLP编码就是自身。

* 2. 否则，如果一个数据串的字节长度是0-55字节，那么它的RLP编码是在数据串开头增加一个字节，
     这个字节的值是0x80加上数据串的字节长度。因此增加的该字节的取值范围为[0x80, 0xb7]。

* 3. 如果一个数据串的字节长度大于55，那么它的RLP编码是在开头增加一个字节，这个字节的值等于0xb7加上数据串字节长度的二进制编码的字节长度，
     然后依次跟着数据串字节长度部分和内容部分。比如：一个长度为1024字节的数据串，其字节长度用16进制表示为0x0400，长度为2个字节，
     因此RLP编码头字节的值为0xb9（0xb7 + 0x02），然后跟着两字节为0x0400，后面再加上数据串的具体内容。
     因此增加的首字节的取值范围为[0xb8, 0xbf]，因此其能编码的最大数据长度为(2^64 - 1)。

* 4. 如果是一个嵌套的列表数据，则需要先将列表中的数据按照单元素的编码规则进行RLP编码后串联得到列表数据的payload。
     如果一个列表数据的payload的字节长度为0-55，那么列表的RLP编码在其payload前加上一个字节，这个字节的值是0xc0加上payload的字节长度。
     因此首字节的取值范围为[0xc0, 0xf7]。

* 5. 如果一个列表数据的payload的长度大于55，那么它的RLP编码是在开头增加一个字节，这个字节的值等于0xf7加上列表payload
     字节长度的二进制编码的字节长度，然后依次跟着payload字节长度部分和payload部分。
     因此首字节的取值范围为[0xf8, 0xff]，因此一个列表中存储的所有元素的字节长度不能超过(2^64-1)。
