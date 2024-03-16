
### Left shift
### Right shift
1. Logical shift(fill with 0)
2. Arithmetic shift(fill with the original sign)

### 整数的编码
1. unsigned
2. Two's Complement(signed) 
	actually the sign bit means Tmin(the smallest number) eg:1000 -> -8
1. can invert mapping(keep the representation and reinterpet )
```
0x1111 15 -16(the highest bit from 8 to -8) -> -1
-1U = + 2^32 to Tmax
we can always calculate two numbers by dec, then throw the extra bits that overflow(equivalent to %, the same from T mode to U mode), and see how it means in its encoding.
0111 + 1 = 1000 mod = 16
```
4. Tmin and Umin
	Tmin = 1<<31 
1.  x + ~x = -1
2. x & (-x) 1110(-2) 0010(2) --> 0010  Brian Kernighan算法
3. The constants are signed numbers by default, but signed values ==implicitly== cast to unsigned if  there is a mix of unsigned and     signed in a single expression, see the example : Tmax(U) < Tmin -1 is true
4. -2147483647 - 1 = Tmin = Umax(the same as -8 to 8)
### Sign Extension and Unsigned Extension
Make k copies of sign bit(Consider why by using T's Comp's understanding )

---
### 有无符号整数 的基本运算
#加法
加法 不区分有符号无符号， 合适的模数,并不同解释
#乘法
k bits * k bits
乘法 不区分有符号无符号 。C语言只保留k位，截断后结果一致（001）
对 2 ^ k取模，再不同解释
0x101 * 0x101 -> 11 001(25 ==11001== 或 9 ==1001==)
***通过位运算实现高效乘法***
x * k 等价于 x与k的各位相乘 再相加 k = 14(14 = 8+4+2),进行转化;
#除法
*只讨论特殊的除以2的k次幂*
1. 无符号整数 ***向下舍入除法*** 右移k位
2. 有符号整数 ***向下舍入除法*** 右移k位
```
x为原数，x1为高w-k位（共w位），x2为低k位，通过对无符号类似的分析
x = 2^k * x1 + x2; 可知向下取整
```
4. 由 向上取整 和 向下取整 之间的数学转换，可以得到**有符号向上舍入**除法
```
ceil(x/y) = floor((x+y-1) / y); ceiling,天花板，向上取整
若x<0,即有符号， (x+(1<<k)-1) >>k 能得到 有符号整数 的向上舍入
```

### 存储方式 
大端存储 和 小端存储 Big Endian / Little Endian

#小端存储 把数据的最低==字节==(不是bit)位置 放在存储端的 最高字节位置
-->假设向右地址增大 则：
0x12 34 56 ff 的存储为：0x ff 56 34 12
#大端存储 相反

### 检查数据的字节顺序(data representation)
按==内存地址顺序== 逐字节打印 看是否与原数的表示相同

### 字符串表示
'0' -> 0x30 

### Integer C Puzzles
 x &7 == 7 -->  x<<30 小于0
(-INT_MAX) * (-INT_MAX) = 1
x< y --> -x > -y
上面均正确


### Floating point（浮点数）
### 表示方法
类似 **原码**表示
s(sign) exp(exponent) M->Significand(尾数) M = f + 1
***公式: (sign)2^(E-Bias) *M***
**解释**：（规格化表示时，）尾数隐含以1开头，因为除了0以外的小数，都可以用==科学计数法== 让整数部分为1(2进制)
![[Pasted image 20240312220229.png]]
浮点数没有补码 的概念，采用类似 原码 的概念
#### 规格化表示

#单精度
指数 8bits 0-255  保留0,255不使用
类似无符号， 但会 -127(biased value/Bias) 来满足负指数的要求
#双精度
Bias = -1023 同样保留0,1023

#### 非规格化表示
为了表示**0和接近0的小数 和特殊数** 非规格化表示
exp = 0 ->10^-126 （**Bias = -126**,(-127+1)==不一样！！！==）
***这样设计的原因***：实现最大非规格化数与最小规格化数 的 平滑转变，作为对非规格化数不隐含开头1的补偿
exp全1  无穷数(inf)(frac=0) 和 非法数(NaN,not a number) (frac!=0)
 ==因此，浮点数溢出会得到正无穷或非法数==(可见上图看区分规则)

