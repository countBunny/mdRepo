- 使用注解赋予这些类库特定的语义，而反射允许你在运行时分析这些类的结构。
- Kotlin增强了Deprecated注解
```Kotlin
@Deprecated("Use removeAt(index) instead.",ReplaceWith("removeAt(index)"))
fun remove(index:Int){}
```
设置了replaceWith，IDEA会提供一个自动的快速修正。ReplaceWith是一个注解，但是把它指定为Deprecated注解的实参时没用@。  
注解实参需要在编译期就是已知的。要把属性当作注解实参使用，需要用`const`修饰符标记。

```Kotlin
const  val  TEST_TIMEOUT=  lOOL

@Test(timeout = TEST_TIMEOUT} fun testMethod() { ... }
```
- Kotlin的注解不仅能应用在类和函数的声明上，还允许对表达式应用注解。`@Suppress`可以抑制被注解的表达式的上下文中的特定编译器警告。
- Kotlin中一些注解代替了java的关键字，`@Volatile`和`@Strictfp`，`@JvmName`可改变由Kotlin生成的Java方法或字段的名字。`@JvmStatic`用在伴生对象的方法上，可以暴露为Java的静态方法。`JvmOverloads`为默认参数的函数生成多个重载。`JvmField`把一个属性暴露为一个没有访问器的共有Java字段。
- annotation class 不能包含任何代码，拥有参数的注解，其参数要用val修饰。
```Kotlin
annotation class JsonSerialize(val name: String)
```
对应着Java中
```Java
public @interface JsonSerialize {
    String value();
}
```
如果需要把Java的注解应用到Kotlin元素上，必须对除了value以外的所有实参使用命名实参语法。`@JsonSerialize(name = "first_name")`

- 可以应用到注解类上的注解被称为元注解。
```Kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation

@BindingAnnotation
annotation class MyBinding
```
在Java代码中无法使用目标为PROPERTY的注解，要让这样的注解可以在Java中使用，可以给它添加第二个目标`AnnotationTarget.FIELD`。这样既可以应用到`Kotlin`中的属性又可以应用到Java的字段上。

- `@Retension`注解用来说明你的注解是否会存储到.class文件，以及在运行时是否可以通过反射来访问它。Java默认会在.class中保留注解但不会让它们在运行时被访问到。而Kotlin的注解拥有RUNTIME保留期。

  ```Kotlin
  annotation class CustomSerializer(val serializerClass: KClass<out ValueSerializer<*>>)
  ```

  如上代码展示了声明类型实参的技巧。

<hr>

- Kotlin 反射API没有仅限于Kotlin类，可以同样访问JVM写成的类。可以访问Java中不存在的概念，属性和可空类型。
- `.memberProperties`可以收集这个类中以及所有超类中定义的全部非扩展属性。

```Kotlin
val kClass = person.javaClass.kotlin
println(kClass.simpleName)
kClass.memberProperties.forEach { println(it.name) }
```

`person.javaClass.kotlin`获取Kotlin的扩展类。

- KCallable的集合存有所有成员，其中声明了call方法，允许调用相应函数或者属性的getter方法。

  ```Kotlin
  fun foo(x:Int) = println(x)
  val kFunction = ::foo
  kFunction.call(42)
  ```

- `call`方法并不提供安全性，可以用编译器的合成类型来约束KCallable接口，通过调用生成的invoke方法来按照设定的类型参数安全调用。

  ```kotlin
  //将函数声明为具体的方法
  val kFunction2: KFunction2<Int, Int, Int> = ::sum
  //这些类型成为合成的编译器生成类型，避免了对函数类型参数数量的人为限制
  println(kFunction2.invoke(1, 2))
  ```

- 属性值也可以调用`call`方法，但是属性接口提供了更好的`get`方法。顶层属性表示为`KProperty0`接口的实例，它有一个无参的get方法。

  ```kotlin
  val kProperty0 = ::counter
  kProperty0.setter.call(21)
  println(kProperty0.get())
  ```

  成员属性由`KProperty1`的实例表示，它拥有一个参数的get方法，要访问该值需要传入所属对象实例。

  ```kotlin
  val memberProperty = Person::age
  println(memberProperty.get(person))
  ```

- `KAnnotationElement`接口定义了属性`annotations`，它是用到源码中元素上所有注解实例的集合。`KProperty`继承了`KAnnotationElement`。因此可以获取到属性的全部注解。

  ```kotlin
  inline fun <reified T> KAnnotatedElement.findAnnotation(): T?
          = annotations.filterIsInstance<T>().firstOrNull()
  ```

  通过上述函数获取到相应注解后，可以通过注解实例，访问到注解的参数等。

  ```kotlin
  val jsonNameAnn = prop.findAnnotation<JsonName>()
  val propName = jsonNameAnn?.name ?: prop.name
  ```

- `KClass`类可以表示普通类和单例类，单例类可以直接通过`objectInstance`获取到实例，否则只能调用`constructor`的无参`call`方法来创建实例。

  ```kotlin
  internal fun <T : Any> KClass<T>.createInstance(): T {
      val noArgConstructor = constructors.find {
          it.parameters.isEmpty()
      }
      noArgConstructor ?: throw IllegalArgumentException(
              "Class must have a no-argument constructor")
  
      return noArgConstructor.call()
  }
  ```

- 如果参数由默认值，那么`param.isOptional`是`true`，就可以省略它的实参。如果一个类型是可空类型，`param.type.isMarkedNullable`会告知这一点。所有形参来说，必须提供对应的实参，否则会抛出异常。（Jkid的设计）

- KClass可以从已知的类中获取`ClassName::class`，也可以从对象实例中获取，`obj.javaClass.kotlin`。

- `KCallable.callBy`方法能用来调用带默认参数值的方法。