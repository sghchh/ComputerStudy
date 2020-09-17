# 基本类型

在Kotlin中，基本类型的一些特性和Java中稍有区别。

### 1. 判断相等

在Kotlin中**==**操作符的实际过程为**当左操作数为非null时调用左操作数的equals函数**，否则判断右操作数是否为null。这就引申出Kotlin中可以为null的基本类型的判断相等问题，在kotlin中，当一个基本类型可以为null的时候，其判断相等的特性更像是Java中的包装类型那样，**===**默认会判断左右操作数是否指向同一个对象，而不是数值大小是否相等;而此时**==**调用equals函数则会判断数值是否相等：

```kotlin
val a: Int? = 1
val b: Int? = 1
println(a == b)  // true
println(a === b)   // false
```

### 2. 类型转换

在Kotlin中，虽然Int的表数比short更大，但是short类型的变量不能被隐式转换为Int变量，**即在Kotlin中将一个short类型的变量赋值给一个Int类型的变量是非法的**，反之也是非法的。在Kotlin中要想实现这之间的转换，需要显式地调用toChar、toInt等函数；同时Char类型的数也不可能与整形数据直接进行运算，即在Kotlin中，Char变量没有对应的整数结果。

> **虽然在Kotlin中，不同尺度的整型变量之间的转换必须是显式地，但是，在Kotlin中，对运算操作符进行过重载，不同尺度的整型变量进行运算的时候会隐式自动进行类型转换**

### 3. range

在Kotlin中想要通过数组的下标来遍历数组，可以不使用**0 until arr.size**这种形式，直接使用**arr.indices**即可,而**arr.withIndex()**则可以成对访问：

```kotlin
for (i in arr.indices) {
	println(arr[i])
}
for ((index, value) in arr.withIndex()) {
    println("the value is ${value}, and the index is ${index}")
}
```

同时，**downTo**产生的range对标**a..b**形式的range，即包括收尾两端，而不是对标until。

# 类结构  

## 一、构造函数  

Kotlin中的构造函数是一个和Java相比有很大不同的地方：一个类可以有一个主构造函数和0至多个二级构造函数。  

### 1. 主构造器  

**主构造器是非必须的**。

对于主构造函数，如果**没有可见性修饰符、没有注解**，那么主构造函数的**constructor**关键字可以去掉  
```kotlin
class One constructor(Params...){}
// 省略掉constructor关键字
class One(Params...){}  
```

在Kotlin中，**主构造器中不能包含任何代码**，初始化的代码可以放在**initializer blocks(用init关键字表示)**中,或者直接使用**初始化器(initializer)**：  
```kotlin
class One(Params...){
	val name = "Hello World!";
	init {
	
	}
}  
```
这里讲解一下各个概念的范围：  

1. 主构造器：主构造器的范围很窄，可以说**只有参数列表与初始化块init**，后面的花括号之后的代码是属于class body的范围，而主构造器是class head，二者没有在范围上没有交集  
2. 初始化器：名字高大上，实际上就是定义的时候直接初始化，例如上面的name = "Hello World"这种   

> 注意：有一个迷惑的行为，**class body中的initializer block中的代码经过jvm编译之后被划给了主构造器**；在后面的介绍中，二级构造器由于要间接或者直接回调主构造器，所以别忘了这时**initializer block中的初始化代码在二级构造器的代码块之前执行(虽然主构造器是非必须的、可以不定义主构造器，但此时initializer block中的代码依然执行在二级构造器之前**)。
>
> **在class body中定义的属性及其初始化器initializer并不属于主构造器**

在Kotlin中，在主构造函数的定义上提供了一种便利的、声明属性的方式，即在主构造器的参数前增加var或者val修饰，那么默认这些参数为类的属性：

```kotlin
class Person(n : String) {
	val name = n
}

class Person(val name : String)
```

上面两种形式的定义是等价的，name都会成为Person对象的一个属性。至此，我们可以明白，在Kotlin中构造函数的参数列表和普通函数的参数列表是一样的，只是用来传递参数，这些参数并不是类的属性，只是类的属性可以通过构造函数传递的这些参数来初始化自己罢了。而在主构造函数的参数列表中**使用var或者val修饰，则是一种语法糖，该参数即为类的属性，且初始化时机就在调用构造函数的时候(早于初始化块和初始化器)，这些属性的初始化是最先开始的。**

### 2. 二级构造器  

二级构造器和主构造器有很大的区别：  

1. 二级构造器必须直接或者间接地调用主构造器  
2. 二级构造器可以有自己的代码块儿
3. 二级构造器的constructor关键字不能省略    
```kotlin
class One(name:String) {
	constructor(name:String, address:String) : this(name){}
}    
```

**在Kotlin中回调构造器只需要在参数列表后面使用冒号接this其他构造器的调用就可**

### 3. 其他区别  

Kotlin中的构造器支持参数的默认值，而且Kotlin中的构造器参数是直接在传递时就初始化的，意思就是，在Java中在构造函数中经常会见到this.xxx = xxx，而在Kotlin中这种代码就没必要了，在传递参数的时候直接相当于有这一步了。   

**如果非抽象类未声明任何构造函数（主构造函数或辅助构造函数），则将生成一个不带参数的主构造函数。**
    
## 二、继承  

在Kotlin中**所有的类的基类是Any**，而不是Object。  

### 1. 可继承性  

在Kotlin中各种class在定义时默认是使用final修饰的，即在定义class的时候默认是不能被继承的，**如果需要可被继承，需要使用open关键字修饰class**   

### 2. 继承父类  

在Kotlin中，子类若想继承父类，那么其构造函数就有些要求：  

1. 如果派生类具有主构造函数，则可以（**并且必须**）在主构造函数的参数列表后调用基类的**构造函数(注意：即使基类没有显式声明构造函数，也会默认有一个无参的主构造函数)**：
```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)  
```
2. 如果派生类没有主构造函数，则每个辅助构造函数都**必须使用super关键字初始化基本类型，或委派给执行此操作的另一个构造函数**：
```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}  
```

在Kotlin中，父类的方法需要主动开放给子类，允许子类重写，默认情况下都是final修饰的，不允许子类重写，同继承一样，若想要允许子类重写父类的方法，**需要使用open关键字修饰方法**。  
```kotlin
open class Shape {
	open fun draw() { /*...*/ }
	fun fill() { /*...*/ }
}
	
class Circle() : Shape() {
	override fun draw() { /*...*/ }
}  
```
在Kotlin中，父类的字段也可以被子类覆盖，其方式同方法重写一样，父类都需要使用open关键字向子类开放相关的权限才行。  
```kotlin
open class Shape {
	open val vertexCount: Int = 0
}
	
class Rectangle : Shape() {
	override val vertexCount = 4
}
```
注意，这是一个Kotlin和Java不同的点，**在Kotlin中，子类的属性名与基类的属性名不能相同，对于相同的属性名编译器会认为你想要override基类的属性，此时会强制你为该属性增加override修饰，同时要求基类增加open修饰(但是，依然可以通过super来访问基类的同名属性)**。

#### 多个基类

在Kotlin中，如果子类直接或者间接继承了多个基类，**对于其基类之间重名的成员，子类必须对该成员进行重写**，而如果想要访问某一个基类的该成员，必须使用**super<Base**格式的语法：

```kotlin
open class Rectangle {
    open fun draw() { /* ... */ }
}

interface Polygon {
    fun draw() { /* ... */ } // interface members are 'open' by default
}

class Square() : Rectangle(), Polygon {
    // The compiler requires draw() to be overridden:
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```



### 3. 委托

