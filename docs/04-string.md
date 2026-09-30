# 第 4 章 串
## 4.1 串的定义和基本操作

串就是字符串。

串是由零个或多个字符组成的有限序列。

例如：

```text
S = "Hello World"
T = "llo"
```

### 基本概念

- 串名：字符串变量的名字。
- 串长：串中字符的个数。
- 空串：长度为 `0` 的串。
- 子串：主串中任意连续字符组成的子序列。
- 主串：包含子串的串。
- 字符在主串中的位置：字符的序号。
- 子串在主串中的位置：子串第一个字符在主串中的位置。

串中的位置从 `1` 开始，不是从 `0` 开始。

特别强调：

> 空串和空格串不同。
>
> `""` 是空串，长度为 `0`。
>
> `"   "` 是空格串，每个空格都是一个字符，长度是空格的个数。

### 串和线性表的联系

串也是一种特殊的线性表。

- 每个字符之间是线性关系。
- 串的数据元素被限制为字符。
- 对线性表常按单个元素操作。
- 对串常按子串进行操作。

例如，搜索引擎搜索时，通常输入一段有意义的字符串，而不是只输入一个字符。

### 串的基本操作

课程中介绍的主要操作包括：

```text
StrAssign(&T, chars)       赋值
StrCopy(&T, S)             复制
StrEmpty(S)                判空
StrLength(S)               求串长
ClearString(&S)            清空
DestroyString(&S)          销毁
Concat(&T, S1, S2)         连接
SubString(&Sub, S, pos, len)  求子串
Index(S, T)                定位
StrCompare(S, T)           比较
```

### 清空和销毁的区别

- 清空：逻辑上把串变为空串，但原来分配的内存空间还可以继续使用。
- 销毁：释放串所占用的存储空间，空间归还给系统。

### 串的比较规则

从第一个字符开始逐个比较：

1. 如果某个位置字符不同，则字符较大的串更大。
2. 如果前面的字符全部相同，则较长的串更大。
3. 字符全部相同且长度相同，则两个串相等。

返回规则：

```text
S > T   返回正数
S == T  返回 0
S < T   返回负数
```

计算机比较字符时，本质上比较字符的编码值。

常见的英文编码是 ASCII。

例如：

- `a` 的编码值小于 `c`。
- 空格也是字符，在 ASCII 中有明确编码，并占用一个字节。

## 4.2 串的存储结构

### 顺序存储

用一组连续空间保存字符。

常见方式：

```cpp
typedef struct {
    char ch[MAXLEN];
    int length;
} SString;
```

**Python 示例：**

```python
from dataclasses import dataclass, field

MAX_LEN = 100

@dataclass
class SString:
    ch: list = field(default_factory=lambda: [""] * MAX_LEN)
    length: int = 0
```

特点：

- 可以随机访问某个字符。
- 修改长度或插入删除通常不方便。
- 串长度受数组容量限制。

### 堆分配存储

在堆区动态申请串空间。

```cpp
typedef struct {
    char *ch;
    int length;
} HString;
```

**Python 示例：**

```python
from dataclasses import dataclass, field

@dataclass
class HString:
    ch: list = field(default_factory=list)
    length: int = 0
```

特点：

- 可以根据实际需要申请空间。
- 初始化、连接、求子串时可能需要动态分配和释放内存。
- 必须注意内存管理。

### 块链存储

用链表保存串，每个结点可以存一个或多个字符。

```cpp
typedef struct StringNode {
    char ch[CHUNKSIZE];
    struct StringNode *next;
} StringNode, *String;
```

**Python 示例：**

```python
from dataclasses import dataclass

CHUNK_SIZE = 4

@dataclass
class StringNode:
    ch: list = None
    next: "StringNode | None" = None
```

特点：

- 插入和删除方便。
- 存储密度受每个结点存放字符数量的影响。
- 随机访问不方便。

## 4.3 朴素模式匹配算法

模式匹配：

> 在主串中查找模式串第一次出现的位置。

朴素算法：

1. 从主串第一个位置开始。
2. 依次与模式串比较。
3. 如果发现某个字符不匹配，主串指针回到本次匹配开始位置的下一个位置。
4. 模式串重新从第一个字符开始。
5. 重复，直到找到或主串扫描完成。

