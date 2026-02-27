# Leetcode练习

# 6. Z 字形变换

## 题目描述

将一个给定字符串 `s` 根据给定的行数 `numRows`，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 `"PAYPALISHIRING"`、行数 `numRows = 3` 时，排列如下：

```plaintext
P   A   H   N
A P L S I I G
Y   I   R
```

之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如：`"PAHNAPLSIIGYIR"`。

请你实现这个将字符串进行指定行数变换的函数：`string convert(string s, int numRows)`。

## 示例

- 示例 1：

  - 输入：`s = "PAYPALISHIRING", numRows = 3`
  - 输出：`"PAHNAPLSIIGYIR"`

- 示例 2：

  - 输入：`s = "PAYPALISHIRING", numRows = 4`

  - 输出：`"PINALSIGYAHRPI"`

  - 解释：

    ```plaintext
    P     I    N
    A   L S  I G
    Y A   H R
    P     I
    ```

- 示例 3：

  - 输入：`s = "A", numRows = 1`
  - 输出：`"A"`

## 提示

- `1 <= s.length <= 1000`
- `s` 由英文字母（小写和大写）、',' 和 '.' 组成
- `1 <= numRows <= 1000`

## 解题

### 我的解题

以`s = "PAYPALISHIRING", numRows = 3`为例

```
P   A   H   N               
A P L S I I G               
Y   I   R                   
```

执行步骤如下

- 令`s`的索引为`index`，用矩阵存储Z字型排序
- 注意到Z字型以`2*numRows-2`为循环的规律存入矩阵
- 前每次循环前`numRows-1`个**下一步**是行坐标向下移动
- 后`numRows-1`个**下一步**是行坐标向上移动，列坐标向右移动

```cpp
class Solution
{
public:
    string convert(string s, int numRows)
    {
        if(numRows < 2 || s.size() < numRows) return s;
        int length = s.size();
        int round = 2 * numRows - 2;                                             // 一个循环的长度
        int col = (length + numRows - 1) / round * (numRows - 1), row = numRows; // s.size()+numRows-1 用于向上取整
        vector<vector<char>> matix(row, vector<char>(col, 0));

        int x = 0, y = 0, index = 0; // x代表行，y代表列

        while (index < length)
        {

            if (index % round < row - 1) // 前numRows-1个下一步x向下
            {
                matix[x++][y] = s[index++];
            }
            else // 后numRows - 2个x向上，y向右
                matix[x--][y++] = s[index++];
        }
        string answer = "";
        for (int i = 0; i < row; ++i)
            for (int j = 0; j < col; ++j)
                answer += (matix[i][j] == 0 ? 0 : matix[i][j]);
        return answer;
    }
};
```

### 官方题解

**思路：**

只记录行移动，遍历s时每次追加s的每个字符，以起到节省空间的作用

```cpp
class Solution
{
public:
    string convert(string s, int numRows)
    {
        if(numRows < 2 || s.size() < numRows) return s;
        int length = s.size();
        int round = 2 * numRows - 2;                                             // 一个循环的长度
        vector<vector<char>> matix(row);

        int x = 0, y = 0, index = 0; // x代表行，y代表列

        while (index < length)
        {

            if (index % round < row - 1) // 前numRows-1个下一步x向下
            {
                matix[x++].push_back(s[index++]);
            }
            else // 后numRows - 2个x向上，y向右
                matix[x--].push_back(s[index++]);
        }
        string answer = "";
        for (int i = 0; i < row; ++i)
            for (int j = 0; j < col; ++j)
                answer += (matix[i][j] == 0 ? 0 : matix[i][j]);
        return answer;
    }
};
```



# 11. 盛最多水的容器

## 题目描述

给定一个长度为 `n` 的整数数组 `height`。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])`。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明**：你不能倾斜容器。

## 示例

- **示例 1**：
  - 输入：`height = [1,8,6,2,5,4,8,3,7]`
  - 输出：`49`
  - 解释：图中垂直线代表输入数组 `[1,8,6,2,5,4,8,3,7]`。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
  - 图示提示：两条线分别是下标 `1`（高度 8）和下标 `8`（高度 7），容器宽度为 7，高度为 7，容积 7×7=49。
- **示例 2**：
  - 输入：`height = [1,1]`
  - 输出：`1`

## 提示

- `n == height.length`
- `2 <= n <= 10^5`
- `0 <= height[i] <= 10^4`

## 解题

- 容器的水量 = 「两条线的水平距离」× 「两条线中较矮的那条的高度」。
- 初始时，将双指针分别放在数组两端（`left=0`，`right=height.length-1`），此时水平距离最大。
- 要增大水量，只能尝试提升「较矮那条线的高度」—— 因此移动 **指向较矮线条的指针**（左指针矮则右移，右指针矮则左移）。
- 每移动一次指针，计算当前容器的水量，并更新最大水量值。
- 循环终止条件：左右指针相遇（`left >= right`）。

**重要**：我们left++和right--都是为了尝试取到更多的水，如果短的板不动的话，取到的水永远不会比上次多。

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int length = height.size();
        int i = 0, j = length - 1;
        int max_num = (j - i) * min(height[i], height[j]);
        while (j > i)
        {
            if (height[i] > height[j])
                j--;
            else
                i++;
            max_num = max((j - i) * min(height[i], height[j]), max_num);
        }
        return max_num;
    }
};
```



# 12. 整数转罗马数字

## 题目描述

罗马数字包含以下七种字符：`I`、`V`、`X`、`L`、`C`、`D`、`M`，对应数值如下：

- `I` = 1，`V` = 5，`X` = 10，`L` = 50
- `C` = 100，`D` = 500，`M` = 1000

罗马数字的表示规则：

1. 通常，字符按**从大到小**的顺序排列，数值为各字符对应数值之和（如 `III` = 1+1+1 = 3，`LVIII` = 50+5+3 = 58）。
2. 特殊情况：为避免重复书写，某些数值采用 “小数字在大数字左侧” 的减法规则表示（仅存在以下六种合法组合）：
   - `IV` = 4（5-1）、`IX` = 9（10-1）
   - `XL` = 40（50-10）、`XC` = 90（100-10）
   - `CD` = 400（500-100）、`CM` = 900（1000-100）

给定一个整数 `num`（1 ≤ num ≤ 3999），将其转换为对应的罗马数字字符串。

