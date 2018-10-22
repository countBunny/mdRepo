### 编译 kotlin 代码

通过如下命令编译成 jar 文件：

```bash
kotlinc <source file or directory> -include-runtime -d <jar name>
```

再使用 java 命令来运行 jar 文件：

```bash
java -jar <jar name>
```

### for 循环中用展开语法增加下标

```kotlin
val list = arrayListOf("10","11","1001")
for((index, element) in list.withIndex()){
    println("$index: $element")
}
```

#### kotlin 的 javaClass 等价于 Java 的 getClass()

#### Java 中没有默认值概念，需要添加@JvmOverloads 在编译时生成 Java 重载函数，从而使 java 代码可以调用 kotlin 的默认值函数。

- kotlin 中可以在函数中嵌套局部函数，并且扩展函数也可以作为局部函数嵌套在函数中。
  ```kotlin
    class User(val id: Int, val name: String, val address: String)
    fun saveUser(user: User) {
    user.validateBeforeSave()
    }

    fun User.validateBeforeSave() {
    fun validate(value:String, fieldName: String){
    if (value.isEmpty()) {
    throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
    }
    }
    validate(name, "Name")
    validate(address, "Address")
    }
  ```
- kotlin嵌套的类默认并不是内部类，没有包含对其外部类的隐式引用。
- 一个类同时实现含有同一默认实现方法的两个接口，它不继承任何默认实现，需要重写该签名相同的方法的具体实现。
  ```kotlin
  class Button:Clickable, Focusable{
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }

    override fun click() = LogUtil.d(TAG, "I was clicked")
    }
  ```
  Java中调用具体父类用`Clickable.super.showOff()`而在kotlin中用`super<Clickable>.showOff()`
- 一个类如果没有特别需要在子类中被重写，该方法应该显式地标注为`final`。
  ```kotlin
  open class RichButton: Clickable{
    fun disable(){}
    
    open fun animate(){}

    final override fun click() { }
  }
  ```
  如果重写了一个基类或者接口的成员，重写的成员默认是open的，如果要阻止子类重写，可以显式的标注为`final`。
- kotlin中没有包私有的概念，但提供了`internal`修饰符，只在一起编译的一组Kotlin文件中可见，优势在于对模块实现细节的真正封装。**Java中这种封装很容易被破坏，只要包名相同就可以取得包私有的访问权限**。
- 顶层声明可用`private`，可以将类、函数和属性限制在当前文件中可见。
- kotlin禁止从public函数访问低可见类型，Kotlin中同一个包中也不能访问`protected`修饰的函数或成员变量，`protected`只在子类中可见。类的扩展函数不能访问`private/protected`的成员。
  在编译成Java代码时，`private`的成员会被编译成包私有。
- kotlin中外部类不能使用内部类的`private`的成员。Kotlin中要作为内部类来用需要添加`inner`修饰符，否则是Java的`static`的嵌套类。
- 在内部类中访问外部类需要使用`this@Outer`
  ```kotlin
    class Outer{
        inner class Inner{
            fun getOuterReference(): Outer = this@Outer
        }
    }
  ```
- kotlin中使用`sealed`可以限制直接子类必须写在父类中作为嵌套类。在when表达式中处理sealed类的子类时不再需要默认分支。sealed修饰符隐含着`open`访问性修饰。
- 构造语句`init{...}`可以多次声明。
<hr>  

- 修改访问器的可见性
  ```Kotlin
    class LengthCounter {
        var counter: Int = 0
            private set

        fun  addWord(word: String){
            counter + word.length
        }
    }
  ```
- Kotlin中的`is`是Java中`instanceof`的模拟。
- 如果两个对象相等，它们必须有着相同的hash值。
  ```Kotlin
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
  ```
  数据类通常需要重写的三个方法，可由kotlin的`data class`自动生成。如果要作为容器的键，属性用`val`是必须的要求。
- 通过`by`可以避免用实现相同接口的形式写模板代码，只需要制定一个该接口的实现实例即可快捷的创建代理模式。
  ```Kotlin
    class CountingSet<T> (val innerSet: MutableSet<T> = HashSet<T>())
    :MutableCollection<T> by innerSet {
        var objectsAdded = 0

        override fun add(element: T): Boolean {
            objectsAdded++
            return innerSet.add(element)
        }

        override fun addAll(elements: Collection<T>): Boolean {
            objectsAdded += elements.size
            return innerSet.addAll(elements)
        }
    }
  ```
- 一个对象声明与类声明类似，唯一不允许的就是构造方法声明，但可以使用初始化语句。可以直接使用该对象名调用该对象的方法。
  ```Kotlin
    object Payroll{
        private val allEmployees = arrayListOf<User>()

        fun calculateSalary(){
            for (person in allEmployees){
                ...
            }
        }
    }
    ...
    Payroll.calculateSalary()
  ```
  可继承，可实现接口，跟类一样用，方便了单例的实现。
<hr>
- 从Java中调用object类的单例，需使用`INSTANCE`。
 
  ```Kotlin
     CaseInsensitiveFileComparator.INSTANCE.compare(file1, file2);
  ```
- 伴生对象可以看作为类的一部分，可以代替类实现一些接口，可以重命名（默认叫做`Companion`），伴生对象实现的接口，可将该类传入作为参数。
  ```Kotlin
    class User2 private constructor(val nickname: String) {

        companion object Loader : JSONFactory<User2> {
            override fun fromJSON(jsonText: String) =
                    User2(JSONObject(jsonText).optString("nickname"))

            fun newSubscribingUser(email: String): User2 = User2(email.substringBefore('@'))
            fun newFacebookUser(accountId: Int) = User2(getFacebookName(accountId))

            private fun getFacebookName(facebookAccountId: Int): String = "facebookUser${facebookAccountId}"
        }
    }

    interface JSONFactory<T> {
        fun fromJSON(jsonText: String): T
    }

    fun <T> String.loadFromJSON(factory: JSONFactory<T>): T = factory.fromJSON(this)
    ...
    //调用处
    val user2 = "{nickname: 'Dmitry'}".loadFromJSON(User2)
    LogUtil.d(TAG, "user2's nickname is ${user2.nickname}")
  ```
- 与Java不同，Kotlin的匿名对象不限于对`final`变量的访问，即使是`var`修饰的局部变量，也是可以访问的。
  ```Kotlin
  registerBean(object : TalkativeButton(), Clickable{
            override fun click() {
                LogUtil.d(TAG, "TalkativeButton inner immplementation been clicked!")
            }

            override fun showOff() {
                super<Clickable>.showOff()
            }
        })
        ...
    private fun registerBean(widget: Focusable){
        ...
    }
  ```