指出：

> 朴素算法的主要问题是失配后主串指针会回退，之前已经比较过的信息被浪费。

最坏时间复杂度：

```text
O(nm)
```

其中：

- `n` 是主串长度。
- `m` 是模式串长度。

## 4.4 KMP 算法

KMP 的核心思想：

> 当某个字符失配时，主串指针不回溯，只移动模式串。

KMP 利用模式串自身的重复前后缀信息，决定模式串应该回退到哪里。

### 最长相等前后缀

前缀：

> 除最后一个字符外，字符串的所有头部子串。

后缀：

> 除第一个字符外，字符串的所有尾部子串。

例如，模式串：

```text
abab
```

最长相等前后缀为：

```text
ab
```

长度为 `2`。

KMP 利用这个信息，避免重复比较已知相同的前后缀。

### next 数组

不同教材的 `next` 定义和下标约定可能不同。

课程采用常见约定：

- 串下标从 `1` 开始。
- `next[1] = 0`。
- `next[2] = 1`。

`next[j]` 表示模式串第 `j` 个位置失配后，模式串应该回退到的位置。

求 `next` 的核心思想：

> 利用模式串自己与自己进行匹配。

常见代码逻辑：

```cpp
void GetNext(SString T, int next[]) {
    int i = 1;
    int j = 0;
    next[1] = 0;

    while (i < T.length) {
        if (j == 0 || T.ch[i] == T.ch[j]) {
            i++;
            j++;
            next[i] = j;
        } else {
            j = next[j];
        }
    }
}
```

**Python 示例：**

```python
def get_next(pattern):
    # pattern 使用 1 基下标，索引 0 作为占位符
    next_array = [0] * (len(pattern) + 1)
    i = 1
    j = 0
    next_array[1] = 0

    while i < len(pattern):
        if j == 0 or pattern[i] == pattern[j]:
            i += 1
            j += 1
            next_array[i] = j
        else:
            j = next_array[j]

    return next_array
```

注意：

> 必须先确认下标从 `0` 还是从 `1` 开始，不能混用公式。

### KMP 匹配过程

设主串指针为 `i`，模式串指针为 `j`。

1. 如果 `j == 0` 或当前字符相等，则 `i++`、`j++`。
2. 如果当前字符不相等，则 `j = next[j]`。
3. 如果 `j` 超过模式串长度，匹配成功。
4. 如果 `i` 超过主串长度，匹配失败。

主串指针 `i` 不需要回退。

时间复杂度：

```text
O(n + m)
```

其中：

- 求 `next` 数组：`O(m)`。
- 扫描主串：`O(n)`。

空间复杂度：

```text
O(m)
```

### next 数组计算示例

模式串：

```text
ababaa
```

常见结果：

```text
j:        1  2  3  4  5  6
模式串:   a  b  a  b  a  a
next:     0  1  1  2  3  4
```

结果随下标约定会变化，使用时必须与教材定义一致。

## 4.5 KMP 算法的进一步优化

如果失配后回退到的字符仍然和当前失配字符相同，继续回退没有意义。

因此可以求 `nextval`：

```cpp
if (T.ch[j] == T.ch[next[j]]) {
    nextval[j] = nextval[next[j]];
} else {
    nextval[j] = next[j];
}
```

**Python 示例：**

```python
if pattern[j] == pattern[next_array[j]]:
    nextval[j] = nextval[next_array[j]]
else:
    nextval[j] = next_array[j]
```

`nextval` 可以减少无效比较。

## 本章结论

- 串是由零个或多个字符组成的有限序列。
- 串中位置从 `1` 开始。
- 空串长度为 `0`，空格串长度不为 `0`。
- 串可以采用顺序存储、堆分配存储或块链存储。
- 朴素模式匹配最坏时间复杂度为 `O(nm)`。
- KMP 的核心是主串指针不回溯。
- `next` 数组利用模式串的最长相等前后缀。
- KMP 时间复杂度为 `O(n + m)`。
- 不同 `next` 定义不能混用。
- `nextval` 可以跳过没有意义的回退。
