---
title: KMP算法
categories: Algorithm
date: 2022-04-14
---

[KMP Algorithm for Pattern Searching](https://www.geeksforgeeks.org/kmp-algorithm-for-pattern-searching/)

屡看屡忘，屡忘屡看...

<!--more-->

Pattern 模式串 p[0,1,..m-1] 

Text 主串 t[0,1,...n-1]

n>=m

We would like to check all the occurrences of the patten string in text string.

**Example**
```
Input:  t[] = "THIS IS A TEST TEXT"
        p[] = "TEST"

Output: 10
```

Using brute force method, for i = 0,1...n-m, we check if the text t[i,i+1...i+m-1] is the same as the pattern p[0,1,..m-1] one character by one character.

The time will be O(m(n-m+1))

## KMP

Let's see an example: 
```
t = "AAAABAAAACB"
p = "AAAAC"
```
We want to know how many occurrences of the p are in t.

We begin from `i = 0, j = 0`, to check if `t[i] == p[j]`, if j reaches p.size(), we find one occurrence. 

When `i=4, j=4`, we have `t[i]!=p[j]`. In naive algorithm, we will rebegin from `i=1, j=0`, however, we can observe that if we apply this change, it will still fail at `i=4, j=3` and we check 3 times additionally before we get the mismatch. 

Is there anything that helps us to know, the first three letters of p is already matched with `t[1...3]` that we don't need to check them? That is, when we come to `p[4]` and there is a mismatch, we know we don't need to restart from `p[0]` but instead, `p[3]`?

Yes, if we know `p[0...2]` == `p[1...3]`, and if there is a mismatch at `p[4]` and `t[i]`, we still know at least `p[1...3]`, and therefore `p[0...2]` matches with `t[i-3...i-1]`. Then we just need to check from `t[i]` and `p[3]`. All we need is a table that records the length of the longest prefix that matches with the suffix up to `p[j]`. We call it `next`.

We could see from the example that, if `next[i]==j`, that means `p[0...j-1] = p[i-j...i-1]`
```
p = "AAAAC"
next = [0,1,2,3,0]
```
When we check for `t[i]` and `p[j]` and there is a mismatch, from the `next` table we know `t[i-next[j-1]...i-1]` and `p[0...next[j-1]-1]` match, and now what we need to do is to check `t[i] == p[next[j-1]]`

When t[4]!=p[4]
```
t = "AAAABAAAACB"
p = "AAAAC"
```
Set `j` to 3 (`next[4-1]`) and check from `t[4]` and `p[3]`
```
t = "AAAABAAAACB"
p = " AAAAC"
```
Still mismatch. Set `j` to 2 and check fomr `t[4]` and `p[2]`
```
t = "AAAABAAAACB"
p = "  AAAAC"
```
Still mismatch. Set `j` to 1 and check fomr `t[4]` and `p[1]`
```
t = "AAAABAAAACB"
p = "   AAAAC"
```
Still mismatch. Set `j` to 0 and check fomr `t[4]` and `p[0]`
```
t = "AAAABAAAACB"
p = "    AAAAC"
```
Now `j==0` and mismatch, do `i++`
```
t = "AAAABAAAACB"
p = "     AAAAC"
```
Find an occurrence and come to `i=10, j=5`. Since `j==p.size()`, set `j` to `next[j-1]`
```
t = "AAAABAAAACB"
p = "          AAAAC"
```
Mismatch at `j==0`, do `i++`, and we come to and end.

### Code

Given the `next` table, `t` and `p`

```C++
int i = 0, j = 0;
while(i<t.size()){
    if (t[i] == p[j]){
        i++;
        j++;
    }
    if (j == p.size()){
        occ.push_back(i-j);
        j = next[j-1];
    }
    else if (i<t.size() && t[i]!=p[j]){
        if (j==0)
            i++;
        else
            j = next[j-1];
    }
}
```

## How to get the `next` table?

We get it by induction. First we know `next[0] = 0`. Now if we have known `next[1...i]`, and we want to know `next[i+1]`

Assume `next[i]=k`. The simple case is, if `p[k] == p[i+1]`, then we know `p[0...k] = p[0...i+1]`, so `next[i+1] = next[i]+1`.

What if `p[k] != p[i+1]`? As the example shows, here `?` cannot be 6.

```
p = "ABCABDABCABC"
next = [0,0,0,1,2,0,1,2,3,4,5,?]
```

Well, now we know `p[0...5] != p[6...11]`, though `p[0...4] == p[6...10]`. Is there any shorter but long enough prefix such that `p[0...l] = p[10-l...10]` so that we could chek if `p[l+1] == p[11]`? 

Yes, and it is `p[0...next[4]-1]` because from the definition of `next` we know `p[0...next[4]-1] == p[4-(next[4]-1)...4] == p[10-(next[4]-1)...10]`, so `p[0...next[4]-1]` is a prefix. Why is it long enough? Because we want to find a prefix that match with the suffix of `p[...10]`, `p[0...4]` is the longest prefix but `p[5] != p[11]` so it cannot be used. What is the next longest prefix? It must have the same prefix and suffix of `p[0...4]`, so we find it by check `next` at `p[4]`. `next[4]` is the length of the prefix so whe prefix is `p[0...next[4]-1]`. Then we check if `p[next[4]] == p[11]`, fortunately it matches, so `next[11] = next[4]+1`. If it does not match, we need to continue looking for shorter prefix, since we have come to `p[next[4]-1]`, the length of the next prefix is `next[next[4]-1]`, so every time if next length `l` is not zero and there is a mismatch, we check for prefix of length `next[l-1]`

```
next[4] = 2
p = "ABCABDABCABC"
next = [0,0,0,1,2,0,1,2,3,4,5,3]
```

### Code

```C++
int i = 1, j = 0;
int next[p.size()];
next[0] = 0;
while(i<p.size()){
    if (p[i] == p[j]){
        next[i] = j+1;
        i++;
        j++;
    }
    else{
        if (j!=0){
            j = next[j-1];
        }
        else{
            next[i] = 0;
            i++;
        }
    }
}
```
