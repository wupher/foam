
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

### View Modifiers

There are 4 built-in shapes in swift:

- retangle
- rounded retangle
- circle
- capsule

#### [Order matters](https://www.hackingwithswift.com/books/ios-swiftui/why-modifier-order-matters)

Whenever we apply a modifier to a SwiftUI view, we actually create a new view with that change applied – we don’t just modify the existing view in place. If you think about it, this behavior makes sense: our views only hold the exact properties we give them, so if we set the background color or font size there is no place to store that data. 

peek here:

```Swift
Button("Hello, world!") {
    print(type(of: self.body))
}    
.background(.red)
.frame(width: 200, height: 200)
```

To read what the type is, start from the innermost type and work your way out:
The innermost type is ModifiedContent`<Button<Text>`, _BackgroundStyleModifier`<Color>`: our button has some text with a background color applied.
    Around that we have ModifiedContent`<…, _FrameLayout>`, which takes our first view (button + background color) and gives it a larger frame.

The best way to think about it for now is to imagine that SwiftUI renders your view after every single modifier. So, as soon as you say .background(.red) it colors the background in red, regardless of what frame you give it. If you then later expand the frame, it won’t magically redraw the background – that was already applied.

> Of course, this isn’t actually how SwiftUI works, because if it did it would be a performance nightmare, but it’s a neat mental shortcut to use while you’re learning.

An important side effect of using modifiers is that we can apply the same effect multiple times: each one simply adds to whatever was there before.

```Swift
Text("Hello, world!")
    .padding()
    .background(.red)
    .padding()
    .background(.blue)
    .padding()
    .background(.green)
    .padding()
    .background(.yellow)
```

#### Environment Modifiers

Many modifiers can be applied to containers, which allows us to apply the same modifier to many views at the same time. 

```Swift
VStack {
    Text("Gryffindor")
    Text("Hufflepuff")
    Text("Ravenclaw")
    Text("Slytherin")
}
.font(.title)
```

From a coding perspective these modifiers are used exactly the same way as regular modifiers. However, they behave subtly differently because if any of those child views override the same modifier, the child’s version takes priority.

```Swift
VStack {
    Text("Gryffindor")
        .font(.largeTitle)
    Text("Hufflepuff")
    Text("Ravenclaw")
    Text("Slytherin")
}
.font(.title)
```

不是所有的 modifier 都是 ENV Modifier，把 font 换成 blur 就会是完全不同的行为：blur() 是 regular modifier.
想知道 modifier 是哪种，只能读文档，或者做试验。

#### Views as Property

using `@ViewBuilder` attribute.

```Swift
@ViewBuilder var spells: some View {
    Text("Lumos")
    Text("Obliviate")
}
```

用于封装视图类型

#### 定义自己的 modifier

继承`ViewModifier` 并实现 `body` 方法。

```Swift
struct Title: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.largeTitle)
            .foregroundColor(.white)
            .padding()
            .background(.blue)
            .clipShape(RoundedRectangle(cornerRadius: 10))
    }
}
```

使用起来很麻烦，得这样用:

```Swift
Text("Hello World")
    .modifier(Title())
```

用起来很 麻烦我们可以使用`extension` 来定义自己的 modifier。

```Swift
extension View {
    func titleStyle() -> some View {
        modifier(Title())
    }
}
```

这下总算能看到和官方相似代码：

```Swift
Text("Hello World")
    .titleStyle()
```

除了扩展现有，我们还能直接修改原有的 view

```Swift
struct Watermark: ViewModifier {
    var text: String

    func body(content: Content) -> some View {
        ZStack(alignment: .bottomTrailing) {
            content
            Text(text)
                .font(.caption)
                .foregroundColor(.white)
                .padding(5)
                .background(.black)
        }
    }
}

extension View {
    func watermarked(with text: String) -> some View {
        modifier(Watermark(text: text))
    }
}
```

水印的封装与使用

```Swift
Color.blue
    .frame(width: 300, height: 200)
    .watermarked(with: "Hacking with Swift")
```

#### 自定义自己的 View 容器: [link](https://www.hackingwithswift.com/books/ios-swiftui/custom-containers)

## Environment

我以为 Environment 是个保存环境变更的地方。其实完全不是，它定义了某些默认操作，比如 dismiss，通过它来绑定调用。
