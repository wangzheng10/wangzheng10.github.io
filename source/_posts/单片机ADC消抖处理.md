---
title: 单片机ADC消抖处理
date: 2021/12/31
categories:
- 单片机
tags:
- 单片机
---

---

# 前言

单片机在实际读取AD值时，读到的值是不稳定的，在不断的变化，如果不经过处理，直接使用，是很难获得其准确的值，因此我们需要对其进行**消抖滤波**处理。

---

# 滤波

一般来说AD值的不稳，我们可以将一段时间读取到的数值取平均值，为了防止数值突变的影响，我们可以减去最大和最小值。

我们10ms读取一次AD值，取10次AD值的平均值即可。

``` c
static u16_t s_wMaxAdVal = 0; // AD最大值
static u16_t s_wMinAdVal = 1023; // AD最小值
static u16_t s_wSumAdVal = 0; // AD和
static u16_t s_wAdFiltVal = 0; // 滤波后的值
static u8_t s_nAdCnt = 0;

void AdFilter(u16_t wAdData)
{
  if (wAdData > s_wMaxAdVal)
  {
    wMaxAdVal = wAdData;
  }
  if (wAdData < s_wMinAdVal)
  {
    s_wMinAdVal = wAdData;
  }
  s_nAdCnt++;
  if (s_nAdCnt <= 10)
  {
    s_wSumAdVal += wAdData;
  }
  else
  {
    s_nAdCnt = 0;
    s_wSumAdVal -=  (s_wMaxAdVal + s_wMinAdVal);
    s_wAdFiltVal = s_wSumAdVal >> 3; // 除以8
    /* 重新计算最大最小值 */
    s_wMaxAdVal = 0;
    s_wMinAdVal = 1023;
    s_wSumAdVal = 0;
  }
}
```

该函数中s_wAdFiltVal的值即是滤波后的AD值。

---

# 温度转换

通常我们可以使用热敏电阻来检测温度，但是热敏电阻的阻值和温度的关系一般不可以通过数学表达式来描述，而是根据热敏电阻的规格书中阻值和温度的关系表来查找。

因此我们需要在程序中编写查找与AD值对应的温度。

---

## 硬件简述

![NTC硬件图](https://img-blog.csdnimg.cn/739639a291d14bda9dbfaf4b3071088b.png#pic_center)
简单的分压计算，通过温度与阻值的关系表，计算出温度与AD值关系表。

---

下面列出一个计算好的关系表：
``` c
#define RT_NUM 160
#define MINTEMP 0
const u16_t  wRTtable[RT_NUM] =
{
 /*  0     1     2     3     4     5     6     7     8     9  */   
     994,  992, 991,   989,  987,  985,  983,  981,  979,  977,
 /*  10    11    12    13    14    15    16    17    18    19 */   
     975,  973,  970,  968,  965,  962,  960,  957,  954,  951,  
 /*  20    21    22    23    24    25    26    27    28    29 */
     947,  944,  941,  937,  934,  930,  926,  922,  918,  913,
 /*  30    31    32    33    34    35    36    37    38    39 */
     909,  905,  900,  895,  890,  885,  880,  875,  870,  864,     
 /*  40    41    42    43    44    45    46    47    48    49 */
     858,  853,  847,  841,  835,  828,  822,  816,  809,  802, 
 /*  50    51    52    53    54    55    56    57    58    59 */
     795,  788,  781,  774,  767,  760,  752,  745,  737,  729, 
 /*  60    61    62    63    64    65    66    67    68    69 */
     721,  713,  705,  697,  689,  681,  673,  665,  656,  648, 
 /*  70    71    72    73    74    75    76    77    78    79 */  
     640,  631,  623,  614,  606,  597,  589,  581,  572,  564,  
 /*  80    81    82    83    84    85    86    87    88    89 */ 
     555,  547,  538,  530,  521,  513,  505,  497,  488,  480,  
 /*  90    91    92    93    94    95    96    97    98    99 */ 
     472,  464,  456,  448,  440,  433,  425,  417,  410,  402, 
 /*  100   101   102   103   104   105   106   107   108   109 */ 
     395,  388,  381,  374,  366,  360,  353,  346,  339,  333, 
 /*  110   111   112   113   114   115   116   117   118   119 */ 
     326,  320,  300,  308,  302,  296,  290,  284,  278,  273, 
 /*  120   121   122   123   124   125   126   127   128   129 */ 
     267,  262,  257,  251,  246,  241,  236,  232,  227,  222, 
 /*  130   131   132   133   134   135   136   137   138   139 */
     218,  213,  209,  205,  200,  196,  192,  188,  184,  181, 
 /*  140   141   142   143   144   145   146   147   148   149 */
     177,  173,  170,  166,  163,  159,  156,  153,  150,  147, 
 /*  150   151   152   153   154   155   156   157   158   159 */    
     144,  141,  138,  135,  132,  130,  127,  124,  122,  119, 
};

```
==注:== 温度应该是连续递增的。

---

## 温度查找

学过C语言的同学，应该都知道二分法查找，每次将当前的值与目标范围的中间值进行比较，从而不断缩小查找范围，时间复杂度为*O($\log_{2}{n}$)*。

``` c
void FindAdcTemp(u16_t wAdcVal, u16_t wTempData)
{
  u8_t nBegin = 0;
  u8_t nEnd= 0;
  u8_t nMiddle = 0;
  /* 首先判断是否超出表格的最大最小值 */
  nEnd = RT_NUM -1;
  if (wAdcVal <= *(wRTtable + nEnd))
  {
    wTempData = RT_NUM + MINTEMP;
  }
  else if (wAdcVal >= *(wRTtable + nBegin))
  {
    wTempData = MINTEMP;
  }
  else /* 当温度在表格范围内 */
  {
    while ((nBegin < nEnd) && (wAdcVal != *(wRTtable + nMiddle)))
    {
      nMiddle = nBegin + (nEnd - nBegin)>>1;
      if (wAdcVal > *(wRTtable + nMiddle))
      {
        nEnd = nMiddle - 1;
      }
      else
      {
        nBegin = nMiddle + 1;
      }
    }
    wTempData = MINTEMP + (nBegin -1);
    /* 接下来要计算当前的值更偏向于前一个还是后一个数 */
    if (wAdcVal < *(wRTtable + nBegin -1))
    {
      wDiffAdcVal = (*(wRTtable + nBegin -1) - wAdcVal)*10;
      wDiffAdcVal /= *(wRTtable + nBegin -1) - *(wRTtable + nBegin);
      if (wDiffAdcVal >= 5) // 四舍五入
      {
        wTempData = MINTEMP + nBegin;
      }
    }
  }
}
```

我们只要调用*FindAdcTemp*这个函数，就可以根据当前的AD值，计算得到最接近的温度值。
