# 100 days of SwiftUI Project 7

## Why @State only works on structs

Using `@State` property transfer data in a UIView is easy. That all works: SwiftUI is smart enough to understand that one object contains all our data, and will update the UI when either value changes.

But, if we want share data between two UIViews, we need to use class rather than struct to store those data, let them point to same instance.

using `@StateObject` to monit object changes.

```Swift
class User{
    @Published var name = "John"
}
```

@Published is more or less half of @State: it tells Swift that whenever either of those two properties changes, it should send an announcement out to any SwiftUI views that are watching that they should reload.

How do those views know which classes might send out these notifications? That’s another property wrapper, @StateObject, which is the other half of @State – it tells SwiftUI that we’re creating a new class instance that should be watched for any change announcements.

```Swift
@StateObject var user = User()
```

the @StateObject property wrapper can only be used on types that conform to the ObservableObject protocol. This protocol has no requirements, and really all it means is “we want other things to be able to monitor this for changes.”

```Swift
class User: ObservableObject {
    @Published var firstName = "Bilbo"
    @Published var lastName = "Baggins"
}
```

