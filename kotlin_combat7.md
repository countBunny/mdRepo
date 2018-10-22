- 使用operator声明`plus`函数之后，就可以使用`+`号来求和。
```kotlin
operator fun plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```
甚至可以通过扩展函数的方式添加操作符函数
```Kotlin
operator fun Char.times(count:Int):String{
    return toString().repeat(count)
}
```
最好不要同时给一个类添加`plus`和`plusAssign`运算，如果类中变量是不可变的，那么应该定义`plus`运算返回一个新值。
- 所有Java中实现了Comparable接口的类，都可以在Kotlin中使用简洁的运算语法，不用再增加扩展函数
```Kotlin
override fun compareTo(other: Person): Int {
    return compareValuesBy(this, other, Person::lastName, Person::firstName)
}
```
`compareValuesBy`函数可以大大简化`compareTo`函数的编写。
- 实现了`Comparable`接口的类，不需要再实现`rangeTo`函数，这个函数返回一个区间，用来检测其他一些元素是否属于它。**rangeTo优先级低于算数优先级**，但是使用时，最好用括号括起来。
- _ClosedRange类实现了iterator方法_，因此闭区间的实例都可以用for(x in ClosedRange)来迭代。
- `componentN`函数可以用来实现解构，将一个复合值赋值给多个变量。此语法被标准库限制在一个对象的**前5个元素**。
- 标准库Map和MutableMap接口定义了getValue和setValue的扩展函数，所以可以直接将Map的实例用作委托属性。