## 示例

- 示例 1：输入 `num = 3`，输出 `"III"`（1+1+1）。
- 示例 2：输入 `num = 4`，输出 `"IV"`（5-1）。
- 示例 3：输入 `num = 9`，输出 `"IX"`（10-1）。
- 示例 4：输入 `num = 58`，输出 `"LVIII"`（50+5+3）。
- 示例 5：输入 `num = 1994`，输出 `"MCMXCIV"`（1000 + 900 + 90 + 4）。

## 提示

- 输入整数 `num` 的范围是 `[1, 3999]`（罗马数字无法表示 0 或负数，且超过 3999 的数值需额外符号，本题不涉及）。

## **解题**：

### **我的解题**

```cpp
class Solution {
public:
    pair<int, string> get_as(int num) {
        pair<int, string> as;
        if (num - 1000 >= 0)
            as = {1000, "M"};
        else if (num - 900 >= 0)
            as = {900, "CM"};
        else if (num - 500 >= 0)
            as = {500, "D"};
        else if (num - 400 >= 0)
            as = {400, "CD"};
        else if (num - 100 >= 0)
            as = {100, "C"};
        else if (num - 90 >= 0)
            as = {90, "XC"};
        else if (num - 50 >= 0)
            as = {50, "L"};
        else if (num - 40 >= 0)
            as = {40, "XL"};
        else if (num - 10 >= 0)
            as = {10, "X"};
        else if (num - 9 >= 0)
            as = {9, "IX"};
        else if (num - 5 >= 0)
            as = {5, "V"};
        else if (num - 4 >= 0)
            as = {4, "IV"};
        else
            as = {1, "I"};
        return as;
    }
    string intToRoman(int num) {
        string roman = "";
        pair<int,string> as;
        while (num) {
           as = get_as(num);
           roman += as.second;
           num-= as.first;
        }
        return roman;
    }
};
```

### 官方题解

```cpp
class Solution
{
public:
    string intToRoman(int num)
    {
        map<int, string, greater<int>> roman_itos = {
            {1000, "M"},
            {900, "CM"},
            {500, "D"},
            {400, "CD"},
            {100, "C"},
            {90, "XC"},
            {50, "L"},
            {40, "XL"},
            {10, "X"},
            {9, "IX"},
            {5, "V"},
            {4, "IV"},
            {1, "I"}
            };
        string roman = "";
        auto it = roman_itos.begin();
        while (num)
        {
            if(num - it->first >=0) {
                roman += it->second;
                num -= it->first;
            }
            else
                it++;
        }
        return roman;
    }
};
```



# 13. 罗马数字转整数

## 题目描述

罗马数字包含以下七种字符：`I`、`V`、`X`、`L`、`C`、`D`、`M`，对应的数值如下：

- `I` = 1
- `V` = 5
- `X` = 10
- `L` = 50
- `C` = 100
- `D` = 500
- `M` = 1000

罗马数字的表示规则：

1. 通常情况下，字符按**从大到小**的顺序排列，数值为各字符对应数值之和。

2. 特殊情况：当小数值字符位于大数值字符

   左边

   时，数值为大数值减去小数值（仅存在以下六种组合）：

   - `IV` = 4（5 - 1）
   - `IX` = 9（10 - 1）
   - `XL` = 40（50 - 10）
   - `XC` = 90（100 - 10）
   - `CD` = 400（500 - 100）
   - `CM` = 900（1000 - 100）

给定一个罗马数字字符串 `s`，将其转换成对应的整数。

## 示例

### 示例 1

- 输入：s = "III"
- 输出：3
- 解释：III = 1 + 1 + 1 = 3。

### 示例 2

- 输入：s = "IV"
- 输出：4
- 解释：IV 是特殊组合，对应 5 - 1 = 4。

### 示例 3

- 输入：s = "IX"
- 输出：9
- 解释：IX 是特殊组合，对应 10 - 1 = 9。

### 示例 4

- 输入：s = "LVIII"
- 输出：58
- 解释：L = 50，V=5，III=3，总和为 50 + 5 + 3 = 58。

### 示例 5

- 输入：s = "MCMXCIV"
- 输出：1994
- 解释：M = 1000，CM = 900，XC = 90，IV = 4，总和为 1000 + 900 + 90 + 4 = 1994。

## 提示

- 1 <= s.length <= 15
- s 仅含字符 ('I', 'V', 'X', 'L', 'C', 'D', 'M')
- 题目保证 s 是一个有效的罗马数字，且输入范围在 [1, 3999] 内

## 解题

### 我的解题:

### 官方题解：

**思路：**

假如`s = "MDCXCV"`,则可得

- `M = 1000`；
- `D = 500`；
- `C = 100`；
- `XC =100 - 90`；
- `V = 5`

即`sum = M+D+C-X+C+V`,可得步骤

- 左边字母`A`大于右边字母`B`，则`sum = sum + roman[A]`
- 右边字母`C`小于左边字母`D`,则`sum = sum - roman[C]`
- 不存在连续小于的情况，一次小于下一次必定大于

```cpp
class Solution {
public:
    int romanToInt(string s) {
        unordered_map<char, int> roman = {
            {'I', 1},
            {'V', 5},
            {'X', 10},
            {'L', 50},
            {'C', 100},
            {'D', 500},
            {'M', 1000}};

        int sum = 0;
      
        for (int i = 0; i < s.size() - 1; ++i)
        {
            int temp = roman[s[i]];
            if (roman[s[i + 1]] > roman[s[i]])
                sum -= temp;
            else
                sum += temp;
        }
            sum += roman[s[s.size() - 1]];
        return sum;
    }
};
```



# 14. 最长公共前缀

## 题目描述

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

## 示例

- 示例 1：
  - 输入：`strs = ["flower","flow","flight"]`
  - 输出：`"fl"`
  - 解释：三个字符串的公共前缀为 "fl"。
- 示例 2：
  - 输入：`strs = ["dog","racecar","car"]`
  - 输出：`""`
  - 解释：三个字符串没有公共前缀，返回空字符串。

## 提示

- `1 <= strs.length <= 200`
- `0 <= strs[i].length <= 200`
- `strs[i]` 仅由小写英文字母组成

## 解题

### 我的解题

**思路：**

