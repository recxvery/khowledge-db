leetcode 

![[Pasted image 20250807040256.png]]

```go
func twoSum(nums []int, target int) []int {
    m := make(map[int]int)
	
    for i, num := range nums {
        _, ok := m[num]
        if ok {
            return []int{i, m[num]}
        }
        m[target - num] = i
    }  
    return nil
}

```
