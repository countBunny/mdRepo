- 函数参数的类型叫作in位置，函数返回类型叫作out位置。out关键字要求所有使用T的方法只能把T放在out位置而不能放在in位置。这种约束保证了对应子类型关系的安全性。  
T上的关键字out有两层含义：
1. 子类型化会被保留
2. T只能用在out位置  

如List<Interface>在Kotlin中是只读的，所以它只有一个返回类型为T的元素的方法get，因此它是协变的。  
还可以当作另一个类型的类型实参。
```Kotlin
fun subList(fromindex: Int, toIndex: Int):  List<T>
```
当T同时出现在in和out两种位置上时，就不能声明为协变的。
- 构造方法的参数既不在in位置，也不在out位置，即使声明成了out也可以在构造方法的参数中声明并使用它。
```Kotlin
class Herd<out T:Animal>(vararg animals:T)
```
var声明会生成get/set方法，导致该类型不安全，从而导致T类型不能协变。
```Kotlin
class Herd<T : Animal>(var leadAnimal: T, vararg animals: T)
```
- 位置规则只覆盖了类外部可见的(public/protected和internal)API，私有方法的参数既不在in位置也不在out位置。变型规则只会防止外部使用者对类的误用但不会对类自己的实现起作用。
- sortedWith函数期望一个Comparator<String>，传递给它一个能比较更一般的类型的比较器是安全的，如果要在特定类型的对象上比较，可以使用能处理该类型或者它的超类型的比较器。这说明Comparator<Any> 是Comparator<String>的子类型，其中Any是String的超类型。**不同类型之间的子类型关系和这些类型的比较器之间的子类型化关系截然相反**。
```Kotlin
interface  Comparator<in  T>{
fun  compare(el:  T,  e2:  T):  Int  {  ...  }
}
```
`in`的意思是，对应类型的值是传递进来给这个类的方法的，并被这些方法消费。在类型参数T上的in关键字意味着子类型化被反转了，而且T只能用在in位置。  
一个类可以在一个类型参数上协变，同时在另外一个类型参数上逆变。
```Kotlin
interface  Functionl<in  P,  out  R>  {
    operator fun  invoke  (p:  P)  : R
}
```
一个辅助理解的例子
```Kotlin
fun enumerateCats(f: (Cat) -> Number){...}
fun Animal.getIndex():Int = this.hashCode()
enumerateCats(Animal::getIndex)
```
- 可以为类型声明中类型参数任意的用法指定变型修饰符。被称为类型投影。只能调用返回类型是泛型类型参数的那些方法，即只在out位置使用它的方法。
```Kotlin
fun <T> copyData(source:MutableList<out T>, destination: MutableList<T>){
    for (item in source){
        destination.add(item)
    }
}
//也可以用in声明来重写
fun <T> copyData(source:MutableList<T>, destination: MutableList<in T>){
    for (item in source){
        destination.add(item)
    }
}
```
标记为out的类型参数，会被编译器禁止调用其使用类型参数做实参的那些方法。
```Kotlin
val list: MutableList<out Number> = ArrayList()
list.add(5)
```
- 如果类型参数已经有out变型，获取它的out投影没有任何意义。如List已经声明了class List<out T>，编译器会发出警告，表明这样做是多余的。  
kotlin的点变形直接对应于Java的限界通配符。in即为`<? super T>`，`out`即为`<? extends T>`。
- 星号投影代表你不知道关于泛型实参的任何信息。如包含未知元素类型的列表List<*>。  
MutableList<Any?>和MutableList<*>不一样，前者包含的是任意类型元素，而后者是某种特定类型的元素，但你并不知道它的类型。因为不知道类型，你无法向其中写入任何东西，因为写入任何值都有可能违反调用代码的期望。但是从中读取值是可行的。
```Kotlin
val list2: MutableList<Any?> = mutableListOf('a', 1, "qwe")
val chars = mutableListOf('a', 'b', 'c')
val unknownElements: MutableList<*> =
        if (Random().nextBoolean()) list2 else chars
//    unknownElements.add(42)
println(unknownElements.first())
```
编译器会把Mutable<*>当成out投影类型。它可以表现为一个消费者，只是类型相当于`<in Nothing>`。
