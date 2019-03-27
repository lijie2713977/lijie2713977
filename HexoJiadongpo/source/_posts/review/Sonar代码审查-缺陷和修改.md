---
title: Sonar代码审查-缺陷和修改
date: 2017-05-30 16:43:49
tags: [review]
categories: [review]
---
## Sonar代码审查-缺陷和修改
Resources should be closed
资源未关闭，打开发现有两处用到的IO流没有关闭

Conditions should not unconditionally evaluate to "TRUE" or to "FALSE"
if/else判断里出现了重复判断，比如在if(a>10)的执行体里面又判断if(a<0)，而后者肯定不会是true

Exception handlers should preserve the original exception
处理异常的时候应该保留原始的异常情况，不要直接来个catch(Exception e)了事

 Throwable.printStackTrace(...) should not be called
不应该直接调用e.printStackTrace()，而是用Loggers来处理（就是打Log）。

Loggers的优势是：Users are able to easily retrieve the logs.
The format of log messages is uniform and allow users to browse the logs easily.
Instance methods should not write to "static" fields6
不要用实例方法改变静态成员，理想情况下，静态变量只通过同步的静态方法来改变
 
"public static" fields should be constant
公共静态成员应该加上final，也就是public static final 一般不分家
 
Thread.run() and Runnable.run() should not be called directly
不应该直接调用Thread和Runnaale对象的run方法，直接调用run会使得run方法执行在当前线程，失去了开启新线程的意义。但有时候可能会这样做，下面有个例子。
 
Generic exceptions should never be thrown
不太理解，大意是说不要直接抛Error,RuntimeException/Throwable/Exception这样的通用的异常。我的具体应用是：throw new Error("Error copying database")，给出的建议是：Define and throw a dedicated exception instead of using a generic one（定义并抛出一个专用的异常来代替一个通用的异常）
 
Class variable fields should not have public accessibility
类变量不要设置为public，而是设为private，再提供get和set方法。
 
Sections of code should not be "commented out"
不要再注释中出现大量的代码段，会使代码可读性变差
 
Package declaration should match source file directory
这个没理解，包的声明应该与源文件目录匹配。
 
Utility classes should not have public constructors
工具类不应该有公共的构造器，也就是说至少要有一个private的构造器，如果没有，默认的构造器是public的。
 
The diamond operator ("<>") should be used
在定义集合的时候，等号右边的<>内不需要再写上元素类型，直接空着就行。
 
Lambdas and anonymous classes should not have too many lines
Lambdas表达式和匿名内部类不要写太多行，一般最多写20行。
 
Anonymous inner classes containing only one method should become lambdas8
只包含一个方法的匿名内部类应该写成Lambdas表达式的形式，增强代码可读性
 
Try-with-resources should be used8
用Try-with-resources的形式取代try/catch/finally的形式，这个有待于以后学习。
```try(Connection con = getConnection()) {
   try (PreparedStatement prep = con.prepareConnection("Update ...")) {
       //prep.doSomething();
       //...
       //etc
con.commit();
   } catch (SQLException e) {
       //any other actions necessary on failure
       con.rollback();
       //consider a re-throw, throwing a wrapping exception, etc
   }
}
```
 
Methods should not be empty
不要写空方法，除非这种情况：An abstract class may have empty methods, in order to provide default implementations for child classes.
 
Source files should not have any duplicated blocks
源文件中不要出现任何重复的代码段或行或字符串等。没理解。
 
"switch case" clauses should not have too many lines
"switch case" 每个case里面的代码不要太长，太长的话可以考虑写个方法代替，主要是为了增强代码可读性
 
Nested blocks of code should not be left empty
嵌套代码块不要是空的，比如 if( a > 0 ) {  doSomething()  } else { }，这时候应该把后面的else{}去掉。
 
Methods should not be too complex
方法不要太复杂，否则难以理解和维护。
 
Unused private fields should be removed
没有使用的private的成员变量应该移除掉。
 
Dead stores should be removed
没有用到的本地变量或其他死存储应该移除掉，也就是写方法的时候，定义的变量如果后来发现根本用不到，要记得删掉那行代码。
 
"switch" statements should end with a "default" clause
switch语句应该以default结束，这是一种defensive programming思想
 
Unused method parameters should be removed
没有用到的方法参数应该移除掉
 
