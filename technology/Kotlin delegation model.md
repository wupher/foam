# Kotlin delegation model

Kotlin 对于代理模式的语法糖。使用 Kotlin 可以快速为已知类通过代理添加各种方便的功能。

[原文](https://blog.frankel.ch/beautify-third-party-api-kotlin/)

传统的 Java 语言中，如果要对资源进行管理，使用，需要使用传统的 try-catch 模式来保证资源关闭。

```java
Component component;
try {
    component = new Component();
    // Use component
} finally {
    component.close();
}

```

Java7 语言中提供 try-with-resouce 模式来简化这个日常使用模式。

```java
try (Component component = new Component()) {
    // Use component
}   
```

但要使用此模式，Component 必须实现 `AutoCloseable` 接口。

Kotlin 语言有类似 try-with-resource 的语法糖，但要求实现 `Closeable` 接口。

```kotlin
Component().use {
  // Use component as it
}       
```

传统情况下，要为已有的类添加诸如 `close` 方法, 你需求搞个代理类。通过代理类添加 `close` 方法，然后通过代理支持原有的方法。

```java
interface class Component {
    void a();
    void b();
    void c();
}

public class CloseableComponent extends Component implements Closeable {

    private final Component component;

    public CloseableComponent(Component component) {
        this.component = component;
    }

    void a() { component.a(); }
    void b() { component.b(); }
    void c() { component.c(); }
    public void close() {}
}
```

而在 Kotlin 语言，可以通过 `by` 关键字来代理。

```kotlin
class CloseableComponent(component: Component) : Component by component,
                                                 Closeable {                  
    override fun close() {}
}
```

`by` 告诉编译器，Component 原有的方法直接使用 componse 来进行代理。
