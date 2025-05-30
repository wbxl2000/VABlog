#H.264描述符

### 简述

H264/AVC文档中存在着大量元素描述符ue(v)，me(v)，se(v)，te(v)等，编码为ue(v)，me(v)，或者 se(v) 的语法元素是指数哥伦布编码，而编码为te(v)的语法元素是舍位指数哥伦布编码。这些语法元素的解析过程是由比特流当前位置比特开始读取，包括非0 比特，直至leading_bits 的数量为0。由于解码过程中需要计算出每个元素的占位(bit数量)，所以我们就不得不理解这些描述符了。

###浅谈哥伦布编码
在说哥伦布编码之前首先先谈谈什么是定长编码？什么是可变长度编码？

#### 定长编码

即固定长度的编码，如：对汉字进行编码时，每个汉字都是2两个字节。

好处是只要读取该固定的字节长度就能取出值。

坏处是会浪费很多不必要的内存，比如用1个字节能存储的值，却必须要2个字节，浪费了一个字节。

#### 可变长编码

长度不固定的编码，随内容大小而改变编码长度。

好处是有多少内容，就占用多少内存。

但是因为内容不固定，所以得在内存中是如何知道这些内容的长度呢？所以一般都是在内容前面加上内容的长度。


#### 为何H264中采用哥伦布编码
在H264编码中，有大量的短数据（在255以内），如：宏块大小，各种标志位等。为了减少码流的长度，把这些短数据使用可变长度编码，而在众多可变长度编码的算法中，哥伦布编码尤其合适H264编码。因为数据的值小，在其二进制的值上+1再补长度个数的0即可，这样就可以把每个bit用到，不会产生多余的浪费。

而为何只是用于短数据呢？因为多少个0代码数据长度，也就是说数据大的，就会产生很多很多0，会造成更大的浪费。具体看下面原理。


### ue(v)

无符号整数指数哥伦布码编码的语法元素，左位在先。计算公式：

$$ codeNum = 2^{leadingZeroBits} − 1 + read_bits( leadingZeroBits ) $$

- **codeNum：** 占的位数（bit数量）
- **leadingZeroBits：** 遇到第一个1前面0的个数；如：0010 1011，leadingZeroBits的值为2；
- **read_bits( leadingZeroBits )：** 遇到第一个1后面leadingZeroBits个组成的无符号二进制值；如：0010 1011，值为...0 1...，即01，即1；

举两个栗子：

（1）0001 1001 => leadingZeroBits==000. ....=3，read_bits(3)==.... 100.=4，所以：$$codeNum=2^3-1+4=13$$
（2）0000 0101  0000 0000 => leadingZeroBits=5，read_bits(5)=.... ..01 000. ....=8，$$codeNum=2^5-1+8=39$$

### se(v)

有符号整数指数哥伦布码编码的语法元素位在先。在按照上面ue(v)公式计算出codeNum后，然后使用该计算公式：

value = $$(-1)^{k+1}$$ Ceil( k÷2 )

- **Ceil：** 返回大于或者等于指定表达式的最小整数，如： Ceil(1.5)= 2

如上例（1）0001 1001 => codeNum=$$2^3$$-1+4=13，value = $$(-1)^{13+1}$$Ceil(13÷2)=Ceil(6.5)=7

### me(v)

映射的指数哥伦布码编码的语法元素，左位在先。在按照上面ue(v)公式计算出codeNum后，然后查表。（在H.264-AVC-ISO_IEC_14496-10的9.1.2 章节中表9-4）

### te(v)

舍位指数哥伦布码编码语法元素，左位在先。明确取值范围在0-x；当x>1时，te(v)就是ue(v)；否则(x=1)，b = read_bits( 1 )，codeNum = !b

### 其他

- **ae(v)：** 上下文自适应算术熵编码语法元素
- **b(8)：** 任意形式的8比特字节。
- **f(n)：** n位固定模式比特串（由左至右），左位在先。
- **i(n)：** 使用n比特的有符号整数。
- **u(n)：** n位无符号整数。

### 参考

(1).[H.264-AVC-ISO_IEC_14496-10](http://103.23.20.16/srs/trunk/doc/H.264-AVC-ISO_IEC_14496-10.pdf)

(2).https://blog.csdn.net/lizhijian21/article/details/80982403