Control flow statements "if", "for", "while", "switch" and "try" should not be nested too deeply4
if /for/while/try这样的嵌套不要太复杂
 
Useless parentheses around expressions should be removed to prevent any misunderstanding3
没有意义的括号不要随便加，以免造成误解，比如"="两边对象类型是相同的，就不要强转。
 
"for" loop stop conditions should be invariant
for循环的结果条件不能是变量，而应该是常量
 
"static" members should be accessed statically
static成员是与类、静态方法相联系的。
 
Catches should be combined
具体参考下面的18，我还没理解
 
Primitives should not be boxed just for "String" conversion
不要使用 4+" "这样的方式将int值转变为字符串，而是使用 Integer.toString(4)这样的方式。
就像Integer.parseInt("我是字符串")这样，不要偷懒。
 
Classes should not be empty
不要写空类
 
Unused local variables should be removed
没有用到的本地变量要删掉
 
"entrySet()" should be iterated when both the key and value are needed
直接看英文更直接：When only the keys from a map are needed in a loop, iterating the keySet makes sense. But when both the key and the value are needed, it's more efficient to iterate theentrySet, which will give access to both the key and value, instead.
也就是说，如果只需要Map的Key，那么直接iterate这个Map的keySet就可以了，但是如果Key和value都需要，就iterate这个Map。具体看下面的19.
 
Method parameters, caught exceptions and foreach variables should not be reassigned
方法参数/捕获的异常/foreach的变量不应该被重新赋值。
 