在很多场景中，我们对于一个接口中定义的方法，想要通过委托给某一个该结构的实现类来调用，而不是自己去实现它，那么就可以使用**委托**。

```kotlin
interface Base {
	fun print();
}
class BaseImpl(val x:Int) : Base {
	override fun print(){print(x)}
}

class Derived(b : Base) : Base by b
fun main() {
	val b = BaseImpl(1)
	Derived(b).print()
}
```

就像上面的代码一样，Derived不是Base的实现类，但是他需要print()方法，而此时有一个Base子类满足了Derived的需求，此时就可以通过传参的形式，将Derived的print()方法委托给BaseImpl的print()方法，这就看起来像是Derived实现了Base一样。

在Kotlin中，甚至对于这种委托还支持对方法进行**override**：

```kotlin
class Derived(b : Base) : Base by b {
	override fun print(print("this is derived"))
}
```

但是，委托终究不是继承，在不进行方法重写的情况下，由于调用的是被委托方的方法，**因此委托方对成员变量的覆盖并不能被捕捉到**：

```kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // This property is not accessed from b's implementation of `print`
    override val message = "Message of Derived"
}

fun main() {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}
```

虽然，Derived看起来覆盖了message成员，但是由于是全权委托给了BaseImpl，所以print方法完全是BaseImpl对象执行的结果。

### 4. 属性委派

上面说的是使用Kotlin中提供的委派机制来实现类似于继承的效果，而关于Kotlin中属性的委派要复杂一些。

委派属性的语法为：**val/var <property name: <Type by <expression**

关于这里的expression，需要提供两个函数：**getValue与setValue**，通过名字就可以知道，对被委派的原变量的访问与赋值，都会委派到该expression的getValue和setValue函数上。

```kotlin
var p: String by Delegate()
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

值得注意的是，这两个函数中的第一个参数thisRef为**被委派的变量的引用**，在本例中为p；而第二个参数为**被委派的变量的属性对象**，在本例中为**String.kclass**.

#### lazy

lazy()函数时官方提供的一种委派expression lambda，它返回一个Lazy<T对象，该对象可以实现的委派机制是：对被委派的变量第一次调用get时，Lazy对象会执行lambda表达式的代码，并且将保存表达式的结果，后续再次调用get时会返回这里保存的结果。

```kotlin
val hello: String by lazy {
            println("this is dele")
            "hello"
        }
```

很明显**lazy()这种官方委派机制，只能用于val变量，因为其没有提供setValue函数**。

#### Observable

这也是一个官方提供的委派机制，这种委派机制有两个参数，**一个代表initialValue，另一个是一个lambda，其参数即为setValue函数对应的参数。看到这里就明白了，我们对被委派属性的每一次赋值操作都会执行该lambda**，因此，该委派机制可以用于var变量。

```kotlin
var hello: String by Delegates.observable("hello") {
            prop, old, new ->
            println("old is ${old}, and new is ${new}")
}
hello = "world"
```

Observable的lambda执行在变量被赋值之后，而**vetoable的lambda执行在变量被赋值之前，除此之外其用法同Observable一样**。

#### 委派给其他属性

这更加简单粗暴，直接将某一个属性委派为另一个属性，所有对该属性的操作，直接转为对委派属性的操作：这种使用方式是使用**：：**

```kotlin
class MyClass {
   var newName: Int = 0
   
   var oldName: Int by this::newName
}
```



## 三、属性  

在Kotlin中，属性语Java中的有很大不同：  

1. **对于非空属性在定义的时候必须立即进行初始化**(可以使用lateinit关键字表明延迟初始化)

   > 注意：这里必须立即初始化的形式有：初始化器、初始化块儿、主构造器中的var和val修饰参数

2. 属性必须使用var或者val之一修饰     


同样，上面说的必须在定义的时候同时赋值，有一些危言耸听，Kotlin也提供了另一种方式，看起来不那么传统，因为使用var或者val修饰的属性，其一定会有一个getter accessor，所以我们可以通过定制这个**getter accessor的方式赋值**：  
```kotlin
val isEmpty: Boolean
	get() = this.size == 0   
```
### 1. accessor  

都知道Kotlin中的var和val属性都是由accessor的，而val只有getter没有setter，var二者都有。当我们访问某一个类的实例的某一个属性的时候实际上调用的是其getter accessor；而在为其赋值的时候实际上调用的是setter accessor，而setter accessor方法是有一个参数的。

### 2. accessor的巧用  

在Kotlin中，如果想要对外屏蔽修改，但是内部却可以对变量进行修改，我们可以将其setter设置为private修饰，这样一来外部就无法调用setter修改了：

```kotlin
var visibility: String = "yes"
	private set
```

 

### 3. Backing Field  

首先声明一下，在Kotlin官方文档中说了在两种情况下一个属性才会**生成Backing Field**：  

* 属性的accessor至少有有一个是默认的实现  
* 在自定义的accessor中显式使用了field字段  

为何要说一下这个，因为如果一个属性没有backing field的话，其initialzer是不允许的：  
```kotlin
var name : String = "sgh"
	get() {return "hello world"}
	set(value) {
	    //field = value
	    println("this is setter")
	}  
```
此时，由于该代码不符合上述的两条规则，此时运行的话会报如下的错误：  
```kotlin
Error:(2, 25) Kotlin: Initializer is not allowed here because this property has no backing field   // name = "sgh"  
```
如果将上面的setter中注释部分加到setter中，那么该属性就会生成backing field字段，此时就不会报错了。因此我们就知道了一个小细节：**initializer在属性含有backing filed的情况下才被允许**。  
    
### 4. 对于accessor的理解  

首先要有这样一个概念：  

* 任何访问属性的实现都是调用getter  
* 任何修改属性的引用(指的是使用=进行赋值操作)的实现都是调用setter  

需要着重说明的一点是，**initializer是独立的，它并不是调用的setter**。 

## 四、接口    

## 五、可见性修饰符  
### 5.1 类和接口的可见性  

* private：只在类的内部可见  
* protected：在类的内部、子类可见(子类override父类的protected成员后，这个overriding成员也是拥有protected级别的可见性)  
* internal：在module内部可见  
* public：任何位置可见    

### 5.2 Top-Level的可见性  

在Kotlin中，函数、类、属性是可以直接定义到一个文件中的，这就是top-level，Top-Level下的可见性修饰符与类中的可见性修饰符有些许不同：  

* private：定义的成员只会在文件中可见  

* internal：在同一个module中可见  

* protected：**该修饰符在top-level下不被允许**  

* public：是默认的修饰符，任何地方可见
  
  
  
## 六、扩展Extensions  

在Kotlin中支持往第三方库或者官方的类中增加扩展方法，同时也可以扩展属性：  

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}  
val list = mutableListOf(1, 2, 3)
list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 
'list'
```

有几点需要注意的：  

1. 扩展的方法的优先级比类的成员方法要低，如果出现了一个扩展方法和该类中一个成员方法同名同参，那么总是会调用成员方法  

2. **调用时只会根据编译时类型来调用对应的扩展方法，而不是根据运行时类型**；也就是说父类和子类都有一个同名同参的扩展方法，那么如果某一个变量定义时声明的是父类对象，运行时是子类对象，那么该方法调用的还是在父类上扩展的方法。  

   

