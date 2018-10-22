### Kotlin中成员引用语法可以快速生成lambda表达式。
```Kotlin
val getName = Person::name
    println(people.joinToString(" ", transform = getName))
```
相当于
```Kotlin
val getName = { person: Person -> person.name }
```
还可以调用顶层函数
```Kotlin
private fun salute() = println("Salute!")
fun invokeSalute() = run(::salute)
```
此种情况是省略了类名，所以`::`开头。
```Kotlin
val createPerson = ::Person
```
创建实例的动作被保存成了值，此外还可以引用扩展函数。
<hr>
- Kotlin1.1支持绑定成员引用。
  
```Kotlin
val aliceAge = people[0]::age
```
- 为了提高效率可以把操作变成使用序列，而不是直接使用集合，可以有效减少中间对象的创建。
```Kotlin
  people.asSequence()
        .map(Person::name)
        .filter { it.startsWith("A") }
        .toList()
```
集合数量庞大时，这种方法可以极大的提高效率。变为sequence后，**在执行末端操作之前，所有的中间操作是不会进行的**。即末端操作触发执行了所有的延期计算。此功能类似于java8的翻版，如果目标版本是Java8，可以用流，因其可以在多个CPU上并行执行流操作。
- 用`generateSequence`生成序列，并执行递归操作
```Kotlin
fun File.isInsideHiddenDirectory() =
        generateSequence(this) { it.parentFile }.any { it.isHidden }
```
- 将lambda表达式传给`inline`的函数是不会生成匿名内部类的。大多数的库函数都标记成了inline。
- 当lambda作为函数返回时，要用SAM构造方法把它包装起来。
```Kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}
```
SAM构造方法名称和底层函数式接口的名称一样。
例如android中重用ClickListener
```Kotlin
val listener = View.OnClickListener {
    val text = when (it.id) {
        R.id.console1 -> "First Console"
        R.id.console2 -> "Second Console"
        else -> "Unknown button"
    }
    toast(text)
}
```
- lambda内部没有匿名对象的this，如果需要在监听事件内部取消它自己，不能使用lambda，而应使用实现了该接口的匿名对象。
- 当lambda传给**重载方法**时，需要使用SAM显式指定该lambda使用的构造方法。