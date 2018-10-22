- lambda的返回值可标记为可空类型
```Kotlin
var canReturnNull: (Int, Int) -> Int? = { a, b -> null }
```
可空的函数声明需要添加括号将函数声明裹起来
```Kotlin
var funOrNull: ((Int, Int) -> Int)? = null
```
- Java8可以直接调用Kotlin中使用了函数类型的函数，而旧版本需要传递实现了函数接口的匿名类实例
```Kotlin
//invoke lambda
LambdaPremierKt.twoAndThree(new Function2<Integer, Integer, Integer>() {
    @Override
    public Integer invoke(Integer a, Integer b) {
        System.out.println("number a=" + a + " number b=" + b);
        return a + b;
    }
});
```
在Java中创建返回是Unit的函数接口时，返回处需显式返回真实的Unit实例。
```Kotlin
List<String> strings = new ArrayList<>();
strings.add("42");
CollectionsKt.forEach(strings, new Function1<String, Unit>() {
    @Override
    public Unit invoke(String s) {
        System.out.println(s);
        return Unit.INSTANCE;
    }
});
```
- 可空函数参数的调用，可以判空后通过安全检查，也可以用`invoke`函数安全调用。
```Kotlin
fun foo(callback: (() -> Unit)?) {
    callback?.invoke()
}
```
- 通过扩展函数，有效提取公共的lambda函数进行封装
```Kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
        filter(predicate).map(SiteVisit::duration).average()
println(log.averageDurationFor({ it.os in setOf(OS.ANDROID, OS.IOS) }))
println(log.averageDurationFor({ it.os == OS.IOS && it.path == "/signup" }))
```
函数类型和lambda表达式简化设计模式，如策略模式，lambda相当于有输入输出的接口，传递不同的lambda表达式作为不同的策略。
- 当一个函数被声明为`inline`时，它的函数体是内联的--换句话说，函数体会被直接替换到函数被调用的地方。参数如果被直接调用或者作为参数传递给另外一个inline函数，则可以被内联。一个lambda可能会包含很多代码或者以**不允许内联的方式使用**，（即接收参数后赋值给变量的lambda参数是不能内联进方法的）。接收这样的参数时，可以用`noinline`修饰符来标记它。
```Kotlin
inline  fun  foo(inlined:  ()  ->  Unit,  noinline  notinlined:  ()  ->  Unit)  {
//...
}
```
集合作为序列处理时，传入的lambda函数不会被内联，还是被赋值给对象，末端操作会使调用链被执行，所以只有在处理大量数据的集合时使用'asSequence'才有用。
如果要内联的函数很大，将它的字节码拷贝到每一个调用点会极大的增加字节码的长度，此时应该将与lambda参数无关的代码抽取到一个独立的非内联函数中。
- lambda作为参数传入时，如果是内联函数，则可以从非局部返回，如果不是内联函数则只能从局部返回。非局部返回可以通过标签转换成局部返回。
```Kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{
        if (it.firstName == "Alice") return@label
    }
    println("Alice might be somewhere")
}
```
lambda作为参数的函数名可以直接作为label来用。
```Kotlin
people.forEach {
    if (it.firstName == "Alice") return@forEach
}
```
一个lambda表达式的标签只能添加一个，添加后**方法名标签会失效**。
- this在lambda中指向作用域内最近的隐式接收者，可以设置外层的显式标签，访问外层this对象。