对于扩展属性，Kotlin中也是支持的，同样，其实现原理也不是往已有的类中插入属性(**因此就没有了backing filed，因此就没法使用initializer**)，而是为扩展的属性维护getter和setter： 

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```



## 七、特殊的Class  
### 1. Data Class  
在我们写代码的时候经常会遇到一些类，这些类的主要功能就是保存数据，这是Kotlin为我们提供了Data Class专门使用在该情况下；对于一个Data Class编译器会自动为**主构造器的参数列表中每一个参数提供如下几个方法实现：**
* equals()和hashCode()方法  
* toString()方法(该toString方法是专门定制的，不会打印内存地址)
* copy()方法实现  
* componentN()方法  
  

但是想要编译器为主构造器中的参数生成上面的函数，需要准许如下几个规则：  
* 主构造器的参数列表至少含有一个参数
* 所有的主构造器参数必须使用var或者val修饰  
* Data Class不可为abstract、sealed或者inner 

另外，**只有在主构造器中声明的属性才会对上述的几个函数生效，在class body中声明的属性不会生效**：

```kotlin
data class(var name: String) {
	var age: Int = 10  // 只有name属性会生效，age不会
}
```



### 2. Sealed Class
### 3. Nested and Inner Class  

在Kotlin中Nested Class是将一个Class定义到另一个Class的内部，但是他和Java中的内部类不同，更像是一个静态内部类：    
```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

可见，其使用方式更像是一个静态内部类那样，这时候**内部类对象并没有持有外部类的引用，同时创建内部类的实例也不需要外部类的实例**  

而在Kotlin中还有一个Inner Class，其更像是对标Java中的普通内部类的：  

```kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

**此时想要创建内部类的实例需要外部类对象，同时内部类对象持有外部类对象的引用**。  

在Kotlin中使用匿名内部类同Java中一样，没有区别。  

### 4. Enum Class

枚举类很简单，我们只需要在定义的时候，所有**枚举常量之间要用逗号分割**：

```kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

枚举类也可以定义其他成员，这时候**枚举常量与这些成员之间要使用分号进行分割**：

```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```

上面的代码也展示了，在Kotlin中使用枚举类时，各个**枚举常量也可以是匿名类对象**。  

在Kotlin中提供了一对方法来操作枚举常量：

* enumValues()：返回一个Array，返回指定枚举类中所有枚举常量组成的数组  
* enumValueOf(name : String):参数代表枚举类中枚举常量的名字，返回该名字对应的枚举常量



# 函数与Lambda

在Kotlin中，函数在定义时使用的关键字为**fun**，其参数列表在定义时的格式为**name:type**。

### 1. 参数默认值

相较于Java，Kotlin支持**为函数的参数设置默认值**，关于这种特性有几种特性需要说一下：

1. 当重写含有默认参数的方法时，子类对应参数的默认值同父类一样，但是在子类的方法声明中可以忽略这些默认值的设置：

```kotlin
open class A {
    open fun foo(i: Int = 10) { /*...*/ }
}

class B : A() {
    override fun foo(i: Int) { /*...*/ }  // no default value allowed
}
```

可以看到，**子类B的foo方法中忽略了将i设置为10的语句**。

2. 当一个参数前面出现了含有默认值的参数时，在函数调用时，如果想要使用这些默认值的话，对于这些参数需要使用命名形式调用

```kotlin
fun foo(bar: Int = 0, baz: Int) { /*...*/ }

foo(baz = 1) // The default value bar = 0 is used
```

可以看到，只有在调用函数的时候显示的使用baz=1才可以体验到默认参数bar=0的便利，而不能直接foo(1)，这样一来这个1到底是谁的值就会产生歧义，因此一种**好的习惯是将所有默认参数放到参数列表的后面**。

3. 当函数的**最后一个参数是一个function时**，在调用该函数时，可以使用一个lambda表达式，而且该lambda表达式的花括号可以放到函数调用的后面，这叫做**尾随lambda**

```kotlin
fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { /*...*/ }

foo(1) { println("hello") }     // lambda直接放到扩号后面
foo(qux = { println("hello") }) // Uses both default values bar = 0 and baz = 1 
foo { println("hello") }    // function参数全部采用默认参数
```

而且，当lambda前面都是默认参数且采用默认值、或者lambda是该函数的唯一参数时，在调用该函数时，圆括号也可以直接省略，就像上面的最后一个调用形式。

### 2. 变长参数

Kotlin中的变长参数使用**vararg**来声明，同时也可直接**结合命名参数调用传递一个array**：

```kotlin
fun foo(vararg strings: String) { /*...*/ }

foo(strings = *arrayOf("a", "b", "c"))
```

**一个函数只能有一个变长参数，如果一个变长参数没有位于参数列表的最后的话，那么对于其后面的参数在传递时必须采用命名参数调用的形式**。

变长参数在传递参数的时候，可以一个个的分开传递，也可以采用**spread操作符(在array前面加一个星号)**多个同时传：

```kotlin
fun foo(vararg strings: String) { /*...*/ }

foo('d', 'e', *arrayOf("a", "b", "c"))
```

上面的例子中，d、e与后面的之间就是一个一个的分开传递，而最后一个采用的是spread操作符。

### 3. Infix notation

满足以下条件的函数可以使用**infix修饰符**声明该函数可以中缀调用：

* 必须是成员函数或者拓展函数extension function
* 必须含有一个参数(不能是可变参数，不能设置默认值)

这样一来，使用了infix修饰的函数，在调用时就可以忽略成员函数调用操作符与圆括号：

```kotlin
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2   // 等同于1.shl(2)
```

但是，想要在该类的其他成员方法中使用这种形式，那么此时的**this就不能省略**：

```kotlin
class MyStringCollection {
    infix fun add(s: String) { /*...*/ }
    
    fun build() {
        this add "abc"   // Correct
        add("abc")       // Correct
        //add "abc"        // Incorrect: the receiver must be specified
    }
}
```

### 4. 尾递归

Kotlin提供**tailrec修饰符**用来修饰一些递归函数，该修饰符会对函数进行优化处理，使得**递归调用不会出现栈溢出**的问题。但是，并不是所有的递归函数都能使用这种优化，只有**递归调用为函数的最后一个表达式时才可以**。

```kotlin
val eps = 1E-10 // "good enough", could be 10^-15

tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (Math.abs(x - Math.cos(x)) < eps) x else findFixPoint(Math.cos(x))
```

注意，这里虽然递归调用位于一个ifelse语句中，但是由于if分支中没有递归调用直接return，而else中才含有递归调用，所以也算满足条件。准确的来说，只要递归调用后面不会出现其他的操作即可。

### 5. 函数的类型

在Kotlin中，函数时最高级别的组件，等同于class。因此，函数也有自己的类型:**(paramslist) -> type**，就是函数的类型，**此时对于返回类型为Unit的函数，这里的Unit不可省略**；我们也可以使用**typealias关键字**为某一个类型的函数设置一个名字，这样一来就可以像声明某一类型的对象那样，声明该类型的函数了：

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

这样一来，就可以使用ClickHandler作为一个类型来声明和初始化变量了。

#### Function Reference

在C语言中，一个函数的名字就是一个指向该函数的指针可以直接赋值给一个变量，但是在Kotlin中是不可以直接使用函数的名字赋值给一个对应类型的函数变量的，需要使用**::操作符**：

```kotlin
fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"
val predicate: (String) -> Boolean = ::isOdd   // 这是正确的
val predicate:(String) -> Boolean = isOdd  // 这是错误的
```

### 6. 匿名函数

匿名函数就是没有函数名的函数，其使用场景基本上同lambda一样，通常也是作为参数传递。其和Lambda很大的一点不同在于其内部的return语句的范围仅仅是其函数体的范围。

### 7. Lambda表达式

Lambda表达式在函数作为参数传递时常用，完整的Lambda表达式的结构如下：

```kotlin
{parameterList -> code}
```

