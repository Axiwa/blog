---
title: 刷题
categories: Algorithm
date: 2022-04-14
katex: true
markup: pandoc
---

正在从 <https://github.com/youngyangyang04/leetcode-master> 开始刷题！

<!--more-->

## 题

### 数组
长度为n的循环数组，从下标i开始倒着跳k格，新坐标咋算？

如果k可以被n整除，等于向前跳了`(k/n)*n-k`格，新坐标

如果k不能被n整除，等于向前跳了`ceil(k/n)*n-k`格，新坐标`(i+ceil(k/n)*n-k)%n`

新坐标：`((i-k)%n+n)%n`

往前跳k格也能用这个

### 二分法
Leetcode: 34, 35, 69, 367, 704
```C
// [left, right]
while (left <= right){
  int mid = left + (right-left)/2; 
  if (nums[mid] < target) left = mid+1;
  else if (nums[mid] > target) right = mid-1;
  else return mid;
}
// [left, right)
while (left < right){
  int mid = left + (right-left)/2; 
  if (nums[mid] < target) left = mid+1;
  else if (nums[mid] > target) right = mid-1;
  else return mid;
}

```
34题，第一个和最后一个位置，分别找第一个nums[left]>=target和nums[right]>target的索引，每一次满足要求，记录下该位置．两个值分别求
```C
int left = nums.size(); int right = nums.size();  // not nums.size()-1!!!
while (from<=to){
  int mid = left+(right-left)/2;
  if (nums[mid]>=target)
    left = mid;
    to = mid-1;
  else 
    from = mid+1;
}
...
```
69/367
找平方根．牛顿法
```C
double t = 1.0
t = (t+x/x)/2
```
二分法，记得用long
```C
if (num<2) 
  ...
else 
  long left = 2, right = num/2;
  while (left<=right){
  int mid = left + (right-left)/2; 
  if (mid*mid < num) left = mid+1;
  else if (mid*mid > num) right = mid-1;
  else ...;
  }
```

### 双指针
Leetcode: 

