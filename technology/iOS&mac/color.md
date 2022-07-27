# Color 不止是颜色

来自邮件列表：[链接](https://www.ethanhuang13.com/p/swiftui-4-not-just-color)

## 排版

那如果我們把多個 Color 擺在一起呢？很簡單，它們會自動平均分配空間。利用这个特性，很多需要等分的场景就可以使用 Color 来轻松解决，舉例來說，當你需要自己做固定數量的 tab bar。

于是，在背景上使用

```Swift
Color.clear.overlay(yourUIComponent)
```

即可让内容等分。如果是 Button，那么除了排版以外，还要考虑点按范围大小，那这里有个变化：

```Swift
        Button(
                action: {},
                label: { Color.clear.overlay(Text("first") )}
        )
```

通过这种方法即可实现热区大小也被 color 平均分布。

## overylay 与 background

其实两者一样可以用于排版， 只是 Z 轴插入的排序不同，一个在上，一个在下。

foregroundColor 与 background 不同，前者只上色，而不具备排版特性。