在lambda表达式中，**没有声明返回类型声明的地方**，会默认表达式最后一句语句的类型为返回类型。

当lambda的参数列表中**只有一个single参数(非vararg)**的时候，表达式中的参数列表和->符号都可以省略，在**code中使用到参数的地方直接使用it指代**。

另外，在lambda中，**对于没有使用到的参数，可以使用一个下划短线作_为占位符**:

```kotlin
map.forEach { _, value -> println("$value!") }
```

#### return语句

在Lambda中使用return语句需要注意，如果不加修饰的话，该语句会导致外层函数的返回而不是仅仅从lambda表达式中返回。想要从lambda表达式中返回，在**return关键字后面需要指明返回的函数**：

```kotlin
ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```

> 上文中说到，lambda的返回值按照其code中的最后一句返回值类型决定，此时其只会从lambda中返回

# 集合

在Kotlin集合分为两大类：**只读的Collection与可变的MutableCollection**。其中Collection及其子类只提供访问其元素的方法，但是不能够增加、删除、修改元素，所有的元素在初始化后就定下来了；而MutableCollection则提供修改元素的方法。

## 一、集合的创建

### 1. 快速创建

Kotlin中提供了一些标准函数，用于快速创建集合对象：listOf()、mutableListOF()、setOf()等等

```kotlin
val numbersSet = setOf("one", "two", "three", "four")
val emptySet = mutableSetOf<String>()
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
```

对于创建一个空的集合，Kotlin也专门提供了相应的函数：emptySet()、emptyList()等等

```kotlin
val empty = emptyList<String>()
```

### 2. list的初始化函数

由于list像是一个数组，每一个元素都有一个对应的下标，所以专门有一个构造器可以通过下标对list进行初始化：

```kotlin
val doubled = List(3, { it * 2 }) 
```

这里List的构造器中第二个参数是一个lambda表达式，其中的it就是指每一个位置的下标

### 3. 集合拷贝

Kotlin中为集合提供的拷贝方法如：toList、toMutableList等都是浅层拷贝，生成的集合都是原来集合的一个快照。在任何一个集合中，修改原来的元素的内容会影响到所有的拷贝集合；但是在其中一个集合中增加和删除元素并不会影响其他的集合。

```kotlin
val sourceList = mutableListOf(1, 2, 3)
val copyList = sourceList.toMutableList()
sourceList.remove(2)   
println("source size: ${sourceList.size}")   // 2
println("Copy size: ${copyList.size}") // 3   
```

### 4. 集合变换

Kotlin也提供了一些方法供集合进行变换操作：**变换操作的意思就是，在基于原来集合中元素的基础上生成一个新的集合**。

#### Map变换

map变换是通过一个lambda表达式操作已知集合中的每一个元素，生成对应新的集合：

```kotlin
val numbers = setOf(1, 2, 3)
println(numbers.map { it * 3 })
println(numbers.mapIndexed { idx, value -> value * idx })
```

其中：

* map()函数中的lambda形参的参数列表只有一个single参数，是集合中的每一个元素
* mapIndexed()函数中的lambda形参的参数列表有两个参数，第一个参数为集合中元素的下标，第二个参数为对应下标位置的元素

另外，上面两个函数生成的新的集合的长度与原来的长度是相等的，如果想要在进行集合变换的时候忽略个别位置的元素，那么可以将他们设置为null，并使用mapNotNull()与mapIndexedNotNull()两个函数分别替代map()与mapIndexed()函数，**这两个函数在进行变换的过程中对于为null的元素不会添加到最后返回的集合汇总**：

```kotlin
val numbers = setOf(1, 2, 3)
println(numbers.mapNotNull { if ( it == 2) null else it * 3 })
println(numbers.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx })
```

而对于**Map集合来说，mapKeys()和mapValues()函数分别用来变换其键和值**。

```kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
println(numbersMap.mapKeys { it.key.toUpperCase() })
println(numbersMap.mapValues { it.value + it.key.length })
--------output---------
{KEY1=1, KEY2=2, KEY3=3, KEY11=11}
{key1=5, key2=6, key3=7, key11=16}
```

注意，mapKeys和mapValues两个函数生成的是两个不同的map，好像并不能同时对keys和values变换生成一个map

#### Zip变换

zip变换用于将两个集合或者两个数组压缩成一个集合或者数组，新的集合或者数组的特性如下：

* 长度为二者中较小的那个
* 每一个位置的元素为一个Pair对象，该Pair对象的第一个元素为第一个集合该位置的元素，Pair对象的第二个元素为第二个集合该位置的元素

```kotlin
val colors = listOf("red", "brown", "grey")

val twoAnimals = listOf("fox", "bear")
println(colors.zip(twoAnimals))

----------输出----------
[(red, fox), (brown, bear)]
```

同时，Kotlin也为zip变换提供了后续的变换，**zip()函数也可以在第一个集合参数后面传递一个lambda表达式作为后续变换的处理函数：**

```kotlin
val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")

println(colors.zip(animals) { color, animal -> "The ${animal.capitalize()} is $color"})
-----------output-------------
[The Fox is red, The Bear is brown, The Wolf is grey]
```

#### Association变换

Association变换可以将一个list或者set变换成一个map；associateWith()函数生成的map中原集合中的元素为key，value是从每个元素中变换过来的，但是由于map中键要保证唯一性，所以如果原来的list中有重复的元素，那么生成的map中只会保留一次出现的该元素生成的键值对：

```kotlin
val numbers = listOf("one", "two", "three", "one")
println(numbers.associateWith { it.length })
--------output---------
{one=3, two=3, three=5}
```

而如果想要集合中原来的元素作为value，变换生成的元素作为key的话，需要使用associateBy()函数：

```kotlin
val numbers = listOf("one", "two", "three", "four")

println(numbers.associateBy { it.first().toUpperCase() })
---------output--------
{O=one, T=three, F=four}
```

同时，associateBy()函数也可以传递第二个参数，用来对value进行变换：

```kotlin
println(numbers.associateBy(keySelector = { it.first().toUpperCase() }, valueTransform = { it.length }))
--------output----------
{O=3, T=5, F=4}
```

#### Flatten变换

该变换负责将一个嵌套的集合展平成一个集合：

```kotlin
val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
println(numberSets.flatten())
---------output---------
[1, 2, 3, 4, 5, 6, 1, 2]
```

#### String替换变换

函数joinToString()可以将一个集合转化为一个字符串对象，原来集合中的元素之间在字符串中使用逗号分割：

```kotlin
val numbers = listOf("one", "two", "three", "four")
        
println(numbers.joinToString())
--------output----------
one, two, three, four
```

### 5. 集合过滤

集合的过滤和集合的变换是一样的做法，但是集合过滤的效果是从原有的集合中筛选符合一定规则的元素，生成一个新的集合，通常来说生成的集合是原来集合的子集。对于List和Set来说，生成的都是List；但是Map生成的还是Map。

对应的Kotlin为集合过滤提供了filter()和filterIndexed()函数分别代表不带下标的lambda与带下标的lambda处理；而filterNot()函数过滤下来的元素为不满足对应规则的元素：

```kotlin
val numbers = listOf("one", "two", "three", "four")
val longerThan3 = numbers.filter { it.length > 3 }
val filteredIdx = numbers.filterIndexed { index, s -> (index != 0) && (s.length < 5)  }
val filteredNot = numbers.filterNot { it.length <= 3 }

println(longerThan3)
println(filteredIdx)
println(filteredNot)
-------output--------
[three, four]
[two, four]
[three, four]
```

