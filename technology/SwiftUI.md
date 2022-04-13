
# SwifUI 笔记

## 10 children

SwiftUI 有个有限的限制，每个子元素的数量不能超过 10 个。比如，你在 form 里面加入 Text，加到 10 个，没问题，加到 11 个？会直接报错。

```Swift
Form { //Xcode 会报错，因为你添加了 11 个元素
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
    Text("Hello, world!")
}
```

非要加？必须通过 Group 来分别封装起来。

```Swift
Form {
    Group {
        Text("Hello, world!")
        Text("Hello, world!")
        Text("Hello, world!")
        Text("Hello, world!")
        Text("Hello, world!")
        Text("Hello, world!")
    }

    Group {
        Text("Hello, world!")
        Text("Hello, world!")
        Text("Hello, world!")
        Text("Hello, world!")
        Text("Hello, world!")
    }
}
```

这要是动态的……，嗯，就有趣了。

## $ binding

> In Swift, we mark these two-way bindings with a special symbol so they stand out: we write a dollar sign before them. This tells Swift that it should read the value of the property but also write it back as any changes happen.

## Loop View

`ForEach` 居然是 SwiftUI 而非 Swift 中的内容。此外, ForEach 没有 10 Item 的限制。

> So, when we’re using ForEach to create many views and SwiftUI asks us what identifier makes each item in our string array unique, our answer is \.self, which means “the strings themselves are unique.” This does of course mean that if you added duplicate strings to the students array you might hit problems, but here it’s just fine.

## Log output

```Swift
let _ = print(Locale.current.currencyCode ?? "NONE")
```

## Button

### ButtonRole

指明按钮的角色，当前默认就两种 cancel, destructive 一般用于取消，删除这类按钮。一般就是加个默认新式。

### Label

Button 中常用的 Label struct 是一个非常值得学习的封装。 

```Swift
struct Label<Title, Icon> : View where Title : View, Icon : View 
```

这个 where 关键字的使用可以参考：[this](https://www.avanderlee.com/swift/where-using-swift/)　AND [this](https://www.appypie.com/swift-where-how-to)

