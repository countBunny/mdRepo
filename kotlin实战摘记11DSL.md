- dsl由多个方法调用创建出的结构功能，并提供了扩展它们的能力。DSL在Kotlin中是完全静态类型的。能比通用程序语言中的等价代码更简洁地表达特定领域的操作。
- 内部DSL不是完全独立的语言，而是使用主要语言的特定方式，同时保留具有独立语法。

##### invoke约定让对象像函数般，可调用

用operator修饰invoke可以让声明的类的对象通过括号，直接调用对象的该invoke函数。

```kotlin
val bavarianGreeter = Greeter("Servus")
bavarianGreeter("Dmitry")

class Greeter(val greeting: String){
    operator fun invoke(name:String){
        println("$greeting, $name")
    }
}
```

会被编译为`bavarianGreeter.invoke("Dmitry")`，可以定义任意数量的参数和任意的返回类型。

- invoke支持安全调用，`lambda?.invoke()`，当lambda被作为函数调用时，会被转换成一次invoke方法调用。

