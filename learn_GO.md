### Go版本切换
使用gvm进行管理[gvm](https://github.com/moovweb/gvm)

### slices和数组区别
- slices: []int{}
- 数组: [...]int{} [3]int{} 

### 闭包问题
```go
list := []Foo{
    {"A"},
    {"B"},
}

list2 := make([]*Foo, len(list))
//错误做法
for index, value := range list {
  list2[index] = &value
}
//正确做法
//for index, value := range list {
//	m := value
//	list2[index] = &m
//}
fmt.Println(list[0], list[1])
fmt.Println(list2[0], list2[1])
```

#### 第一步，看定义
先来读一下关于 range 的文档。
在 Go 规范文档`For statements with range clause`

**还有一点就是记住，在Go里面所有的都是值copy，即使是range里面的是指针数据，那么这个指针也是一个copy，只是指向了同一个地址而已。**

#### range 变量
 我们应该都知道，对于 range 左边的循环变量（上面这个例子里的 i）可以用以下方式来赋值：
 - 等号直接赋值 (=)
 - 短变量申明赋值 (:=)
 
当然也可以什么都不写来完全忽略迭代遍历到的值。

如果使用短变量申明（:=），Go 会在每次**循环**的迭代中重用申明的变量（只在循环内的作用域里有效）

#### range 表达式
range 右边（上述例子里的 a）表达式的结果，可以是以下这些数据类型：

 - 数组
 - 指向数组的指针
 - 切片
 - 字符串
 - map
 - 可以接收传输的 channel, 比如：chan int or chan<- int
 
range 表达式会在开始循环前被 evaluated 一次。但有一个例外情况：**如果对一个数组或者指向数组的指针做 range 并且只用到了数组索引：此时只有 len(a) 被 evaluated。**

#### range只是语法糖
```go
for index, value := range list {
    m := value
    list2[index] = &m
}
// 会转换成
len_temp := len(list)
range_temp := list
for index_temp = 0; index_temp < len_temp; index_temp++ {
    value_temp = range_temp[index_temp]
    index = index_temp
    value = value_temp
    original body
}
```
[Go-Range-内部实现](http://newt0n.github.io/2017/04/06/Go-Range-%E5%86%85%E9%83%A8%E5%AE%9E%E7%8E%B0/)

#### defer关键字

#### function & method
```go
type Point struct {
	X, Y float64
}
//traditional function
func Distance(p, q Point) float64 {
	return math.Hypot(q.X-p.X, q.Y-q.Y)
}
// a method of the Point type
func (p Point) Distance(q Point) float64 {
	return math.Hypot(q.X-p.X, q.Y-q.Y)
}
```
上面的代码里那个附加的参数p,叫做方法的接收器(receiver),早期的面向对象语言留下的遗产将调用 一个方法称为“向一个对象发送消息”。
在Go语言中,我们并不会像其它语言那样用this或者self作为接收器;我们可以任意的选择接收器的名 字。由于接收器的名字经常会被使用到,所以保持其在方法间传递时的一致性和简短性是不错的主意。 这里的建议是可以使用其类型的第一个字母,比如这里使用了Point的首字母p。

#### channel
- `var c chan T` 双向channel
- `var c <-chan T` 只读channel
- `var c chan<- T` 只写channel

##### 坑点
- 这样会deadlock,因为`range`不断等待`c`的输入，但是外部又没有其他驱动，所以会deadlock,要`close(c)`
```go 
var c <-chan String
for f := range c{
    fmt.Println(f)
}
```