上面的三个函数与集合变换很类似，对于map来说集合变换提供了两个函数分别对key和value进行变换，在集合过滤中，Kotlin也提供了**filterKeys()和filterValues()**两个函数来实现相对应的过滤效果；同时，filter()函数在作用于map时，其lambda的参数有两个，第一个是key，第二个是value，该函数可以同时定义key和value的过滤规则，从而生成一个map：

```kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
println(filteredMap)
-------output-------
{key11=11}
```

另外，Kotlin还专门提供了两个特殊的过滤函数：filterIsInstance()和filterNotNull()，前者负责从一个List<Any>中过滤出特定泛型类的元素；后者直接过滤出不为null的元素：

```kotlin
val numbers = listOf(null, 1, "two", 3.0, "four")
println(numbers.filterIsInstance<String>())
val numbers = listOf(null, "one", "two", null)
println(numbers.filterNotNull())
-------output--------
[two, four]
[one, two]
```

### 6. 集合的加减运算

在Kotlin中，List和Set的加运算就是取集合元素的并集，而减运算就是左操作数减去其与右操作数的交集，这两个很好理解没什么好说的。

对于Map字典来说，加法与减法则有很大的不同。Map的加法要求右操作数是一个Pair对象或者也是一个map，加法的规则是：

* 如果右操作数中某一个元素的key在左操作数中已经存在，那么新的map中该key对应的value以右操作数的value为准
* 对于左操作数中没有出现过的key，全部添加到新的map中

```kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap + Pair("four", 4))
println(numbersMap + Pair("one", 10))
println(numbersMap + mapOf("five" to 5, "one" to 11))
--------output---------
{one=1, two=2, three=3, four=4}
{one=10, two=2, three=3}
{one=11, two=2, three=3, five=5}
```

Map字典的加法更加特殊，右操作数不再是map了，而是单个key值或者由一系列key组成的list，减法规则为：

* 右操作数中包含的key如果在左操作数中存在，那么删除该键值对

```kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap - "one")
println(numbersMap - listOf("two", "four"))
------output--------
{two=2, three=3}
{one=1, three=3}
```

### 7. 集合分类

Kotlin提供了一个groupBy()函数可以为List中的元素进行分类，该函数的返回类型是一个map，该map的key值根据制定的规则从元素中生成，map的value则是对元素采取的自定义变换(当然该变换也可以不传递，则直接采用原来的元素)后符合key特性的所有元素组成的list：

```kotlin
val numbers = listOf("one", "two", "three", "four", "five")

println(numbers.groupBy { it.first().toUpperCase() })
println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.toUpperCase() }))
-------output--------
{O=[one], T=[two, three], F=[four, five]}
{o=[ONE], t=[TWO, THREE], f=[FOUR, FIVE]}
```

### 8. 集合检索

#### slice

Kotlin中的slice函数用于List和Set，可以为该函数传递一个range或者一个包含下标数字的list或者set，表示根据range或者包含下标的list或者set取出对应的元素组成一个新的list：

```kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")    
println(numbers.slice(1..3))
println(numbers.slice(0..4 step 2))
println(numbers.slice(setOf(3, 5, 0)))    
--------output-----------
[two, three, four]
[one, three, five]
[four, six, one]
```

#### take和drop

其中take(num)函数表示从list或者set的第一个元素开始从前往后数，取num个元素返回；而takeLast(num)表示从list或者set的最后一个元素开始从后往前数，去num个元素返回。

```kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.take(3))
println(numbers.takeLast(3))
----------output------------
[one, two, three]
[four, five, six]
```

而drop(num)与dropLast(num)之间的关系同take相似，只不过take是挑选，而drop是舍弃，即drop(num)代表从第一个元素开始从前往后数舍弃num个元素，将剩下的返回；而dropLast为从后往前数。

另外还提供**takeWhile()、takeLastWhile()、dropWhile()、dropLastWhile()**等通过向其传递一个lambda表达式来根据对应的元素指定取舍规则。

### 9. 排序

集合中的元素想要进行排序，同Java中一样，需要在调用**sortedWith函数时传递一个Comparator接口的实现类**，同时Kotlin也提供一种快速的生成Comparator接口的函数**compareBy()**：

```kotlin
val lengthComparator = Comparator { str1: String, str2: String -> str1.length - str2.length }
println(listOf("aaa", "bb", "c").sortedWith(lengthComparator))
println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))
-------output---------
[c, bb, aaa]
[c, bb, aaa]
```

如上，两种传递Comparator的方式是等价的。



# 其他

## 一、空指针安全

在Kotlin中要求程序员在编写代码时就得保证该变量的空指针安全，String类型的变量与String?类型的变量是不同的，二者也不可能进行强制类型转换，其中String类型变量保证永远不会为null；而String?类型的变量则可能会为null，因此在使用该类型的变量时可能需要检查其不为null才可。

### 1. 安全访问操作符

对于可以为null的类型变量，在访问时提供了**?.操作符**，该操作符的语义为如果变量为null那么返回null，否则正常调用：

```kotlin
var b : String? = "hello"
var a = b ?. length
```

由于这种语义，b ?. length可能返回为null，所以这里**a的类型为Int？**

同时，可以将安全访问操作符与**let**连用，可以对不为null的元素进行一些操作：

```kotlin
val listWithNulls: List<String?> = listOf("Kotlin", null)
for (item in listWithNulls) {
    item?.let { println(it) } // prints Kotlin and ignores null
}
```

上面的代码只会打印集合中不为null的元素。

### 2. Elvis 操作符

对于下面的条件判断：

```kotlin
val l: Int = if (b != null) b.length else -1
```

Kotlin专门提供了一个操作符来简化书写：**expression1 ?: expression2**

该操作符的语义为：**如果表达式1的结果为null，那么返回表达式2的结果；否则返回表达式1**

### 3. 抛出异常

Kotlin也提供操作符，当变量为null时会抛出空指针异常：**!!.**

### 4. 类型转换

上面说到String与String?之间不能进行强制类型转换，但是Kotlin提供了一个:**as?**操作符，可以实现Parent?与Child?之间的类型转换.

```
val b: Int? = a as? Int   // 其中a
```

## 二、Scope Function

Kotlin中提供的五个作用域函数，得益于[文章](https://juejin.im/post/5edfd7c9e51d45789a7f206d
)中介绍的方式，我们对这5个作用域函数进行分类和记忆的时候可以依据如下三个标准：

* 是否是拓展函数
* 参数是this还是it
* 返回值是最后一行还是调用者本身

| 函数    | 是否是扩展函数 | 函数参数 | 返回值   |
| ------- | -------------- | -------- | -------- |
| with    | 不是           | this     | 最后一行 |
| T.run   | 是             | this     | 最后一行 |
| T.let   | 是             | it       | 最后一行 |
| T.also  | 是             | it       | 调用本身 |
| T.apply | 是             | this     | 调用本身 |

首先介绍一下对于返回值为**最后一行**与**调用本身**之间的适用场景的不同。

很显然，对于返回值为调用本身的函数，该函数的作用更像是对调用者本身做一些配置性的工作；对于返回值为最后一行的函数，该函数的作用**更像使用调用者的自身属性进行一些计算**，然后返回一个计算结果。

在之后的讲解实例中我们都是用Student这个类来做例子：

```kotlin
data class Student(var name : String?, var num : String?, var clazz : String?, var score : Int?)
```

#### 调用本身

对于返回值为调用本身的场景，其适用情况在本例子中更像是对一个student的信息进行更改：

```kotlin
val student = Student().apply {
	this.name = "li ming"
	this.clazz = "001"
	this.score = 90
}
```

#### 最后一行

对于返回值为最后一行的场景，其适用情况在本例中可以选择获取该student的不同信息：