- 依次比对第i个字符，若出现不同则直接返回当前`substr`
- for循环完成仍无返回说明最长公共前缀为`strs[0]`;

```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if(strs.empty()) return "";
        if(strs.size() == 1) return strs[0];
        int length = strs.size();
        int strLength = strs[0].size();

        for (int i = 0; i < strLength; ++i) {
            char topc = strs[0][i];
            for (int j = 1; j < length; ++j) {
                if(topc != strs[j][i]) return strs[0].substr(0,i);
            }
        }

        return strs[0];
    }
};
```

### 官方题解

**思路：**

- 先寻找第1个和第2个字符串公共前缀`prefix`
- 再获取第3个字符串和`prefix`的公共前缀赋值给`prefix`
- 重复上述步骤，遍历完字符串数组即可获取公共前缀

```cpp
class Solution {
public:
    string longestCommonPrefix(string s1,string s2)
    {
        int length = min(s1.size(),s2.size());
        int preLength = 0;
        for(int i = 0;i<length;++i){
            if(s1[i] == s2[i]) preLength++;
            else break;
        }
        return s1.substr(0,preLength);
    }
    string longestCommonPrefix(vector<string>& strs) {
        int length = strs.size();
        int strLength = strs[0].size();
        int preLength = 0;
        string prefix = strs[0];
        for(int i = 0;i <length;++i)
        {
            prefix = longestCommonPrefix(prefix,strs[i]);
            preLength = prefix.size();
            if(preLength == 0  ) return "";
        }
        return strs[0].substr(0,preLength);
    }
};
```

### 更高效的解题

**思路：**

在一个字符串数组中，最长公共前缀必定是数组中 “字典序最小” 和 “字典序最大” 的两个字符串的最长公共前缀。

字典序是一种排序规则：

1. `**"apple" vs "banana"**
   - 比较第一个字符：'a' vs 'b'。
   - 因为 'a' < 'b'，所以 **"apple" < "banana"**。
2. **"apple" vs "app"**
   - 比较第一个字符：'a' == 'a'。
   - 比较第二个字符：'p' == 'p'。
   - 比较第三个字符：'p' == 'p'。
   - 此时，"app" 已经没有更多字符了，而 "apple" 还有。
   - 因为 "app" 是 "apple" 的前缀，所以 **"app" < "apple"**。

```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if(strs.empty()) return "";
        const auto p = minmax_element(strs.begin(), strs.end());
        for(int i = 0;i <p.first->size();++i){
            if(p.first->at(i)!=p.second->at(i)) return p.first->substr(0,i);
        }
        return *p.first;
    }
};
```





# 15. 三数之和

## 题目描述

给你一个包含 `n` 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 `a`、`b`、`c`，使得 `a + b + c = 0`。请你找出所有和为 `0` 且不重复的三元组。

**注意**：答案中不可以包含重复的三元组。

## 示例

- **示例 1**：
  - 输入：`nums = [-1, 0, 1, 2, -1, -4]`
  - 输出：`[[-1, 0, 1], [-1, -1, 2]]`
  - 解释：
    - `(-1) + 0 + 1 = 0`
    - `(-1) + (-1) + 2 = 0`
- **示例 2**：
  - 输入：`nums = []`
  - 输出：`[]`
- **示例 3**：
  - 输入：`nums = [0]`
  - 输出：`[]`

## 提示

- `0 <= nums.length <= 3000`
- `-10^5 <= nums[i] <= 10^5`

## 解题

### 我的解题

**问题：**

暴力解法

时间复杂度过高*`O(N³)`*，去重依靠`find`函数,数据多时超时

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        
        vector<vector<int>> answer;
        sort(nums.begin(),nums.end());
        int length = nums.size();
        for (int i = 0; i < length; ++i) {
            for (int j = i+1; j < length; ++j) {
                    for (int k =j+1; k < length; k++) {
                            if (nums[i] + nums[j] + nums[k] == 0) {
                                vector<int> temp = {nums[i], nums[j], nums[k]};
                                auto it  = find(answer.begin(),answer.end(),temp);
                                if(it == answer.end())
                                answer.push_back(temp);
                        }
                    }
            }
        }
        return answer;
    }
};
```

### 官方题解

**思路**：

- **排序**：先对数组进行排序，这样可以方便地跳过重复元素，并使用双指针法高效查找。
- **遍历 + 双指针：**
  - 固定一个元素 `nums[i]`（作为三元组的第一个元素）。
  - 使用双指针 `low = i + 1` 和 `high = nums.size() - 1` 查找另外两个元素。
  - 根据三数之和调整指针：
    - 如果和为 0，加入结果集，并跳过左右指针的重复元素。
    - 如果和小于 0，移动左指针增大 sum。
    - 如果和大于 0，移动右指针减小 sum。
