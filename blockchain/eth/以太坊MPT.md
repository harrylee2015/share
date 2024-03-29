# 以太坊MPT树

MPT（Merkle Patricia Tree）是以太坊用来将Key-Value进行紧凑编码的一种数据组织形式。基于该数据组织形式，MPT上任何存储数据的细微变化都会导致MPT的根节点发生变更，因此可以校验数据的一致性。



在MPT中，存在如下三类节点。



1. 叶子节点：用于数据存储的节点。其Key值是一个对应插入数据的特殊16进制编码（需要剔除掉从根节点到当前叶子节点的前缀部分内容），
 Value值对应插入数据的RLP编码。

2. 扩展节点：扩展节点用来处理具有共同前缀的数据，通过扩展节点可以扩展出一个多分叉的分支节点。其Key值存储的是共同的前缀部分的16进制，
 Value值存储的是扩展出的分支节点的hash值（sha3（RLP（分支节点数据List）））。

3. 分支节点：由于MPT存储的Key是16进制的编码数据，那么在不具备共同前缀时就通过分支节点进行分叉。分支节点的key是一个16个数据的数组，
 数组的下标对应16进制的0-F，用来扩展不同的数据。


数组中存储的的是分叉节点的hash值，同时数组的下标也对应一个数据的一个16进制位（4bit）。分支节点的Value一般为空，
如果有数据的Key值在其上的扩展节点终止，那么数据Key对应的Value值则存储在分支节点的Value属性上。

扩展节点和叶子节点的Key值前面存在一个4bit或者8bit nibble，用来标示节点类型和后续的数据长度的奇偶性（原始数据Key值16进制后长度为偶数，
但是经过扩展节点和分支节点的前缀剔除后剩下的长度可能是奇数，也可能为偶数）。
## 关于编码

* Raw编码
MPT对外提供的API采用的就是Raw编码方式，这种编码方式不会对key进行修改，如果key是“foo”, value是"bar"，编码后的key就是["f", "o", "o"]。


假设我们要把`a`作为key放入MPT树，key可以直接用`a`的ASCII表示97就可以了。

* Hex编码
可以发现采用Raw编码以后，从a-z一共26个字母，如果采用`分支节点（BranchNode）`存储的话需要26个空间，如果再加上0-9一共10个数字和一个value，总共需要37个空间，以太坊的开发者权衡了一下觉得太多了，于是就改良了编码方式，有了Hex编码。



以太坊先定义了一个新单位`nibble`，一个`nibble`表示4个bit，0.5个byte。然后按照如下规则编码；

将Raw输入的每个字符（1byte）拆分成2个nibble，前4位和后4位各一个nibble；
将每个nibble扩展为1个byte（8个bit）；
然后分别将Raw编码后的十六进制结果的每个`b`进行如下操作
1. b / 16；

2. b % 16；

如下示例：
```
a的ASCII编码为99（十进制），转换十六进制为63
采用Hex编码
[0] = 61 / 16 = 3
[1] = 61 % 16 = 13
编码后的结果 [3, 13]

在举个例子
输入"cat"，Raw编码后 [63, 61, 74]
63 / 16  = 3
63 % 16 = 15
61 / 16 = 3
61 % 16 = 13
74 / 16 = 4
74 % 16 = 10
编码后的结果 [3，15，3，13，4，10]
```
树的最后一位value是一个标识符，如果存储的是真实的数据项，即该节点是叶子节点，则在末尾添加一个ASCII值为16的字符作为终止标识符。
添加后的结果是` [3，15，3，13，4，10, 16]`

* HP编码（Hex-Prefix Encoding 16进制前缀编码）

如果要对Hex编码后的数据进行持久化，就会发现一个问题，我们对原数据进行了扩展，本来1个byte的数据被我们变成了2个byte，
显然这对于存储来说是不可接受的，于是就又有了HP编码。

HP编码的过程如下；

输入key（Hex编码的结果）如果有标识符，则去掉这个标识符。
key的头部填充一个nibble，填充的规则如下
1. 如果key的nibble长度是偶数则最后一位0

2. 如果key的nibble长度是奇数则最后一位1

3. 如果key是`扩展节点`则倒数第二位是0

4. 如果key是`叶子节点`则倒数第二位是1

举个例子
```
"cat"经过Hex编码后的结果 [3，15，3，13，4，10, 16]
再用HP编码
1. 去掉16，同时表明这个是叶子节点。
2. 叶子节点，nibble数量是奇数个，这两个条件得出需要填充的值为 0010 0000
3. 将HP编码后的结果用二进制表示 [0010, 0000，0011, 1111, 0011, 1101, 0100, 1010]
4. 将HP编码后的结果合并成byte，为[00100000, 00111111, 00111101, 01001010]转换为十进制是[32, 63, 61, 74]
```

nibble|nibble含义
---|------
00000000/0x00|该节点为扩展节点，并且数据长度为偶数
0001/0x1|该节点为扩展节点，并且数据长度为奇数
00100000/0x20|该节点为叶子结点，并且数据长度为偶数
0011/0x3|该节点为叶子节点，并且数据长度为奇数


* 在MPT树上节点存储的数据是原始数据的如下映射：

`Key = sha3(原始数据key)`

`Value = RLP(原始数据Value)`


