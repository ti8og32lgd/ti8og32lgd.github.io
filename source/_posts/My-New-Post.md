title: 一个计算某个数的二进制表示中位值为1的位的个数
tags:
  - 位运算
commetns: true
permalink: perma
categories:
  - 编程基础
date: 2020-05-08 18:04:00
---
《深入理解计算机系统》练习题(3.49)中出现了一个函数：

{% codeblock lang:cpp %}
long fun_c(unsigned long x)
{
    long val = 0;
    int i;
    for (i = 0; i < 8; i++)
    {
        val += x & 0x0101010101010101L;
        x >>= 1;
    }
    val += (val >> 32);
    val += (val >> 16);
    val += (val >> 8);
    return val & 0xFF;
}
{% endcodeblock %}
首先说明这个函数的作用。计算某个unsigned long数的二进制表示中，位值为1的位的个数，一个直观的方法是，将这个数右移64次，每次和0x1交来从低（右）向高（左）判断该位是否为1:

```cpp
int bits_qual_1(unsigned long x)
{
    int rs = 0;
    int i;
    for (i = 0; i < 64; i++)
    {
        rs += x & 0x1L;
        x >>= 1;
    }
    return rs;
}
```

函数bits_qual_1为了得到结果需要执行循环64次，而func_c只需要8次！

这个函数采用了很隐晦的技巧，直接看上去很难理解。
第5行for循环中的作用是将x中的每8位(2Byte)中的1的数目加到val中的对应2Byte中。
例：
x:
0x2A00,0000,0000,0000
0b[0011,1010]00000...

mask:
0x0101010101010101L;
0b[0000,0001]0000,0001...

对x中的高8bit为例，for循环内执行了8次，x右移了8位，最后一次移位后结果没有与mask&操作并加到val上，等于被抛弃了。
每次&操作把x第8位上的的值加到了val的第8位上。
第1次循环，[0011,1010]中最后的0加到了val的第八位，
第2次循环，[0001,1101]中最后的1加到了val的第八位，
第3循环，[000,1110]中最后的0加到了val的第八位，
......
对于x中后面的每个8bit，都是进行着这样的操作。

退出for循环后，val中两个字节保存了x中对应两个字节中位值为1的位的数量，0=<数量(十进制)<=8,因为两个字节中最多8个1。但是，这个两个字节可以表示的最大数量是64(从1算起)，也就是整个x中位值为1的位的数量的最大值。

接下来将val对折3次，每次将对折的结果相加。
(解释）


在本书的第一个实验DataLab中需要大量运用这种技巧。最后再举一个相似的例子：
[leetcode 190. 颠倒二进制位](https://leetcode-cn.com/problems/reverse-bits/)
常规的解法是遍历每一位进行修改。类似于本文中介绍的方法，有一种不用遍历的方法：
```cpp
class Solution {
public:
    uint32_t reverseBits(uint32_t n) {
        auto ret = ((n & 0xaaaaaaaa) >> 1) | ((n & 0x55555555) << 1);
        ret = ((ret & 0xcccccccc) >> 2) | ((ret & 0x33333333) << 2);
        ret = ((ret & 0xf0f0f0f0) >> 4) | ((ret & 0x0f0f0f0f) << 4);
        ret = ((ret & 0xff00ff00) >> 8) | ((ret & 0x00ff00ff) << 8);
        return ((ret & 0xffff0000) >> 16) | ((ret & 0x0000ffff) << 16);
    }
};
```
1. 两两相邻的1位对调
2. 两两相邻的2位对调
3. 两两相邻的4位对调
4. 两两相邻的8位对调
5. 两两相邻的16位对调