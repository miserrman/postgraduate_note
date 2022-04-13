# 字符串相关算法

## 后缀自动机

**如果用有向无环图表示所有字串，最简单的方法就是使用trie树 ** 

Trie又被称为前缀树、字典树，所以当然是一棵树。上面这棵Trie树包含的字符串集合是{in, inn, int, tea, ten, to}。每个节点的编号是我们为了描述方便加上去的。树中的每一条边上都标识有一个字符。这些字符可以是任意一个字符集中的字符。比如对于都是小写字母的字符串，字符集就是’a’-‘z’；对于都是数字的字符串，字符集就是’0’-‘9’；对于二进制字符串，字符集就是0和1

给定一个字符串，输出最长的重复子串

举例：ask not what your country **can do for you**,but what you**can do for you\**r\**** country

最长的重复子串：can do for you

思路：使用后缀数组解决

> 

1、由于要求最长公共子序列，则需要找到字符串的所有子串，即通过产生字符串的后缀数组实现。

2、由于要求最长的重复子串，则需要对所有子串进行排序，这样可以把相同的字符串排在一起。

3、比较相邻字符串，找出两个子串中，相同的字符的个数。
原文链接：https://blog.csdn.net/insistGoGo/article/details/7830972

#### 后缀自动机思路

一个子串，右端点出现的集合称为endpos

abcab endpos(ab) = {2, 5}

1. 如果两个子串的endpos相同，其中一个子串为另一个子串的后缀
2. 对于endpos相同的子串，我们把它归为endpos等价类，对于任意一个endpos等价类，其包含的其中子串的长度从大到小排序，每一个子串均为上一个子串-1
3. endpos等价类的个数级别是O(n)

> 构建一个相应的后缀树

![](D:\document\postgraduate\note\pic\后缀自动1.PNG)

延parent tree往下走是在字符串前面增加字符，延自动机往下走是在字符串后面添加字符

**后缀自动机每一个节点代表什么？**

定义一个子串的所有出现位置结尾的集合为这个子串的right集合。
每一个节点识别的子串right集合相同，right集合相同的子串用同一个点接受。有一个很有意思的性质是right集合相同的字符串他们的长度一定是连续的。可以感性理解一下，aba和ba，aba出现的位置ba一定出现过，所以ba出现的位置数一定大于等于aba。

**后缀自动机的parent链是什么？**

如果B节点的right集合包含A节点right集合的最小集合，那么A节点有一条parent链连向B节点。可以把parent链当做fail链，但是他们之间有一些微妙的区别。
构造的意义：顺着parent链跳可以逐渐增大right集合以便找到所有可以转移的状态。
匹配的意义：我在匹配时记录已经匹配的长度，和当前在哪一个节点，那么我就可以知道我匹配最长是哪一个串，跳parent链时通过增大right集合，减小匹配长度，从而找到合法转移。这个等一会儿详细讲。
parent链是一棵树，它组成了原串反串的一棵后缀树（这里不研究后缀树）。

###### 举个例子吧：abb

a:1
ab:2
b:2,3
bb:3
abb:3

我们把right集合相同的放在一起：
1,a
2,ab
3,b
4,abb,bb
把子集包含的连上parent链（红色代表parent链），这样我们就得到了一个后缀自动机：

![](D:\document\postgraduate\note\pic\后缀自动机2.PNG)

## AC自动机

同时搜索所有子串

AC自动机是一个树型结构

![](D:\document\postgraduate\note\pic\AC1.PNG)

fail指针，AC自动机的核心

![](D:\document\postgraduate\note\pic\AC2.PNG)

i->fail->j

word[j]是word[i]的最长后缀

[AC自动机]: https://www.bilibili.com/video/BV1uJ411Y7Eg?p=3&amp;spm_id_from=pageDriver





![image-20211004142324934](C:\Users\26583\AppData\Roaming\Typora\typora-user-images\image-20211004142324934.png)

```c
typedef struct _ACNode {
    _AC *child[26];
    _AC *fail;
    vector<int>exit//如果是单词，存储它的长度
}AC_NODE
```

#### 先构建一个字典树

![](D:\document\postgraduate\note\pic\AC3.PNG)

#### 使用bfs构建AC自动机

* 只有一个字母fail指针指向root
* 访问到3号节点，观察3号节点的父亲指针的fail指针



使用AC自动机进行字符串匹配

![](D:\document\postgraduate\note\pic\AC34.PNG)