Collection.isEmpty() should be used to test for emptiness
当判断集合是否为空的时候，不要使用if (myCollection.size() == 0) 这样的方式，而是使用if (myCollection.isEmpty()这样的方式，后者性能更高。
 
Standard outputs should not be used directly to log anything
标准输出不直接打印任何东西，也就是打log的时候，不要使用System.out.println("My Message")这样的方式，而是使用logger.log("My Message")这种方式。
 
Generic wildcard types should not be used in return parameters
通配符不应该出现在返回声明中。比如这句：List <? extends Animal>getAnimals(){...}， 我们无法知道“是否可以把a Dog, a Cat 等加进去”，等之后用到这个方法的时候，我们没必要去考虑这种问题（前面引号里面的）。
 
Synchronized classes Vector, Hashtable, Stack and StringBuffer should not be used1
不要使用同步的Vector/HashTable/Stack/StringBuffer等。在早期，出于线程安全问题考虑，java API 提供了这些类。但是同步会极大影响性能，即使是在同一个线程中使用他们。
通常可以这样取代：
`ArrayList  or  LinkedList   instead of  Vector
Deque  instead of  Stack
HashMap  instead of  Hashtable
StringBuilder  instead of  StringBuffer
Exit methods should not be called`
尽量不要调用system.exit()方法。
 
Local Variables should not be declared and then immediately returned or thrown
本地变量如果赋值之后直接return了，那就直接return本地变量的赋值语句。
 
Field names should comply with a naming convention
命名要规范
 
Local variable and method parameter names should comply with a naming convention
命名要规范
 
String literals should not be duplicated5
字符串不应该重复，如果多次用到同一字符串，建议将该字符串定义为字符串常量，再引用。
 
Return of boolean expressions should not be wrapped into an "if-then-else" statement3
不要写if (  a > 4  ) {  return false  } else { return true }这样的代码，直接写return a > 4。
 
Static non-final field names should comply with a naming convention
命名要规范
 
Modifiers should be declared in the correct order
修饰符等要按约定俗成的顺序书写 ，例如，写成public static 而不是static public 
 
The members of an interface declaration or class should appear in a pre-defined order2
与前面的一个问题类似，根据Oracle定义的Java代码规范中，不同代码的出现位置应该如下所示：
class and instance variables--Constructors--Methods
 
Array designators "[]" should be on the type, not the variable
数组的括号要写在类型后面，而不是变量后面，例如 int[] a 而不是int a[]
 
Multiple variables should not be declared on the same line1
不要在同一行定义多个变量
 
"switch" statements should have at least 3 "case" clauses
当至少有3种或者3种以上的情况时，才考虑用switch，否则用if/else的形式。
 
Overriding methods should do more than simply call the same method in the super class
既然在子类中重写了父类的某个方法，那就再这个方法中做些与父类方法不同的事情，否则没必要重写。
 
Statements should be on separate lines
不要把这样的代码写在同一行：if(someCondition)    doSomething()；而是应该写成下面的形式
if(someCondition) {
doSomething()
} 
Method names should comply with a naming convention1
命名要规范 
"TODO" tags should be handle    TODO标签要及时处理，该做的事情不要忘了做


## 部分规则详细说明
#### 1.The members of an interface declaration or class should appear in a pre-defined order
正确的顺序如下所示：静态成员变量→成员变量→构造器→方法

```public class Foo{
public static final int OPEN = 4;  //Class and instance variables
private int field = 0;
public Foo() {...}    //Constructors
public boolean isTrue() {...}    //Methods
}

```
#### 2.The diamond operator ("<>") should be used
Noncompliant Code Example：不规范的示例

```List<String>  strings = new ArrayList<String>();  // Noncompliant
Map<String, List<Integer>> map = new HashMap<String, List<Integer>>();  // Noncompliant
Compliant Solution ：规范的示例
List<String> strings = new ArrayList<>();
Map<String, List<Integer>> map = new HashMap<>();
```

#### 3.Sections of code should not be "commented out"
代码片段不应该出现在注释中，这样会bloat程序，可读性变差
`Programmers should not comment out code as it bloats programs and reduces readability.
Unused code should be deleted and can be retrieved from source control history if required.`

#### 4.Utility classes should not have public constructors
工具类不应该有public的构造器，也就是工具类至少要定义一个non-public的构造器
Utility classes, which are a collection of static members, are not meant to be instantiated. Even abstract utility classes, which can be extended, should not have public constructors.
Java adds an implicit public constructor to every class which does not define at least one explicitly. Hence, at least one non-public constructor should be defined.

```class StringUtils { // Noncompliant Code Example
    public static String concatenate(String s1, String s2) {
          return s1 + s2;
    }
}
```


```class StringUtils { //Compliant Solution
    private StringUtils() {
    }
    public static String concatenate(String s1, String s2) {
    return s1 + s2;
    }
}
```
#### 5."public static" fields should be constant
公共的静态成员应该加上final来修饰
There is no good reason to declare a field "public" and "static" without also declaring it "final". Most of the time this is a kludge to share a state among several objects. But with this approach, any object can do whatever it wants with the shared state, such as setting it to null.
public static Foo foo = new Foo();//不规范的
public static final Foo FOO = new Foo();//规范的

6.Class variable fields should not have public accessibility

```public class MyClass {
public static final int SOME_CONSTANT = 0;    // Compliant - constants are not checked
public String firstName;                      // Noncompliant
}
```


```public class MyClass {
public static final int SOME_CONSTANT = 0;    // Compliant - constants are not checked
private String firstName;                      // Compliant
public String getFirstName() {
return firstName;
}
public void setFirstName(String firstName) {
this.firstName = firstName;
}
}
#### 7.Static non-final field names should comply with a naming convention
public final class MyClass {//Noncompliant Code Example
      private static String foo_bar;
}
class MyClass {//Compliant Solution
private static String fooBar;
}

```

#### 8."switch" statements should have at least 3 "case" clauses
当有3种或3种情况以上的时候，才用switch，否则用if/else
switch statements are useful when there are many different cases depending on the value of the same expression.
For just one or two cases however, the code will be more readable with if statements.

#### 9.String literals should not be duplicated

```prepare("action1");     // Noncompliant - "action1" is duplicated 3 times
execute("action1");
release("action1");

private static final String ACTION_1 = "action1";  // Compliant
prepare(ACTION_1);                                            // Compliant
execute(ACTION_1);
release(ACTION_1);
```


#### 10.Return of boolean expressions should not be wrapped into an "if-then-else” statement
Replace this if-then-else statement by a single return statement

```if (expression) {//Noncompliant Code Example
      return true;
} else {
     return false;
}
return expression;//Compliant Solution
return !!expression;
```

#### 11.Method parameters, caught exceptions and foreach variables should not be reassigned
方法参数，捕获的异常，foreach里的变量，都不应该重新赋值

```class MyClass {//Noncompliant Code Example：不规范代码示例
    public String name;
    public MyClass(String name) {
            name = name;          // Noncompliant - useless identity assignment
    }
    public int add(int a, int b) {
        a = a + b;                // Noncompliant
        return a;                 // Seems like the parameter is returned as is, what is the point?
   }
    public static void main(String[] args) {
        MyClass foo = new MyClass();
        int a = 40;
        int b = 2;
        foo.add(a, b);                  // Variable "a" will still hold 40 after this call
    }
}

```

```class MyClass {//Compliant Solution：规范代码示例
    public String name;
    public MyClass(String name) {
         this.name = name;              // Compliant
    }
    public int add(int a, int b) {
        return a + b;                  // Compliant
    }
    public static void main(String[] args) {
    MyClass foo = new MyClass();
        int a = 40;
        int b = 2;
        foo.add(a, b);
     }
}
```


#### 12.Local Variables should not be declared and then immediately returned or thrown
Noncompliant Code Example：不规范代码示例

```public long computeDurationInMilliseconds() {
long duration = (((hours * 60) + minutes) * 60 + seconds ) * 1000 ;
return duration;
}
public void doSomething() {
RuntimeException myException = new RuntimeException();
throw myException;
}
```

Compliant Solution：规范代码示例

```public long computeDurationInMilliseconds() {
return (((hours * 60) + minutes) * 60 + seconds ) * 1000 ;
}
public void doSomething() {
throw new RuntimeException();
}
```

#### 13.Thread.run() and Runnable.run() should not be called directly
The purpose of theThread.run()andRunnable.run()methods is to execute code in a separate, dedicated thread. Calling those methods directly doesn't make sense because it causes their code to be executed in the current thread.
Thread和Runnable里面的run方法设计的目的是让run方法里面的代码在不同的线程中执行。如果直接调用
```run方法，就会导致run方法里的代码在当前线程中执行，失去意义
Noncompliant Code Example：不规范的代码示例
Thread myThread = new Thread(runnable);
myThread.run(); // Noncompliant

Compliant Solution：规范代码示例
Thread myThread = new Thread(runnable);
myThread.start(); // Compliant
```

这部分内容为个人理解，可以略过
但在有些情况，也会直接调用Runnable的run方法，
下面这个postTaskSafely方法会保证task永远在主线程中执行

```public static void postTaskInMainThread(Runnable task) {
     int curThreadId= android.os.Process.myTid();//得到当前线程的id
    if(curThreadId==getMainThreadId()) {// 如果当前线程是主线程
            task.run();//直接执行
    }else{// 如果当前线程不是主线程
        getMainThreadHandler().post(task);//用主线程的Handler来post
}
```

#### 14.Lambdas and anonymous classes should not have too many lines
Anonymous classes and lambdas (with Java 8) are a very convenient and compact way to inject a behavior without having to create a dedicated class. But those anonymous inner classes and lambdas should be used only if the behavior to be injected can be defined in a few lines of code, otherwise the source code can quickly become unreadable.
anonymous class number of lines ： at most 20

#### 15.Resources should be closed：该关闭的一定记得关闭
Java's garbage collection cannot be relied on to clean up everything. Specifically, connections, streams, files and other classes that implement theCloseableinterface or it's super-interface,AutoCloseable, must be manually closed after creation. Failure to do so will result in a resource leak which could bring first the application and then perhaps the box it's on to their knees.
Noncompliant Code Example：不规范的代码示例
   
``` OutputStream stream = null;
    try{
        for (String property : propertyList) {
        stream = new FileOutputStream("myfile.txt");  // Noncompliant
        // ...
        }
    }catch(Exception e){
        // ...
    }finally{
        stream.close();  // Multiple streams were opened. Only the last is closed.
    }

```
Compliant Solution：规范代码示例
  
```  OutputStream stream = null;
    try{
        stream = new FileOutputStream("myfile.txt");
        for (String property : propertyList) {
            // ...
        }
   }catch(Exception e){
        // ...
   }finally{
       stream.close();
   }

```
#### 16.Exception handlers should preserve the original exception
Noncompliant Code Example:不规范的代码示例
// Noncompliant - exception is lost
try { /* ... */ } catch (Exception e) { LOGGER.info("context"); }
// Noncompliant - exception is lost (only message is preserved)
try { /* ... */ } catch (Exception e) { LOGGER.info(e.getMessage()); }
// Noncompliant - exception is lost
try { /* ... */ } catch (Exception e) { throw new RuntimeException("context"); }

Compliant Solution:规范的代码示例

```try { /* ... */ } catch (Exception e) { LOGGER.info(e); }
try { /* ... */ } catch (Exception e) { throw new RuntimeException(e); }
try {
/* ... */
} catch (RuntimeException e) {
doSomething();
throw e;
} catch (Exception e) {
// Conversion into unchecked exception is also allowed
throw new RuntimeException(e);
}

```

#### 17.Catches should be combined
Since Java 7 it has been possible to catch multiple exceptions at once. Therefore, when multiplecatchblocks have the same code, they should be combined for better readability.
Note that this rule is automatically disabled when the project'ssonar.java.sourceis lower than7.


Noncompliant Code Example：不规范代码示例

```catch (IOException e) {
    doCleanup();
    logger.log(e);
}catch (SQLException e) { //Noncompliant
    doCleanup();
    logger.log(e);
 }catch (TimeoutException e) {  // Compliant; block contents are different
     doCleanup();
     throw e;
 }

```
Compliant Solution：规范代码示例

```catch (IOException|SQLException e) {
    doCleanup();
    logger.log(e);
 }catch (TimeoutException e) {
    doCleanup();
    throw e;
}
```
18."entrySet()" should be iterated when both the key and value are needed
Noncompliant Code Example：不规范的代码示例

```public void doSomethingWithMap(Map map) {
for (String key : map.keySet()) {  // Noncompliant; for each key the value is retrieved
Object value = map.get(key);
// ...
}   
}   

```
Compliant SolutionL：规范代码示例

```public void doSomethingWithMap(Map map) {
for (Map.Entry entry : map.entrySet()) {
String key = entry.getKey();
Object value = entry.getValue();
// ...
}   
}   
```

Use a logger to log this exception
You should probably clarify which logger are you using.

org.apache.commons.logging.Log interface has method void error(Object message, Throwable t) (and method void info(Object message, Throwable t)), which logs the stack trace together with your custom message. Log4J implementation has this method too.

So, probably you need to write:
logger.error("BOOM!", e);
If you need to log it with INFO level (though, it might be a strange use case), then:
logger.info("Just a stack trace, nothing to worry about", e);
Hope it helps.

Rename 'i' as this name is already used in declaration at line 26

## 阻断

#### 1、Close this"FileInputStream" in a "finally" clause.
在finally中关闭FileInputStream，这个最为常见，主要是关闭方式不对，finally代码块中，应该要对每个stream进行单独关闭，而不能统一写在一个try-catch代码中，jdk 7 可以考虑try-resources方式关闭，代码相对优雅。
另外数据库操作的statement和resultRs没有关闭的情况也非常多。这个也是要重点留意的地方。

#### 2、A"NullPointerException" could be thrown; "tom" is nullablehere
空指针，极为讨厌的问题，主要是编码经验缺少的体现。一般的高手在编码过程中，就会第一时间考虑到这类情况，并做相应的处理。解决方式无它，先判断或者先实例化，再访问里面的属性或者成员。

## 严重
#### 1、Define and throw a dedicated exception instead of using a generic one
定义并抛出一个专用的异常来代替一个通用的异常。
#### 2、Removethis hard-coded password
移除代码里硬编码的密码信息。会有少许的误判的情况，一般是变量包含：PWD或者password，所以如果真的不是硬编码，可以考虑更换变量名称，比如PWD改PW等等。
#### 3、Eitherlog or rethrow this exception
catch异常之后，使用log方式或者throw异常的方式解决。如果业务上真的没有throw或者记录日志的话，可以使用log.debug的方式填充来解决问题。

#### 4、Makethis IP "127.0.0.1" address configurable
将IP弄到配置文件中，不要硬编码到代码里。个人觉得改动稍大！
#### 5、Make this"public static JSAPI" field final
如果你将这个变量设置为public访问方式，同时又是静态Static方式，就要考虑将它设置为final了，因为这个是共享变量，其它类可以随时随地将它设置为别的值。所以如果是只是当前类使用，可以考虑将公开访问方式改为私有。
#### 6、Makethe enclosing method "static" or remove this set
见代码：

```public class MyClass {
  private static int count = 0;
  public void doSomething() {
    //...
    count++;  // Noncompliant
  }
}

```不要使用非静态方法去更新静态字段，这样很难获得正确的结果，如果有多个类实例和/或多个线程，则很容易导致错误。理想情况下，静态字段仅从同步静态方法中更新。
#### 7、Override"equals(Object obj)" to comply with the contract of the"compareTo(T o)" method
如果重写了compareTo方法，同时也应重写equals方法。
#### 8、Make"body" transient or serializable.

```public class Address {
  //...
}
public class Person implements Serializable {
  private static final long serialVersionUID = 1905122041950251207L;
  private String name;

  private Address address;  // Noncompliant; Address isn't serializable
}

```
如果person已经序列化，其成员变量Address也进行序列化。不然转化时会有问题。
#### 9 Floating point numbers should not be tested for equality

浮点类型的数字，不要通过==或者!=方式其它类型比较，因为浮点是不精确的，所以在比较时，会进行类型升级升级原则如下：
·  如果运算符任意一方的类型为double，则另一方会转换为double
·  否则，如果运算符任意一方的类型为float，则另一方会转换为float
·  否则，如果运算符任意一方的类型为long，则另一方会转换为long
·  否则，两边都会转换为int
 

以下的方式得到的结果都是false。

```
float myNumber = 3.146;
if ( myNumber == 3.146f ) { //Noncompliant. Because of floating point imprecision, this will be false
 
if ( myNumber != 3.146f ) { //Noncompliant. Because of floating point imprecision, this will be true
if (myNumber < 4 || myNumber > 4) { // Noncompliant; indirect  
float zeroFloat = 0.0f;
if (zeroFloat == 0) {  // Noncompliant. Computations may end up with a value close but not equal to zero.
}
 
```

所以，要比较浮点数是否相等，需要做的事情是：

    排除NaN和无穷

    在精度范围内进行比较

正确的例子：


```public boolean isEqual(double a, double b) {
    if (Double.isNaN(a) || Double.isNaN(b) || Double.isInfinite(a) || Double.isInfinite(b)) {
       return false;
    }
    return (a - b) < 0.001d;
}
```

 

#### 10、Thiscall to "contains()" may be a performance hot spot if the collectionis large.
如果collection的记录数非常大的话，它的contains方法的时间复杂度是很高的。所以开发过程中要注意这一点。下面的是列表：


```·  ArrayList

contains
remove
·  LinkedList

get
contains
·  ConcurrentLinkedQueue

size
contains
·  ConcurrentLinkedDeque

size
contains
·  CopyOnWriteArrayList

add
contains
remove
·  CopyOnWriteArraySet

add
contains
remove

```

##### 1.Equality tests should not be made with floating point value
　　代码举例： if (result == num) //result和num均为double 之间比较会有精度损失
　　解决：BigDecimal data1 = new BigDecimal(totalArea);
　　　　　BigDecimal data2 = new BigDecimal(s1);
　　　　    int num = data1.compareTo(data2);//num =0 相等  >0前者大于后者 ，反之 <0 前者小于后者
##### 2.This class overrides "equals()" and should therefore also override "hashCode()".　
　　代码举例：public boolean equals(Object obj){...}  //需要添加对应的hashCode方法
　　解决：可以添加一个最简单的hashCode方法　　
　　　　　public int hashCode() {return 0;}　　　
##### 3.Synchronize on a new "Object" instead
　　代码举例：synchronized ("实例化") {...}   //里边必须是对象
　　解决： private Object obj ="实例化"；
　　　　　synchronized (obj ) {...}
##### 4.Close this"FileInputStream" in a "finally" clause.
　　解决方法: 在finally中关闭FileInputStream，主要是关闭方式不对，finally代码块中，应该要对每个stream进行单独关闭，而不能统一写在一个try-catch代码中。

##### 5.A"NullPointerException" could be thrown; "tom" is nullablehere
　　空指针，解决方式：先判断或者先实例化，再访问里面的属性或者成员。
##### 6.Makethis IP "127.0.0.1" address configurable
　　解决方法:不要把IP地址写在此类中，应该在对应的系统文件或者相应的配置文件中配置
##### 7.Either log or rethrow this exception.
　　解决方法: 把对应的输出写成Logger.error("aaa“);的形式 






