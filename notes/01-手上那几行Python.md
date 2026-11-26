# 手上那几行 Python

只记每道题还没想清算法时就要写出来的那几行。排序之后怎么扫、窗口怎么缩、表怎么填，都在后面的卡片里，这里不重复。

本题库的解法是 `solve_method(...)` 加断言。考场把样例喂进标准输入。先把函数写对，再包一层读入。

## 读入

先看样例的分隔符，再选一种。不要三种混用。

```python
import sys
input = sys.stdin.readline

# 一行空格
nums = list(map(int, input().split()))

# 一行逗号。样例常带方括号：[1, 2, 3]
line = input().strip().strip("[]")
nums = [int(x) for x in line.split(",") if x.strip() != ""]

# 第一行 n，后面 n 行
n = int(input())
matrix = [list(map(int, input().split())) for _ in range(n)]

# 第一行 n m
n, m = map(int, input().split())

# 不规则文本先整行拿，再自己拆
s = input().strip()
```

方括号、引号、末尾空格先 `strip`。`split()` 不带参数会吃掉连续空白；`split(",")` 不会。数字和单位粘在一起时（`10M`、`3.2G`）用正则，不要 `int()` 硬转。

```python
import re
num, unit = re.fullmatch(r"(\d+)([a-zA-Z]+)", token).groups()
```

多组输入直到空行：

```python
for line in sys.stdin:
    line = line.strip()
    if not line:
        break
```

输出跟题面走，不要自作主张。常见三种：

```python
print(ans)                         # 一个数
print(" ".join(map(str, ans)))     # 空格
print(",".join(map(str, ans)))     # 逗号
```

题面写 `-1`、`No`、`NULL`、`empty`、`ERROR!` 时，原样输出这几个字符串，大小写照抄。

## 计数、分组、去重

频数、下标、分组是这套题里出现次数最多的三行，先写它们，很多 100 分题到这里已经结束。

```python
from collections import Counter, defaultdict

cnt = Counter(nums)          # 每个值出现几次
cnt.most_common(1)           # [(值, 次数)]，次数最高的一个
cnt.most_common()            # 全部，已按次数降序

pos = defaultdict(list)
for i, x in enumerate(nums):
    pos[x].append(i)         # 每个值出现在哪些下标

groups = defaultdict(list)
for row in records:
    groups[row[0]].append(row)
```

交集、并集用集合，比手写双循环短，也是本题库的实际写法（找字符、匿名信、VLAN 删掉已申请的号）：

```python
both = set(a) & set(b)
only = set(a) - set(b)
```

两个数组的配对次数（统计匹配的二元组）是频数相乘，不是两层循环：

```python
ca, cb = Counter(a), Counter(b)
ans = sum(ca[x] * cb[x] for x in ca if x in cb)
```

`defaultdict(int)` 用来做差分和「时刻 → 人数」。普通字典要先判断键在不在，考场上容易漏。

## 排序键

比较规则本身就是题的时候，`key` 写完这题就结束了。后面 03 还有区间和配对；这里先把语法固定成一种，避免考场上混用 `cmp`。

```python
# 次数降序，次数相同则字典序升序
ordered = sorted(cnt.items(), key=lambda kv: (-kv[1], kv[0]))

# 多个字段：身高升、体重升、编号升
people.sort(key=lambda p: (p[1], p[2], p[0]))

# 离目标最近，距离相同则价格更小
hotels.sort(key=lambda price: (abs(price - target), price))

# 版本号：主、次、增量按数字，里程碑按字符串
parts.sort(key=lambda v: (v.major, v.minor, v.patch, v.milestone))

# 字符串比大小组成最小数：a+b 和 b+a 比
from functools import cmp_to_key
arr.sort(key=cmp_to_key(lambda a, b: 1 if a + b > b + a else -1 if a + b < b + a else 0))
```

负数放进 `key` 表示降序，比 `reverse=True` 再处理并列更不容易写反。并列规则一定写进同一个元组，不要排完再稳定排序第二次，除非两种键的方向你非常有把握。

`heapq` 只在「随时取当前最小」时用。打印文件、任务调度、前 K 小的组合是这种。Python 的堆是小根堆，优先级高的用负数放进去：

```python
import heapq
heap = []
heapq.heappush(heap, (-priority, order, task_id))
priority, order, task_id = heapq.heappop(heap)
priority = -priority
```

元组比较从左到右。同一优先级要先来先服务时，把到达顺序放在第二位。

## 字符串上天天用的几行

```python
s.lower()
s.isalpha()
s.isdigit()
ord("a")            # 97
chr(ord("a") + 1)   # b
(ord(ch) - 97 + shift) % 26 + 97

words = re.findall(r"[A-Za-z]+", text)   # 英文输入法、评论里抠单词
```

回文、逆序不要先写循环：

```python
word[::-1]
"".join(reversed(words))
```

连续相同字符拉成一段，后面最长连续、第 k 长子串都从这里开始：

```python
runs = []
i = 0
while i < len(s):
    j = i
    while j < len(s) and s[j] == s[i]:
        j += 1
    runs.append((s[i], j - i))
    i = j
```

## 格子和下标

四方向背下来。五子棋、矿堆、消消乐要八方向时，把四个对角加上。

```python
DIRS = [(-1, 0), (1, 0), (0, -1), (0, 1)]

def inside(x, y, n, m):
    return 0 <= x < n and 0 <= y < m
```

完全二叉树若输入从下标 1 开始：左 `2*i`，右 `2*i+1`，父 `i//2`。从下标 0 开始：左 `2*i+1`，右 `2*i+2`，父 `(i-1)//2`。本题库两种都有，先看样例第 0 个是不是占位的 0。

## 整数习惯

Python 整数不会溢出，大数直接乘。考场上不要为了「防溢出」去写取模，除非题面写了模数。

```python
a // b          # 整除，向负无穷取整。正数就是向下
(a + b - 1) // b   # 正数向上取整，施肥天数用这个
abs(a)
```

负数取模和 C++ 不同。数字加减游戏那题先取绝对值再枚举，避开负数模。

浮点只在题面出现平均值、几何平均时使用。比较用差的绝对值，本题库几何平均用的是 `1e-10`：

```python
if abs(x - y) <= 1e-10:
    ...
```

## 递归深度

网格 DFS、目录大小、依赖链会超过默认 1000 层。文件开头加一行：

```python
sys.setrecursionlimit(1_000_000)
```

步数类不要靠递归硬走，改队列。队列写法在 05。

## 写完对一下样例

```python
if __name__ == "__main__":
    assert solve_method(...) == ...
```

本题库每个文件末尾都是这样。自己做题时留下断言，改边界时不会把样例改坏。断言过了再接标准输入。
