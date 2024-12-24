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