27.[移除元素](https://leetcode-cn.com/problems/remove-element/)

左边的指针从0开始，右边的指针从nums.size()-1开始．
```c
while (left <= right){
  if (nums[left] == target){
    if (nums[right] != target){
      nums[left] = nums[right];
    }
    else 
      right--;
  }
  else
    left++;
}
```

26.[删除重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array)

区别空数组和非空; 非空的`right`从１开始，`left`从０开始
```c
while (right < nums.size()){
  if (nums[right] != nums[left]){
    left = left+1;
    nums[left] = nums[right];
  }
  right++;
}
```

80.[删除重复项II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii/)

若可以保留k个重复项，区别数组长度<=k和>k; `left`和`right`下标都从k开始. 如果`nums[left-k]!=nums[right]`，说明`nums[left]`应该被`nums[right]`替换，否则增加`right`

```c
int left = k; int right = k;
while (right < nums.size()){
  if (nums[right] != nums[left-k]){
    nums[left] = nums[right];
    left++;
  }
  right++;
}
```

844.[Backspace String Compare](https://leetcode.com/problems/backspace-string-compare/)

<https://leetcode.com/problems/container-with-most-water>

3Sum/3Sum2/4Sum...先排序，最后一层循环用双指针，只能降一个数量级
[4SumII](https://leetcode-cn.com/problems/4sum-ii/)　用哈希表！少２个数量级！

142.[环形链表II](https://github.com/youngyangyang04/leetcode-master/blob/master/problems/0142.%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8II.md)

### 滑动窗口
209.[Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/)

76.[最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

用map记录需要满足的字母和相应个数，用`satisfied`记录满足的字母总数，      `satisfied`满足时左指针右移至不再满足，记录满足的最小长度，`satisfied`不满足时右指针右移直到满足．一旦更改记录的`table`，检查相应元素的个数是否刚好满足/刚好不满足(`table[nums[left]] == 1 or table[nums[right]] == 0`)来改变对应的`satisfied`

### 旋转矩阵

### 快乐数
快乐数不是数学问题，是找循环的问题．`unordered_set` / `快慢指针` 

### 丑数和超级丑数 
最小堆/动态规划

### 回溯
对于二叉数，只能用前序遍历/后序遍历！

#### 组合＆去重
```c
for(int i = l; i< r; i++){
  ...
}
//下标从传入的l开始
//找幂集: 选，或者不选，不需要for
```
去重的方法
* 用`for`横向检查的时候，用unordered_set检查某一重复元素在本层是否被使用过，如果有,`continue`
* **排序**之后，每一次用`for`横向检查的时候`if (i!=l && nums[i] == nums[i-1]) continue;`
* [递增子序列](https://leetcode-cn.com/problems/increasing-subsequences/)不能排序，如果不用hash，需要保证有同样的元素时，只有一种顺序的可能．比如`[1,1,1]`，只可能有`[0,0,0][0,0,1],[0,1,1],[1,1,1]`，则
```c
if (nums[l] >= mm){ // if nums[l] >= m 符合要求，可以选
  path.push_back(nums[l]);
  helper(nums, l+1, nums[l])
  path.pop_back();
}
if (nums[l] != mm){　// if nums[l] == m，说明之前选过和nums[l]相等的元素，则nums[l]不能再不选．否则，可以不选
  helper(nums, l+1, mm);
}
```

#### 排列＆去重

排列是选了`nums[i]`以后在剩下所有的`size-1`里面选．

1. 对[1,2,3,4,5]，`l=0`时选1，从`l=1`开始挑([2,3,4,5])，选2，交换1和2，仍然从`l=2`开始挑[1,3,4,5]．．．
```c
for(int i = l; i<r; i++){
  path.puch_back(nums[i]);
  swap(nums[l], nums[i]);
  helper(nums, l+1, r);
  swap(nums[l], nums[i]);
  path.pop_back();
}

2. 用`used`数组表示某个元素在这一分支里是否已经选过，每次从0开始挑，遇到`used[i] == true`跳过

第一种方法无法去重，因为我们需要有序的集合．
```c
for(int i = 0; i<r; i++){
  if (i!=0 && nums[i]==nums[i-1] && used[i-1]==false) // used[i-1]==false说明这一层已经用过第一个出现的该元素了，跳过
    continue;
  if (!used[i]){
    used[i] = true;
    path.push_back(nums[i]);
    helper(nums, l+1, r, used);
    path.pop_back();
    used[i] =false;
  }
}

用set也可以，但是要先将数组排序
```

### 二叉搜索树

Morris遍历

### 图
#### DFS&BFS

### 位运算
按位或： |

按位与： &

判断某数的某一位i为1/0：`(a>>i)&1`. NOTE: 位运算的优先级最低（先算术运算，后移位运算，最后位运算），记得加括号！！

#### 状态压缩
用一个n位的k进制数，代表n个元素的k个状态
[好子集的个数](https://leetcode-cn.com/problems/the-number-of-good-subsets/solution/hao-zi-ji-de-shu-mu-by-leetcode-solution-ky65/) 状态压缩+动态规划

## C++
* __gcd ()函数是内置于algorithm头文件中的函数，主要是用于求两个数的最大公约数
* 创建二维数组
```C
#include<vector>
vector<vector<int>> matrix(n, vector<int>(m, 0));
// matrix.size() = n
```

* map
```C
#include<map>
map<char, int> m;
m['a'] = 1;
m['a']++;
//可以直接用
//不希望存在的key，不要用m[key]，用m.find(key) == m.end()．否则会被创建出来！
```

* 定义链表
```c
struct ListNode{
  int val;
  ListNode *next;
  ListNode() : val(0), next(nullptr) {}
  ListNode(int x): val(x), next(nullptr) {}
  ListNode(int x, ListNode *next): val(x), next(next) {}
};

```

* 定义二叉树
```c
struct TreeNode{
    int val;
    TreeNode * left;
    TreeNode * right;
    TreeNode(int x): val(x), left(nullptr), right(nullptr) {};
}

```

* Hash & map
```c
std::set　//AVL实现
std::multiset
//红黑树，有序.查询和增减效率O(logn)
std::unordered_set //哈希表．查询和增减效率O(1)
std::map　//AVL实现
std::multimap　//key可重复
std::unordered_map　//哈希表实现．key无序　O(1)
```

* 排序
```c
#include<algorithm>
string a = "12345";
sort(a.begin(), a.end());
string a[n];
sort(a,a+n);
```

* INT_MAX / INT_MIN
```C
#include<limits.h>
// INT_MAX =  2^31-1
// INT_MIN = -2^31
```
* Iterator是个啥
```c
//正向
map<string, int> mm;
map<string, int>::iterator iter;
for(iter = mm.begin(); iter!= mm.end(); iter++){
    cout<<iter->first<<" "<<iter->second<<endl;
}
//反向
map<string, int> mm;
map<string, int>::reverse_iterator iter;
for(iter = mm.rbegin(); iter != mm.rend(); iter++){
	cout << iter->first << ", " << iter->second << endl;
}
```

* 不知道的函数．．．
```c
unique
reverse //reverse(rev.begin(), rev.end());
sort
resize // s.resize(newSize);
replace //
emplace
substr // s.substr(starting pos: int, length): string
//原来swap函数是自带的！
```

* 位操作
```c
void Swap(int &a, int &b){
    if (a != b){
        a ^= b;
        b ^= a;
        a ^= b;
    }
}



```

* 引用和指针

* 容器和容器适配器，底层的实现，性能

容器适配器的底层使用不同的容器，使得容器适配器内的数据在内存中不连续．默认的底层容器是deque.

## N-Queens
*For a chessboard of any size, there can't be two queens in the same row, column, or diagonal.*

We can solve this problem with a recursive algorithm. Suppose each queen is placed in each row, i.e. n queens takes row 1 to row n, and we should determine their columns.

```scala
def queens(n: Int)= 
    def placeQueens(k: Int): Set[List[Int]] = // with k queens, the set of solutions
        if k==0 then Set(List())
        else 
            for 
                queens <- placeQueens(k-1) // with k-1 queens, the set of solutions
                col <- 0 until n // the column of the k th queen
                if isSafe(col, queens) // safe means a solution
            yield col::queens
    placeQueens(n)

    def isSafe(col: Int, queens: List[Int]): Boolean = 
        !checks(col, 1, queens)
    def checks(col: Int, delta: Int, queens: List[Int]): Boolean = queens match
        case Nil => false // no other queens, no check
        case qcol::others =>
            qcol == col                     // same column
            || (qcol-col).abs == delta      // diagonal check
            || checks(col, delta+1, others) // check next queen
```

* [fprintf vs. fwrite](https://blog.csdn.net/godenlove007/article/details/7721647)

## KMP
[从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827#t10)


## Water pouring puzzle