- **去重：**遇到相同元素的时移动指针

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> answer;
        //排序用于后续去重
        sort(nums.begin(), nums.end());
        int length = nums.size();
        

        for (int i = 0; i < length-2; ++i) {
            int low = i + 1;
            int high = length - 1;
            while (high > low) {
                int sum = nums[i] + nums[low] + nums[high];
                if (sum == 0) {
                    vector<int> temp = {nums[i], nums[low], nums[high]};
                    answer.push_back(temp);
                    //去重1
                    while(low < high && nums[low] == nums[low+1]) low++;
                    low++;
                    while( low < high && nums[high]==nums[high-1]) --high;
                    --high;
                    
                } else if (sum > 0 && high > low) {
                    --high;
                } else if (sum < 0 && high > low) {
                    ++low;
                }
            }
            //去重2
            while(i<length-2 && nums[i]==nums[i+1]) ++i;
        }
        return answer;
    }
};
```





# 17. 电话号码的字母组合

## 题目描述

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。

数字到字母的映射如下（与电话按键相同）。注意，1 不对应任何字母。

```plaintext
2: abc
3: def
4: ghi
5: jkl
6: mno
7: pqrs
8: tuv
9: wxyz
```

## 示例

- **示例 1**：
  - 输入：`digits = "23"`
  - 输出：`["ad","ae","af","bd","be","bf","cd","ce","cf"]`
- **示例 2**：
  - 输入：`digits = ""`
  - 输出：`[]`
- **示例 3**：
  - 输入：`digits = "2"`
  - 输出：`["a","b","c"]`

## 提示

- `0 <= digits.length <= 4`
- `digits[i]` 是范围 `['2', '9']` 的字符。

## 解题

### 我的解题：

**思路**：

递归(实际上是`dfs`)

```cpp
class Solution {
public:
    // 非递归
    Solution() {
        map_digits = {{'2', "abc"}, {'3', "def"},  {'4', "ghi"}, {'5', "jkl"},
                      {'6', "mno"}, {'7', "pqrs"}, {'8', "tuv"}, {'9', "wxyz"}};
    }
    // 递归
    void fac(string& str, string& digits) {
        if (str.size() == digits.size()) {
            answer.push_back(str);
            return;
        }
        string Letters = map_digits[digits[str.size()]];
        for (auto& it : Letters) {
            string newStr = str + it;
            fac(newStr, digits);
        }
    }
    vector<string> letterCombinations(string digits) {
        string str = "";
        fac(str, digits);
        return answer;
    }

private:
    unordered_map<char, string> map_digits;
    vector<string> answer;
};
```



# 28. 找出字符串中第一个匹配项的下标

## 题目描述

给你两个字符串 `haystack` 和 `needle`，请你在 `haystack` 字符串中找出 `needle` 字符串的第一个匹配项的下标（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回 `-1`。

## 示例

- **示例 1**：
  - 输入：`haystack = "sadbutsad", needle = "sad"`
  - 输出：`0`
  - 解释：`"sad"` 在下标 `0` 和 `6` 处匹配。第一个匹配项的下标是 `0`。
- **示例 2**：
  - 输入：`haystack = "leetcode", needle = "leeto"`
  - 输出：`-1`
  - 解释：`"leeto"` 没有在 `haystack` 中出现，返回 `-1`。

## 提示

- `1 <= haystack.length, needle.length <= 10^4`
- `haystack` 和 `needle` 仅由小写英文字符组成

## 进阶

你能实现一个时间复杂度为 `O(n + m)` 的算法吗？（其中 `n` 是 `haystack` 的长度，`m` 是 `needle` 的长度）

## 解题

### 我的解题

直接调用c++的search函数

```cpp
class Solution {
public:
    int strStr(string haystack, string needle) {
        auto it = search(haystack.begin(),haystack.end(),needle.begin(),needle.end());
        if(it == haystack.end()) return -1;
        return distance(haystack.begin(),it);
    }
};
```

### 官方题解

`KMP`算法

**`next`数组**

`next[i+1]`等于前`i`个长度的字符串的最长公共字符串

**实现方式**

- `i`：指向当前正在处理的子串的末尾（后缀的末尾），`j`：指向前缀的末尾，同时也代表了当前找到的最长相等前缀的长度。

- **初始化**：

  - 创建一个与模式串 `needle` 长度相同的 `next` 数组。
  - 令 `i = 0`，`j = -1`。
  - 设置 `next[0] = -1`。这是一个约定的初始值，表示在第一个字符之前没有任何前缀。

- **循环处理**：当 `i` 小于模式串长度减 1 时（`i < m - 1`）：

  a. **情况 1：`j == -1` 或 `needle[i] == needle[j]`**

  - 如果 `j == -1`，说明之前没有找到任何匹配的前缀，或者已经回溯到了起点。
  - 如果 `needle[i] == needle[j]`，说明找到了更长的一对相等的前缀和后缀。
  - 操作
    - `i` 和 `j` 都向后移动一位（`i++`, `j++`）。
    - 将 `j` 的值赋给 `next[i]`。这表示 `needle[0..i]` 的最长相等前缀后缀长度为 `j`。

  b. **情况 2：`needle[i] != needle[j]`**

  - 当前字符不匹配，说明不能延长当前的最长前缀。
  - 操作
    - 将 `j` 回溯到 `next[j]` 的位置。这是 `KMP` 算法的精髓所在。它利用已经计算出的 `next` 数组信息，让我们不需要从头开始重新比较，而是直接跳到可能与 `needle[i]` 匹配的位置。

**`nextval`数组**

**实现方式**

- 令`j` = `next[i]`
- 若`needle[i] != needle[j]`，那么`nextval[i]= j`
- 若`needle[i] == needle[j]`，那么循环令`j = nextval[j]`，直至`needle[i] != needle[j]`

```cpp