#### 实现C语言的双精度正无穷、负无穷、0
```
//粗略实现（答案是这么写的）
#define POS_INFINITY 1e400
#define NEG_INFINITY (-POS_INFINITY)
#define NEG_ZERO (-1.0/POS_INFINITY)
```

### 浮点运算
浮点加法不具有结合性（误差）
浮点加法与乘法具有单调性


# 位运算技巧
#实现掩码，获取需要的位
**1.取出最低有效字节**
```
x & 0xff
```
**2.取出除最低字节以外其它字节**
```
x & ~0xff
```
**2.5 不使用右移，取出符号位并转化为标准01**
```
//不使用右移
!!(x&(1<<31)) //10000 -> 1, 0->0
```
**3.判断是否为Umax**(为真返回1，后同)
```
return ! ~x
```
**3.5 常用掩码：2\^k-1***
```
通常由移位得到，可以得到大量的、分界的1和0
如果k=0;得到全0(2^0 - 1=0)
```

**4.判断最低有效字节是否全1**
结合1，3
```
return ! ~(x&0xff)
```
**5.判断是否奇数位全1**
```
//mask四个字节全为aa(1010)
mask = (0xAA<<24) | (0xAA<<16) | (0xAA<<8) // A -> 10
//清除x的偶数位 置为0
x = x & mask;
//异或运算来判等
return !(x^mask)
```
**6.用移位实现符号扩展**
```
//1字节到4字节(输入为unsigned char x)
int b = x;//举例，x=15; b=0x000f
b = b<<(3<<2) //b = 0xf000
return b>>(3<<2);//b = 0xffff;
```
**7.用算术右移来完成逻辑右移，或相反(注意怎么生成掩码）**
```
//算术右移实现逻辑右移
unsigned srl(unsigned x, int k){
	/*给出的算术右移值，之后操作不允许再右移*/
	unsigned xsra = (int)x>>k;
	//只需把高k位用掩码置零即可
	//生成掩码方式：让第k位为1，再整体减一（快速出现大量的1！）
	int x = 1<<(32-k) - 1;
	return xsra & x;
}
//逻辑右移实现算术右移
int sra(int x, int k){
	/*给出的逻辑右移值,之后操作不允许再右移*/
	int xsrl = (unsigned)x >> k;
	//高k位用掩码置为符号位 与 sss000 相或 (s表示sign)
	//利用2.5的知识
	int sign = !!(x & (1<<31));//得到标准0,1的s
	unsigned tem = ~((a<<(32-k)) - 1);
	int xsra = xsrl | tem;
	return xsra;
}
```
**8. 实现除以2^k（向0舍入）**
```
即上面提到的 
若x<0,即有符号， (x+(1<<k)-1) >>k 能得到 有符号整数 的向上舍入
若x>=0 直接是 x>>k
写成C表达式：
return x>=0?(x:(x+(1<<k)-1))>>k;
不能用条件判断，考虑怎么让x符号位为0时x+0，反之+(1<<k)-1
-->使用掩码
return (x + sign*((1<<k)-1))>>k //使用了不能使用的乘法
//根据符号位 生成全1 或 全0掩码 来实现乘1或乘0
s=1时，不好得到全1，先得到全0，再取反
s=0时，不好得到全0，先得到全1，再取反
统一为 ~(s-1)
return (x + ~(s-1) & (1<<k-1))>>k;
```
**判断是否含奇数个1**
```
显然异或
对与 前一半与后一半异或的结果， 只看后一半位数，后一半位数与整体含有1的个数的奇偶性相等
选择性忽略前一半即可
x ^= (x>>16);
x ^= (x>>8);
x ^= (x>>4);
x ^= (x>>2);
x ^= (x>>1); //只看最低位，选择性忽略其它位
return x& 0x1
```
**提取最左位置的1状态**
```
如 0xff00 返回0x8000，注意：if(x===0) return 0
//提示：为得到00010000 可先得到 00001111 or 00011111(再右移1位即可)
//得到 00011111更容易（雾）
//想到 得到交叉01的式子，再 位或 到一起
// 00010101 & (00010101 >>1) --> 00011111
// 利用最左位置的1可以得到 00010101
x |= x >> 16
x |= x >> 8
…
x |= x >> 1
此时得到了包括了最左位置靠右全1，先移位
x = （x >> 1） + 1
```