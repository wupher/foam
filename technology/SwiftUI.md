
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