```



# 30. 串联所有单词的子串

## 题目描述

给定一个字符串 `s` 和一个字符串数组 `words`，`words` 中的所有字符串 **长度相同**。

找出 `s` 中所有由 `words` 中所有单词串联形成的子串的起始索引。返回的索引可以按 **任意顺序** 排列。

**说明**：

- 串联是指将 `words` 中的所有单词按任意顺序连接形成的字符串（每个单词仅使用一次）。
- `s` 的子串必须完全匹配串联后的字符串，不能有额外字符。

## 示例

- **示例 1**：
  - 输入：`s = "barfoothefoobarman", words = ["foo","bar"]`
  - 输出：`[0,9]`
  - 解释：
    - 起始索引 0：子串 "barfoo" 是 "bar" + "foo" 的串联。
    - 起始索引 9：子串 "foobar" 是 "foo" + "bar" 的串联。
- **示例 2**：
  - 输入：`s = "wordgoodgoodgoodbestword", words = ["word","good","best","word"]`
  - 输出：`[]`
  - 解释：`s` 中没有由所有单词串联形成的子串。
- **示例 3**：
  - 输入：`s = "barfoofoobarthefoobarman", words = ["bar","foo","the"]`
  - 输出：`[6,9,12]`
  - 解释：
    - 起始索引 6：子串 "foobarthe" 是 "foo" + "bar" + "the" 的串联。
    - 起始索引 9：子串 "barthefoo" 是 "bar" + "the" + "foo" 的串联。
    - 起始索引 12：子串 "thefoobar" 是 "the" + "foo" + "bar" 的串联。

## 提示

- `1 <= s.length <= 10^4`
- `1 <= words.length <= 5000`
- `1 <= words[i].length <= 30`
- `s` 和 `words[i]` 仅由小写英文字母组成

## 解题

### 我的解题

# 42. 接雨水

## 题目描述

给定 `n` 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

简单来说，柱子之间的 “凹槽” 可以储存雨水，需计算所有凹槽能容纳的雨水总量（宽度均为 1，高度为凹槽两侧最高柱子的最小值减去当前凹槽底部高度）。

## 示例

### 示例 1

- 输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]

- 输出：6

- 解释：

  

  该高度图对应的柱子排列如：

  

<img src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rainwatertrap.png" style="zoom: 150%;" />

具体储水量计算：

- 索引 2（高度 0）：左右最高分别为 1 和 2，储水 1-0=1

- 索引 4（高度 1）：左右最高分别为 2 和 3，储水 2-1=1

- 索引 5（高度 0）：左右最高分别为 2 和 3，储水 2-0=2

- 索引 6（高度 1）：左右最高分别为 2 和 3，储水 2-1=1

- 索引 9（高度 1）：左右最高分别为 3 和 2，储水 2-1=1

  

  总储水量：1+1+2+1+1=6。

### 示例 2

- 输入：height = [4,2,0,3,2,5]

- 输出：9

- 解释：

  

  柱子排列形成的凹槽储水总量为 9，核心凹槽包括：

  - 索引 2（高度 0）：左右最高 4 和 5，储水 4-0=4

  - 索引 1（高度 2）：左右最高 4 和 3，储水 3-2=1

  - 索引 3（高度 3）：左右最高 4 和 5，储水 4-3=1

  - 索引 4（高度 2）：左右最高 4 和 5，储水 4-2=3

    

    总储水量：4+1+1+3=9。

## 提示

- `n == height.length`
- 1 <= n <= 2 * 10^4
- 0 <= height[i] <= 10^5

## 解题

### 我的解题

**思路**：

- 找到左边最高的柱子高度 `left_max[i]`
- 找到右边最高的柱子高度 `right_max[i]`
- 该位置能装的水 = `min(left_max[i], right_max[i]) - height[i]`（如果结果为正，否则为 0）
- ![](https://assets.leetcode-cn.com/solution-static/42/1.png)

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int length = height.size();
        vector<int> left_max(length, 0);
        int left = 0, right = 0;
        vector<int> right_max(length, 0);
        for (int i = 0; i < length; ++i) {
            if (height[i] > left)
                left = height[i];
            int j = length - 1 - i;
            if (height[j] > right)
                right = height[j];
            left_max[i] = left;
            right_max[j] = right;
        }
        int sum = 0;
        for (int ii = 0; ii < length; ++ii) {
            sum += min(left_max[ii], right_max[ii]) - height[ii];
        }
        return sum;
    }
};
```

### 官方题解

1、**单调栈**

- **压栈**：遇到比栈顶矮或相等的柱子，直接压入（保持递减）；
- **弹栈**：遇到比栈顶高的柱子，开始弹栈，每弹出一个栈顶（凹槽底部），就计算一个凹槽的储水量；
- **计算**：每个凹槽的储水量 = （左右边界最小值 - 凹槽底部高度）× （右边界索引 - 左边界索引 - 1）；
- **终止**：栈为空或当前高度 <= 新栈顶高度时，停止弹栈，将当前索引压入栈。