```kotlin
val student = Student("li ming", "201600001", "001", 90)
var nameAndNum = student.run {
	"my name is:${this.name}, num is:${this.num}"
}
var nameAndClazz = student.run {
	"my name is:${this.name}, clazz is:${this.clazz}"
}
var nameAndScore = student.run {
	"my name is:${this.name}, score is :${this.score}"
}
```

我们当然可以在Student这个类中定义不同的成员函数或者拓展函数来获取student的不同信息，但是这种工作量太大，而且其适配能力有限，假如想要获取学生的序号与班级、或者姓名学号班级等等，有很多种不同的组合形式，我们不可能为每一种组合都准备一个函数，因此，作用域函数这种随用随调的特性就会使得编程变得非常容易，而且其调用形式就像是一个类的成员函数一样，遵守了面向对象语言的特点。

#### with函数

with函数的参数为this，其返回值为最后一行，其与其他的作用域函数最大的不同就在于，它是一个普通的函数，并不是一个拓展函数，因此，使用with的场景更像是使用参数对象做一些运算，由于其调用的时候需要将对象传递过去，因此其场景更像是**基于该对象与其他外部数据做一些额外的计算**，然后返回计算的结果。

#### 关于it与this的区别

关于此二者的不同，我们沿袭文章中的理解，区别在于可读性。

## 三、类型检查

在Kotlin中，判断一个变量是否是某种类型的变量，通过**is**操作符：

```kotlin
a is String
```

而强制类型转换在Kotlin中使用**as**操作符实现：

```kotlin
val x : String = y as String
```

只不过这种类型转换在y为null的时候会抛出异常(对于非String类型的变量y抛出异常是无法避免的)，针对于**空安全**这一要点，提供了**as?**操作符：

```kotlin
val x: String? = y as? String
```

当y为null的时候，等号右侧的结果为null。

#### 泛型情况下的类型检查

对于泛型情况下的类型检查，普通场景下分为两大类：

* 在编译期能确定泛型类型
* 在编译期不能确定泛型类型

对于第一种情况，只能通过**星号代替泛型参数进行类型检查**：

```kotlin
list is List<Int>  // no
list is List<*>   // yes，and the items are Any？
```

对于第二种情况，可以使用类型检查，但是**不可以带上泛型参数**：

```kotlin
fun handleStrings(list: List<String>) {
    if (list is ArrayList) {
        // `list` is smart-cast to `ArrayList<String>`
    }
}
```

而且，此时也可以使用**as**操作符：list as ArrayList。

## 四、判断相等

在Kotlin中判断相等的操作符有两种：**==**与**===**，其中后者等价于Java、C等中的==操作符，在Kotlin中的==操作符有不同的含义--Kotlin中的==操作符被解释为：

```kotlin
a ?. equals(b) ?: (b === null)
```

即当a不为null的时候，调用equals方法判断两个变量是否相等；当a为null的时候，判断此时b是否为null，若同为null则相等，若b不为null则不等。

## 五、反射

### 1. Class Reference

在kotlin中使用反射是通过**::**操作符来完成的，值得注意的是此时得到的是**KClass**对象，而不是java中的Class对象，要想得到java中的class对象，只需要在KClass对象的基础上调用**.java**即可：

```kotlin
var a = String::class
var b = a.java
println(a)
println(b)
---------output---------
class kotlin.String
class java.lang.String
```

### 2. Function Reference

函数引用在前面匿名函数与lambda表达式的时候提到过：

```kotlin
fun isOdd(x: Int) = x % 2 != 0
```

很明显，isOdd是一个(Int) -> Boolean类型的函数，但是去不能直接将其作为参数传递给其他的特殊函数。只能通过::isOdd才可以。

如果想要得到一个类的成员函数或者extension函数，那么需要在前面加上类名：

```kotlin
var a = String::toCharArray
```

### 3. Property Reference

对于类成员属性的访问，同成员函数一样，需要**把类名带上**。但是这只是得到了一个KMutableProperty或者KProperty类型的对象，要想访问该类具体的对象下该属性的值还需要调用该Property的get方法：

```kotlin
class A(val p: Int)
val prop = A::p
println(prop.get(A(1)))
```

## 六、操作符重载

在Kotlin中，有**很多操作符都具有相对应的函数名**，而Kotlin支持操作符的重载，重载方式就是通过重写这些函数来实现的。可以选择将这些函数**定义为拓展函数**，也**可以定义为成员函数**；但是，只有在定义函数前增加修饰符**operator**才能表明这是一个操作符重载的函数，而不是普通函数。

> 要想实现特定的操作符重载，则operator与特定的函数定义二者缺一不可

### 1. 二元计算操作符

```kotlin
class Test(val value : Int) {
	operator fun plus(a : Test) : Test = Test(value + a.value)
}

operator fun Test.minus(a : Int) :Int = value - a
```

从上面的实例代码中可以知道，对于二元计算操作符：

* 函数的参数可以为任意类型
* 函数的返回值可以为任意类型

上面的两点也向我们说明了，**在Kotlin中，一个类对于同一个操作符，如+，可以定义多个plus函数，它们的函数原型不同，用来适配不同的场景**。这样一来，该类对象就可以与不同类型的数据直接进行+操作，也可以返回不同的类型。

## 七、Object表达式与声明

有时候我们在使用一个类的时候，需要对其进行微调，但是又不至于大动干戈地专门code一个子类出来，这时候Object表达式与Object声明的作用就体现出来了。

### 1. Object表达式

Object表达式首先是一个表达式，因此，**其可以作为右值使用**。Object表达式通常的**使用场景为：匿名类对象、对类的微调实现**

#### 匿名类

Object表达式用作匿名类与lambda最大的区别就是，lambda只能是用在接口中只定义一个函数的情景，而Object则没有这些限制：

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { /*...*/ }

    override fun mouseEntered(e: MouseEvent) { /*...*/ }
})
```

#### 微调类

正如开头所说，当我们想要对一个现有的类进行微调，却又不至于专门定义一个子类的时候，可以使用Object表达式现用：

```kotlin
open class A(x: Int) {
    public open print() {
    	print("hello world")
    }
}

interface B { /*...*/ }

