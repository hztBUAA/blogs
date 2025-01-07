
一直觉得这个很神秘 只知道跟 前后缀相关
今天`最近修改于 2025-01-04 16:21:10` 也算是 通过看过[youtube的算法爷的手把手带过一遍算法](https://www.youtube.com/watch?v=V5-7GzOfADQ)+ [leetcode 看到的机器详细的解法](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/solutions/2600821/kan-bu-dong-ni-da-wo-kmp-suan-fa-chao-qi-z1y0/?envType=study-plan-v2&envId=top-interview-150)   理解了其中的思想

本质上 求next数组 存在朴素和进阶两种解法 复杂度：前者n^2  后者 n



# 朴素匹配
![[Pasted image 20250104163328.png | 450]]


# 求next数组
示意图
![[Pasted image 20250104162401.png|450]]

> next数组在KMP中的用法 [[#^41a733 | 查看]]

## 朴素代码
```cpp
// 求解字符串p的 next数组
void get_next(vector<int>& next, string p)
{
    int n = p.size();
    next.assign(n + 1, 0);	
    for (int j = 2; j <= n; j++) {     // 遍历p的所有子串
        string tmp = p.substr(0, j);      // p的子串
        // 因为求的是最大公共前后缀，因此让前后缀长度从大到小遍历，
        // 这样找到的第一个相等的前后缀的长度len，就是next[j]的值
        for (int len = j - 1; len >= 1; len--) {
            if (tmp.substr(0, len) == tmp.substr(j - len, len))  // 比较p的子串的前后缀是否相等
                next[j] = len;
                break;
            }	
    }
}
```

## 进阶代码
```cpp
void get_next(vector<int>& next, string p)
{
    int n = p.size();
    next.assign(n + 1, 0);  // next数组初始化为0
    for (int j = 2, i = 0; j <= n; j++) {      //从j = 2开始计算
        while (i > 0 && p[i] != p[j - 1]) i = next[i];
        if (p[i] == p[j - 1]) i++;				 
        next[j] = i;			
    }
}

```

核心思想 类似于动态规划
正好是一个子问题
但是现在又遇到一些问题   就是这里的i？
由于i是累积的  这里可以记录类似于`next[j-1]` 的值 
注意next的`j-1` 也就是previous的值   一步步 其实都是i



# KMP匹配 使用next

>源串的指针i是不回退的 [[#源串i不回退|具体原因]]



# 完整next+KMP匹配
```cpp
class Solution {
public:
    int strStr(string s, string p) {
        vector<int> next(p.size() + 1, 0);  // next 数组 
        vector<int> res;		            // res数组:记录p在s中出现的位置下标
        for (int j = 2, i = 0; j <= p.size(); j++) {   // 求next数组过程
            while (i > 0 && p[j - 1] != p[i]) i = next[i];
            if (p[j - 1] == p[i]) i++;
            next[j] = i;
        }
        for (int i = 0, j = 0; i < s.size(); i++) {   // p和s匹配的过程
            while (j > 0 && s[i] != p[j]) j = next[j];
            if (s[i] == p[j]) j++;
            if (j == p.size()) {  // 如果找到了一个匹配的位置
                res.push_back(i - j + 1);   // 记录结果
                j = next[j];
            }
        }
        if (res.size() > 0)
            return res[0];  // 题目要求返回第一个出现的位置
        return -1;
    }
};


```




# 一些解答


## 源串i不回退
匹配到某个失败时：
![[Pasted image 20250104165042.png|300]]

也正是在这里 用到了 蓝色区域的最长前后缀 在模式串P中进行计算即可  而不是源

![[Pasted image 20250104165238.png | 400]]
上面文字解释next数组在`p`串的作用 以及`p[0]`的终止的作用 ^41a733
 > 用到最长  因为不够长  相当于`j`就迁移更多了 ` j`的前移距离是 `j（current） - next[j]`

## next数组的本质  
`最近修改于 2025-01-07 17:16:17`
> 注意 `j`是模式串的模式指针  表示考虑`模式串p[0..j-1]` => `next[j]` 表示匹配`模式串p[0..j-1]`也就是`p[j-1]`时不匹配时 `j`应该跳到模式串的哪个位置  使得`i`不需要后移

![[Pasted image 20250107171354.png]]


## 需要理解两层交互
>其实也是next数组的理解


代码虽然写的相似  但是思路不太一样
1. next数组的构建 [[#模式串与next数组的交互 (更深层理解)]]
2. KMP的匹配 [[#KMP匹配的源串跟模式串交互]]
## 模式串与next数组的交互  (更深层理解)

>注意next存储的值 是长度！  不是下标

> 所以 `最近修改于 2025-01-07 17:27:43` 注意到 `i = next[i]`其实 现在的i作为下标索引到的模式串`p[i]`  是需要跟 `p[j-1]`进行比较的 （此时`next`创建进行到`j` 我们比较的是`p[0..j-1]`）  所以可以理解成开区间

![[Pasted image 20250107172102.png]]

## KMP匹配的源串跟模式串交互

[[#源串i不回退]] 参考这个的理解  是一样的解答

> 这时候使用开区间`next[j]` 表示j`[p[0..j-1]`的最大前后缀长度就体现出来了 

## 关于next的长度的形式
![[Pasted image 20250104165640.png]]
因为`next[0]`本身没有作用