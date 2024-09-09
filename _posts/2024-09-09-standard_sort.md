---
title: go标准库-sort 
date: 2024-09-09 23:46:12 +0800
categories: [go]
tags: [go_standard_library]     # TAG names should always be lowercase
typora-root-url: ../../chp0304.github.io/assets/pictures
---

##### 该包用于给slice或者用户自定义的集合进行排序

### 排序底层算法

go的底层排序算法使用的是pattern-defeat quicksort(pdqSort)算法，PDQSort（Pattern-defeating quicksort）是一种现代的排序算法，它结合了快速排序（Quicksort）、插入排序（Insertion Sort）和堆排序（Heapsort）的优点，并通过模式识别和优化来避免最坏情况的性能退化。

[go source code](https://cs.opensource.google/go/go/+/refs/tags/go1.23.1:src/sort/zsortinterface.go)

[pdqSort](https://www.easemob.com/news/8361)

### Types（类型）

#### ```sort.Interface```

​	sort.Interface是Sort包中的一个接口，该接口定义了3个方法：

- ```Len() int``` : 返回切片的长度
- ```Less(i,j int) bool```:  用于判断切片中两个值的大小
- ```Swap(i,j int)```: 交换索引i和j处的元素

实现该接口的类型可以使用该包的```Sort```函数进行排序

```go
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int

	// Less reports whether the element with index i
	// must sort before the element with index j.
	//
	// If both Less(i, j) and Less(j, i) are false,
	// then the elements at index i and j are considered equal.
	// Sort may place equal elements in any order in the final result,
	// while Stable preserves the original input order of equal elements.
	//
	// Less must describe a transitive ordering:
	//  - if both Less(i, j) and Less(j, k) are true, then Less(i, k) must be true as well.
	//  - if both Less(i, j) and Less(j, k) are false, then Less(i, k) must be false as well.
	//
	// Note that floating-point comparison (the < operator on float32 or float64 values)
	// is not a transitive ordering when not-a-number (NaN) values are involved.
	// See Float64Slice.Less for a correct implementation for floating-point values.
	Less(i, j int) bool

	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

#### sort包内置实现sort.Interface接口的类型

- ```Float64Slice```

- ```IntSlice```

- ```StringSlice```

  以上三个类型均实现下面的函数：

  - Len()
  - Less(i,j int) bool
  - Search(x) 查找数组中元素，若x在集合中，则返回对应的index，否则返回x应该在集合中的位置
  - Sort() 对Slice进行排序
  - Swap(i,j)

##### 	例子

```go
func SortSlice() {
	arr := []int{1, 2, 4, 5, 61, 13, 2} // 定义一个整形slice
	slice := sort.IntSlice(arr)         // 类型转换
	slice.Sort()                        // 对slice进行排序
	fmt.Println(arr)
	ans := slice.Search(12) // 查找数组中元素，若x在集合中，则返回对应的index，否则返回x应该在集合中的位置
	fmt.Println(ans)
	ans = slice.Search(2) // 查找数组中元素，若x在集合中，则返回对应的index，否则返回x应该在集合中的位置
	fmt.Println(ans)
}
```

### Functions

- Find函数

``` go
func Find(n int, cmp func(int) int) (i int, found bool)
// 基于binary search 查找在集合[0,n)范围内满足cmp(i) <= 0 最小的索引
// 如果查找失败，说明没有满足cmp条件的index，则返回i=n，此时found = false
// 如果查找成功，则i<n && cmp(i) <= 0,此时found = true
// cmp函数用于比较待查找的值target与集合x的中元素x[i]的大小,例如:
// target < x[i] return <0
// target == [i] return =0
// target > x[i] return > 0
// Example
func TestFind(t *testing.T) {
	arr := []int{11, 23, 1, 4, 13}
	fmt.Println(arr)
	slice := sort.IntSlice(arr)
	slice.Sort()
	fmt.Println(arr)
	target := 22
	ans, found := sort.Find(len(arr), func(i int) int {
		return target - arr[i]
	})
	fmt.Println(found)
	fmt.Println(ans)
}
```

- Float64s函数

```go
func Float64s(x []float64)
// 将float类型的集合升序排序
// Example
func TestFloat64s(t *testing.T) {
	arr := []float64{1, 2, 34, 4, 2}
	fmt.Println(arr)
	sort.Float64s(arr)
	fmt.Println(arr)
}
```

- Float64sAreSorted函数

```go
func Float64sAreSorted(x []float64) bool
// 判断给定的集合是否已经排序
func TestFloat64sAreSorted(t *testing.T) {
	arr := []float64{1, 2, 34, 4, 2}
	fmt.Println(sort.Float64sAreSorted(arr)) // return false
	sort.Float64s(arr)
	fmt.Println(sort.Float64sAreSorted(arr)) // return true
}
```

- Ints函数 (同float64s)
- IntsAreSorted函数 (同Float64sAreSorted函数)

- IsSorted函数

```go
func IsSorted(data Interface) bool
// 判断sort.Interface类型的集合是否有序
// Example
func TestIsSorted(t *testing.T) {
	arr := []int{1, 2, 34, 4, 2}
	slice := sort.IntSlice(arr)
	ans := sort.IsSorted(slice)
	fmt.Println(ans)
	slice.Sort()
	ans = sort.IsSorted(slice)
	fmt.Println(ans)
}
```

- Search函数

```go
func Search(n int, f func(int) bool) int
// Search函数基于binary search 查找[0,n)范围内满足f(i) == true 的最小下标元素
// 假设在范围[0, n)内，若f(i)为真，则f(i+1)也为真。
// 也就是说，搜索要求函数f对于输入范围[0, n)的某个（可能为空）前缀部分返回假值，然后对其余（可能为空）部分返回真值；搜索将返回第一个使结果为真的索引。
// 如果这样的索引不存在，搜索将返回n。（注意：“未找到”的返回值不是-1，例如strings.Index中的情况。）搜索仅对区间[0, n)内的i调用f(i)。
// Example 
func TestSearch(t *testing.T) {
	arr := []int{1, 2, 34, 4, 2}
	target := 3
	sort.Ints(arr)
	fmt.Println(arr)
	ans := sort.Search(len(arr), func(i int) bool { // 如果存在满足条件的元素，则返回对应元素的下标；否则，return n
		return target <= arr[i]
	})
	if ans < len(arr) && ans == target { // ans = 3
		fmt.Println(ans)
	} else {
		fmt.Println(ans)
	}
}
```

- SearchFloat64s函数

```
func SearchFloat64s(a []float64, x float64) int
// 查找集合中固定值所在的索引
// 若x存在于集合中，则返回对应的下标 反之 返回应该插入的位置
// 集合必须是升序排列
// Example 
func TestSearchFloat64s(t *testing.T) {
	arr := []float64{1, 2, 34, 4, 2}
	slice := sort.Float64Slice(arr)
	slice.Sort()
	ans := sort.SearchFloat64s(arr, 5) // ans = 4
	fmt.Println(ans)
}
```

- SearchInts函数（同SearchFloat64）
- SearchStrings函数（同SearchFloat64）
- Slice函数

```go
func Slice(x any, less func(i, j int) bool)
// Slice函数按照less函数对集合进行排序
// 注意：该方法不是稳定排序
// Example
func TestSlice(t *testing.T) {
	arr := []float64{1, 2, 34, 4, 2}
	sort.Slice(arr, func(i, j int) bool {
		return arr[i] < arr[j]
	})
	fmt.Println(arr)
}
```

- SliceIsSorted函数

```go
func SliceIsSorted(x any, less func(i, j int) bool) bool
// 判断按照给定less排序方法 集合是否已经有序
```

- SliceStable函数

```
func SliceStable(x any, less func(i, j int) bool)
// 稳定的排序算法
```

- Sort函数

```
func Sort(data Interface)
// 对实现的sort.Interface的类型集合进行排序
// Example 
type myArr []int

func (arr myArr) Len() int {
	return len(arr)
}
func (arr myArr) Swap(i, j int) {
	arr[i], arr[j] = arr[j], arr[i]
}
func (arr myArr) Less(i, j int) bool {
	return arr[i] < arr[j]
}
func TestSort(t *testing.T) {
	arr := []int{1, 2, 34, 4, 2}
	sort.Sort(myArr(arr))
	fmt.Println(arr)
}
```

- Stable函数

```go
func Stable(data Interface)
// 采用稳定的排序算法
```

- Strings函数

```
func Strings(x []string)
// 对字符串进行排序
func TestStrings(t *testing.T) {
	arr := []string{"b", "a"}
	sort.Strings(arr)
	fmt.Println(arr)
	fmt.Println(sort.StringsAreSorted(arr))
}
```

- StringsAreSorted

```
func StringsAreSorted(x []string) bool
// 判断字符串集合是否有序
```