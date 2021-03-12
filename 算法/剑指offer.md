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

* #### [替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

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

* #### [从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

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

* #### [重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */

// 前序遍历第一个元素是root，确定root后，到中序遍历中，可以区分出左右子树
// 在中序遍历中，继续递归进行区分操作
// 这时的root，要根据上一次中序遍历的结果，上一次的root+1就是下一次递归的左子树root，root+1+左子树的长度就是右子树的root
func buildTree(preorder []int, inorder []int) *TreeNode {

    // 先用map记录下前序遍历的root位置
    rootMap := make(map[int]int)
    for i, v := range inorder {
        rootMap[v] = i
    }

    // 递归递归查找子树
    return recursive(preorder, 0, 0, len(preorder)-1, rootMap)
}

func recursive(preorder []int, root, left, right int, rootMap map[int]int) *TreeNode {
    if left > right {
        return nil
    }

    // 根节点中序遍历中的位置
    // root是前序遍历中根节点的位置
    rootIndex := rootMap[preorder[root]]
    
    // 构建树
    node := &TreeNode{preorder[root], nil, nil}

    // 递归左子树，前序遍历的根节点+1，得到下一次递归的左子树根节点
    node.Left = recursive(preorder, root+1, left, rootIndex-1, rootMap)

    // 递归右子树
    // 左子树的长度为rootIndex-left，那么右子树root位置在rootIndex-left+root+1
    node.Right = recursive(preorder, rootIndex-left+root+1, rootIndex+1, right, rootMap)

    return node
}
```

* #### [用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

```go
// 左右两个栈
type CQueue struct {
    left []int
    right []int
}


func Constructor() CQueue {
    return CQueue{}
}

// 入栈时，永远进左栈
func (this *CQueue) AppendTail(value int)  {
    this.left = append(this.left, value)
}

// 出栈时，弹出右栈顶元素
// 如果右栈没有元素，则把左栈的元素全部弹出，压入右栈
func (this *CQueue) DeleteHead() int {
    if len(this.right) != 0 {
        r := this.right[len(this.right)-1]
        this.right = this.right[0:len(this.right)-1]
        return r
    } else {
        if len(this.left) == 0 {
            return -1
        }
        for len(this.left) != 0 {
            this.right = append(this.right, this.left[len(this.left)-1])
            this.left = this.left[0:len(this.left)-1]
        }
        r := this.right[len(this.right)-1]
        this.right = this.right[0:len(this.right)-1]
        return r
    }
}
```

* #### [斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

```go
// 递归解法超时，采用动态规划解法
func fib(n int) int {
    dp := []int{0, 1}
    for i:=2; i<=n; i++ {
        dp = append(dp, dp[i-1] % 1000000007 + dp[i-2] % 1000000007)
    }
    return dp[n] % 1000000007
}
```

* #### [青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

```go
// 与斐波那契数列类似，动态规划解法
func numWays(n int) int {
    dp := []int{1, 1}
    for i:=2; i<=n; i++ {
        dp = append(dp, dp[i-1] % 1000000007 + dp[i-2] % 1000000007)
    }
    return dp[n] % 1000000007
}
```

* #### [旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

```go
// 二分查找变种
func minArray(numbers []int) int {
    // if len(numbers) == 0 {
    //     return -1
    // }
    // if len(numbers) == 1 {
    //     return numbers[0]
    // }
    left, right := 0, len(numbers)-1
    for left < right {
        mid := left + (right - left) / 2
        if numbers[mid] > numbers[right] {
            // 中位数大于最右的数，说明最小元素在右边
            left = mid + 1
        } else if numbers[mid] < numbers[right] {
            // 中位数小于最右的数，说明右半边的元素有序，那么最小数字在左半边
            // 这里不能确定mid是不是最小的，所以右指针只移动到mid
            right = mid
        } else {
            // 中位数和最右元素相等，判断不了,但可以确定右边的元素全部相同，只是不知道是不是最小元素，那么将right左移一位继续查找
            right--
        }
    }
    return numbers[left]
}
```

* #### [矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

```go
// 深度优先搜索，遍历矩阵每个点，依次进行上下左右深度优先搜索，找到一个结果就退出
// 时间复杂度，每个点都要走3个方向，来的方向不用走，格子全部都要走到，O(3^len(word)*M*N)
// 空间复杂度O(K)，栈深度最多不超过K
func exist(board [][]byte, word string) bool {
    if len(board) == 0 {
        return false
    }

    w := []byte(word)
    for row:=0; row<len(board); row++ {
        for column:=0; column<len(board[0]);column++ {
            if dfs(&board, &w, 0, column, row) {
                return true
            }
        }
    }
    return false
}

