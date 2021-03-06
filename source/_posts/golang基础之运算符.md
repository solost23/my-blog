---
title: golang基础之运算符
date: 2021-12-10 11:00
tags:
    - golang
---

运算符用于在程序运行时执行数学或逻辑运算。

golang内置的运算符有：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 位运算符
- 赋值运算符

### <center>算术运算符</center>

| 运算符 |   含义    |
| :----: | :-------: |
|   +    |   相加    |
|   -    |   相减    |
|   *    |   相乘    |
|   /    | 相除/求商 |
|   %    |   求余    |

<font color=red>注意:</font>

- "/"两边必须都是浮点数才会获得准确值，否则为求商。
- "%"两边必须都是整数。
- ++(自增)和--(自减)在golang中是单独的语句，并不是运算符，且只支持后置。

### <center>关系运算符</center>

| 运算符 |                             含义                             |
| :----: | :----------------------------------------------------------: |
|   ==   |     检查两个值否是相等，若相等返回 true，否则返回 false      |
|   !=   |   检查两个值是否不相等，若不相等返回 true，否则返回 false    |
|   \>   |  检查左边值是否大于右边值，若大于返回 true，否则返回 false   |
|  \>=   | 检查左边值是否大于等于右边值，若大于等于返回 true，否则返回 false |
|   <    |  检查左边值是否小于右边值，若小于返回 true，否则返回 false   |
|   <=   | 检查左边值是否小于等于右边值，若小于等于返回 true，否则返回 false |

### <center>逻辑运算符</center>

| 运算符 |                             含义                             |
| :----: | :----------------------------------------------------------: |
|   &&   | 逻辑 AND 运算符，若两边的操作数都为 true，则为 true，否则为 false。 |
|  \|\|  | 逻辑 OR 运算符，若两边的操作数有一个为 true，则为 true，否则为 false |
|   !    | 逻辑 NOT 运算符，右边操作数为 true，则为 false，否则为 true。 |

### <center color=red>位运算符</center>

位运算符对整数在内存中的二进制位进行操作。

| 运算符 |                             含义                             |
| :----: | :----------------------------------------------------------: |
|   &    |  参与运算的两数各对应的二进制位相与。（两位均为 1 才位 1）   |
|   ｜   | 参与运算的两数各对应的二进制位相或。（两位有一个为 1 就为 1） |
|   ^    | 参与运算的两数各对应的二进制位相异或，当俩对应的二进制位相异时，结果为 1（两位不一样则为 1） |
|   <<   | 左移 n 位就是乘以 2 的 n 次方。“a<<b” 就是把 a 的各二进制位全部左移 b 位，高位丢弃，低位补 0. |
|  \>>   | 右移 n 位就是除以 2 的 n 次方。“a>>b” 就是把 a 的各二进制位全部右移 b 位，高位补零，低位丢弃。 |

### <center>赋值运算符</center>

| 运算符 |                      含义                      |
| :----: | :--------------------------------------------: |
|   =    | 简单的赋值运算符，将一个表达式的值赋给一个变量 |
|   +=   |                  相加后再赋值                  |
|   -=   |                  相减后再赋值                  |
|   *=   |                  相乘后再赋值                  |
|   /=   |                   相除后赋值                   |
|   %=   |                   取余后赋值                   |
|  <<=   |                   左移后赋值                   |
|  \>>=  |                   右移后赋值                   |
|   &=   |                  按位与后赋值                  |
|  \|=   |                  按位或后赋值                  |
|   ^=   |                 按位异或后赋值                 |