[42. 接雨水 - 力扣（LeetCode）](https://leetcode.cn/problems/trapping-rain-water/solutions/692342/jie-yu-shui-by-leetcode-solution-tuvc/?envType=study-plan-v2&envId=top-interview-150)

```cpp
```



# 58. 最后一个单词的长度

## 题目描述

给你一个字符串 `s`，由若干单词组成，单词前后用一些空格字符隔开。返回字符串中 **最后一个单词** 的长度。

**单词** 是指仅由字母组成、不包含任何空格字符的最大子字符串。

## 示例

- 示例 1：输入 `s = "Hello World"`，输出 `5`，解释：最后一个单词是 "World"，长度为 5。
- 示例 2：输入 `s = " fly me to the moon "`，输出 `4`，解释：最后一个单词是 "moon"，长度为 4。
- 示例 3：输入 `s = "luffy is still joyboy"`，输出 `6`，解释：最后一个单词是 "joyboy"，长度为 6。

## 提示

- 1 <= s.length <= 10^4
- `s` 仅包含英文字母和空格 `' '`
- `s` 中至少存在一个单词

## 解题

### 我的解题

**思路：**

- 倒序遍历字符串
- 遇到空格就记录长度answer，且将长度计数count归零，若answer不为零提前返回
- 对于像"a","a "的字符串，count记录的就是最后一个单词长度，因此for循环外可以直接返回count

```cpp
class Solution {
public:
    int lengthOfLastWord(string s) {
        int count = 0;
        int answer = 0;
        for (int i = s.size() - 1; i >= 0; --i) {
            if (s[i] != ' ') {
                count++;
            } else {

                answer = count;
                count = 0;
            }
            if(answer) return answer;
        }

        return count;
    }
};
```

### 官方题解

**思路：**

- 倒序遍历字符串
- 先遍历字符串后的空格，去除字符串后的空格
- 再遍历最后一个单词，遇到空格或者索引小于0时停止遍历，记录的wordLength就是最后一个单词长度

```cpp
class Solution {
public:
    int lengthOfLastWord(string s) {
        int index = s.size() - 1;

        while (s[index] == ' ') {
            index--;
        }
        int wordLength = 0;
        while (index >= 0 && s[index] != ' ') {
            wordLength++;
            index--;
        }

        return wordLength;
    }
};

```



# 68. 文本左右对齐

## 题目描述

给定一个单词数组 `words` 和一个长度 `maxWidth`，请你重新排版单词，使其成为每行恰好有 `maxWidth` 个字符，且左右两端对齐的文本。

你需要使用以下规则进行排版：

1. 每个单词之间的空格必须尽可能均匀分布。如果单词间的空格不能均匀分配，则左侧的空格数要多于右侧的空格数。
2. 最后一行应为左对齐，且单词之间不插入额外的空格，末尾补满空格至 `maxWidth` 长度。

## 示例

- **示例 1**：

  - 输入：`words = ["This", "is", "an", "example", "of", "text", "justification."], maxWidth = 16`

  - 输出：

    ```plaintext
    ["This    is    an",
     "example  of text",
     "justification.  "]
    ```

    

  - 解释：

    - 第一行有 3 个单词，需分配 16 - (4+2+2) = 8 个空格，均匀分配为 4、4，故单词间各 4 个空格。
    - 第二行有 3 个单词，需分配 16 - (7+2+4) = 3 个空格，左侧多分配 1 个，故空格数为 2、1。
    - 第三行是最后一行，左对齐，单词间 1 个空格，末尾补 2 个空格至 16 长度。

- **示例 2**：

  - 输入：`words = ["What","must","be","acknowledgment","shall","be"], maxWidth = 16`

  - 输出：

    ```plaintext
    ["What   must   be",
     "acknowledgment  ",
     "shall be        "]
    ```

    

  - 解释：

    - 第一行 3 个单词，分配 16 - (4+4+2) = 6 个空格，均匀分配为 3、3。
    - 第二行 1 个单词，左对齐（最后一行前的单行也需左对齐补空格），末尾补 2 个空格。
    - 第三行是最后一行，左对齐，单词间 1 个空格，末尾补 6 个空格。

- **示例 3**：

  - 输入：`words = ["Science","is","what","we","understand","well","enough","to","explain","to","a","computer.","Art","is","everything","else","we","do"], maxWidth = 20`

  - 输出：

    ```plaintext
    ["Science  is  what we",
     "understand      well",
     "enough to explain to",
     "a computer. Art is",
     "everything else we do"]
    ```

## 提示

- `1 <= words.length <= 300`
- `1 <= words[i].length <= 20`
- `words[i]` 由英文字母和符号组成
- `1 <= maxWidth <= 100`
- `words[i].length <= maxWidth`

## 解题

### 我的解题

纯浪费时间的题

# 134. 加油站

## 题目描述

在一条环路上有 `n` 个加油站，其中第 `i` 个加油站有汽油 `gas[i]` 升。

你有一辆油箱容量无限的汽车，从第 `i` 个加油站开往第 `i+1` 个加油站需要消耗汽油 `cost[i]` 升。你从其中一个加油站出发，开始时油箱为空。

如果你可以绕环路行驶一周，则返回出发时加油站的编号；否则，返回 `-1`。

**注意**：题目数据保证答案唯一。

## 示例

### 示例 1

- 输入：gas = [1,2,3,4,5], cost = [3,4,5,1,2]
- 输出：3
- 解释：
  1. 从 3 号加油站（索引为 3）出发，油箱有 0 升汽油。
  2. 加 4 升汽油，油箱剩 4 升。开往 4 号加油站，消耗 1 升，剩 3 升。
  3. 加 5 升汽油，油箱剩 8 升。开往 0 号加油站，消耗 2 升，剩 6 升。
  4. 加 1 升汽油，油箱剩 7 升。开往 1 号加油站，消耗 3 升，剩 4 升。
  5. 加 2 升汽油，油箱剩 6 升。开往 2 号加油站，消耗 4 升，剩 2 升。
  6. 加 3 升汽油，油箱剩 5 升。开往 3 号加油站，消耗 5 升，剩 0 升。
  7. 刚好绕环路行驶一周，因此返回 3。

### 示例 2

- 输入：gas = [2,3,4], cost = [3,4,3]
- 输出：-1
- 解释：
  1. 从 0 号加油站出发，加 2 升汽油，开往 1 号加油站消耗 3 升，油箱为空，无法继续行驶。
  2. 从 1 号加油站出发，加 3 升汽油，开往 2 号加油站消耗 4 升，油箱为空，无法继续行驶。
  3. 从 2 号加油站出发，加 4 升汽油，开往 0 号加油站消耗 3 升，剩 1 升。再开往 1 号加油站消耗 3 升，油箱不足，无法继续行驶。
  4. 因此返回 -1。

## 提示

- `n == gas.length == cost.length`
- 1 <= n <= 10^5
- 0 <= gas[i], cost[i] <= 10^4

## 解题

### **我的解题：**

**问题**：超时

```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
       int length = cost.size();
        std::vector<int> r_cost(length, 0);

        for (int i = 0; i < length; ++i)
        {
            r_cost[i] = gas[i] - cost[i];
        }
        int answer = -1;
        for (int i = 0; i < length; i++)
        {
            if (r_cost[i] < 0)
                continue;
            int sum = 0;
            for (int j = 0; j < length; j++)
            {
                sum += r_cost[(i + j) % length];
                if (sum < 0)
                    break;
            }
            if (sum >= 0)
            {
                answer = i;
                break;
            }
        }
        return answer;
    }
};
```

### **官方题解：**

**解题思路：**贪心算法

- 如果从起点 `start` 出发，到某个加油站 `k` 时，油箱里的油量 `tank` 第一次变成了负数，那么说明`start`到`k`之间的任一点都不能被当做起点。
- 为什么？假如选择`start`到`k`之间一点`g`作为起点，那么`start`到`g`时的油箱油量`tank`是大于0的，由此易得`g`是到达不了`k`点的
- 下一次循环选择`k+1`作为起点

```c++
#include <vector>
using namespace std;

class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int n = gas.size();  // 加油站数量（gas 和 cost 长度相同）
        int totalGas = 0;    // 总油量 - 总消耗（判断是否存在可行解）
        int currentGas = 0;  // 当前油箱剩余油量（模拟从当前起点出发的行驶过程）
        int startIndex = 0;  // 候选起点索引

        for (int i = 0; i < n; ++i) {
            // 累加总油量差和当前油箱油量
            totalGas += gas[i] - cost[i];
            currentGas += gas[i] - cost[i];

            // 关键判断：如果当前油箱油量为负，说明从 startIndex 到 i 的路径不可行
            // 所有介于 startIndex 和 i 之间的起点也都不可行，直接将起点更新为 i+1
            if (currentGas < 0) {
                startIndex = i + 1;  // 重置起点
                currentGas = 0;      // 重置当前油箱（从新起点开始，初始油量为 0）
            }
        }

        // 最终判断：如果总油量差 >= 0，说明存在可行解，且 startIndex 就是唯一解（题目保证答案唯一）
        // 否则，返回 -1（总油量不足，无法绕一圈）
        return totalGas >= 0 ? startIndex : -1;
    }
};
```



# 135. 分发糖果

## 题目描述

`n` 个孩子站成一排。给你一个整数数组 `ratings` 表示每个孩子的评分。

你需要按照以下要求，给这些孩子分发糖果：

1. 每个孩子至少分配到 **1** 个糖果。
2. 相邻两个孩子评分更高的孩子会获得更多的糖果。

请你给每个孩子分发糖果，计算并返回需要准备的 **最少糖果数目**。

## 示例

### 示例 1：

- **输入**: `ratings = [1,0,2]`

- **输出**: `5`

- 解释

  :

  - 1 号孩子（索引 0）得到 2 个糖果。
  - 2 号孩子（索引 1）得到 1 个糖果。
  - 3 号孩子（索引 2）得到 2 个糖果。
  - 总共需要 `2 + 1 + 2 = 5` 个糖果。
  - 分配结果：`[2, 1, 2]`。

### 示例 2：

- **输入**: `ratings = [1,2,2]`

- **输出**: `4`

- 解释

  :

  - 1 号孩子（索引 0）得到 1 个糖果。
  - 2 号孩子（索引 1）得到 2 个糖果。
  - 3 号孩子（索引 2）得到 1 个糖果。
  - 总共需要 `1 + 2 + 1 = 4` 个糖果。
  - 分配结果：`[1, 2, 1]`。
  - 注意：第三个孩子和第二个孩子评分相同，所以不需要比第二个孩子多。

## 提示

- `n == ratings.length`
- `1 <= n <= 2 * 10^4`
- `0 <= ratings[i] <= 2 * 10^4`

## 解题：

### 我的解题：

**思路：**

1. 从左到右遍历，确保每个孩子都比他左边评分低的邻居得到更多糖果。
2. 从右到左遍历，确保每个孩子都比他右边评分低的邻居得到更多糖果。

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int length = ratings.size();
        int answer = 0;
        vector<int> candy_nums(length, 1);//记录每个孩子的candy数量
        if (length == 1)
            return 1;
        else {
            //正序发糖
            for (int i = 0; i < length - 1; ++i) {
                if (ratings[i + 1] > ratings[i])
                    candy_nums[i + 1] = candy_nums[i] + 1;
            }
            //逆序发糖
            for (int i = length - 1; i > 0; --i) {
                if (ratings[i - 1] > ratings[i] && candy_nums[i-1]<=candy_nums[i])
                    candy_nums[i - 1] = candy_nums[i] + 1;
                    answer += candy_nums[i];//计算已经确定数量的糖
            }
            answer += candy_nums[0];//加上剩余数量的糖
        }
        return answer;
    }
};
```

### 官方题解

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();
        vector<int> left(n);
        for (int i = 0; i < n; i++) {
            if (i > 0 && ratings[i] > ratings[i - 1]) {
                left[i] = left[i - 1] + 1;
            } else {
                left[i] = 1;
            }
        }
        int right = 0, ret = 0;
        for (int i = n - 1; i >= 0; i--) {
            if (i < n - 1 && ratings[i] > ratings[i + 1]) {
                right++;
            } else {
                right = 1;
            }
            ret += max(left[i], right);
        }
        return ret;
    }
};
```



