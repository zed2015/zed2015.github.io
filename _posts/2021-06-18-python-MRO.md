---
layout:     post
title:      python new style class mro 
subtitle:   python mro prescription
date:       2021-06-17
author:     ZC
header-img: 
catalog: 	 true
tags:
    - python
    - algorithm
---
### python 方法解析顺序【MRO】
By zc
> 当写 python 多重继承时，会碰到方法的继承覆盖问题，因此需要研究 python 关于方法的解析顺序问题，也就是MRO 问题

#### MRO 算法
----
##### 定义
> MRO 需要满足 local precedence order（局部有限顺序）、monotocity（单调性）这两个顺序

1. 在一个复杂的多继承体系中，给定类C，很难确定哪些方法被重写了，例如：确定类C的祖先顺序。
2. 类C的祖先列表，包含类C本身，从最近的祖先开始，一直到最远的祖先；成为类C的类优先序列(class precedence list)或者线性化(linearization)。
3. 方法解析顺序(MRO)是一系列用于构建线性化序列的规则；在Python中，术语"the MRO of C和类C的线性化一个意思。
4. 例如，在单继承的层次中，如果C是C1的子类、C1是C2的子类，那么C的线性化就是[C, C1, C2];然而，在多继承层次中，构建类的线性化列表就很麻烦，因为构建一个满足局部优先顺序(local precedence ordering)和单调性(monotonicity)的现象化会更困难。
5. 我会在后文讨论局部优先顺序，但要在这先说下单调性。当满足"如果在C的线性化中，C1在C2之前，那么在C的任意子类的线性化中，C1都在C2之前"时, 一个MRO就是单调性的。
6. 不是所有的类都会有线性化。在复杂的层次中，存在着一些不可能从一个类派生出一个新的子类使得该子类的序列化满足要求情况。

##### 算法举例描述

异常case:
```
 -----------
|           |
|    O      |
|  /   \    |
 - X    Y  /
   |  / | /
   | /  |/
   A    B
   \   /
     ? python2.3 raise error because merge(AXY0, BYXO, AB) = A + B + merge(XYO, YXO), X, Y 此时均不是好的头部元素，抛出异常
```
     
MRO 基本公式：
  - L[C(B1 ... BN)] = C + merge(L[B1] ... L[BN], B1 ... BN)
  - L[object] = object.
  - merge 公式算法：
    - take the head of the first list, i.e L[B1][0]; if this head is not in the tail of any of the other lists, then add it to the linearization of C and remove it from the lists in the merge, otherwise look at the head of the next list and take it, if it is a good head. Then repeat the operation until all the class are removed or it is impossible to find good heads. In this case, it is impossible to construct the merge, Python 2.3 will refuse to create the class C and will raise an exception.
    - 取出第一数组的头部元素，例如L[B1][0]; 如果这个头元素不在其它数组的的任何元素的尾部， 那么就将这个元素添加到C的线性化里然后从merge函数里的所有列表移出它，否则就查看下一个列表的头部元素，如果是一个真的头部元素(不在其它元素尾部)， 那么就重复这个这个步骤直到所有的元素（类）都被移除，或者不可能找到一个真的头部元素。在这个案例里，就是不可能构造merge, python2.3 以上会直接抛出异常；

正常case:
```
6
                         ---
Level 3                 | O |                  (more general)
                      /  ---  \
                     /    |    \                      |
                    /     |     \                     |
                   /      |      \                    |
                  ---    ---    ---                   |
Level 2        3 | D | 4| E |  | F | 5                |
                  ---    ---    ---                   |
                   \  \ _ /       |                   |
                    \    / \ _    |                   |
                     \  /      \  |                   |
                      ---      ---                    |
Level 1            1 | B |    | C | 2                 |
                      ---      ---                    |
                        \      /                      |
                         \    /                      \ /
                           ---
Level 0                 0 | A |                (more specialized)
                           ---
```

  - MRO 计算过程：
    -  1
    ```
    L[O] = O
    L[D] = D O
    L[E] = E O
    L[F] = F O
    ```
    -  2
    ```
    L[B] =  B D E O
    ```
    -  3
    ```
    L[C] = C + merge(DO,FO,DF)
     = C + D + merge(O,FO,F)
     = C + D + F + merge(O,O)
     = C D F O
    ```
    -  4
    ```
    L[A] = A + merge(BDEO,CDFO,BC)
     = A + B + merge(DEO,CDFO,C)
     = A + B + C + merge(DEO,DFO)
     = A + B + C + D + merge(EO,FO)
     = A + B + C + D + E + merge(O,FO)
     = A + B + C + D + E + F + merge(O,O)
     = A B C D E F O
    ```
#### 日常CODING 注意事项
- 尽量避免过于复杂的继承关系
- 扩展功能的父类优先放在左边
- 可以用组合模式减少复杂的继承

#### Resource 引用

python mro 官方文档描述 https://www.python.org/download/releases/2.3/mro/

中文翻译参考： https://zhuanlan.zhihu.com/p/25321349

