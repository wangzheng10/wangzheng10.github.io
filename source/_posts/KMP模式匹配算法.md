---
title: KMP模式匹配算法
date: 2022/4/12
categories:
- 数据结构
---

@[TOC](目录)

---

# 朴素模式匹配

串的模式匹配指的是子串在主串中的定位匹配，例如主串A = "abcdef"，子串B = "cd"，那么子串与主串的第三、四位相同。
思路：
（1）首先应该让主串的第一位与子串的第一位对比，相等则对比第二位，否则子串向后移一位，子串第一位与主串第二位对比。
（2）子串的第一位依次与主串的第1-5位匹配对比。

```c
/****************************************\
*输入：char Fat[]--主串，char Son[]--子串
*输出：无
*返回：匹配成功的位置，不成功则返回-1
\****************************************/
int Index(char Fat[], char Son[])
{
    int i = 0;
    int j = 0;

    int FLen = strlen(Fat);//获取主串和子串的长度
    int SLen = strlen(Son);

    while ((i < FLen) && (j < SLen)) //当子串和主串均未匹配完
    {
        if (Fat[i] == Son[j]) //当前字符相同，则匹配下一位字符
        {
            i++;
            j++;
        }
        else
        { /*当前字符不相同，下一个应该匹配的应该是，上次主串匹配的
            下一位字符i-j+2（上次主串匹配的前一个字符位置：i-j）。
            子串下次匹配应该重新是第0位*/
            i = i - j + 2;
            j = 0;
        }
    }

// 匹配结束，如果子串匹配完最后一位，则匹配成功。
//否则，此时是主串已经全部匹配完，没有找到相同的部分，匹配失败
    if (j == SLen)
    {
        return (i - j);
    }
    else
    {
        return -1;
    }
}
```

---
---

# KMP模式匹配

KMP模式匹配，主要是分析子串的字符相似度，减少没有必要的匹配。
例如主串A = "abcdefg"，子串B = "abcx"，在`d`与`x`匹配失败后，没有必要从主串的`b,c`开始匹配了，因为前三位匹配成功，但子串`b,c`均不与首字符相同，则主串相应位置也不会相同。

另一种是子串相似度较高的情况，例如主串A = "abababc"，子串B = "ababc"，在第一次匹配失败后，不需要再从主串的第1，2，3位开始匹配，而是直接对主串的第4位`a`和子串的第2位`a`进行匹配。
![匹配样例](https://img-blog.csdnimg.cn/371fe0910f7b469ebf7fef8c4501cc36.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
因为子串B的第0-1字符与2-3字符相同，且子串B与主串A前4个字符相等，那么主串A的2-3字符自然与子串B的0-1字符相等，只要从主串的第4位和子串的第2位开始比较即可。

匹配时，主串的匹配位置是不回溯的，而子串的匹配位置是会回溯的，因此用next数组表示对应的位置匹配失败时，回溯的位置。

数学表达式如下：
$$
next[j] = \left\{
\begin{array}{l}
-1 &   &{j = 0}\\
Max\{k | 0<k<j, B[0]B[1]...B[k-1] = B[j-k+1]B[j-k+2]...B[j-1]\} &   &{集合不为空}\\
0  &   &{其他情况}
\end{array}
\right.
$$

next数组推导，例如子串B = "ababacc"。
（1）next[0] = -1；next[1] = 0；
（2）当j = 3时，前面的字符为`ab`，前缀`a`与后缀`b`不等，next[3] = 0；
（3）当j = 4时，前面的字符为`aba`，前缀`a`与后缀`a`相等，next[4]  = 1；
（4）当j = 5时，前面的字符为`abab`，前缀`ab`与后缀`ab`相等，next[5] = 2；
（5）当j = 6时，前面的字符为`ababa`，前缀`aba`与后缀`aba`相等，next[6]  = 3；
（6）当j = 7时，前面的字符为`ababac`，前缀`a`与后缀`c`不等，next[7] = 0。
所以该子串的next数组为{-1，0，0，1，2，3，0}。


程序如下：
```c
/****************************************\
*输入：char Son[]--子串
*输出：int next[]
*返回：无
\****************************************/
void Get_Next(char Son[], int next[])
{
    int i = 1;
    int j = 0;

    next[0] = -1;
    next[1] = 0;

    int Len = strlen(Son);

    while (i < Len-1)
    {
        if ((j == -1) || Son[i] == Son[j])
        {
            i++;
            j++;
            next[i] = j;
        }
        else
        {
            j = next[j];
        }
    }
}
```

---

KMP模式匹配程序如下：
```c
/****************************************\
*输入：char Fat[]--主串，char Son[]--子串
*输出：无
*返回：匹配成功的位置，不成功则返回-1
\****************************************/
int Index_KMP(char Fat[], char Son[])
{
    int i = 0;
    int j = 0;

    int FLen = strlen(Fat);
    int SLen = strlen(Son);

    int next[SLen];
    Get_Next(Son, next);


    while ((i < FLen) && (j < SLen))
    {
        if ((j == -1) || (Fat[i] == Son[j])) //j=-1时: 子串的第0位与主串的第i位匹配失败，继续与主串的第i+1位匹配
        {
            i++;
            j++;
        }
        else
        {
            j = next[j]; //子串回溯到对应的next[j]位置
        }
    }

    if (j == SLen)
    {
        return (i - j);
    }
    else
    {
        return -1;
    }
}
```

---

测试用例：
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>

#define N (100000)
int main(void)
{
	char B[] = "ababaabaaaababa";

    char ch[N+1];

    srand((unsigned)time(NULL));
    for (int i=0; i < N; i++) //随机产生由a,b组成的串，长度为N
    {
        ch[i] = 'a' + rand()%2;
    }
    ch[N] = '\0';

    int Pos = Index_KMP(ch, B);

    printf("%d\n", Pos);

    return 0;
}
```

---

KMP模式匹配是通过跳过子串前部相同的一段，但是对于正要匹配的字符，如果与主串不匹配，而其next位的字符和它相等，则没有必要再与主串匹配了。
例如主串A = "aaaabcabc"，子串B = "aaaaac"，在主串第4位`b`与子串的第4位`a`匹配失败时，根据next数组，next[] = {-1, 0, 1, 2, 3, 4}，之后会依次将子串的第3、2、1、0位与主串第4位进行匹配，这是没有必要的。

因此我们可以在计算next数组时，判断当前位置的字符是否与原先next[i]位置的字符相等，如果相等，就将前缀字符的next赋值给当前位置的next。

```c
void Get_NextImprov(char Son[], int next[])
{
    int i = 1;
    int j = 0;

    next[0] = -1;
    next[1] = 0;

    int Len = strlen(Son);

   
    while (i < Len-1)
    {
        if ((j == -1) || Son[i] == Son[j])
        {
            i++;
            j++;
            if (Son[i] != Son[j])
            {
                next[i] = j;
            }
            else
            {
                next[i] = next[j];
            }
        }
        else
        {
            j = next[j];
        }
    }
}
```