# 151. 反转字符串中的单词

## 题目描述

给你一个字符串 `s` ，请你反转字符串中 **单词** 的顺序。

**单词** 是由非空格字符组成的字符串。`s` 中使用至少一个空格将单词分隔开。

返回 **单词顺序颠倒且单词之间用单个空格连接的结果字符串**。

**注意：** 输入字符串 `s`中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

## 示例

- **示例 1：**
  - 输入：`s = "the sky is blue"`
  - 输出：`"blue is sky the"`
- **示例 2：**
  - 输入：`s = " hello world "`
  - 输出：`"world hello"`
  - 解释：反转后的字符串中不能存在前导空格和尾随空格。
- **示例 3：**
  - 输入：`s = "a good example"`
  - 输出：`"example good a"`
  - 解释：如果两个单词间有多个空格，反转后单词间应仅用单个空格分隔。

## 提示

- `1 <= s.length <= 10^4`
- `s` 包含英文大小写字母、数字和空格 `' '`
- `s` 中 **至少存在一个** 单词

## 进阶

如果字符串在你使用的编程语言中是一种可变数据类型，请尝试使用 O (1) 额外空间复杂度的原地解法。

## 解题

# 209. 长度最小的子数组

## 题目描述

给定一个含有 `n` 个正整数的数组和一个正整数 `target` 。

找出该数组中满足其总和大于等于 `target` 的长度最小的 **连续子数组** `[nums[l], nums[l+1], ..., nums[r]]` ，并返回其长度。如果不存在符合条件的子数组，返回 `0` 。

## 示例

- **示例 1**：
  - 输入：`target = 7, nums = [2,3,1,2,4,3]`
  - 输出：`2`
  - 解释：子数组 `[4,3]` 的总和是 7，且长度最小。
- **示例 2**：
  - 输入：`target = 4, nums = [1,4,4]`
  - 输出：`1`
  - 解释：子数组 `[4]` 的总和是 4，长度为 1。
- **示例 3**：
  - 输入：`target = 11, nums = [1,1,1,1,1,1,1,1]`
  - 输出：`0`
  - 解释：不存在总和大于等于 11 的连续子数组，返回 0。

## 提示

- `1 <= target <= 10^9`
- `1 <= nums.length <= 10^5`
- `1 <= nums[i] <= 10^5`

## 进阶

如果你已经实现 `O(n)` 时间复杂度的解法，请尝试设计一个 `O(n log n)` 时间复杂度的解法。

## 解题

### 我的解题

滑动窗口算法的**核心思想**是：通过维护一个动态窗口，在 O (n) 时间内解决子数组 / 子字符串问题。

**关键点**：

1. **窗口的定义**：由 `left` 和 `right` 指针确定。
2. **窗口的移动**：右指针扩大窗口，左指针缩小窗口。

```c++
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int n = nums.size();
        int ans = 100001;
        int left = 0,right = 1,sum = nums[0];
        while(right>left){
            if(sum <target&&right<n){
                sum += nums[right++];
            }
            else{
                if(sum >= target) ans =min(ans,right - left);
                sum -= nums[left++];
            }
        }
        return ans==100001? 0:ans;
    }
};
```



# 238. 除自身以外数组的乘积

## 题目描述

给你一个整数数组 `nums`，返回 数组 `answer`，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积。

题目数据 **保证** 数组 `nums` 之中任意元素的全部前缀元素和后缀的乘积都在 **32 位整数** 范围内。

