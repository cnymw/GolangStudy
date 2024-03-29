# leetcode 27 移除元素

## 题目

给定一个数组 nums 和一个值 val，你需要原地移除所有数值等于 val 的元素，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

示例 1:

```text
给定 nums = [3,2,2,3], val = 3,

函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2

你不需要考虑数组中超出新长度后面的元素
```

示例 2:

```text
给定 nums = [0,1,2,2,3,0,4,2], val = 2,

函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4

注意这五个元素可为任意顺序

你不需要考虑数组中超出新长度后面的元素
```

## 题解

该题目对于 go 语言来说是非常轻松的，因为 go 的 slice 语法自带移除元素功能：

```go
q := []int{1, 2, 3}
q = append(q[:1], q[2:]...)
fmt.Println(q) // [1,3]
```

数字 2 位于索引 1 位置上，append(q[:1], q[2:]...) 方法即可返回一个移除索引 1 的新的数组。

所以如果要移除索引 index 位置的数字，那么 append(nums[:index], nums[index+1:]...) 即可满足要求。

实现如下：

```go
func removeElement(nums []int, val int) int {
    for index := 0; index < len(nums); {
        if nums[index] == val {
            nums = append(nums[:index], nums[index+1:]...)
        } else {
            index++
        }
    }
    return len(nums)
}
```