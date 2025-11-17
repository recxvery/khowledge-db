Грубо говоря вариация Two Pointers.

![[Pasted image 20251116233828.png]]
Просто создаём окно с двумя рамками по которому например проверяем являются ли элементы в окне субстрокой?

## **Временная сложность: O(n)**


## Постоянный паттерн к плавающему окну

![[Pasted image 20251117000747.png]]
window_state - объявляем начальную границу окна и дальше сдвигаем его в цикле

**window size**:
```go
wsize := end - begin + 1 // в задаче ниже это просто K
```

### Maximum Average Subarray I 

```go
func findMaxAverage(nums []int, k int) float64 {
	currentSum := 0
	for i := 0; i < k; i++ {
		currentSum += nums[i]
	} //считаем сумму один раз а не бесконечно пересчитываем её

	MaxSum := currentSum

	for i := k; i < len(nums); i++ {
		//[1, 2, 3, 4, 5], k = 3
		currentSum = currentSum + nums[i] - nums[i - k] //вычитаем первый элемент прошлого окна. то есть
		//как это всё выглядит на итерациях
		//1. 1 + 2 + 3 = 6, 2. 6 + 4 (k = 3, nums[3 - 3 = 0), соответственно вычитаем первый элемент
		//почему именно так? Экономит кучу памяти. 
		if currentSum > MaxSum {
			MaxSum = currentSum
		}
	}
	return float64(MaxSum) / float64(k)
}
```
Сложность **O(1)**, 

#### Неправильное решение этой задачи чтобы не ошибаться далее

Я правильно понял паттерн, но слишком увеличил потребление ресурсов, в итоге задача не прошла из-за того что там было миллион элементов в input'e

```go
func findMaxAverage(nums []int, k int) float64 {
    left, right := 0, k
    max := sumArray(nums[left:right]) 
    for right <= len(nums) {           // O(n - k) итераций ≈ O(n)
        if max < sumArray(nums[left:right]) {  // sumArray работает за O(k)
            max = sumArray(nums[left:right])   // Ещё раз O(k)
        }
        left++
        right++
    }
    return float64(max) / float64(k)
}

func sumArray(n []int) int {  // O(k) - проходит по всем k элементам
    s := 0
    for _, v := range n {
        s += v
    }
    return s
}
```
**O(n x k)** - сложность этого алгоритма. И очевидно с  такой задачей лит код послал меня куда подальше. SumArray здесь вызывался по нескольку раз.

- **Было:** O(n × k) — на каждом шаге пересчитываем всю сумму