**进阶**：你可以在 O (n) 时间复杂度和 O (1) 空间复杂度内完成此题吗？（出于对空间复杂度分析的目的，输出数组 **不被视为** 额外空间。）

## 示例

### 示例 1

- 输入：nums = [1,2,3,4]
- 输出：[24,12,8,6]

### 示例 2

- 输入：nums = [-1,1,0,-3,3]
- 输出：[0,0,9,0,0]

## 提示

- 2 <= nums.length <= 10^5
- -30 <= nums[i] <= 30
- 保证 数组 `nums` 之中任意元素的全部前缀元素和后缀的乘积都在 32 位整数 范围内

## 说明

- 请 **不要使用除法**，且在 O (n) 时间复杂度内完成此题。

## 解题

### 我的解法：

**思路：**引入俩个数组分别记录前n个数的乘积和后n个数的乘积，获取第n个数的结果只需要前n-1个的乘积和n+1后的数的乘积

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int lenth = nums.size();
        std::vector<int> product_front(lenth,1);//记录前n个数的乘积
        std::vector<int> product_back(lenth,1);//记录后n个数的乘积
        int acc_f = 1;
        int acc_b = 1;
        for(int i = 0;i<lenth;++i)
        {
            acc_f *= nums[i] ;
            product_front[i] = acc_f;
            acc_b *= nums[lenth-1-i];
            product_back[lenth-1-i] = acc_b;
        }
        
        for(int i =0;i<nums.size();++i){
            if(i == 0) nums[i] = product_back[1];
            else if(i == nums.size()-1 ) nums[i] =product_front[i-1];
            else nums[i] = product_front[i-1]*product_back[i+1];
        }
        
        return nums;
    }
};
```

### **官方题解：**

**解决方案：**answer[i]存储i之前元素的乘积，之后用int R计算后1,2,3...数的乘积，同时令answer[i] = answer[i] * R

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int length = nums.size();
        vector<int> answer(length);

        // answer[i] 表示索引 i 左侧所有元素的乘积
        // 因为索引为 '0' 的元素左侧没有元素， 所以 answer[0] = 1
        answer[0] = 1;
        for (int i = 1; i < length; i++) {
            answer[i] = nums[i - 1] * answer[i - 1];
        }

        // R 为右侧所有元素的乘积
        // 刚开始右边没有元素，所以 R = 1
        int R = 1;
        for (int i = length - 1; i >= 0; i--) {
            // 对于索引 i，左边的乘积为 answer[i]，右边的乘积为 R
            answer[i] = answer[i] * R;
            // R 需要包含右边所有的乘积，所以计算下一个结果时需要将当前值乘到 R 上
            R *= nums[i];
        }
        return answer;
    }
};
```

## 文案



# 380. O (1) 时间插入、删除和获取随机元素

## 题目描述

实现 `RandomizedSet` 类：

- `RandomizedSet()` 初始化 `RandomizedSet` 对象。
- `bool insert(int val)` 当元素 `val` 不存在时，向集合中插入该项，并返回 `true`；否则，返回 `false`。
- `bool remove(int val)` 当元素 `val` 存在时，从集合中移除该项，并返回 `true`；否则，返回 `false`。
- `int getRandom()` 随机返回集合中的一项（测试用例保证调用此方法时集合中至少存在一个元素）。每个元素应该有 **相同的概率** 被返回。

你必须实现类的所有函数，并满足每个函数的 **平均时间复杂度为 O (1)** 。

## 示例

### 输入

["RandomizedSet", "insert", "remove", "insert", "getRandom", "remove", "insert", "getRandom"]

[[], [1], [2], [2], [], [1], [2], []]

### 输出

[null, true, false, true, 2, true, false, 2]

### 解释

1. RandomizedSet randomizedSet = new RandomizedSet();
2. randomizedSet.insert (1); // 向集合中插入 1。返回 true，表示 1 成功插入。
3. randomizedSet.remove (2); // 集合中不存在 2，返回 false。
4. randomizedSet.insert (2); // 向集合中插入 2。返回 true，表示 2 成功插入。
5. randomizedSet.getRandom (); // 随机返回 1 或 2（两者概率相同）。
6. randomizedSet.remove (1); // 从集合中移除 1。返回 true，表示 1 成功移除。
7. randomizedSet.insert (2); // 2 已在集合中，返回 false。
8. randomizedSet.getRandom (); // 集合中只有 2，返回 2。

## 提示

- `-2^31 <= val <= 2^31 - 1`
- 最多调用 `insert`、`remove` 和 `getRandom` 函数 `2 * 10^5` 次
- 在调用 `getRandom` 方法时，集合中至少存在一个元素

## 解题

### 我的解法：

**问题：**插入和删除没有做到O(1)时间复杂度,查询时间复杂度高O(n)

```cpp
#include<vector>
class RandomizedSet {
public:
    RandomizedSet() {
        
    }
    
    bool insert(int val) {
        auto it = std::find(set_int_.begin(),set_int_.end(),val);
        if(it!=set_int_.end()) return false;
        else
        {
            set_int_.push_back(val);
            return true;
        }
    }
    
    bool remove(int val) {
        if(set_int_.size()==0) return false;
        auto it = std::find(set_int_.begin(),set_int_.end(),val);
        if(it!=set_int_.end())
        {
            set_int_.erase(it);
            return true;
        }
        else return false;
    }
    
    int getRandom() {
        
        return set_int_[rand()%set_int_.size()];
    }
private:
    std::vector<int> set_int_;

};
```

### 官方题解：

**解决方案：**引入unordered_map使查询时间复杂度为O(1)

```cpp
class RandomizedSet {
public:
    RandomizedSet() {
        srand((unsigned)time(NULL));
    }
    
    bool insert(int val) {
        if (indices.count(val)) {
            return false;
        }
        int index = nums.size();
        nums.emplace_back(val);
        indices[val] = index;
        return true;
    }
    
    bool remove(int val) {
        if (!indices.count(val)) {
            return false;
        }
        int index = indices[val];
        int last = nums.back();
        nums[index] = last;
        indices[last] = index;
        nums.pop_back();
        indices.erase(val);
        return true;
    }
    
    int getRandom() {
        int randomIndex = rand()%nums.size();
        return nums[randomIndex];
    }
private:
    vector<int> nums;
    unordered_map<int, int> indices;
};
```



