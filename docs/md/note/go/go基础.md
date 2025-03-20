# 1.1 深拷贝&浅拷贝

```go
original := []int{1, 2, 3}
shallowCopy := original // 浅拷贝

shallowCopy[0] = 10
fmt.Println("Original:", original)       // [10, 2, 3]
fmt.Println("Shallow Copy:", shallowCopy) // [10, 2, 3]
```

```go
original := []int{1, 2, 3}

// 手动实现深拷贝
deepCopy := make([]int, len(original))
copy(deepCopy, original)

deepCopy[0] = 10
fmt.Println("Original:", original)    // [1, 2, 3]
fmt.Println("Deep Copy:", deepCopy)   // [10, 2, 3]
```

# 1.2值传递，还是引用传递、指针传递

1. 值传递
   
   值传递是将变量的副本传递给函数。在函数内部修改该副本，不会影响到原始变量。
   
   ```go
   func modifyValue(x int) {
   	x = 10
   }
   
   func main() {
   	a := 5
   	modifyValue(a)
   	fmt.Println(a) // 输出: 5
   }
   ```

2. 指针传递
   
   通过传递指针（即内存地址），函数可以直接修改原始变量的值。这种传递方式仍然是值传递，因为传递的是指针的副本，但由于指针指向了原始数据，函数可以通过指针操作原始数据。
   
   ```go
   func modifyPointer(x *int) {
       *x = 10
   }
   
   func main() {
       a := 5
       modifyPointer(&a)
       fmt.Println(a) // 输出: 10
   }
   
   
   func modifySlice(x []int) {
   	x[0] = 10
   }
   
   func main() {
   	a := []int{1, 2, 3}
   	modifySlice(a)
   	fmt.Println(a) // 输出: [10,2,3]
   }
   ```

3. 引用传递
   
   Go 语言并不直接支持引用传递，而是通过值传递和指针传递间接实现类似的效果。

# 1.3 数组与切片

结论：数组是值类型，适合处理固定大小的数据集，而切片是引用类型，更适合处理动态数据。

```go
// 数组示例
func modifyArray(arr [3]int) {
    arr[0] = 10
}

a := [3]int{1, 2, 3}
modifyArray(a)
fmt.Println(a) // 输出: [1, 2, 3]，原数组未被修改

// 切片示例
func modifySlice(s []int) {
    s[0] = 10
}

b := []int{1, 2, 3}
modifySlice(b)
fmt.Println(b) // 输出: [10, 2, 3]，原切片被修改
```

# 1.4 切片容量怎么增长

- 切片的容量小于 1024 时，切片的容量会按照 **倍数** 增长，即每次扩容时容量会翻倍。
- 当切片的容量达到或超过 1024 时，容量增长不再是翻倍，而是每次增加约 25%。

这一增长策略既能保证扩容的效率，又能在大容量的情况下避免过度分配内存。

在 Go 的源码中，`runtime.growslice` 函数实现了切片的扩容逻辑。以下是简化后的关键代码片段：

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {    newcap = cap
} else {    if old.len < 1024 {        newcap = doublecap
    } else {        for newcap < cap {            newcap += newcap / 4
        }    }}
```

通过这个代码片段可以看到，Go 在扩容时，首先会检查当前容量是否小于 1024。如果是，则新容量直接翻倍。如果容量已经较大，则每次增长 25%。
