## data-lab

### 规则

表达式规则：

- 整数的范围在0~255之间，不能使用大整数
- 只能使用局部变量和参数
- 允许使用单目运算符! ~
- 允许使用双目运算符& ^ | + << >>

禁止使用：

- 非顺序执行的语句
- 宏
- 调用函数
- 使用不在规定范围内的操作符
- 使用强制类型转换
- 使用非int类型，使用数组，结构体等

**每个题目都会给出上面之外的具体规则**。

### 一些小坑

使用dlc时出现bits.c:xx:parse error bits.c:xx:undeclared variable xx 是因为该编译器仅仅支持C89的语法,**需要将变量全部在代码块的开头声明**。

使用dlc时出现bits.c:284: Warning: suggest parentheses around arithmetic in operand of x 是因为运算符优先级肯存在歧义，建议为优先运算块加括号

#### 1.按位异或

```
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */

// 仅用 ~ 和 & （按位取反和按位与运算）完成 ^ （按位异或运行）
```

思路：

1.通过真值表写出异或的逻辑表达式

```
1.	找出输出为 1 的行。
2.	根据输入值写出对应的逻辑表达式。
3.	用逻辑或将这些表达式连接起来，得到最终的逻辑表达式。
```

真值表：

| x  | y | x ^ y  |
|:--------|---------:|--------:|
| 0   | 0     | 0   |
| 0   | 1     | 1   |
| 1   | 0     | 1   |
| 1   | 1     | 0   |

所以，得出 x ^ y = (~x & y) | (x & ~y)

2. 德摩根定律如下：

```
~(x & y) = (~x) | (~y)
~(x | y) = (~x) & (~y)
```

所以，得出 x ^ y = (~x & y) | (x & ~y) = ~(~(~x & y) & ~(x & ~y))

解答如下：

```
int bitXor(int x, int y) {
  return ~(~(~x & y) & ~(x & ~y));
}
```

#### 2.最小的int

```
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
```

思路：

最小的int即 1000 0000 0000 0000 = -2^31，将1左移31位即可。

解答如下：

```
int tmin(void) {

  return 1 << 31;

}
```

#### 3.是否最大int

```
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
```

思路：

最大的int为：x = 0x7FFFFF,x_plus_1 = x + 1 = 0x80000000,x_plus_1 + x_plus_1 = 0（溢出）

但是x = -1也有类似的性质，所以要保证x != -1;

即要保证y = x_plus_1 + x_plus_1为0并且x_plus_1不为0

解答如下：

注意这里!的用法，!可以把非0值转为0，把0转为非0值

```
int isTmax(int x) {
    int x_plus_1 = x + 1; 
    int y = x_plus_1 + x_plus_1;
    return (!(y)) & !!x_plus_1; 
    // 检查 y 是否为 0 且 x + 1 不为 0
}
```

#### 4.判断所有奇数位是否为1

```
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */

// 如果一个整数（int类型）中所有奇数位（从最低有效位 0 到最高有效位 31 计算）都被设置为 1，则返回 1，否则返回 0。
```

思路：

只需要创建一个奇数位全为1，偶数为全为0的掩码0xAAAAAAAA,但是只能使用0～0xFF范围的常量，所以需要构造mask = 0xAA + (0xAA << 8) + (0xAA << 16) + (0xAA << 24);

如果符合条件，mask & x == mask，判断是否符合只需要对结果与mask本身取异或，符合结果为0

解答如下：

```
int allOddBits(int x) {
  int mask = 0xAA + (0xAA << 8) + (0xAA << 16) + (0xAA << 24);
  int x_and_mask = x & mask;
  int result = x_and_mask ^ mask; // 如果result为0，则说明x的所有奇数位都为1
  return !result;
}
```

#### 5.返回输入的负数

```
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
```

思路：

补码的性质：-x = ~x + 1

解答如下：

```
int negate(int x) {
  return ~x+1;
}
```

#### 6.判断是否是数字

```
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
```

思路：

通过上下界来判断，通过x - y = x + ~y + 1来计算减法（补码的性质）

通过符号位来判断结果大于0还是小于0，只需右移31位（算数移位，复制符号位）

解答如下：

```
int isAsciiDigit(int x) {
    int lower_bound = 0x30; // 下界
    int upper_bound = 0x39; // 上界

    int is_ge_lower = !((x + ~lower_bound + 1) >> 31); // 判断是否不小于下界
    int is_ge_upper = !((upper_bound + ~x + 1) >> 31); // 判断是否不大于上界

    return is_ge_lower & is_ge_upper; // 如果小于下界或大于上界，则返回0，否则返回1
}
```

