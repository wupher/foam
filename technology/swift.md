# Swift 笔记

## 数组

Swift 中的数组，具有 __值__ 语义。将一个数组赋值给另外一个数组，可以想像成数字赋值给数字，是深拷贝。不会有像 NSArray 那样的对数组内容进行修改。同时 A 修改的内容，也不会传导至 B。虽然是深拷贝，但“写时复制”保证不会在赋值和函数传参时出现大量的复制行为，只是需要的时候才会复制。

Swift 中的数值，不鼓励你通过数组下标进行访问，而推荐使用迭代器和 for。与使用数组相比，first 与 last 以及诸如 filter 这样的操作不会出现下标越界的传统问题。此外，不能访问时会简单返回 nil。

为什么用 map 而不用之前的 for 来对数组进行变换：

1. 代码更简单
2. 声明时可以直接 let，而不用 for 中由于赋值或者修改必须声明 var
3. 最后通过诸如 map 和 filter 等进行组合，可以将复杂变换简单定义
综上所述，swift 中也优先使用闭包而多于函数

数组的切片不是数组，它的类型是 ArraySlice，而不是 Array。ArraySlice 只是数组的一种 __表示方式__，背后的数据仍是数组。如果想将切片转换为数组，将其传递给 Array 的构造函数即可。

## map

map 也是值语义，和数组类似。

## struct

内置 observable 支持对属性进行监视

```swift
struct App {
    var contacts = [String](){
        didSet {
            print("contacts changed, old value was \(oldValue) ")
        }
        willSet{
            print("contacts will change to \(newValue) ")
        }
    }
}
```

## class

对于修改类成员变量的函数，需要添加 `mutating` keyword.

大体上 class 和 struct 的区别是：

- class 可以被继承，struct 不可以
- class 可以被扩展，struct 不可以
- 潜拷贝，深拷贝

using copy func instead copy constructor

class deinit func is a destructor function

like cpp, declare a class as `final` to stop others subclassing it.

## opaque return types

当想把 protocol 用作函数返回值时，swift 有个语法要求。由于 protocol 并不是真正的实体类，而只是 contract 或者说 接口。所以你要使用 `some` 关键字，标识返回的是对应 protocol 的一个实现。

```swift
func getRandomNum() -> some Equatable {
    return Int.random(in: 0...10)
}

func getRandomBool() -> some Equatable{
    return Bool.random()
}
```

> DESIGN GUIDELINE: If you return a new value rather than changing it in place, you should use world endings like `ed` or `ing`, like `reversed()`

## Extensions

- add method to existing type
- add __properties__ to existing type: they must only be computed properties, not stored properties.

### extensions + protocol

#### 1. 扩展可以用于向 protocol 添加默认实现函数。

```Swift
protocol Person{
    var name: String {get}
    func sayHello()
}

extension Person{
    func sayHello(){
        print("Hi, I'm \(name)")
    }
}
```

#### 2. 扩展有时要考虑一点点，嗯，泛型

```Swift
extension Numberic{
    func squared() -> Self{ //注意这里的返回是 self 而非 Int, Double ，因此此时不知道具体类是什么，所以返回 self
        self * self
    }
}
```

> 注意： Self 这里代表`类型`， 而 self 这里代表`实例`。前者是首字母大写。