val ab: A = object : A(1), B {
    override print(){
    	println("hello world")
    } 
}
```

我们在使用的时候，需求只需要将A的print方法换一种实现，对于其他部分没有必要修改，这时候显然没必要专门定义一个A的子类，Object表达式正好派上用场。

同时，上面也展示了，**Object表达式在使用的时候，可以继承多个类和接口，唯一特殊的是对于类的构造器，需要传入具体的参数值(毕竟是作为右值使用，所以需要传递具体的参数值进行初始化)**。

### 2. Object声明

Object声明与表达式不同，其**就是在一个kotlin源文件下，或者在一个class下的一个声明**。

Object声明可以随时随地声明一个“结构”，之后人们可以直接使用该“结构”。

#### 直接在Kotlin文件

直接在Kotlin文件下使用Object声明，其在使用时就像是Java中的静态成员一样，可以直接调用：

```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```

这样一来，我们可以**直接导入该Kotlin文件，然后使用DataProviderManager.registerDataProvider()的形式来访问Object声明中提供的代码**。

而且，我们依然可以在使用Object声明的时候选择继承一些类，只需要在ObjectName后面加上继承的语法即可。

在Kotlin中，**我们就可以通过这种方式实现Java中的Singleton模式的代码，而且，这种形式的Singleton保证是安全的。**

#### 在class下使用

Object在class里声明的格式同在Kotlin文件下是一样的，只不过在使用时需要加上**companion**，其在调用时**更像是Java中的static方法那样，直接通过类名进行调用**：

```kotlin
class ITest {
    init {
        println("this is constructor")
    }
    companion object {
        fun myprintln() {
            println("this is object")
        }
    }
}
fun main(args: Array<String>?) {
    ITest.myprintln()
}
```

需要注意的一点是，虽然在Kotlin中，这种形式的Object声明在使用上很像是Java中的静态方法，**但是在jvm中，该Object声明依然被认为是成员函数(这一点比较难懂，虽然是成员函数，但是调用该函数并不会触发任何对象的构造函数)**；想要将其完全变为静态函数那样，需要使用**JvmStatic注释**。

# 协程

协程是轻量级的线程，之所以是轻量级的线程是因为后面会看到，协程也是运行在线程之中的，而位于同一线程的不同协程之间是并发甚至并行执行的。

## 一、快速使用

在Kotlin中，协程没有专门对应的类来表示，而是通过一系列的类相互合作共同来实现一个协程的工作，必须需要**协程上下文CoroutineContext、协程范围CoroutineScope**等。

快速使用协程的方式有：launch、async、runBlocking等。

### 1. launch

该函数是CoroutineScope的扩展函数，该函数可以启动一个协程，而**协程的code部分就是该函数传递的lambda表达式，在launch的函数原型中，该参数是一个无参数无返回值的Function对象**：

```kotlin
fun main() {
    GlobalScope.launch { 
        delay(1000L) 
        println("World!") 
    }
    println("Hello,") 
    Thread.sleep(2000L) 
}
```

上面的输出为Hello，World！，正体现了launch启动的协程与main函数中其他代码之间是**异步的关系**，launch启动的协程的工作**并不会阻塞当前的线程**。从这一点来看，启动一个协程很像启动一个线程。

同时，launch函数的返回值是一个**Job**对象，该Job对象指向协程，我们可以通过该**job对象执行join()函数来实现对协程的同步调用**：

```kotlin
fun main() {
    val job = GlobalScope.launch { 
        delay(1000L) 
        println("World!") 
    }
    println("Hello,")
    job.join()
}
```

该代码的执行结果和上面的一样，如果把job.join在println("hello")之前调用，那么输出结果就是World！Hello了。

> 上面的GlobalScope是CoroutineScope的一个实现类，关于launch函数的更多细节这里先不讨论

#### cancel

上面提到了，launch函数的返回类型是一个**Job**对象，而在该对象是可以被**cancel**的，能够被正确的取消的前提是，在**协程的code部分要去检查cancelation状态**才可：

```kotlin
val startTime = System.currentTimeMillis()
val job = launch(Dispatchers.Default) {
    var nextPrintTime = startTime
    var i = 0
    while (i < 5) {
        if (System.currentTimeMillis() >= nextPrintTime) {
            println("job: I'm sleeping ${i++} ...")
            nextPrintTime += 500L
        }
    }
}
delay(1300L) 
println("main: I'm tired of waiting!")
job.cancelAndJoin() 
println("main: Now I can quit.")
```

该代码在调用job.cancelAndJoin之后协程中的代码依然会执行，这是因为在协程的code部分没有去检查cancelation状态，虽然执行了cancel函数，但是并不会响应，只需要对协程中的循环条件改为while(isActive)即可

> 这里的isActive是CoroutineScope的扩展属性

另一个有用的处理是：**在协程中，可以将代码使用try-cache-finally来进行包裹**，这样一来，在其被cancel之后，如果有使用到suspand函数的话，其会抛出异常，因此在这种场景下，**finally块可以用来在协程关闭之前进行一些资源释放工作**。

### 2. runBlocking

runBlocking和launch首先的不同点在于，runBlocking并不是一个扩展函数；其次，他也会创建一个协程，但是该协程会阻塞当前线程，该协程的工作与当前线程的其他工作之间不再是异步的关系：

```kotlin
fun main() { 
    runBlocking { 
        delay(1000L)
        println("World!")
    }
    println("Hello,") 
}
```

### 3. async

这个函数同launch一样，也是CoroutineScope的扩展函数，与launch不同的一点是，其**协程的code部分是一个有返回值的Function对象**，而在多数情况下希望协程是与其他代码之间是并发运行的，因此，在其他地方想要获得该协程code的结果需要使用**该Job的await()函数**：

```kotlin
suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}
```



### 4. withTimeOut

该函数也可以启动一个协程，该协程的执行有一个时间限制，超时的话会被cancel，抛出TimeoutCancellationException：

```kotlin
withTimeout(1300L) {
    repeat(1000) { i ->
        println("I'm sleeping $i ...")
        delay(500L)
    }
}
```

我们也同样可以将其包裹进try-catch-finally块中进行资源释放。

而**withTimeoutOrNull**与withTimeout的不同点在于，超时并不会抛出异常，而是返回一个null。

## 二、协程工作环境

在CoroutineScope中有一个关键的属性：**CoroutineContext**，在官方文档中把该上下文描述为一个包含很多元素的set。

其中最重要的就是**CoroutineDispatcher**和**Job**了，Job在前面的launch函数中提到过，而这里的CoroutineDispatcher就是**为协程分发到不同运行时线程的类**。

* **Dispatchers.Default**：使用一个共享的后台线程池来运行协程中的任务，**适用于计算密集型任务**
* **Dispatchers.IO**：也是使用一个共享的守护线程池来执行协程中的任务，不同于Default的线程池；**适用于IO密集型的操作**
* **Dispatchers.Unconfined**：对于其协程内部的第一个suspend操作会在调用该协程的线程执行；对于其后的suspend操作，会根据具体的情况分配线程或者线程池执行。
* **newSingleThreadContext**：为协程创建一个新的线程去执行
* **newFixedThreadPoolContext**：为协程分配到一个私有的线程池去执行
* 对于任意一个**Executor**对象，都可以使用**asCoroutineDispatcher**函数转为一个dispatcher

> 通过最后两个形式的Dispatcher，可以发现该类内部应该是使用线程池来为协程分配工作线程的

CoroutineDispatcher属于CoroutineContext，而后者是CoroutineScope的属性，所以在**不同的Scope下，其默认的Dispatcher是不同的**。GlobalScope默认的Dispatcher为Default，而runBlocking的Dispatcher为runBlocking函数被调用的线程。

## 三、异步流机制

Kotlin中的Asynchornous Flow机制同RxJava非常相似，我们可以将思想照搬。

**Kotlin中的Flow通过flow函数产生一个Flow对象，而其内部调用emit函数可以顺序地发送一些列数据；在接收端，通过Flow的collect函数逐个进行接收，期间可以对数据流进行map变换、filter过滤等中间处理**。其思想和形式和RxJava中的Observable非常像。

```kotlin
fun simple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    simple().collect { value -> println(value) } 
}
```

需要注意的是，Flow对象生成时，并不会立即调用emit发送数据，只有在调用collect函数时，其emit函数才会被启动，才会发送数据；可以认为调用flow函数只是生成了一个数据发生器，只有在调用该Flow对象的collect时才会启动该Flow发送数据，在collect中进行接收并处理。

同上面介绍的launch普通协程一样，Flow也可以被cancel。

### 1. builder

像上面的launch一样，想要创建一个Flow，也有一些特定的builder函数

* flow函数，就如上面的示例代码所示
* flowOf函数，将数组作为一个Flow产生
* asFlow函数，集合对象调用该函数，集合元素作为Flow发送的数据

### 2. 中间操作符

中间操作函数有：**map、filter、transform**。

#### map

其使用同集合中的map相类似，该函数的lambda的参数为Flow通过emit函数发送的数据，我们可以再map函数中对该数据进行**简单的变换**：

```kotlin
suspend fun performRequest(request: Int): String {
    delay(1000) 
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
}
```

这里就将Flow发送的Int数据延迟一秒发送，并且变换为了固定格式的字符串。

#### transform

transform和map与filter不同的一点是，该函数返回值为Unit，该函数可以**拦截上游Flow发送的数据**，然后可以在transform中根据拦截到的数据再次emit处理后的数据，而且在tranform中**可以再次emit多个数据**：

```kotlin
(1..3).asFlow() // a flow of requests
    .transform { request ->
        emit("Making request $request") 
        emit(performRequest(request)) 
    }
    .collect { response -> println(response) }
