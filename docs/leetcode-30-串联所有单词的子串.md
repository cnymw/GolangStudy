# leetcode 30 串联所有单词的子串

## 题目

给定一个字符串 s 和一些长度相同的单词 words。找出 s 中恰好可以由 words 中所有单词串联形成的子串的起始位置。

注意子串要与 words 中的单词完全匹配，中间不能有其他字符，但不需要考虑 words 中单词串联的顺序。

示例 1
```go
输入:
  s = "barfoothefoobarman",
  words = ["foo","bar"]
输出:[0,9]
解释:
从索引 0 和 9 开始的子串分别是 "barfoo" 和 "foobar" 
输出的顺序不重要, [9,0] 也是有效答案
```

示例 2：
```go
输入:
  s = "wordgoodgoodgoodbestword",
  words = ["word","good","best","word"]
输出:[]
```

## 题解：

该题目实际意思是将输入 s 划分为长度一定的连续字符组，例如输入 s = "barfoothefoobarman"，实际含义是将 s 转换为 s1 = [bar foo the foo bar man]，s2 = [arf oot hef oob arm]...

然后将 words 转换为记录单词出现次数的哈希表， words_map = [foo:1 bar:1]

最后将转换后的 s(s1,s2,s3...) 序列按顺序放入到 words 构成的哈希表中去进行对比，例如将 s1 与 words_map 进行对比，可以发现 s1 满足 words_map 单词出现的次数(s1 中包含 [bar foo])，于是获得 s1 对应的单词起始位置 0

代码实现：

```go
func findSubstring(s string, words []string) []int {
    if len(s) == 0 || len(words) == 0 {
        return []int{}
    }
    wordsCnt := make(map[string]int, 0)
    for _, word := range words {
        wordsCnt[word]++
    }
    wordsLength := len(words)
    length := len(words[0])
    result := make([]int, 0)
    for index := 0; index+wordsLength*length <= len(s); index++ {
        ch := s[index : index+length]
    
        if _, ok := wordsCnt[ch]; ok {
            sCnt := make(map[string]int, 0)
            var j int
            for j = 0; j < wordsLength; j ++ {
                temp := s[index+j*length : index+(j+1)*length]
                sCnt[temp]++
                if sCnt[temp] > wordsCnt[temp] {
                    break
                }
            }
            if j == wordsLength {
                result = append(result, index)
            }
        }
    }
    return result
}
```