func dfs(board *[][]byte, word *[]byte, index, column, row int) bool {

    // 行列越界或者当前的值跟word字母不一样
    if row < 0 || row >= len(*board) || column < 0 || column >= len((*board)[0]) ||(*word)[index] != (*board)[row][column] {
        return false
    }

    // 索引到了最后，说明之前的推出逻辑全部没走到，也就是之前的字母全部找到了，找到了结果
    if index == len(*word)-1 {
        return true
    }

    // 这里将当前位置设置成空的byte，表示这里被访问过了，防止下面递归的时候往回走又找到这个元素
    var tmp byte
    (*board)[row][column] = tmp
    rs := dfs(board, word, index+1, column+1, row) || dfs(board, word, index+1, column, row+1) ||  dfs(board, word, index+1, column-1, row) || dfs(board, word, index+1, column, row-1)

    // 还原矩阵
    (*board)[row][column] = (*word)[index]
    return rs
}
```

* #### [剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

```go
// 拆出来的3越多越大，如果拆完最后只剩1，那么少拆一个3，最后一个数字变为4
func cuttingRope(n int) int {

    // 小于等于3的时候，直接返回n-1
    if n <= 3 {
        return n-1
    }


    // 计算能拆出多少个3
    b := n / 3

    // 最后剩下1，那么少拆一个3
    if n % 3 == 1 {
        return int(math.Pow(float64(3), float64(b-1)) * float64(4))
    } else if n % 3 == 0 {
        return int(math.Pow(float64(3), float64(b)))
    } else {
        return int(math.Pow(float64(3), float64(b)) * float64(2))
    }
}
```

* #### [剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)

```go
// 拆出来的3越多越大，如果拆完最后只剩1，那么少拆一个3，最后一个数字变为4
// 这里需要自己实现pow函数
func cuttingRope(n int) int {

    // 小于等于3的时候，直接返回n-1
    if n <= 3 {
        return n-1
    }


    // 计算能拆出多少个3
    b := n / 3

    // 最后剩下1，那么少拆一个3
    if n % 3 == 1 {
        return pow(3, b-1) * 4 % 1000000007
    } else if n % 3 == 0 {
        return pow(3, b) % 1000000007
    } else {
        return pow(3, b) * 2 % 1000000007
    }
}

func pow(x, y int) int {
    r := 1
    for i:=0; i<y; i++ {
        r = (r * x) % 1000000007
    }
    return r
}
```

* #### [二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

```go
// 位运算
// &，都为1则为1， | 有一个为1则为1， ^不同则为1
func hammingWeight(num uint32) int {
    var r int
    for num>0 {
        if num & 1 == 1 {
            r++
        }
        // 右移一位
        num>>=1
    }
    return r
}
```

* #### [打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

```go
// 最大的范围应该是1～pow(10, n)
func printNumbers(n int) []int {
    var r []int
    for i:=1; i<int(math.Pow(10.0, float64(n))); i++ {
        r = append(r, i)
    }
    return r
}
```

* #### [删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func deleteNode(head *ListNode, val int) *ListNode {

    pivot := new(ListNode)
    pivot.Next = head
    cur := pivot
    for cur.Next != nil {
        if cur.Next.Val == val {
            cur.Next = cur.Next.Next
            return pivot.Next
        }
        cur = cur.Next
    }
    return pivot.Next
}
```

* #### [调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

```go
// 双指针，左右各一个，如果左边碰到了偶数，那么右边指针开始向左走，找到一个奇数，然后交换
func exchange(nums []int) []int {
    left, right := 0, len(nums)-1
    for left < right {
        if nums[left] % 2 == 0 {
            for left < right && nums[right] % 2 == 0 {
                right--
            }
            nums[left], nums[right] = nums[right], nums[left]
        }
        left++
    }
    return nums
}
```

* #### [链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */

// 双指针，前面的指针提往前多走k步，然后同步往后走，当前指针到达链表尾部的时候，后指针的节点即是结果
func getKthFromEnd(head *ListNode, k int) *ListNode {
    pivot := new(ListNode)
    pivot.Next = head
    cur := pivot

    for i:=0; i<k; i++ {
        cur = cur.Next
    }

    for cur != nil {
        pivot = pivot.Next
        cur = cur.Next
    }
    return pivot
    
}
```

* #### [反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    cur := head
    for cur != nil {
        // 存一下当前节点的下一个值
        tmp := cur.Next 

        // 两步交换，反转两个节点
        cur.Next = prev
        prev = cur

        // 指针向后移动
        cur = tmp
    }
    return prev
}
```

