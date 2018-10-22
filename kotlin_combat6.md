- 检查过类型并拒绝了null值，就可以相应的使用它。
```kotlin
class Person(val name: String, val company: Company?) {
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false
        return otherPerson.name == name && otherPerson.company == company
    }
```
- 不要在同一行写多个非空断言。否则会搞不清错误发生在哪个表达式。
```Kotlin
//不要写这样的代码
person.company!!.address!!.country
```
- 用`lateinit`来延迟初始化，并且修饰符只能用`var`。
```Kotlin
class MyTest {
    /**
     * 延迟初始化属性都是var
     */
    private lateinit var myService: MyService
    @Before
    fun setUp(){
        myService = MyService()
    }

    @Test fun testAction(){
        Assert.assertEquals("foo", myService.performAction())
    }
```
- 可空类型扩展函数是不需要安全检查就可以调用的。在可空类型扩展函数中，this也可能为null。
- 当Kotlin不能识别出Java类型的可空性信息时，Kotlin就会把它当作**平台类型**来处理，此时须由你判断这个变量是否为空。
- 一种Java集合接口在Kotlin中都有两种表示：一种是只读的，一种是可变的，但这种隔离**只是接口隔离**，只在Kotlin起作用，Java中可以照常对只读列表进行增删操作，甚至会对可空性造成影响。
- 通过`toTypedArray`可以将list变为Array。
```Kotlin
val strings = listOf("a", "b", "c")
        println("%s/%s/%s".format(*strings.toTypedArray()))
```
- Kotlin提供了基本数据类型的专用数组，而用Array创建的都是包装类，IntArray可以被编译成int[]的基本类型数组，用size参数初始化数组，可用`intArrayOf`函数创建变长参数的数组，可以lambda生成有规律的数组。
```kotlin
val squares = IntArray(5){ i -> ( i + 1 ) * ( i + 1 ) }
```
也可以用`toIntArray`函数转换包装类型的数组到基本类型数组。