#### 7.实现条件选择

```
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
```

思路：

根据x生成全1/全0掩码，再通过|进行实现选择

解答如下：

```
int conditional(int x, int y, int z) {
  int mask;
  x = !!x; // 0 if x is false, 1 if x is true
  mask = (~x) + 1;
  // 全0 mask if x is false, 全1 mask if x is true
  return (mask & y) | (~mask & z);
}
```

#### 8.小于等于

```
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
```

思路：

x - y = x + ~y + 1来计算减法

通过判断符号位来判断大于还是小于0

还要考虑两种溢出的情况：

如果x为正数，y为负数，直接返回0

如果x为负数，y为正数，直接返回1

解决如下：

```
int isLessOrEqual(int x, int y) {
    int diff = x + (~y + 1);  // 计算 x - y
    int signX = (x >> 31) & 1; // x 的符号位
    int signY = (y >> 31) & 1; // y 的符号位
    int signDiff = (diff >> 31) & 1; // x - y 的符号位
    int diffIsZero = !diff; // x - y 是否为 0
    int x_pos_y_neg = (!signX) & signY; // x 为正数，y 为负数
    int x_neg_y_pos = signX & (!signY); // x 为负数，y 为正数

    return ((signDiff | x_neg_y_pos | diffIsZero)) & (!x_pos_y_neg);
}
```

#### 9.实现逻辑非

```
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
```

思路：

如果如果x为非0，x | (-x) 的符号位为非0，否则为0

-x = ~x + 1

(x | (~x + 1)) >> 31 为全1，如果x为非0,全1 + 1 = 0

(x | (~x + 1)) >> 31 为全0，如果x为0，全0 + 1 = 1

解决如下：

```
int logicalNeg(int x) {
  return ((x | (~x + 1)) >> 31) + 1;
}
```

#### 10.最小能生成该数的位

```
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
```

思路：

对于正数，最少位数是其二进制表示加上一个符号位。

对于负数，需要包括符号位以及其他有效位（高位的连续 1 是符号扩展，不需要计入有效位数）。

1.统一符号位处理

对于正数，最左侧的 0 是符号位。

对于负数，最左侧的 1 是符号位。

因此，为了统一处理，可以通过 x 和符号位的异或 (x ^ sign) 将正数保持不变，而将负数转为等效的正数(按位取反)：

```
int sign = x >> 31; // 符号位：正数为0，负数为-1
x = x ^ sign;       // 如果是负数，相当于取反，如果是正数不变
```

2.查找最高有效位

为了找到需要的最少位数，可以通过逐步二分法确定 x 的最高有效位位置。

从高到低检查 x 的位。

使用位移和按位与操作，逐步确定是否存在高位为 1。

二分法的思想：

- 如果 x >> 16 不为零，则最高有效位在高 16 位。
- 如果 x >> 8 不为零，则在高 8 位。
- 依次类推，直到找到最高有效位。

解答如下：

```
int howManyBits(int x) {
  int bits, high16, high8, high4, high2, high1;
  int sign = x >> 31;   // 符号位：正数为0，负数为-1
  x = x ^ sign;         // 如果是负数，转为等效正数；正数保持不变

  // 开始二分查找最高有效位位置
  bits = 0;

  // 如果高 16 位有有效位
  high16 = !!(x >> 16) << 4; // 如果高 16 位不为零，结果加上 16
  bits += high16;
  x >>= high16; // 右移高 16 位

  // 如果高 8 位有有效位
  high8 = !!(x >> 8) << 3; // 如果高 8 位不为零，结果加上 8
  bits += high8;
  x >>= high8; // 右移高 8 位

  // 如果高 4 位有有效位
  high4 = !!(x >> 4) << 2; // 如果高 4 位不为零，结果加上 4
  bits += high4;
  x >>= high4; // 右移高 4 位

  // 如果高 2 位有有效位
  high2 = !!(x >> 2) << 1; // 如果高 2 位不为零，结果加上 2
  bits += high2;
  x >>= high2; // 右移高 2 位

  // 如果高 1 位有有效位
  high1 = !!(x >> 1); // 如果高 1 位不为零，结果加上 1
  bits += high1;
  x >>= high1; // 右移高 1 位

  // 最低 1 位
  bits += x; // 如果最低位不为零，加上 1

  // 返回所需位数，加上符号位
  return bits + 1;
}
```

!!(x >> 16)将高16位转化为bool值，如果为0代表高16为没有有效位，相应的high16也会为0

