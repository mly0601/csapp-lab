## csapp-lab学习笔记

### data-lab

#### 1、按位异或

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

#### 2、最小的int

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

#### 3、是否最大int

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

#### 7.

```
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
```
