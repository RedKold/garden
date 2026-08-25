# Trie（字典树）

> [!summary] 类型总结
> Trie 把字符串按字符逐层存入树中，适合**前缀匹配、公共前缀统计**。
>
> - 820 单词压缩编码：后缀问题反转成前缀问题，插入时记录"经过节点的字符数 count"；末尾节点 count==0 说明没有更长单词覆盖它，需要单独编码。
>
> 通用思路：把不擅长的匹配方向（后缀）转化成 Trie 擅长的形态（前缀）→ 插入时维护额外信息（count/结尾标记）→ 按末尾节点统计答案。注意 count 自增的位置是"当前节点"而非"子节点"。

---
## 前缀和+Trie（反向插入）

### [820. 单词的压缩编码](https://leetcode.cn/problems/short-encoding-of-words/)

#### 题面

给定一个单词列表，将这个列表压缩到一个**索引字符串** `S` 中，每个单词以 `#` 结尾。例如 `["time", "me", "bell"]` 可以压缩为 `"time#bell#"`（`me` 是 `time` 的后缀，不需要重复编码）。返回 `S` 的最小长度。

#### 思路

**核心洞察**：如果一个单词是另一个单词的**后缀**，则它不需要被编码（已被覆盖）。

将每个单词**反转**后插入 Trie，则后缀匹配转化为**前缀匹配**。

- 插入时，Trie 节点的 `count` 记录有多少字符"经过"了该节点（即有多少个单词以该节点为前缀）
- 插入完成后，检查每个单词的**末尾节点**：
  - `count == 0`：没有其他单词经过该节点 → 该单词不是任何单词的后缀 → 需要编码
  - `count > 0`：有其他单词经过该节点 → 该单词是某个更长单词的后缀 → 不需要编码

> **为什么要反转？** 后缀匹配在 Trie 中不好直接做（Trie 擅长前缀匹配），反转后就变成前缀匹配了。

#### Code

```cpp
class TrieNode{
    TrieNode* children[26];
public:
    int count;  // 经过该节点的字符数
    TrieNode(){
        for(int i = 0;i<26;i++){
            children[i] = NULL;
            count=0;
        }
    }

    TrieNode* get(char c){
        if(!children[c-'a']){
            children[c-'a'] = new TrieNode();
        }
        count++;  // 有字符经过此节点
        return children[c-'a'];
    }
};

class Solution {
public:
    int minimumLengthEncoding(vector<string>& words) {
        TrieNode* root = new TrieNode();
        int n = words.size();
        unordered_map<TrieNode*, int> Hash;

        for(int i=0;i<n;i++){
            string& word = words[i];
            TrieNode* cur = root;
            // 反转插入
            for(int j=word.size()-1; j>=0; j--){
                cur = cur->get(word[j]);
            }
            Hash[cur] = i;  // 记录单词末尾节点
        }

        int ans=0;
        for(auto&[node, i]: Hash){
            // count == 0: 没有其他单词经过 → 不是后缀 → 需要编码
            if(node->count == 0){
               ans += words[i].size() + 1;  // +1 是 '#'
            }
        }
        return ans;
    }
};
```

#### 关键细节

- 末尾节点的 `count` 永远为 0（因为 `get()` 是在**当前节点**上自增，然后返回**子节点**，末尾节点从未被调用 `get()`）
- 如果一个单词是另一个单词的后缀，反转后它会在 Trie 中"中途"结束，其末尾节点的 `count > 0`
- 时间复杂度 $O(\sum |word|)$，空间复杂度 $O(\sum |word| \cdot \Sigma)$

---

