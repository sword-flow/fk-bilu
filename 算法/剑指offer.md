* #### [数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

```go
// 用一个map记录下扫描过的元素，遇到重复的返回
// 时间复杂度O(n)，空间复杂度O(n)
func findRepeatNumber(nums []int) int {
    scaned := make(map[int]bool)
    for _, i := range nums {
        if _, ok := scaned[i]; ok {
            return i
        } else {
            scaned[i] = true
        }
    }
    return -1
}
```

* #### [二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

```go
// 从左下角开始遍历
// 比左下角元素i大的都在右边，比i小的都在上面
// 遍历中止条件,找到元素或者遍历到了右上角
func findNumberIn2DArray(matrix [][]int, target int) bool {

    if len(matrix) == 0 || len(matrix[0]) == 0 {
        return false
    }
    column := 0
    row := len(matrix)-1
    for column <= len(matrix[0])-1 && row >= 0 {
        if matrix[row][column] == target {
            return true
        } else if  matrix[row][column] > target {
            row--
        } else {
            column++
        }
    }
    return false
}
```

#### [替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

```go
func replaceSpace(s string) string {
    var ans []byte
    for _, v := range []byte(s) {
        if v == ' ' {
            i := []byte("%20")
            ans = append(ans, i...)
        } else {
            ans = append(ans, v)
        }
    }
    return string(ans)
}
```

#### [从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */

// 遍历链表，用栈记录下遍历的结果，然后挨个出栈
// 时间复杂度O(n)，空间复杂度O(n)
func reversePrint(head *ListNode) []int {
    stack := []int{}
    for head != nil {
        stack = append(stack, head.Val)
        head = head.Next
    }

    ans := []int{}
    for i := len(stack)-1; i >= 0; i-- {
        ans = append(ans, stack[i])
    }
    return ans
}
```