```

#### take操作

take操作同集合中的相类似，Flow可能会发送很多个数据，**take操作可以选择只接受前n个**，由于，**Flow通过emit发送数据的时机是collect等类型的函数启动的，而take操作发生在collect等函数之前，所以对于Flow中n个数据之后的emit根本不会被调用执行**：

```kotlin
fun numbers(): Flow<Int> = flow {
    try {                          
        emit(1)
        emit(2) 
        println("This line will not execute")
        emit(3)    
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking<Unit> {
    numbers() 
        .take(2) // take only the first two
        .collect { value -> println(value) }
} 
```

### 3. 终端操作符

终端操作符负责启动Flow的emit函数调用，同时接受并处理Flow发送的数据。

上面多次提到的**collect函数就是逐个接受并处理数据**。

#### toList和toSet

这两个操作符很容易理解，即将流发送的所有数据封装成一个list或者set

#### reduce

该操作符负责将flow发送的数据流reduce到一个数据，该操作符最终导致Flow的处理结果是返回了一个**数据**：

```kotlin
val sum = (1..5).asFlow()                      
    .reduce { a, b -> 
        a + b } // sum them (terminal operator)
println(sum)
```

最终的打印结果是sum为1到5的求和，而**reduce参数中的a为到b之前所有数据的求和**，比如当发送数据5时，此时a为1+2+3+4=10.

### 4. Flow的上下文

通过**flow{}**创建的Flow对象的运行上下文由调用**collect**函数的上下文决定，也就是说，flow函数发送数据与collect函数处理数据默认情况下位于同一个线程。

但是，在大多数使用场景中，数据发送与处理一般位于不同的线程，这也就要求flow函数与collect函数用有不同的上下文，我们可以**对Flow对象使用flowOn函数来指定flow函数执行的线程环境**：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it in CPU-consuming way
        log("Emitting $i")
        emit(i) // emit next value
    }
}.flowOn(Dispatchers.Default) // RIGHT way to change context for CPU-consuming code in flow builder

fun main() = runBlocking<Unit> {
    simple().collect { value ->
        log("Collected $value") 
    } 
}  
```

### 5. Flow的缓冲策略

在官方文档中，将这一部分的内容叫做缓冲Buffering，且先不管其各个操作符底层是否涉及到缓冲以及如何实现的缓冲，首先要明白这一部分的内容的目的是处理：**Flow在使用过程中遇到的，emit产生数据的速率与collect等函数处理数据的速率不等的优化方案**。

*注意：一下的例子中都以emit数据速率为100ms一次，而collect处理数据速率为300ms一次*；而且总共会产生3个数据。

在不使用buffering策略的时候，默认情况为：collect处理完一个数据之后，才会通知Flow来生产下一个数据。这是一种，完全的顺序策略，在例子中，整个Flow处理完成需要(100+300)x3=1200ms左右。

#### buffer

该操作符**可以使得在collect函数代码处理上一个emit过来的数据的时，通知Flow同时可以产生下一个emit的数据了，使得Flow的数据生成与处理可以部分并行进行**，节约了时间。因此在这种情况下，处理完整个Flow需要(100+300x3)=1000ms左右。

```kotlin
val time = measureTimeMillis {
    simple()  // simple每100ms发送一个数据
        .buffer() 
        .collect { value -> 
            delay(300) 
            println(value) 
        } 
}   
println("Collected in $time ms")
```

#### conflate

buffer操作符可以实现Flow中数据的产生与处理部分并行执行，但是这种机制还很保守，即在collect处理过程中，只会让Flow准备下一个数据的生产，对于更之后的数据并不会提前开始；即在我们的场景中，collect处理一个数据需要300ms，这个时间足够产生3个数据了，但是Flow只会在下一次collect时提前准备好一个数据，所以说buffer是一种保守的策略。

而conflate的策略则是，**在collect处理Flow产生的数据时间内，Flow可以准备后续数据的产生，提前准备的个数不限**。

> 首先，提前说明一下，该操作符只有在Flow处理数据耗时大于其产生数据的耗时时同buffer operator才有区别，因此对于二者没有区别的场景不做讨论。

但是这就带来了一个问题，那**就是Flow数据产生太快，collect不及时，导致了数据的堆积**。conflate operator采取的策略为：**在下一次collect执行开始时，只会处理最后一个被准备好的数据，其之前的会直接被舍弃**。在我们的场景中，collect处理耗时为300ms，因此在处理第一个数据时，Flow可以直接产生到最后的3号数据，而下一次调用collect时，就会传递进去3号数据，而2号就会被舍弃了；而如果还有4号数据的话，那么3号也会被舍弃，collect会处理4号。

```kotlin
val time = measureTimeMillis {
    simple() // simple每100ms发送一个数据
        .conflate() 
        .collect { value -> 
            delay(150) 
            println(value) 
        } 
}   
println("Collected in $time ms")
```

#### collectLastest

该操作符和conflate的思想一致，但是不同点在于，对于Flow提前准备好的数据，collectLatest在处理时，并不会直接舍弃最后一个之前的数据，而是都会尝试为每一个准备好的数据执行collectLatest，但是只会在最后一个数据处完成

### 6. 拼接Flow

这一部分负责将多个Flow拼接成一个。

#### zip函数

很简单，将多个Flow中数据序列逐个对应起来进行整合，最终生成的Flow中数据流中数据的个数为原来的Flow中数据流最小的那一个：

```kotlin
val nums = (1..2).asFlow()
val strs = flowOf("one", "two", "three") 
nums.zip(strs) { a, b -> "$a -> $b" } 
    .collect { println(it) } 
----------output------------
1 -> one
2 -> two
```

#### combine函数

### 7. Flatten变换

对于一个Flow对象来说，真正有用的是其发送的数据本身，因此Flow对象之间的嵌套关系并没有好处，而Flatten变换的作用就是将嵌套的Flow展开成独自的Flow。

#### flatMapConcat

该操作符的作用只有一个，那就是简单把嵌套的Flow按照emit的数据来展开：

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapConcat { requestFlow(it) }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
---------output--------
1: First at 121 ms from start
1: Second at 622 ms from start
2: First at 727 ms from start
2: Second at 1227 ms from start
3: First at 1328 ms from start
3: Second at 1829 ms from start
```

可以看到，只是将嵌套的Flow<Flow<Int>>展开成一层Flow，这里数据的emit与collect之间也没有并行动作。

#### flatMapMerge

该操作符就像是在flatMapConcat的基础上增加了conflate操作，即展开的Flow的各个数据的emit与collect之间是连续并行的关系：

```kotlin
val startTime = System.currentTimeMillis() // remember the start time 
(1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
    .flatMapMerge { requestFlow(it) }                                                                           
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
---------output----------
1: First at 136 ms from start
2: First at 231 ms from start
3: First at 333 ms from start
1: Second at 639 ms from start
2: Second at 732 ms from start
3: Second at 833 ms from start
```

#### flatMapLatest