#### 11.浮点数乘2

```
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
```

思路：

熟悉单精度浮点数的组成：[IEEE浮点表示](https://github.com/mly0601/Wiki/blob/main/CSAPP/2_信息的表示与处理.md#ieee浮点表示)

单精度浮点数：32位（1位符号位s+ 8位指数位exp编码E + 23尾数位frac编码M）V = (-1)^s * M * 2^E

首先处理特殊情况（INF、NAN），特点为exp为0xff，直接返回这个数

然后处理规格数和非规格数

处理非规格数，需要确定是否会变成规格数，只需要看frac是否超过0x7fffff，若超过，frac需要去掉溢出部分，exp变成exp+1

处理规格数，只需要exp+1

解答如下：

```
unsigned floatScale2(unsigned uf) {
  unsigned s, exp, frac, ans;
  // 分解出s, exp, frac
  s = uf >> 31; // 符号位
  exp = (uf >> 23) & (0xff); // 指数位
  frac = uf & (0x7fffff); // 尾数位

  if (exp == 0xff) { // 如果是 NaN 或无穷大，直接返回原值
    return uf;
  } else if (exp == 0) { // 如果是非规格化数
    frac <<= 1; // 尾数左移一位，相当于乘2
    if (frac & (1 << 23)) { // 如果尾数溢出，说明由非规格数转化成规格数，指数加1（由0变成1）
      exp = 1;
      frac &= ~(1 << 23); // 清除尾数溢出的位
    }
  } else { // 规格化的情况
    exp = exp + 1; // 指数位加1，相当于乘2
  }

  // 将s, M, E重新组合成浮点数
  ans = (s<<31) | (exp << 23) | frac;
  return ans;
}
```

#### 12.将浮点数转化为整数

```
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
```

思路：

int()的规则，直接舍弃小数位

本题需要熟悉计算从浮点数转换为整数的规则，经过分析可以分为以下几种情况，

特殊值 NaN 和 infinity 的情况，会返回0x80000000u

非规格化的情况会返回0，因为非规格化表示的是小于1的数，后面是规格化的情况：

实际阶数 E 满足E < 0的数同样返回0，因为表示的数同样小于1

实际阶数 E 满足0 <= E <= 23的情况，保留整数部分的数（只有这个时候才能精确表示，可以参考书上练习），具体计算规则见代码

实际阶数 E 满足E > 23，根据题目要求返回0x80000000u

其中非特殊情况还要加上符号位。

解答如下：

```
int floatFloat2Int(unsigned uf) {
  unsigned s, exp, frac;
  int E, ans;
  //分解出s, exp, frac
  s = uf >> 31; // 符号位
  exp = (uf >> 23) & (0xff); // 指数位
  frac = uf & (0x7fffff); // 尾数位

  if (exp == 0xff) { // 如果是 NaN 或无穷大，直接返回 0x80000000u
    return 0x80000000u;
  } 
  
 if (exp == 0) { // 如果是非规格化数
    return 0; // 非规格化数直接返回0
  } 

  // 规格化的情况
  E = exp - 127; // 实际阶数
  if (E < 0) { // 表示的数小于1，直接返回0
      return 0;
  } 

  if (E <= 23) { // 符合条件的阶数，通过隐含的1和尾数位可以组成整数
    ans = (frac >> (23 - E)) + (1 << E);
  } else { // 阶数大于23，表示的数大于2^23，溢出，返回0x80000000u
      return 0x80000000u;
  }

  if (s) { // 如果是负数，取反加1
    ans = -ans;
  }
  return ans;
}
```

#### 13.返回单精度浮点数2^x的位级表示

```
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
```

思路：

**这里不考虑非规格的情况**！

规格化时，V = (-1)^s * M * 2^E，这里s = 0，frac = 0

E的范围在(-126,127),这个范围内exp = 127 + x

解答如下：

```
unsigned floatPower2(int x) {
    int exp; // 存储指数部分
    if (x < -126) {
        // 小于最小规格化数范围，可能是非规格化数或更小
        return 0;
    } else if (x > 127) {
        // 超过最大范围，返回正无穷
        return 0x7f800000; // +INF 的表示：符号位为 0，指数全为 1，尾数为 0
    } else {
        // 规格化范围内，计算指数部分
        exp = x + 127; // 加上偏移量 127
        return exp << 23; // 尾数为 0，因此只需将指数部分左移到正确位置
    }
}
```

#### 运行结果

运行结果如下：

![data-lab运行结果.png](pictures/data-lab运行结果.png)
