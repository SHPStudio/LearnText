# Go语言基础
## 变量声明
`var [名称] [类型]`
eg: `var a, b *int` 表示a,b为int的指针类型

go语言的基本类型：
- bool
- string
- int int8 int16 int32 int64
- uint uint8 uint16 uint32 uint64 uintptr
- byte // uint8的别名 
- rune // int32的别名代表一个Unicode码
- float32 float64
- complex64 complex128

变量声明后，系统自动赋予该类型的零值，int为0，float为0.0 bool为false string为空字符串 指针为nil。

变量命名遵循驼峰命名

变量声明的几种方式：
- var [名字] [类型]
- 批量
```
var (
  a int
  b string
  c []float32
  d func() bool
  e struct {
     x int
  }
)
```
- 简短格式 `名字:=表达式`，不过它有一些限制 
1. 定义变量同时需要显式初始化
2. 不能提供数据类型
3. 只能用在函数内部

## 匿名变量
`_`，任何赋给该特殊标识符的变量的值都会被抛弃，后续代码也无法使用或者进行某些运算。
匿名变量不占用命名空间，不会分配内存。

## 内置类型
内置类型是由语言提供的一组类型。它们分别是数值类型、字符串类型和布尔类型。这些类型本质上是原始的类型。因此，当对这些值进行增加或者删除的时候，会创建一个新值。所以，当把这些类型的值传递给方法时，应该传递一个对应值的副本。同样的string类型本质也是一种很原始的数据值，也应该传递字符串的副本。

## 引用类型
Go的引用类型类型：切片(数组),映射(map)，通道，接口和函数类型。声明上述变量时，创建的变量被称为标头值(指针)。

引用类型的值如果像原始类型一样传入方法或者函数`xxxfunc(ip IP(引用类型))`，就代表是传入了该引用类型值的副本，并不是应用类型本身。

### 引用类型传递
#### 值传递
是否传递引用类型使用指针`*`，这个要看该引用类型本质上是否属于原始类型，属于不可变结构。如果属于不可变结构完全可以当成原始类型来使用。例子如下
```
type Time struct {
    // sec 给出自公元 1 年1 月1 日00:00:00
    // 开始的秒数
    sec int64

    // nsec 指定了一秒内的纳秒偏移，
    // 这个值是非零值，
    // 必须在[0, 999999999]范围内
    nsec int32

    // loc 指定了一个Location，
    // 用于决定该时间对应的当地的分、小时、
    // 天和年的值
    // 只有Time 的零值，其loc 的值是nil
    // 这种情况下，认为处于UTC 时区
    loc *Location
}
```
该时间结构体本身就属于不可更改的对象。所以可以当作原始类型来对待。
```
func Now() Time {
    sec, nsec := now()
    return Time{sec + unixToInternal, nsec, Local}
}
```
Now方法的实现就没有通过使用指针让Time值共享，而是返回了Time值的副本。
```
func (t Time) Add(d Duration) Time {
    t.sec += int64(d / 1e9)
    nsec := int32(t.nsec) + int32(d%1e9)
    if nsec >= 1e9 {
        t.sec++
        nsec -= 1e9
    } else if nsec < 0 {
        t.sec--
        nsec += 1e9
    }
    t.nsec = nsec
    return t
}
```
上面代码的Add方法就展示了如何将Time作为原始类型的例子。这个方法使用值接收者，并返回了一个新的Time值。因为之前说过没有使用指针的话就表示传入的是Time值的副本，方法内操作的就是Time的值副本，返回的也是副本。
所以该方法返回的值可以用于替代原本的time的值，也可以用一个新的变量来保存，具体是由调用者决定。

#### 指针传递
大多数情况，结构类型的本质并不是不可变的，这种情况下，对这个类型做增加和修改删除登操作应该改这个值的本身，而不是副本。这个时候就需要传递指针来共享这个值。例如文件操作。
```
// File 表示一个打开的文件描述符
type File struct {
    *file
}

// file 是*File 的实际表示
// 额外的一层结构保证没有哪个os 的客户端
// 能够覆盖这些数据。如果覆盖这些数据，
// 可能在变量终结时关闭错误的文件描述符
type file struct {
    fd int
    name string
    dirinfo *dirInfo // 除了目录结构，此字段为nil
    nepipe int32     // Write 操作时遇到连续EPIPE 的次数
}
```

```
func Open(name string) (file *File, err error) {
    return OpenFile(name, O_RDONLY, 0)
}
```

