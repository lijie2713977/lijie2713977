---
title: Findbug代码审查-缺陷和修改
date: 2017-05-30 16:43:49
tags: [review]
categories: [review]
---

# FindBugs使用实践-缺陷和修改
Confidence 是fingbug团队认为该代码导致bug的可能性。

## 分类
1 Malicious code vulnerability 恶意代码
2 Performance 性能问题
3 Security 安全性问题
4 Dodgy code 小问题
5 Correctness 代码的正确性
6 Bad practice 不好的习惯
7 Internationalization 国际化问题
8 Experrimental 实验性问题
无     Multithreaded currectness 线程问题

## 常见问题
#### 1 Bad practice 坏的实践
一些不好的实践，下面列举几个： HE：类定义了equals()，却没有hashCode()；或类定义了equals()，却使用Object.hashCode()；或类定义了hashCode()，却没有equals()；或类定义了hashCode()，却使用Object.equals()；类继承了equals()，却使用Object.hashCode()。 
SQL：Statement 的execute方法调用了非常量的字符串；或Prepared Statement是由一个非常量的字符串产生。 
DE：方法终止或不处理异常，一般情况下，异常应该被处理或报告，或被方法抛出。 
Malicious code vulnerability 可能受到的恶意攻击
如果代码公开，可能受到恶意攻击的代码，下面列举几个： FI：一个类的finalize()应该是protected，而不是public的。MS：属性是可变的数组；属性是可变的Hashtable；属性应该是package protected的。
#### 2 Correctness 一般的正确性问题
可能导致错误的代码，下面列举几个： NP：空指针被引用；在方法的异常路径里，空指针被引用；方法没有检查参数是否null；null值产生并被引用；null值产生并在方法的异常路径被引用；传给方法一个声明为@NonNull的null参数；方法的返回值声明为@NonNull实际是null。 Nm：类定义了hashcode()方法，但实际上并未覆盖父类Object的hashCode()；类定义了tostring()方法，但实际上并未覆盖父类Object的toString()；很明显的方法和构造器混淆；方法名容易混淆。 SQL：方法尝试访问一个Prepared Statement的0索引；方法尝试访问一个ResultSet的0索引。 UwF：所有的write都把属性置成null，这样所有的读取都是null，这样这个属性是否有必要存在；或属性从没有被write。
#### 3 Dodgy 危险的
具有潜在危险的代码，可能运行期产生错误，下面列举几个： CI：类声明为final但声明了protected的属性。 DLS：对一个本地变量赋值，但却没有读取该本地变量；本地变量赋值成null，却没有读取该本地变量。 ICAST：整型数字相乘结果转化为长整型数字，应该将整型先转化为长整型数字再相乘。 INT：没必要的整型数字比较，如X <= Integer.MAX_VALUE。 NP：对readline()的直接引用，而没有判断是否null；对方法调用的直接引用，而方法可能返回null。 REC：直接捕获Exception，而实际上可能是RuntimeException。 ST：从实例方法里直接修改类变量，即static属性。
#### 4 Performance 性能问题
可能导致性能不佳的代码，下面列举几个： DM：方法调用了低效的Boolean的构造器，而应该用Boolean.valueOf(…)；用类似Integer.toString(1) 代替new Integer(1).toString()；方法调用了低效的float的构造器，应该用静态的valueOf方法。 SIC：如果一个内部类想在更广泛的地方被引用，它应该声明为static。 SS：如果一个实例属性不被读取，考虑声明为static。 UrF：如果一个属性从没有被read，考虑从类中去掉。 UuF：如果一个属性从没有被使用，考虑从类中去掉。
#### 5 Multithreaded correctness 多线程的正确性多线程编程时，可能导致错误的代码，下面列举几个：
ESync：空的同步块，很难被正确使用。 MWN：错误使用notify()，可能导致IllegalMonitorStateException异常；或错误的使用wait()。 No：使用notify()而不是notifyAll()，只是唤醒一个线程而不是所有等待的线程。 SC：构造器调用了Thread.start()，当该类被继承可能会导致错误。
#### 6 Internationalization 国际化当对字符串使用upper或lowercase方法，如果是国际的字符串，可能会不恰当的转换。

API： http://findbugs.sourceforge.net/api/index.html
技术手册： http://findbugs.sourceforge.net/manual/index.html
更多请参见官网： http://findbugs.sourceforge.net/bugDescriptions.html
##### 6.1、       ES_COMPARING_PARAMETER_STRING_WITH_EQ
     ES: Comparison of String parameter using == or != (ES_COMPARING_PARAMETER_STRING_WITH_EQ)
This code compares a java.lang.String parameter for reference equality using the == or != operators. Requiring callers to pass only String constants or interned strings to a method is unnecessarily fragile, and rarely leads to measurable performance gains. Consider using the equals(Object) method instead.
     使用 == 或者 != 来比较字符串或interned字符串，不会获得显著的性能提升，同时并不可靠，请考虑使用equals()方法。
##### 6.2、       HE_EQUALS_NO_HASHCODE
     HE: Class defines equals() but not hashCode() (HE_EQUALS_NO_HASHCODE)
This class overrides equals(Object), but does not override hashCode().  Therefore, the class may violate the invariant that equal objects must have equal hashcodes.
     类定义了equals()方法但没有重写hashCode()方法，这样违背了相同对象必须具有相同的hashcodes的原则
##### 6.3、IT_NO_SUCH_ELEMENT
     It: Iterator next() method can't throw NoSuchElement exception (IT_NO_SUCH_ELEMENT)
This class implements the java.util.Iterator interface.  However, its next() method is not capable of throwing java.util.NoSuchElementException.  The next() method should be changed so it throws NoSuchElementException if is called when there are no more elements to return.
     迭代器Iterator无法抛出NoSuchElement异常，类实现了java.util.Iterator接口，但是next()方法无法抛出java.util.NoSuchElementException异常，因此，next()方法应该做如此修改，当被调用时，如果没有element返回，则抛出NoSuchElementException异常
##### 6.4、J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION
     J2EE: Store of non serializable object into HttpSession (J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION)
This code seems to be storing a non-serializable object into an HttpSession. If this session is passivated or migrated, an error will result.
     将没有实现serializable的对象放到HttpSession中，当这个session被钝化和迁移时，将会产生错误，建议放到HttpSession中的对象都实现serializable接口。
##### 6.5、ODR_OPEN_DATABASE_RESOURCE
     ODR: Method may fail to close database resource (ODR_OPEN_DATABASE_RESOURCE)
The method creates a database resource (such as a database connection or row set), does not assign it to any fields, pass it to other methods, or return it, and does not appear to close the object on all paths out of the method.  Failure to close database resources on all paths out of a method may result in poor performance, and could cause the application to have problems communicating with the database.
     方法可能未关闭数据库资源，未关闭数据库资源将会导致性能变差，还可能引起应用与服务器间的通讯问题。
##### 6.6、OS_OPEN_STREAM
     OS: Method may fail to close stream (OS_OPEN_STREAM)
The method creates an IO stream object, does not assign it to any fields, pass it to other methods that might close it, or return it, and does not appear to close the stream on all paths out of the method.  This may result in a file descriptor leak.  It is generally a good idea to use a finally block to ensure that streams are closed.
     方法可能未关闭stream，方法产生了一个IO流，却未关闭，将会导致文件描绘符的泄漏，建议使用finally block来确保io stream被关闭。
##### 6.7、       DMI_CALLING_NEXT_FROM_HASNEXT
     DMI: hasNext method invokes next (DMI_CALLING_NEXT_FROM_HASNEXT)
The hasNext() method invokes the next() method. This is almost certainly wrong, since the hasNext() method is not supposed to change the state of the iterator, and the next method is supposed to change the state of the iterator.
##### 6.8、       IL_INFINITE_LOOP
     IL: An apparent infinite loop (IL_INFINITE_LOOP)
This loop doesn't seem to have a way to terminate (other than by perhaps throwing an exception).
     明显的无限循环.
##### 6.9、       IL_INFINITE_RECURSIVE_LOOP
     IL: An apparent infinite recursive loop (IL_INFINITE_RECURSIVE_LOOP)
This method unconditionally invokes itself. This would seem to indicate an infinite recursive loop that will result in a stack overflow.
     明显的无限迭代循环,将导致堆栈溢出.
##### 6.10、   WMI_WRONG_MAP_ITERATOR
     WMI: Inefficient use of keySet iterator instead of entrySet iterator (WMI_WRONG_MAP_ITERATOR)
This method accesses the value of a Map entry, using a key that was retrieved from a keySet iterator. It is more efficient to use an iterator on the entrySet of the map, to avoid the Map.get(key) lookup.
     使用了keySet iterator和Map.get(key)来获取Map值,这种方式效率低,建议使用entrySet的iterator效率更高.
##### 6.11、   IM_BAD_CHECK_FOR_ODD
     IM: Check for oddness that won't work for negative numbers (IM_BAD_CHECK_FOR_ODD)
The code uses x % 2 == 1 to check to see if a value is odd, but this won't work for negative numbers (e.g., (-5) % 2 == -1). If this code is intending to check for oddness, consider using x & 1 == 1, or x % 2 != 0.
     奇偶检测逻辑,未考虑负数情况.
#### 7.实际项目中Bug类型统计
##### 7.1、       Call to equals() comparing different types
id : EC_UNRELATED_TYPES, type : EC, category : CORRECTNESS This method calls equals(Object) on two references of different class types with no common subclasses. Therefore, the objects being compared are unlikely to be members of the same class at runtime (unless some application classes were not analyzed, or dynamic class loading can occur at runtime). According to the contract of equals(), objects of different classes should always compare as unequal; therefore, according to the contract defined by java.lang.Object.equals(Object), the result of this comparison will always be false at runtime.
原因分析：
这缺陷的意思是，大部分都是类型永远不会有这种情况， 比如a为DOUBLE类型所以EQUALS只匹配字符串 if(a.equals())或if(a.quals())这类判断是根本不会有用的；
示例：if("1".equals(DAOValue.valueofSuccess()))
##### 7.2、       Dead store to local variable 
id: DLS_DEAD_LOCAL_STORE, type: DLS, category: STYLE
This instruction assigns a value to a local variable, but the value is not read or used in any subsequent instruction. Often, this indicates an error, because the value computed is never used.
Note that Sun's javac compiler often generates dead stores for final local variables. Because FindBugs is a bytecode-based tool, there is no easy way to eliminate these false positives.
原因分析：
DLS问题指的是给本地变量赋了一个值，但随后的代码并没有用到这个值。
##### 7.3、       Method call passes null for nonnull parameter
id: NP_NULL_PARAM_DEREF, type: NP, category: CORRECTNESS
This method call passes a null value for a nonnull method parameter. Either the parameter is annotated as a parameter that should always be nonnull, or analysis has shown that it will always be dereferenced.
原因分析：对参数为null的情况未作处理。
例如：

```public void method1() {
    String ip = null;
    try {
        ip = InetAddress.getLocalHost().getHostAddress();
    } catch (UnknownHostException e) {
        e.printStackTrace();
    }
    long ipCount = countIpAddress(ip);//可能会传入空引用 
    //....
}

long countIpAddress(String ip) {
    long ipNum = 0;
    String[] ipArray = ip.split("\\.");
}
```

```public void method1() {
        String ip = null;
        try {
            ip = InetAddress.getLocalHost().getHostAddress();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        long ipCount = countIpAddress(ip);//可能会传入空引用
        //....
    }

    long countIpAddress(String ip) {
        long ipNum = 0;
        if(ip==null){
            return 0;//抛出异常
        }
        String[] ipArray = ip.split("\\.");
    }
```
对于接口需要对参数校验合法性

##### 7.4、Method with Boolean return type returns explicit null  
id: NP_BOOLEAN_RETURN_NULL, type: NP, category: BAD_PRACTICE
A method that returns either Boolean.TRUE, Boolean.FALSE or null is an accident waiting to happen. This method can be invoked as though it returned a value of type boolean, and the compiler will insert automatic unboxing of the Boolean value. If a null value is returned, this will result in a NullPointerException.
原因分析：
方法如果定义为返回类型Boolean，则可以返回Boolean.TRUE, Boolean.FALSE or null （如果 return 的是 true or  false， 则AutoBoxing 成 Boolean.TRUE, Boolean.FALSE）。因为JDK 支持 基本类型和装箱类型的自动转化， 所以下面的代码中：
boolean result = test_NP_BOOLEAN_RETURN_NULL();
因为此时test_NP_BOOLEAN_RETURN_NULL() 返回的是NULL， 所以 JDK 做 automatic unboxing 的操作时， 即调用了 object. booleanValue() 方法时，抛出了空指针。
改成：boolean result = test_NP_BOOLEAN_RETURN_NULL()==null?false:true;
##### 7.5、       No relationship between generic parameter and method argument
id: GC_UNRELATED_TYPES, type: GC, category: CORRECTNESS
This call to a generic collection method contains an argument with an incompatible class from that of the collection's parameter (i.e., the type of the argument is neither a supertype nor a subtype of the corresponding generic type argument). Therefore, it is unlikely that the collection contains any objects that are equal to the method argument used here. Most likely, the wrong value is being passed to the method.
In general, instances of two unrelated classes are not equal. For example, if the Foo and Bar classes are not related by subtyping, then an instance of Foo should not be equal to an instance of Bar. Among other issues, doing so will likely result in an equals method that is not symmetrical. For example, if you define the Foo class so that a Foo can be equal to a String, your equals method isn't symmetrical since a String can only be equal to a String.
In rare cases, people do define nonsymmetrical equals methods and still manage to make their code work. Although none of the APIs document or guarantee it, it is typically the case that if you check if a Collection<String> contains a Foo, the equals method of argument (e.g., the equals method of the Foo class) used to perform the equality checks.
原因分析：调用Collection类中的contains方法比较时，所比较的两个参数类型不致；

##### 7.6、       Null pointer dereference in method on exception path
id: NP_ALWAYS_NULL_EXCEPTION, type: NP, category: CORRECTNESS
A pointer which is null on an exception path is dereferenced here.  This will lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
原因分析：在异常处理时，调用一个空对象的方法时可能引起空指针异常。
##### 7.7、       Nullcheck of value previously dereferenced
id: RCN_REDUNDANT_NULLCHECK_WOULD_HAVE_BEEN_A_NPE, type:RCN, category: CORRECTNESS
A value is checked here to see whether it is null, but this value can't be null because it was previously dereferenced and if it were null a null pointer exception would have occurred at the earlier dereference. Essentially, this code and the previous dereference disagree as to whether this value is allowed to be null. Either the check is redundant or the previous dereference is erroneous.
原因分析：前面获取的对象，现在引用的时候没有交验是否为null。

##### 7.8、       Possible null pointer dereference
id: NP_NULL_ON_SOME_PATH, type: NP, category: CORRECTNESS
There is a branch of statement that, if executed, guarantees that a null value will be dereferenced, which would generate a NullPointerException when the code is executed. Of course, the problem might be that the branch or statement is infeasible and that the null pointer exception can't ever be executed; deciding that is beyond the ability of FindBugs.
原因分析：可能存在空引用。
 
##### 7.9、       Possible null pointer dereference in method on exception path
id: NP_NULL_ON_SOME_PATH_EXCEPTION, type: NP, category:CORRECTNESS
A reference value which is null on some exception control path is dereferenced here.  This may lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
原因分析：
代码调用时， 遇到异常分支， 可能造成一个对象没有获得赋值依旧保持NULL空指针。 接下来如果对这个对象有引用， 可能造成NullPointerException 空指针异常。
例如：

##### 7.10、   Test for floating point equality
id: FE_FLOATING_POINT_EQUALITY, type: FE, category: STYLE
This operation compares two floating point values for equality. Because floating point calculations may involve rounding, calculated float and double values may not be accurate. For values that must be precise, such as monetary values, consider using a fixed-precision type such as BigDecimal. For values that need not be precise, consider comparing for equality within some range, for example: if ( Math.abs(x - y) < .0000001 ). See the java Language Specification, section 4.2.4.
原因分析：
Float类型的数据比较时，会存在的定的误差值，用!=来比较不是很准确，建议比较两个数的绝对值是否在一定的范围内来进行比较。如，if ( Math.abs(x - y) < .0000001 )
例如：
 
```  if(kpje.doubleValue()!=Doble.valueOf(coreCrdeInfo.getOrderpayInfo().getyeje（）)){
        return ResultVo.valueOfError("订单编号");
    }

```
##### 7.11、   Useless assignment in return statement
id: DLS_DEAD_LOCAL_STORE_IN_RETURN, type: DLS, category: STYLE
This statement assigns to a local variable in a return statement. This assignment has effect. Please verify that this statement does the right thing.
原因分析：
在return的对象中，没有必要通过对象赋值再进行返回。
例如：

``` DAOValue daoValue = DAOValue.valueof(false,-1,"批量删除联系人业务大类信息失败");
    if(lxrCpdlVoList.size()==0){
        return daoValue = DAOValue.valueof(true,1,"没有批量删除联系人业务大类信息失败");
    }
```
##### 7.12、   Write to static field from instance method
id: ST_WRITE_TO_STATIC_FROM_INSTANCE_METHOD, type: ST, category:STYLE
This instance method writes to a static field. This is tricky to get correct if multiple instances are being manipulated, and generally bad practice.
原因分析：向static字段中写入值。
例如： 
``` private static DBRBO dbrBO; 
 public final void refresh() { 
        danskeBankBO = null; 
        dbrBO = null; 
        fileAndPathBO = null; 
    }
``` 
建议改为：去掉static。
##### 7.13、   Incorrect lazy initialization and update of static field
id: LI_LAZY_INIT_UPDATE_STATIC, type: LI, category: MT_CORRECTNESS
This method contains an unsynchronized lazy initialization of a static field. After the field is set, the object stored into that location is further updated or accessed. The setting of the field is visible to other threads as soon as it is set. If the futher accesses in the method that set the field serve to initialize the object, then you have a very seriousmultithreading bug, unless something else prevents any other thread from accessing the stored object until it is fully initialized.
Even if you feel confident that the method is never called by multiple threads, it might be better to not set the static field until the value you are setting it to is fully populated/initialized.
原因分析：
该方法的初始化中包含了一个迟缓初始化的静态变量。你的方法引用了一个静态变量，估计是类静态变量，那么多线程调用这个方法时，你的变量就会面临线程安全的问题了，除非别的东西阻止任何其他线程访问存储对象从直到它完全被初始化。
##### 7.14、   Method ignores return value
id: RV_RETURN_VALUE_IGNORED, type: RV, category: CORRECTNESS
The return value of this method should be checked. One common cause of this warning is to invoke a method on an immutable object, thinking that it updates the object. For example, in the following code fragment,
String dateString = getHeaderField(name);
dateString.trim();
the programmer seems to be thinking that the trim() method will update the String referenced by dateString. But since Strings are immutable, the trim() function returns a new String value, which is being ignored here. The code should be corrected to:
String dateString = getHeaderField(name);
dateString = dateString.trim();
原因分析：方法忽略了设置返回值。
例如：
 
```String dateString = getHeaderField(name);
    dateString.trim();
    
    if(CustomActionEnum.Agee_SRT.equals(operationType)||CustomActionEnum.DISSAgee_SRT.equals(operationType){
        //....
    }
```
##### 7.15、   Method might ignore exception
id: DE_MIGHT_IGNORE, type: DE, category: BAD_PRACTICE
This method might ignore an exception.Â  In general, exceptions should be handled or reported in some way, or they should be thrown out of the method.
原因分析：应该将异常 处理、打印或者抛出
例如：
```try{
     //....
 }catch(Exception e){
     
 }
```

##### 7.16、   Unwritten field
id: UWF_UNWRITTEN_FIELD, type: UwF, category: CORRECTNESS
This field is never written.Â  All reads of it will return the default value. Check for errors (should it have been initialized?), or remove it if it is useless.
原因分析：从未被初始化的变量，调用它时，将返回默认值，要么初始化，要么删掉它。
例如：
```try{
        //....
    }catch(Exception e){
        
    }
```

##### 7.17、   Value is null and guaranteed to be dereferenced on exception path
id: NP_GUARANTEED_DEREF_ON_EXCEPTION_PATH, type: NP, category:CORRECTNESS
There is a statement or branch on an exception path that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).
原因分析：exception分支上，存在引用一个null对象的方法，引发空指针异常。
例如：
``` PrintWriter out = null;
    try{
        response.setContentType("text/html;charset=GBK");
        response.setHeader("Cache-Control;no-cache");
        out = response.getWriter();
        out.flush();
    }catch(Exception ex){
        logger.debug("获取树的XML代码出错。"+ex.getMessage());
    }finally{
        out.close();//out可能为空
    }
```

##### 7.18、   Very confusing method names
id: NM_VERY_CONFUSING, type: Nm, category: CORRECTNESS
The referenced methods have names that differ only by capitalization. This is very confusing because if the capitalization were identical then one of the methods would override the other.
原因分析：被引用的方法中存在容易混淆的变量。
例如：fzgsdm改成 fzgsDm 即可。
 
```public String getFzgsdm() {
    return getFzgsdm;
}
```

##### 7.19、   Method invokes inefficient new String() constructor
id: DM_STRING_VOID_CTOR, type: Dm, category: Performance Creating a new java.lang.String object using the no-argument constructor wastes memory because the object so created will be functionally indistinguishable from the empty string constant "".  Java guarantees that identical string constants will be represented by the same String object.  Therefore, you should just use the empty string constant directly.
原因分析：不使用new String()定义空的字符串
例如：
  
```String alarmCodeCond = new String();
应当
String alarmCodeCond = "";
```      

##### 7.20、   Load of known null value
id: NP_LOAD_OF_KNOWN_NULL_VALUE, type: Np, category: Dodgy
The variable referenced at this point is known to be null due to an earlier check against null. Although this is valid, it might be a mistake (perhaps you intended to refer to a different variable, or perhaps the earlier check to see if the variable is null should have been a check to see if it was nonnull).
原因分析：null值的不当使用。
例如：

``` 
if(devIds ==null && devIds.size()==0){ //.....} 

if(null ==tempList||tempList.size()!=0){ //.....}

if(batchNo ==null){
   throw new Exception("the No. "+batchNo + "is not exists1"）)
}
```
##### 7.21、   Method concatenates strings using + in a loop  
id: SBSC_USE_STRINGBUFFER_CONCATENATION, type: SBSC, category: Performance
The method seems to be building a String using concatenation in a loop. In each iteration, the String is converted to a StringBuffer/StringBuilder, appended to, and converted back to a String. This can lead to a cost quadratic in the number of iterations, as the growing string is recopied in each iteration. Better performance can be obtained by using a StringBuffer (or StringBuilder in Java 1.5) explicitly.
For example:
  // This is bad
  String s = "";
  for (int i = 0; i < field.length; ++i) {
    s = s + field[i];
  }
  // This is better
  StringBuffer buf = new StringBuffer();
  for (int i = 0; i < field.length; ++i) {
    buf.append(field[i]);
  }
  String s = buf.toString();
原因分析：在循环里使用字符串连接，效率低，应该使用StringBuilder/StringBuffer
 
### Bug列表
#### BUG-0001
Bug: Field only ever set to null: com.bettersoft.admin.BtCorpManager.ps 
All writes to this field are of the constant value null, and thus all reads of the field will return null. Check for errors, or remove it if it is useless.
Confidence: Normal, Rank: Troubling (12)
Pattern: UWF_NULL_FIELD 
Type: UwF, Category: CORRECTNESS (Correctness)
代码片段：、
```public class BtCorpManager {
    private BtCorp btcorp=null;
    private Connection con = null;
    private Statement st = null;
    private PreparedStatement ps = null;
    private ResultSet rs = null;
    private void setConnection(String centerno) throws Exception{
        //con = DBManager.getConnection(centerno);
        con = DBManager.getConnection();
    }
```
解释说明：在BtCorpManager类里面定了一个私有的成员变量PreparedStatement ps,但是这个成员变量ps在实例范围内没有得到任何的初始化(采用默认的构造方法)，始终为null,所以在实例范围内使用该成员变量时，如果不先对其进行初始化操作或者无意识的行为忘了初始化操作，那肯定是要报空指针异常，所以这无疑是一个bug 
推荐修改： 自己看着办
 
#### BUG-0002
Bug: Nullcheck of form at line 36 of value previously dereferenced in com.bettersoft.admin.CorpEditAction.execute(ActionMapping, ActionForm, HttpServletRequest, HttpServletResponse)
A value is checked here to see whether it is null, but this value can't be null because it was previously dereferenced and if it were null a null pointer exception would have occurred at the earlier dereference. Essentially, this code and the previous dereference disagree as to whether this value is allowed to be null. Either the check is redundant or the previous dereference is erroneous.
Confidence: High, Rank: Scary (9)
Pattern: RCN_REDUNDANT_NULLCHECK_WOULD_HAVE_BEEN_A_NPE 
Type: RCN, Category: CORRECTNESS (Correctness)
代码片段：

```public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request, HttpServletResponse response) throws Exception {
        //throw new UnsupportedOperationException("Method is not implemented");
        ActionErrors errors = new ActionErrors();
        CreateCorpActionForm createCorp = new CreateCorpActionForm();
        createCorp = (CreateCorpActionForm)form;
        CreateCorpActionForm webcorp=new CreateCorpActionForm();
        BudgetWebcorpManager budgetWebcorpManager=new BudgetWebcorpManager();
        webcorp=budgetWebcorpManager.getCWebcorp(createCorp.getId());
        createCorp.setFbsaddapproveid(webcorp.getFbsaddapproveid());
        createCorp.setFbsinputapproveid(webcorp.getFbsinputapproveid());
        createCorp.setFbsprocessapproveid(webcorp.getFbsprocessapproveid());
 
        boolean b=false;
        if(createCorp!=null){

```
解释说明：注意到有个局部变量 CreateCorpActionForm createCorp;再看下它的初始化过程，先是通过new给它分配了内存空间，紧接着有让它引用了了另一个未知的变量，这里说未知是指这个新的引用可能为空，显然 createCorp有可能指向一个空的地址，所以在接下来的引用中极可能报空指针异常（在引用之前不进行判空操作的话）！ 在接下来的代码，如下
`if(createCorp!=null){
`
其实也就没有存在的必要，因为如果为空的话，上面这行代码根本不可能执行到，所以findbug说这是冗余的空指针检查。当然考虑到特殊情况，这里显然是struts1的action,所以只要web应用正常启动，通常以下代码

`createCorp = (CreateCorpActionForm)form;`

是不会导致createCorp指向空的，唯一的缺陷就是之前的new操作是多余的。
推荐修改：自己看着办

#### BUG-0003
Bug: con is null guaranteed to be dereferenced in com.bettersoft.admin.leftAction.getLeft(int, String, String) on exception path
There is a statement or branch on an exception path that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).
Confidence: Normal, Rank: (Troubling 11)
Pattern: NP_GUARANTEED_DEREF_ON_EXCEPTION_PATH 
Type: NP, Category: CORRECTNESS (Correctness)
代码片段：

```finally
        {
            try
            {
                if(rs != null)
                    rs.close();
                if(ps!=null)
                    ps.close();
                con.close();
            }catch(Exception ee)
            {
                ee.printStackTrace();
            }
        }
```

解释说明：这应该是大家很熟悉的代码片段了，可以想象Connection con是在catch代码块中进行的初始化操作，findbug对该bug说的很明白，说如果出现异常，Connection con将保证为空，因为很显然如果出现异常，con将得不到正确的初始化，即便初始化了，因为异常的出现，引用也会被解除，回到一开始定义处的null状态，那么在这里的finally代码块中调用Connection con的close()方法，将报空指针异常 
推荐修改：自己看着办 
 
#### BUG-0004 
Bug: Possible null pointer dereference of dbVersion in com.bettersoft.admin.LoginAction.loadVersion(HttpServletRequest, ActionErrors) on exception path 
A reference value which is null on some exception control path is dereferenced here. This may lead to aNullPointerExceptionwhen the code is executed. Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
Confidence: Normal, Rank: Troubling (11)
Pattern: NP_NULL_ON_SOME_PATH_EXCEPTION 
Type: NP, Category: CORRECTNESS (Correctness)
代码片段：

```VersionInfo dbVersion = null;
    try {
        con = DBManager.getConnection();
        dbVersion = vm.getVersionInfo(con);
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    } finally {
        try {
            con.close();
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
if (dbVersion.equals(programVersion)) {
    programVersion.setCorrent(true);
} else {
    programVersion.setCorrent(false);
}
```

解释说明：如果try catch 中捕获异常，那么dbVersion将为空， 

`dbVersion.equals(programVersion)`

上面这行代码，将报空指针异常 
 
#### BUG-0005
Bug: Dead store to am in com.bettersoft.approve.action.CheckAction.execute(ActionMapping, ActionForm, HttpServletRequest, HttpServletResponse)
This instruction assigns a value to a local variable, but the value is not read or used in any subsequent instruction. Often, this indicates an error, because the value computed is never used.
Note that Sun's javac compiler often generates dead stores for final local variables. Because FindBugs is a bytecode-based tool, there is no easy way to eliminate these false positives.
Confidence: High, Rank: Of Concern (15)
Pattern: DLS_DEAD_LOCAL_STORE 
Type: DLS, Category: STYLE (Dodgy code)
代码片段：

`ApproveManager am = new ApproveManager();`

解释说明：am这个局部变量创建出来后，没有在任何地方被引用。这里确实没有被引用，所以是个bug,但是findbug又说明了，这有可能是误报，因为javac编译器在编译局部常量时，也会产生dead stroes。所以这个要视情况而定，不能过于纠结 
 
 
#### BUG-0006
Bug: The class name com.bettersoft.approve.form.BtWebCorp shadows the simple name of the superclass com.bettersoft.admin.BtWebCorp
This class has a simple name that is identical to that of its superclass, except that its superclass is in a different package (e.g.,alpha.Fooextendsbeta.Foo). This can be exceptionally confusing, create lots of situations in which you have to look at import statements to resolve references and creates many opportunities to accidently define methods that do not override methods in their superclasses.
Confidence: High, Rank: Troubling (14)
Pattern: NM_SAME_SIMPLE_NAME_AS_SUPERCLASS 
Type: Nm, Category: BAD_PRACTICE (Bad practice)
代码片段： 

```public class BtWebCorp extends com.bettersoft.admin.BtWebCorp{
 
}
```

解释说明：这里子类和父类名称一样，findbug认为这回导致很多混淆。显然一旦出问题，将很难发现，运行结果将出乎意料 
 
#### BUG-0007
Bug: Comparison of String objects using == or != in com.byttersoft.admin.persistence.dao.MessageOpenDao.addOpenSave(MessageOpenForm)
This code comparesjava.lang.Stringobjects for reference equality using the == or != operators. Unless both strings are either constants in a source file, or have been interned using theString.intern()method, the same string value may be represented by two different String objects. Consider using theequals(Object)method instead.
Confidence: Normal, Rank: Troubling (11)
Pattern: ES_COMPARING_STRINGS_WITH_EQ 
Type: ES, Category: BAD_PRACTICE (Bad practice)
代码片段：

```if(id==null || id==""){
        sql = "insert into xx values (1,?,?,?)";
}else{
        sql = "insert into xx values ((select max(id) + 1 from xx),?,?,?)";
}
```

解释说明： 直接使用==进行对象实例比较，而没有使用equals,本来没觉得这个bug咋样，但是发现项目里居然最多的bug就是这个，不管是很多年前的代码还是最近的代码，都存在这大量这样的问题。看来这是一个通病，所以大家注意一下，不光是我们公司的项目有这样的问题，这应该是一个普遍的问题，尤其实在比较String类型的时候，注意只要不是java基本类型都需要使用equals进行比较，哪怕是自动解封的Integer，Double等
 
#### BUG-0008
Bug: Call to String.equals(Double) in com.byttersoft.amm.util.BalanceInterzoneRateUtil.formatRate(Double)
This method calls equals(Object) on two references of different class types with no common subclasses. Therefore, the objects being compared are unlikely to be members of the same class at runtime (unless some application classes were not analyzed, or dynamic class loading can occur at runtime). According to the contract of equals(), objects of different classes should always compare as unequal; therefore, according to the contract defined by Java.lang.Object.equals(Object), the result of this comparison will always be false at runtime.
Confidence: High, Rank: Scariest (1)
Pattern: EC_UNRELATED_TYPES 
Type: EC, Category: CORRECTNESS (Correctness)
代码片段：
```private static String  formatRate(Double r){
         
        if(r==null ||  ("undefined").equals(r)){
            return null;
        }

```
解释说明：使用equals比较不同类型的数据，"undefined"是String类型，r是Double类型，这两个比较肯定返回false

`("undefined").equals(r)`

上面这行代码完全没有必要 ，不可能存在这种情况
 
#### BUG-0009
Bug: Class com.byttersoft.util.CertInfo defines non-transient non-serializable instance field subjectDnAttr
This Serializable class defines a non-primitive instance field which is neither transient, Serializable, orjava.lang.Object, and does not appear to implement theExternalizableinterface or thereadObject()andwriteObject()methods. Objects of this class will not be deserialized correctly if a non-Serializable object is stored in this field.
Confidence: High, Rank: Troubling (14)
Pattern: SE_BAD_FIELD 
Type: Se, Category: BAD_PRACTICE (Bad practice)
代码片段： 

```public class CertInfo implements java.io.Serializable
{
    private String subjectDN="";
    private String issuerDN="";
    private String notAfterDate="";
    private String notBeforeDate="";
    private String serialNumber="";
    private String sigAlgName="";
    private String sigAlgOID="";
    private String version="";
    private String publicKeyFormat="";
    private String publicKeyAlgorithm="";
    private Names subjectDnAttr=null;
｝
public class Names
{}<span></span>
```

解释说明： CertInfo实现的序列话，但是他的成员变量Names subjectDnAttr没有实现序列化，这将会导致序列化失败，String已经默认实现了序列化。注意，序列化时所有的成员变量都必须递归的实现序列化，否则将导致序列化失败。如果某个成员变量不想被序列化要么标注为瞬态要么重写readObj方法
 
#### BUG-0010
Bug: Dead store to corpGourps rather than field with same name in com.byttersoft.admin.form.CorpGroupsForm.setCorpGourps(CorpGourps)
This instruction assigns a value to a local variable, but the value is not read or used in any subsequent instruction. Often, this indicates an error, because the value computed is never used. There is a field with the same name as the local variable. Did you mean to assign to that variable instead?
Confidence: High, Rank: Scary (9)
Pattern: DLS_DEAD_LOCAL_STORE_SHADOWS_FIELD 
Type: DLS, Category: STYLE (Dodgy code)
代码片段： 

```
public void setCorpGourps(CorpGourps corpGourps) {
        corpGourps = corpGourps;
    }
```
解释说明：成员变量和局部变量重名
 
#### BUG-0011
Bug: Invocation of toString on labelValue in com.byttersoft.approve.persistence.dao.MesAppDao.getMapByPara(String)
The code invokes toString on an array, which will generate a fairly useless result such as [C@16f0472. Consider using Arrays.toString to convert the array into a readable String that gives the contents of the array. See Programming Puzzlers, chapter 3, puzzle 12.
Confidence: Normal, Rank: Troubling (10)
Pattern: DMI_INVOKING_TOSTRING_ON_ARRAY 
Type: USELESS_STRING, Category: CORRECTNESS (Correctness)
代码片段：

```for (String parameter : parameters) {
            String[] labelValue = parameter.split("=");
            if (labelValue.length == 2) {
                String key = labelValue[0];
                String value = labelValue[1];
                hashMap.put(key, value);
            } else {
                logger.debug("参数 " + labelValue + " 配置错误。");
            }
        }
```

解释说明：在进行日志输出时，直接输出对象，将默认调用对象的toString方法，而默认是输出对象的内存地址，所以这里显然有问题，本意应该是输出数组中的字符串 
 
#### BUG-0012
Bug: Write to static field com.byttersoft.admin.service.BtSysResService.map from instance method com.byttersoft.admin.service.BtSysResService.hashCatchOfSysRes(boolean)
This instance method writes to a static field. This is tricky to get correct if multiple instances are being manipulated, and generally bad practice.
Confidence: High, Rank: Of Concern (15)
Pattern: ST_WRITE_TO_STATIC_FROM_INSTANCE_METHOD 
Type: ST, Category: STYLE (Dodgy code)
代码片段： 

```static Map<String, BtSysRes> map = new HashMap<String, BtSysRes>();
public Map<String, BtSysRes> hashCatchOfSysRes(boolean isRefresh) {
        if(isRefresh == true){
            map = hashCatchOfSysRes();
        }else{
            if(map == null || map.isEmpty()){
                map = hashCatchOfSysRes();
            }
        }
        return map;
    }

```
解释说明：在实例方法中修改类变量的引用，这会导致共享问题，因为其他实例也会访问该静态变量，但是却不知道某个实例已经修改了该静态变量的引用，导致不可预知的问题
推荐修改：将该方法改为类方法
 
 
#### BUG-0013
Bug: Unwritten field: com.byttersoft.admin.service.importservice.ImportServices.bank
This field is never written. All reads of it will return the default value. Check for errors (should it have been initialized?), or remove it if it is useless.
Confidence: Normal, Rank: Troubling (12)
Pattern: UWF_UNWRITTEN_FIELD 
Type: UwF, Category: CORRECTNESS (Correctness)
代码片段：

```public class ImportServices {
private IBankAccServices bank;
public IBankAccServices getBank() {
        return bank;
    }
｝
```

解释说明： bank对象为空，getBank方法返回了一个肯定为空的对象实例
 
 
#### BUG-0014
Bug: There is an apparent infinite recursive loop in com.byttersoft.amm.dao.impl.CheckLoanOrProvideInfoDaoImpl.addBatch(List)
This method unconditionally invokes itself. This would seem to indicate an infinite recursive loop that will result in a stack overflow.
Confidence: High, Rank: Scary (9)
Pattern: IL_INFINITE_RECURSIVE_LOOP 
Type: IL, Category: CORRECTNESS (Correctness)
代码片段： 
```public void addBatch(List<CmsPLoanToBean> cmsPLoanBeans) {
        this.addBatch(cmsPLoanBeans);
    }
```
解释说明：出现了递归调用addBatch,将出现死循环
 
### High
#### 1.DM_DEFAULT_ENCODING
 
##### 1.1 Found reliance on default encoding in com.cmcc.aoi.httprequest.service.HttpRequest.sendGet(String, String): new java.io.InputStreamReader(InputStream)
Found a call to a method which will perform a byte to String (or String to byte) conversion, and will assume that the default platform encoding is suitable. This will cause the application behaviour to vary between platforms. Use an alternative API and specify a charset name or Charset object explicitly.
 
new BufferedReader(new InputStreamReader(connection.getInputStream()));
 
修改为： InputStreamReader fileData = new InputStreamReader(file ,"utf-8");
 
 
 
##### 1.2 Found reliance on default encoding in com.cmcc.aoi.httprequest.service.HttpRequest.sendPost(String, JSONObject): new java.io.PrintWriter(OutputStream)
Found a call to a method which will perform a byte to String (or String to byte) conversion, and will assume that the default platform encoding is suitable. This will cause the application behaviour to vary between platforms. Use an alternative API and specify a charset name or Charset object explicitly.
out = new PrintWriter(conn.getOutputStream());
 
修改为： out = new PrintWriter(new OutputStreamWriter(conn.getOutputStream(), "utf-8"));
 
 
##### 1.3 Found reliance on default encoding in com.cmcc.aoi.selfhelp.action.DeliverWebRequestAction.calculateUserCount(HttpServletRequest): String.getBytes()
Found a call to a method which will perform a byte to String (or String to byte) conversion, and will assume that the default platform encoding is suitable. This will cause the application behaviour to vary between platforms. Use an alternative API and specify a charset name or Charset object explicitly.
fileName = new String(req.getParameter("fileName").getBytes(), "UTF-8");
修改为
fileName = new String(req.getParameter("fileName").getBytes("UTF-8"), "UTF-8");
 
##### 1.4 Found reliance on default encoding in com.cmcc.aoi.selfhelp.action.servlet.AoeRegAction.report(HttpServletRequest, HttpServletResponse): java.io.ByteArrayOutputStream.toString()
Found a call to a method which will perform a byte to String (or String to byte) conversion, and will assume that the default platform encoding is suitable. This will cause the application behaviour to vary between platforms. Use an alternative API and specify a charset name or Charset object explicitly.
logger.info("RECV STR: " + baos.toString());
修改为
logger.info("RECV STR: " + baos.toString("utf-8"));
 
##### 1.5 Found reliance on default encoding in com.cmcc.aoi.selfhelp.action.servlet.AoeUploadLogAction.report(HttpServletRequest, HttpServletResponse): new java.io.FileWriter(File)
Found a call to a method which will perform a byte to String (or String to byte) conversion, and will assume that the default platform encoding is suitable. This will cause the application behaviour to vary between platforms. Use an alternative API and specify a charset name or Charset object explicitly.
new FileWriter(f).append(baos.toString("UTF-8")).close();
修改为
BufferedWriter out = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(f, true))); 
out.write(baos.toString("UTF-8")); 
out.close();
 
BufferedWriter bw= new BufferedWriter( new OutputStreamWriter(new FileOutputStream(filePath, true), "utf-8"));
 
##### 1.6 Found reliance on default encoding in new com.cmcc.aoi.util.TokenZipFileUtil(String): new java.io.FileReader(String)
Found a call to a method which will perform a byte to String (or String to byte) conversion, and will assume that the default platform encoding is suitable. This will cause the application behaviour to vary between platforms. Use an alternative API and specify a charset name or Charset object explicitly.
FileReader in = new FileReader(file);
改为
BufferedReader reader = new BufferedReader(new InputStreamReader(newFileInputStream(file), "UTF-8")); 
       
#### 2.MS_SHOULD_BE_FINAL
 
com.cmcc.aoi.selfhelp.action.DeliverWebRequestAction.logger isn't final but should be
This static field public but not final, and could be changed by malicious code or by accident from another package. The field could be made final to avoid this vulnerability.
 
protected static   Logger logger = LoggerFactory.getLogger(DeliverWebRequestAction.class);
修改为 protected static final Logger logger = LoggerFactory.getLogger(DeliverWebRequestAction.class);
 
####  3.DLS_DEAD_LOCAL_STORE
 
Dead store to s in com.cmcc.aoi.selfhelp.action.DeliverWebRequestAction.textSend(WebSendTextForm, HttpServletRequest)
This instruction assigns a value to a local variable, but the value is not read or used in any subsequent instruction. Often, this indicates an error, because the value computed is never used.
Note that Sun's javac compiler often generates dead stores for final local variables. Because FindBugs is a bytecode-based tool, there is no easy way to eliminate these false positives.
ShopMappingDeliver shopMappingDeliver = null;
删除即可
 
#### 4.ST_WRITE_TO_STATIC_FROM_INSTANCE_METHOD
Write to static field com.cmcc.aoi.selfhelp.action.MultipleMediaAoeAction.linkRoot from instance method com.cmcc.aoi.selfhelp.action.MultipleMediaAoeAction.afterPropertiesSet()
This instance method writes to a static field. This is tricky to get correct if multiple instances are being manipulated, and generally bad practice.
linkRoot = sysConfigService.getDomainName() + "/";
修改改为：
```public static String getLinkRoot() {
        returnlinkRoot;
    }
 
    publicstaticvoid setLinkRoot(String linkRoot) {
        MultipleMediaAoeAction.linkRoot = linkRoot;
    }
   MultipleMediaAoeAction.setLinkRoot(sysConfigService.getDomainName() + "/");

``` 

#### 5. J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION
Store of non serializable com.cmcc.aoi.selfhelp.action.UploadFileAction$FileUploadStatus into HttpSession in new com.cmcc.aoi.selfhelp.action.UploadFileAction$MyProgressListener(UploadFileAction, HttpServletRequest)
This code seems to be storing a non-serializable object into an HttpSession. If this session is passivated or migrated, an error will result.
修改为 FileUploadStatus implements Serializable
 
 
#### 6.  RCN_REDUNDANT_NULLCHECK_OF_NONNULL_VALUE
Redundant nullcheck of rtr, which is known to be non-null in com.cmcc.aoi.selfhelp.action.servlet.AoeReportApplistAction.device(HttpServletRequest, HttpServletResponse)
This method contains a redundant check of a known non-null value against the constant null.
```if (rtr != null) {
            Writer writer;
            try {
                writer = response.getWriter();
                if (rtr != null) {
                    try {
 
                        String s = JSONUtil.objToJson(rtr);
                        if (LOGGER.isDebugEnabled()) {
                            LOGGER.debug("SEND STR: " + s);
                        }
                        writer.write(s);
                        writer.flush();
                    } catch (IOException e) {
                        LOGGER.warn("", e);
                        if (writer != null) {
                            try {
                                writer.write(JSONUtil.objToJson(rtr));
                            } catch (IOException e1) {
                                LOGGER.warn("", e1);
                            }
                        }
                    }
                } else {
                    response.getWriter().write("{\"errorCode\":401}");
                }
            } catch (IOException e2) {
                LOGGER.warn("", e2);
            }
        }
```
修改为
```if (rtr != null) {
            Writer writer;
            try {
                writer = response.getWriter();
                try {
 
                    String s = JSONUtil.objToJson(rtr);
                    if (LOGGER.isDebugEnabled()) {
                        LOGGER.debug("SEND STR: " + s);
                    }
                    writer.write(s);
                    writer.flush();
                } catch (IOException e) {
                    LOGGER.warn("", e);
                    if (writer != null) {
                        try {
                            writer.write(JSONUtil.objToJson(rtr));
                        } catch (IOException e1) {
                            LOGGER.warn("", e1);
                        }
                    }
                }
 
            } catch (IOException e2) {
                LOGGER.warn("", e2);
            }
        } else {
            response.getWriter().write("{\"errorCode\":401}");
        }
```
 
#### 7. RU_INVOKE_RUN
com.cmcc.aoi.selfhelp.action.servlet.UploadTokensAction$TokenFileThread.run() explicitly invokes run on a thread (did you mean to start it instead?)
This method explicitly invokes run() on an object.  In general, classes implement the Runnable interface because they are going to have their run() method invoked in a new thread, in which case Thread.start() is the right method to call.
 
`ti.run();
`
修改为：

```ti.start();
try {
    ti.join();
} catch (InterruptedException e) {
     e.printStackTrace();
}
```
 
 
#### 8. NM_SAME_SIMPLE_NAME_AS_SUPERCLASS
The class name com.cmcc.aoi.selfhelp.dao.BaseDao shadows the simple name of the superclass org.slave4j.orm.hibernate.BaseDao
This class has a simple name that is identical to that of its superclass, except that its superclass is in a different package (e.g., alpha.Foo extends beta.Foo). This can be exceptionally confusing, create lots of situations in which you have to look at import statements to resolve references and creates many opportunities to accidentally define methods that do not override methods in their superclasses
com.cmcc.aoi.selfhelp.dao.BaseDao
修改为
com.cmcc.aoi.selfhelp.dao.BasisDao
 
#### 9. SE_BAD_FIELD_INNER_CLASS
com.cmcc.aoi.selfhelp.action.UploadFileAction$FileUploadStatus is serializable but also an inner class of a non-serializable class
This Serializable class is an inner class of a non-serializable class. Thus, attempts to serialize it will also attempt to associate instance of the outer class with which it is associated, leading to a runtime error.
If possible, making the inner class a static inner class should solve the problem. Making the outer class serializable might also work, but that would mean serializing an instance of the inner class would always also serialize the instance of the outer class, which it often not what you really want.
修改外部类
UploadFileAction extends BaseAction implements Serializable
 
 
#### 10. DM_BOXED_PRIMITIVE_FOR_PARSING
Boxing/unboxing to parse a primitive com.cmcc.aoi.selfhelp.dao.StatAppEveryHourDao.findWeekList(String)
A boxed primitive is created from a String, just to extract the unboxed primitive value. It is more efficient to just call the static parseXXX method.

```statAppEveryHour.setNewnumber(Integer.valueOf(String.valueOf(objects[2])));               statAppEveryHour.setAccnumber(Integer.valueOf(String.valueOf(objects[3])));
```
修改为

```statAppEveryHour.setStattime(sdf.parse(String.valueOf(objects[1])));
                    statAppEveryHour
                            .setNewnumber(Integer.parseInt(String.valueOf(objects[2]) != null
                                    && !"".equals(String.valueOf(objects[2]))
                                            ? String.valueOf(objects[2]) : "0"));
                    statAppEveryHour
                            .setAccnumber(Integer.parseInt(String.valueOf(objects[3]) != null
                                    && !"".equals(String.valueOf(objects[3]))
                                            ? String.valueOf(objects[3]) : "0"));
```
 
###  Normal
#### 1.SBSC_USE_STRINGBUFFER_CONCATENATION
com.cmcc.aoi.httprequest.service.HttpRequest.sendGet(String, String) concatenates strings using + in a loop
The method seems to be building a String using concatenation in a loop. In each iteration, the String is converted to a StringBuffer/StringBuilder, appended to, and converted back to a String. This can lead to a cost quadratic in the number of iterations, as the growing string is recopied in each iteration.
Better performance can be obtained by using a StringBuffer (or StringBuilder in Java 1.5) explicitly.
For example:
  // This is bad
  String s = "";
  for (int i = 0; i < field.length; ++i) {
    s = s + field[i];
  }
 
  // This is better
  StringBuffer buf = new StringBuffer();
  for (int i = 0; i < field.length; ++i) {
    buf.append(field[i]);
  }
  String s = buf.toString();
 
#### 2. WMI_WRONG_MAP_ITERATOR

```for (String key : map.keySet()) {
                System.out.println(key + "--->" + map.get(key));
}
```
改为

```for (  Map.Entry<String, List<String>> entry : map.entrySet()) {
                System.out.println(entry.getKey() + "--->" + entry.getValue());
            }

``` 
#### 3.  EI_EXPOSE_REP
com.cmcc.aoi.selfhelp.entity.Activation.getValidUntil() may expose internal representation by returning Activation.validUntil
Returning a reference to a mutable object value stored in one of the object's fields exposes the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Returning a new copy of the object is better approach in many situations.
 
 
    public Date getValidUntil() {
        returnvalidUntil;
}
 
修改为
public Date getValidUntil() {
        if(validUntil == null) {
            returnnull;
        }
        return (Date) validUntil.clone();
}
 
#### 4. EI_EXPOSE_REP2
com.cmcc.aoi.selfhelp.entity.Activation.setValidUntil(Date) may expose internal representation by storing an externally mutable object into Activation.validUntil
This code stores a reference to an externally mutable object into the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Storing a copy of the object is better approach in many situations.

```publicvoid setValidUntil(Date validUntil) {
this.validUntil = validUntil;
}
```
修改为
```publicvoid setValidUntil(Date validUntil) {
        if(validUntil == null) {
            this.validUntil = null;
        }else {
            this.validUntil = (Date) validUntil.clone();
        }
    }

``` 
 
#### 5. BC_VACUOUS_INSTANCEOF
instanceof will always return true for all non-null values in com.cmcc.aoi.selfhelp.entity.AppType.compareTo(AppType), since all com.cmcc.aoi.selfhelp.entity.AppType are instances of com.cmcc.aoi.selfhelp.entity.AppType
This instanceof test will always return true (unless the value being tested is null). Although this is safe, make sure it isn't an indication of some misunderstanding or some other logic error. If you really want to test the value for being null, perhaps it would be clearer to do better to do a null test rather than an instanceof test.
 
#### 6. MS_MUTABLE_ARRAY
com.cmcc.aoi.selfhelp.entity.DeviceType.CURRENTUSEDDEVICES is a mutable array
A final static field references an array and can be accessed by malicious code or by accident from another package. This code can freely modify the contents of the array.
 
public static final int[] CURRENTUSEDDEVICES = new int []{Device.iOS.ordinal()，Device.Android.ordinal()，Device.WP.ordinal()}；
修改为
 Public > protected
 
#### 7. EQ_COMPARETO_USE_OBJECT_EQUALS
com.cmcc.aoi.selfhelp.entity.AppType defines compareTo(AppType) and uses Object.equals()
This class defines a compareTo(...) method but inherits its equals() method from java.lang.Object. Generally, the value of compareTo should return zero if and only if equals returns true. If this is violated, weird and unpredictable failures will occur in classes such as PriorityQueue. In Java 5 the PriorityQueue.remove method uses the compareTo method, while in Java 6 it uses the equals method.
From the JavaDoc for the compareTo method in the Comparable interface:
It is strongly recommended, but not strictly required that (x.compareTo(y)==0) == (x.equals(y)). Generally speaking, any class that implements the Comparable interface and violates this condition should clearly indicate this fact. The recommended language is "Note: this class has a natural ordering that is inconsistent with equals."
 
修改
添加 hashcode() 和 equals() 代码即可
 
#### 8. BC_VACUOUS_INSTANCEOF
instanceof will always return true for all non-null values in com.cmcc.aoi.selfhelp.entity.AppType.compareTo(AppType), since all com.cmcc.aoi.selfhelp.entity.AppType are instances of com.cmcc.aoi.selfhelp.entity.AppType
This instanceof test will always return true (unless the value being tested is null). Although this is safe, make sure it isn't an indication of some misunderstanding or some other logic error. If you really want to test the value for being null, perhaps it would be clearer to do better to do a null test rather than an instanceof test.
  
```@Override
    publicint compareTo(AppType o) {
        if (oinstanceof AppType) {
            AppType p = (AppType) o;
            returnthis.typeId > p.typeId ? 1 : this.typeId == p.typeId ? 0 : -1;
        }
        return 1;
    }

``` 

修改为
  
```@Override
    publicint compareTo(AppType o) {
        if (null != o) {
            AppType p  = (AppType) o ;
            returnthis.typeId > p.typeId ? 1 : this.typeId == p.typeId ? 0 : -1;
        }
        return 1;
 
    }

``` 
#### 9. ME_ENUM_FIELD_SETTER
com.cmcc.aoi.selfhelp.dto.ActivationSituation.setSituation(String) unconditionally sets the field situation
This public method declared in public enum unconditionally sets enum field, thus this field can be changed by malicious code or by accident from another package. Though mutable enum fields may be used for lazy initialization, it's a bad practice to expose them to the outer world. Consider removing this method or declaring it package-private.
    publicvoid setCode(String code) {
        this.code = code;
    }
 
修改
 删除该无用代码
 
 
#### 10.  IM_BAD_CHECK_FOR_ODD
Check for oddness that won't work for negative numbers in com.cmcc.aoi.selfhelp.dto.WebSendTextForm.toDeliverWebRequest()
The code uses x % 2 == 1 to check to see if a value is odd, but this won't work for negative numbers (e.g., (-5) % 2 == -1). If this code is intending to check for oddness, consider using x & 1 == 1, or x % 2 != 0.

```DeliverFactory
                                    .createTextOpenApp(this.msgtype, "", this.content,
                                            this.isRingAndVibrate % 2 == 1,
                                            isRingAndVibrate / 2 >= 1, this.activity)
                                    .toJsonString());

``` 
修改为

```DeliverFactory
                                    .createTextOpenApp(this.msgtype, "", this.content,
                                            this.isRingAndVibrate % 2 != 0,
                                            isRingAndVibrate / 2 >= 1, this.activity)
                                    .toJsonString());

``` 
 
#### 11. MS_EXPOSE_REP
Public static com.cmcc.aoi.selfhelp.dict.DeviceSupported.getSupportedDevs() may expose internal representation by returning DeviceSupported.DEVS
A public static method returns a reference to an array that is part of the static state of the class. Any code that calls this method can freely modify the underlying array. One fix is to return a copy of the array.

```public static Device[] getSupportedDevs() {
        return DEVS;
    }
```
修改为

```publicstatic Device[] getSupportedDevs() {
        return DeviceSupported.DEVS.clone();
    }

``` 
#### 12.URF_UNREAD_PUBLIC_OR_PROTECTED_FIELD
Unread public/protected field: com.cmcc.aoi.selfhelp.dict.OperatorDict.countryCode
This field is never read.  The field is public or protected, so perhaps it is intended to be used with classes not seen as part of the analysis. If not, consider removing it from the class.
publicintcode;
    
```public String enName;
    public String cnName;
    public String countryCode;
 
    public OperatorDict() {
    }
 
    /**
     *
     * @param code
     *            运营商代码,一般是5位
     * @param enName
     *            英文名
     * @param countryCode
     *            国家英文代码
     * @param cnName
     *            中文名
     */
    public OperatorDict(intcode, String enName, String countryCode, String cnName) {
        this.code = code;
        this.enName = enName;
        this.countryCode = countryCode;
        this.cnName = cnName == null ? Integer.toString(code) : cnName;
    }

```修改为
Public  -》 private
  
#### 13. ES_COMPARING_STRINGS_WITH_EQ
Comparison of String objects using == or != in com.cmcc.aoi.selfhelp.entity.Provider.compareTo(Object)
This code compares java.lang.String objects for reference equality using the == or != operators. Unless both strings are either constants in a source file, or have been interned using the String.intern() method, the same string value may be represented by two different String objects. Consider using the equals(Object) method instead.
 
return this.spid.compareTo(p.spid) > 0 ? 1 : this.spid == p.spid ? 0 : -1;
修改为
this.spid.compareTo(p.spid) > 0 ? 1 : this.spid.equals(p.spid) ? 0 : -1;
14.DB_DUPLICATE_BRANCHES
com.cmcc.aoi.selfhelp.dao.ShStatTerminalDao.getListQuery(String, int, Date, Date, boolean, int) uses the same code for two branches
This method uses the same code to implement two branches of a conditional branch. Check to ensure that this isn't a coding mistake.

```if (bool) {
                query.setInteger(i++, nodeType);
                query.setInteger(i++, nodeType);
            } else {
                query.setInteger(i++, nodeType);
                query.setInteger(i++, nodeType);
            }

```修改为

```query.setInteger(i++, nodeType);
query.setInteger(i++, nodeType);
```
 
#### 15. SE_COMPARATOR_SHOULD_BE_SERIALIZABLE
 
com.cmcc.aoi.selfhelp.task.entity.StatAppHabitComparator implements Comparator but not Serializable
This class implements the Comparator interface. You should consider whether or not it should also implement the Serializable interface. If a comparator is used to construct an ordered collection such as a TreeMap, then the TreeMap will be serializable only if the comparator is also serializable. As most comparators have little or no state, making them serializable is generally easy and good defensive programming.
修改为
implements Serializable
 
#### 16.  UWF_UNWRITTEN_PUBLIC_OR_PROTECTED_FIELD
 
Unwritten public or protected field: com.cmcc.aoi.selfhelp.task.entity.StatDevice.keyname
No writes were seen to this public/protected field.  All reads of it will return the default value. Check for errors (should it have been initialized?), or remove it if it is useless.
Public  String keyname;
修改为
Private  String keyname;
 
#### 18.  RV_RETURN_VALUE_IGNORED_BAD_PRACTICE
Exceptional return value of java.io.File.mkdirs() ignored in com.cmcc.aoi.util.FileUtil.moveFile(File, String)
This method returns a value that is not checked. The return value should be checked since it can indicate an unusual or unexpected function execution. For example, the File.delete() method returns false if the file could not be successfully deleted (rather than throwing an Exception). If you don't check the result, you won't notice if the method invocation signals unexpected behavior by returning an atypical return value.
tmp.mkdirs()
修改为
booleanmkdirs = tmp.mkdirs();
logger.debug("debug",mkdirs);
 
REC_CATCH_EXCEPTION
Exception is caught when Exception is not thrown in com.cmcc.aoi.selfhelp.task.fileparser.TokenIncrease.parseLine(String[])
This method uses a try-catch block that catches Exception objects, but Exception is not thrown within the try block, and RuntimeException is not explicitly caught. It is a common bug pattern to say try { ... } catch (Exception e) { something } as a shorthand for catching a number of types of exception each of whose catch blocks is identical, but this construct also accidentally catches RuntimeException as well, masking potential bugs.
A better approach is to either explicitly catch the specific exceptions that are thrown, or to explicitly catch RuntimeException exception, rethrow it, and then catch all non-Runtime Exceptions, as shown below:
 
``` try {
    ...
  } catch (RuntimeException e) {
    throw e;
  } catch (Exception e) {
    ... deal with all non-runtime exceptions ...
  }

``` 
 
#### 19. ICAST_IDIV_CAST_TO_DOUBLE
Integral division result cast to double or float in com.cmcc.aoi.selfhelp.service.BaseAnalysisService.getInterval(Date, Date, int)
This code casts the result of an integral division (e.g., int or long division) operation to double or float. Doing division on integers truncates the result to the integer value closest to zero. The fact that the result was cast to double suggests that this precision should have been retained. What was probably meant was to cast one or both of the operands to double before performing the division. 
```Here is an example:
int x = 2;
int y = 5;
// Wrong: yields result 0.0
double value1 =  x / y;
 
// Right: yields result 0.4
double value2 =  x / (double) y;
```





## FindBugs规则整理
FindBugs是基于Bug Patterns概念，查找javabytecode（.class文件）中的潜在bug，主要检查bytecode中的bug patterns，如NullPoint空指针检查、没有合理关闭资源、字符串相同判断错（==，而不是equals）等
### 一、Security 关于代码安全性防护
1.Dm: Hardcoded constant database password (DMI_CONSTANT_DB_PASSWORD)
代码中创建DB的密码时采用了写死的密码。
2.Dm: Empty database password (DMI_EMPTY_DB_PASSWORD)
创建数据库连接时没有为数据库设置密码，这会使数据库没有必要的保护。
3.HRS: HTTP cookie formed from untrusted input (HRS_REQUEST_PARAMETER_TO_COOKIE)
此代码使用不受信任的HTTP参数构造一个HTTP Cookie。
4.HRS: HTTP Response splitting vulnerability (HRS_REQUEST_PARAMETER_TO_HTTP_HEADER)
在代码中直接把一个HTTP的参数写入一个HTTP头文件中，它为HTTP的响应暴露了漏洞。
5.SQL: Nonconstant string passed to execute method on an SQL statement (SQL_NONCONSTANT_STRING_PASSED_TO_EXECUTE)
该方法以字符串的形式来调用SQLstatement的execute方法，它似乎是动态生成SQL语句的方法。这会更容易受到SQL注入攻击。
6.XSS: JSP reflected cross site scripting vulnerability (XSS_REQUEST_PARAMETER_TO_JSP_WRITER)
在代码中在JSP输出中直接写入一个HTTP参数，这会造成一个跨站点的脚本漏洞。
### 二、Experimental
 
1.LG: Potential lost logger changes due to weak reference in OpenJDK (LG_LOST_LOGGER_DUE_TO_WEAK_REFERENCE)
OpenJDK的引入了一种潜在的不兼容问题，特别是，java.util.logging.Logger的行为改变时。它现在使用内部弱引用，而不是强引用。–logger配置改变，它就是丢失对logger的引用，这本是一个合理的变化，但不幸的是一些代码对旧的行为有依赖关系。这意味着，当进行垃圾收集时对logger配置将会丢失。例如：
public static void initLogging() throws Exception {
 Logger logger = Logger.getLogger("edu.umd.cs");
 logger.addHandler(new FileHandler()); // call to change logger configuration
 logger.setUseParentHandlers(false); // another call to change logger configuration
}
该方法结束时logger的引用就丢失了，如果你刚刚结束调用initLogging方法后进行垃圾回收，logger的配置将会丢失（因为只有保持记录器弱引用）。
public static void main(String[] args) throws Exception {
 initLogging(); // adds a file handler to the logger
 System.gc(); // logger configuration lost
 Logger.getLogger("edu.umd.cs").info("Some message"); // this isn't logged to the file as expected
}
2.OBL: Method may fail to clean up stream or resource (OBL_UNSATISFIED_OBLIGATION)
这种方法可能无法清除（关闭，处置）一个流，数据库对象，或其他资源需要一个明确的清理行动。
一般来说，如果一个方法打开一个流或其他资源，该方法应该使用try / finally块来确保在方法返回之前流或资源已经被清除了。这种错误模式基本上和OS_OPEN_STREAM和ODR_OPEN_DATABASE_RESOURCE错误模式相同，但是是在不同在静态分析技术。我们正为这个错误模式的效用收集反馈意见。
 
### 三、Bad practice代码实现中的一些坏习惯
 
1.AM: Creates an empty jar file entry (AM_CREATES_EMPTY_JAR_FILE_ENTRY)
调用putNextEntry()方法写入新的 jar 文件条目时立即调用closeEntry()方法。这样会造成JarFile条目为空。
2.AM: Creates an empty zip file entry (AM_CREATES_EMPTY_ZIP_FILE_ENTRY)
调用putNextEntry()方法写入新的 zip 文件条目时立即调用closeEntry()方法。这样会造成ZipFile条目为空。
3.BC: Equals method should not assume anything about the type of its argument (BC_EQUALS_METHOD_SHOULD_WORK_FOR_ALL_OBJECTS)
equals(Object o)方法不能对参数o的类型做任何的假设。比较此对象与指定的对象。当且仅当该参数不为 null，并且是表示与此对象相同的类型的对象时，结果才为 true。
4.BC: Random object created and used only once (DMI_RANDOM_USED_ONLY_ONCE)
随机创建对象只使用过一次就抛弃
5.BIT: Check for sign of bitwise operation (BIT_SIGNED_CHECK)
检查位操作符运行是否合理
((event.detail & SWT.SELECTED) > 0)
If SWT.SELECTED is a negative number, this is a candidate for a bug. Even when SWT.SELECTED is not negative, it seems good practice to use '!= 0' instead of '> 0'.
6.CN: Class implements Cloneable but does not define or use clone method (CN_IDIOM)
按照惯例，实现此接口的类应该使用公共方法重写 Object.clone（它是受保护的），以获得有关重写此方法的详细信息。此接口不 包含 clone 方法。因此，因为某个对象实现了此接口就克隆它是不可能的,应该实现此接口的类应该使用公共方法重写 Object.clone
7.CN: clone method does not call super.clone() (CN_IDIOM_NO_SUPER_CALL)
一个非final类型的类定义了clone()方法而没有调用super.clone()方法。例如：B扩展自A，如果B中clone方法调用了spuer.clone（），而A中的clone没有调用spuer.clone()，就会造成结果类型不准确。要求A的clone方法中调用spuer.clone()方法。
8.CN: Class defines clone() but doesn't implement Cloneable (CN_IMPLEMENTS_CLONE_BUT_NOT_CLONEABLE)
类中定义了clone方法但是它没有实现Cloneable接口
9.Co: Abstract class defines covariant compareTo() method (CO_ABSTRACT_SELF)
抽象类中定义了多个compareTo()方法，正确的是覆写Comparable中的compareTo方法，方法的参数为Object类型，如下例：
int compareTo(T o)  比较此对象与指定对象的顺序。
10.Co: Covariant compareTo() method defined (CO_SELF_NO_OBJECT)
类中定义了多个compareTo()方法，正确的是覆写Comparable中的compareTo方法，方法的参数为Object类型
11.DE: Method might drop exception (DE_MIGHT_DROP)
方法可能抛出异常
12.DE: Method might ignore exception (DE_MIGHT_IGNORE)
方法可能忽略异常
13.DMI: Don't use removeAll to clear a collection (DMI_USING_REMOVEALL_TO_CLEAR_COLLECTION)
不要用removeAll方法去clear一个集合
14.DP: Classloaders should only be created inside doPrivileged block (DP_CREATE_CLASSLOADER_INSIDE_DO_PRIVILEGED)
类加载器只能建立在特殊的方法体内
15.Dm: Method invokes System.exit(...) (DM_EXIT)
在方法中调用System.exit(...)语句，考虑用RuntimeException来代替
16.Dm: Method invokes dangerous method runFinalizersOnExit (DM_RUN_FINALIZERS_ON_EXIT)
在方法中调用了System.runFinalizersOnExit 或者Runtime.runFinalizersOnExit方法，因为这样做是很危险的。
17.ES: Comparison of String parameter using == or != (ES_COMPARING_PARAMETER_STRING_WITH_EQ)
用==或者!=方法去比较String类型的参数
18.ES: Comparison of String objects using == or != (ES_COMPARING_STRINGS_WITH_EQ)
用==或者！=去比较String类型的对象
19.Eq: Abstract class defines covariant equals() method (EQ_ABSTRACT_SELF)
20.Eq: Equals checks for noncompatible operand (EQ_CHECK_FOR_OPERAND_NOT_COMPATIBLE_WITH_THIS)
equals方法检查不一致的操作。两个类根本就是父子关系而去调用equals方法去判读对象是否相等。
public boolean equals(Object o) {
  if (o instanceof Foo)
    return name.equals(((Foo)o).name);
  else if (o instanceof String)
    return name.equals(o);
  else return false;
21.Eq: Class defines compareTo(...) and uses Object.equals() (EQ_COMPARETO_USE_OBJECT_EQUALS)
类中定义了compareTo方法但是继承了Object中的compareTo方法
22.Eq: equals method fails for subtypes (EQ_GETCLASS_AND_CLASS_CONSTANT)
类中的equals方法可能被子类中的方法所破坏，当使用类似于Foo.class == o.getClass()的判断时考虑用this.getClass() == o.getClass()来替换
23.Eq: Covariant equals() method defined (EQ_SELF_NO_OBJECT)
类中定义了多个equals方法。正确的做法是覆写Object中的equals方法，它的参数为Object类型的对象。
24.FI: Empty finalizer should be deleted (FI_EMPTY)
为空的finalizer方法应该删除。一下关于finalizer的内容省略
25.GC: Unchecked type in generic call (GC_UNCHECKED_TYPE_IN_GENERIC_CALL)
This call to a generic collection method passes an argument while compile type Object where a specific type from the generic type parameters is expected. Thus, neither the standard Java type system nor static analysis can provide useful information on whether the object being passed as a parameter is of an appropriate type.
26.HE: Class defines equals() but not hashCode() (HE_EQUALS_NO_HASHCODE)
方法定义了equals方法却没有定义hashCode方法
27.HE: Class defines hashCode() but not equals() (HE_HASHCODE_NO_EQUALS)
 类定义了hashCode方法去没有定义equal方法
28.HE: Class defines equals() and uses Object.hashCode() (HE_EQUALS_USE_HASHCODE)
一个类覆写了equals方法，没有覆写hashCode方法，使用了Object对象的hashCode方法
29.HE: Class inherits equals() and uses Object.hashCode() (HE_INHERITS_EQUALS_USE_HASHCODE)
子类继承了父类的equals方法却使用了Object的hashCode方法
30.IC: Superclass uses subclass during initialization (IC_SUPERCLASS_USES_SUBCLASS_DURING_INITIALIZATION)
子类在父类未初始化之前使用父类对象实例
public class CircularClassInitialization {
        static class InnerClassSingleton extends CircularClassInitialization {
static InnerClassSingleton singleton = new InnerClassSingleton();
        }        
        static CircularClassInitialization foo = InnerClassSingleton.singleton;
}
31.IMSE: Dubious catching of IllegalMonitorStateException (IMSE_DONT_CATCH_IMSE)
捕捉违法的监控状态异常，例如当没有获取到对象锁时使用其wait和notify方法
32.ISC: Needless instantiation of class that only supplies static methods (ISC_INSTANTIATE_STATIC_CLASS)
为使用静态方法而创建一个实例对象。调用静态方法时只需要使用类名+静态方法名就可以了。
33.It: Iterator next() method can't throw NoSuchElementException (IT_NO_SUCH_ELEMENT)
迭代器的next方法不能够抛出NoSuchElementException
34.J2EE: Store of non serializable object into HttpSession (J2EE_STORE_OF_NON_SERIALIZABLE_OBJECT_INTO_SESSION)
在HttpSession对象中保存非连续的对象
35.JCIP: Fields of immutable classes should be final (JCIP_FIELD_ISNT_FINAL_IN_IMMUTABLE_CLASS)
 The class is annotated with net.jcip.annotations.Immutable, and the rules for that annotation require that all fields are final. .
36.NP: Method with Boolean return type returns explicit null (NP_BOOLEAN_RETURN_NULL)
返回值为boolean类型的方法直接返回null，这样会导致空指针异常
37.NP: equals() method does not check for null argument (NP_EQUALS_SHOULD_HANDLE_NULL_ARGUMENT)
变量调用equals方法时没有进行是否为null的判断
38.NP: toString method may return null (NP_TOSTRING_COULD_RETURN_NULL)
toString方法可能返回null
39.Nm: Class names should start with an upper case letter (NM_CLASS_NAMING_CONVENTION)
类的名称以大写字母名称开头
40.Nm: Class is not derived from an Exception, even though it is named as such (NM_CLASS_NOT_EXCEPTION)
类的名称中含有Exception但是却不是一个异常类的子类，这种名称会造成混淆
41.Nm: Confusing method names (NM_CONFUSING)
令人迷惑的方面命名
42.Nm: Field names should start with a lower case letter (NM_FIELD_NAMING_CONVENTION)
非final类型的字段需要遵循驼峰命名原则
43.Nm: Use of identifier that is a keyword in later versions of Java (NM_FUTURE_KEYWORD_USED_AS_IDENTIFIER)
验证是否是java预留关键字
44.Nm: Use of identifier that is a keyword in later versions of Java (NM_FUTURE_KEYWORD_USED_AS_MEMBER_IDENTIFIER)
验证是否时java中的关键字
45.Nm: Method names should start with a lower case letter (NM_METHOD_NAMING_CONVENTION)
方法名称以小写字母开头
46.Nm: Class names shouldn't shadow simple name of implemented interface (NM_SAME_SIMPLE_NAME_AS_INTERFACE)
实现同一接口实现类不能使用相同的名称，即使它们位于不同的包中
47.Nm: Class names shouldn't shadow simple name of superclass (NM_SAME_SIMPLE_NAME_AS_SUPERCLASS)
继承同一父类的子类不能使用相同的名称，即使它们位于不同的包中
48.Nm: Very confusing method names (but perhaps intentional) (NM_VERY_CONFUSING_INTENTIONAL)
很容易混淆的方法命名，例如方法的名称名称使用使用大小写来区别两个不同的方法。
49.Nm: Method doesn't override method in superclass due to wrong package for parameter (NM_WRONG_PACKAGE_INTENTIONAL)
由于错误引用了不同包中相同类名的对象而不能够正确的覆写父类中的方法
import alpha.Foo;
public class A {
  public int f(Foo x) { return 17; }
}
import beta.Foo;
public class B extends A {
  public int f(Foo x) { return 42; }
  public int f(alpha.Foo x) { return 27; }
}
50.ODR: Method may fail to close database resource (ODR_OPEN_DATABASE_RESOURCE)
方法中可能存在关闭数据连接失败的情况
51.OS: Method may fail to close stream (OS_OPEN_STREAM)
方法中可能存在关闭流失败的情况
52.OS: Method may fail to close stream on exception (OS_OPEN_STREAM_EXCEPTION_PATH)
方法中可能存在关闭流时出现异常情况
53.RC: Suspicious reference comparison to constant (RC_REF_COMPARISON_BAD_PRACTICE)
当两者为不同类型的对象时使用equals方法来比较它们的值是否相等，而不是使用==方法。例如比较的两者为java.lang.Integer, java.lang.Float
54.RC: Suspicious reference comparison of Boolean values (RC_REF_COMPARISON_BAD_PRACTICE_BOOLEAN)
使用== 或者 !=操作符来比较两个 Boolean类型的对象，建议使用equals方法。
55.RR: Method ignores results of InputStream.read() (RR_NOT_CHECKED)
InputStream.read方法忽略返回的多个字符，如果对结果没有检查就没法正确处理用户读取少量字符请求的情况。
56.RR: Method ignores results of InputStream.skip() (SR_NOT_CHECKED)
InputStream.skip()方法忽略返回的多个字符，如果对结果没有检查就没法正确处理用户跳过少量字符请求的情况
57.RV: Method ignores exceptional return value (RV_RETURN_VALUE_IGNORED_BAD_PRACTICE)
方法忽略返回值的异常信息
58.SI: Static initializer creates instance before all static final fields assigned (SI_INSTANCE_BEFORE_FINALS_ASSIGNED)
在所有的static final字段赋值之前去使用静态初始化的方法创建一个类的实例。
59.Se: Non-serializable value stored into instance field of a serializable class (SE_BAD_FIELD_STORE)
非序列化的值保存在声明为序列化的的非序列化字段中
60.Se: Comparator doesn't implement Serializable (SE_COMPARATOR_SHOULD_BE_SERIALIZABLE)
Comparator接口没有实现Serializable接口
61.Se: Serializable inner class (SE_INNER_CLASS)
序列化内部类
62.Se: serialVersionUID isn't final (SE_NONFINAL_SERIALVERSIONID)
关于UID类的检查内容省略
63.Se: Class is Serializable but its superclass doesn't define a void constructor (SE_NO_SUITABLE_CONSTRUCTOR)
子类序列化时父类没有提供一个void的构造函数
64.Se: Class is Externalizable but doesn't define a void constructor (SE_NO_SUITABLE_CONSTRUCTOR_FOR_EXTERNALIZATION)
Externalizable 实例类没有定义一个void类型的构造函数
65.Se: The readResolve method must be declared with a return type of Object. (SE_READ_RESOLVE_MUST_RETURN_OBJECT)
readResolve从流中读取类的一个实例，此方法必须声明返回一个Object类型的对象
66.Se: Transient field that isn't set by deserialization. (SE_TRANSIENT_FIELD_NOT_RESTORED)
This class contains a field that is updated at multiple places in the class, thus it seems to be part of the state of the class. However, since the field is marked as transient and not set in readObject or readResolve, it will contain the default value in any deserialized instance of the class.
67.SnVI: Class is Serializable, but doesn't define serialVersionUID (SE_NO_SERIALVERSIONID)
一个类实现了Serializable接口但是没有定义serialVersionUID类型的变量。序列化运行时使用一个称为 serialVersionUID 的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。如果接收者加载的该对象的类的 serialVersionUID 与对应的发送者的类的版本号不同，则反序列化将会导致 InvalidClassException。可序列化类可以通过声明名为 "serialVersionUID" 的字段（该字段必须是静态 (static)、最终 (final) 的 long 型字段）显式声明其自己的 serialVersionUID： 
 ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
68.UI: Usage of GetResource may be unsafe if class is extended (UI_INHERITANCE_UNSAFE_GETRESOURCE)
当一个类被子类继承后不要使用this.getClass().getResource(...)来获取资源
 
### 四、Correctness关于代码正确性相关方面的
1.BC: Impossible cast (BC_IMPOSSIBLE_CAST)
不可能的类转换，执行时会抛出ClassCastException
2.BC: Impossible downcast (BC_IMPOSSIBLE_DOWNCAST)
父类在向下进行类型转换时抛出ClassCastException
3.BC: Impossible downcast of toArray() result (BC_IMPOSSIBLE_DOWNCAST_OF_TOARRAY)
集合转换为数组元素时发生的类转换错误。
This code is casting the result of calling toArray() on a collection to a type more specific than Object[], as in: 
String[] getAsArray(Collection<String> c) {
  return (String[]) c.toArray();
  }
This will usually fail by throwing a ClassCastException. The toArray() of almost all collections return an Object[]. They can't really do anything else, since the Collection object has no reference to the declared generic type of the collection. 
The correct way to do get an array of a specific type from a collection is to use c.toArray(new String[]); or c.toArray(new String[c.size()]); (the latter is slightly more efficient). 
4.BC: instanceof will always return false (BC_IMPOSSIBLE_INSTANCEOF)
采用instaneof方法进行比较时总是返回false。前提是保证它不是由于某些逻辑错误造成的。
5.BIT: Incompatible bit masks (BIT_AND)
错误的使用&位操作符，例如(e & C)
6.BIT: Check to see if ((...) & 0) == 0 (BIT_AND_ZZ)
检查恒等的逻辑错误
7.BIT: Incompatible bit masks (BIT_IOR)
错误的使用|位操作符，例如(e | C)
8.BIT: Check for sign of bitwise operation (BIT_SIGNED_CHECK_HIGH_BIT)
检查逻辑运算符操作返回的标识。例如((event.detail & SWT.SELECTED) > 0)，建议采用!=0代替>0
9.BOA: Class overrides a method implemented in super class Adapter wrongly (BOA_BADLY_OVERRIDDEN_ADAPTER)
子类错误的覆写父类中用于适配监听其他事件的方法，从而导致当触发条件发生时不能被监听者调用
10.Bx: Primitive value is unboxed and coerced for ternary operator (BX_UNBOXED_AND_COERCED_FOR_TERNARY_OPERATOR)
在三元运算符操作时如果没有对值进行封装或者类型转换。例如：b ? e1 : e2
11.DLS: Dead store of class literal (DLS_DEAD_STORE_OF_CLASS_LITERAL)
以类的字面名称方式为一个字段赋值后再也没有去使用它，在1.4jdk中它会自动调用静态的初始化方法，而在jdk1.5中却不会去执行。
12.DLS: Overwritten increment (DLS_OVERWRITTEN_INCREMENT)
覆写增量增加错误i = i++
13.DMI: Bad constant value for month (DMI_BAD_MONTH)
hashNext方法调用next方法。
14.DMI: Collections should not contain themselves (DMI_COLLECTIONS_SHOULD_NOT_CONTAIN_THEMSELVES)
集合没有包含他们自己本身。
15.DMI: Invocation of hashCode on an array (DMI_INVOKING_HASHCODE_ON_ARRAY)
数组直接使用hashCode方法来返回哈希码。
int [] a1 = new int[]{1,2,3,4};
        System.out.println(a1.hashCode());
        System.out.println(java.util.Arrays.hashCode(a1));
16.DMI: Double.longBitsToDouble invoked on an int (DMI_LONG_BITS_TO_DOUBLE_INVOKED_ON_INT)
17.DMI: Vacuous call to collections (DMI_VACUOUS_SELF_COLLECTION_CALL)
集合的调用不能被感知。例如c.containsAll(c)总是返回true，而c.retainAll(c)的返回值不能被感知。
18.Dm: Can't use reflection to check for presence of annotation without runtime retention (DMI_ANNOTATION_IS_NOT_VISIBLE_TO_REFLECTION)
Unless an annotation has itself been annotated with @Retention(RetentionPolicy.RUNTIME), the annotation can't be observed using reflection (e.g., by using the isAnnotationPresent method). .
19.Dm: Useless/vacuous call to EasyMock method (DMI_VACUOUS_CALL_TO_EASYMOCK_METHOD)
While ScheduledThreadPoolExecutor inherits from ThreadPoolExecutor, a few of the inherited tuning methods are not useful for it. In particular, because it acts as a fixed-sized pool using corePoolSize threads and an unbounded queue, adjustments to maximumPoolSize have no useful effect.
20.EC: equals() used to compare array and nonarray (EC_ARRAY_AND_NONARRAY)
数组对象使用equals方法和非数组对象进行比较。即使比较的双方都是数组对象也不应该使用equals方法，而应该比较它们的内容是否相等使用java.util.Arrays.equals(Object[], Object[]);
21.EC: equals(...) used to compare incompatible arrays (EC_INCOMPATIBLE_ARRAY_COMPARE)
使用equls方法去比较类型不相同的数组。例如：String[] and StringBuffer[], or String[] and int[]
22.EC: Call to equals() with null argument (EC_NULL_ARG)
调用equals的对象为null
23.EC: Call to equals() comparing unrelated class and interface (EC_UNRELATED_CLASS_AND_INTERFACE)
使用equals方法比较不相关的类和接口
24.EC: Call to equals() comparing different interface types (EC_UNRELATED_INTERFACES)
调用equals方法比较不同类型的接口
25.EC: Call to equals() comparing different types (EC_UNRELATED_TYPES)
调用equals方法比较不同类型的类
26.EC: Using pointer equality to compare different types (EC_UNRELATED_TYPES_USING_POINTER_EQUALITY)
This method uses using pointer equality to compare two references that seem to be of different types. The result of this comparison will always be false at runtime.
27.Eq: equals method always returns false (EQ_ALWAYS_FALSE)
使用equals方法返回值总是false
28.Eq: equals method always returns true (EQ_ALWAYS_TRUE)
equals方法返回值总是true
29.Eq: equals method compares class names rather than class objects (EQ_COMPARING_CLASS_NAMES)
使用equals方法去比较一个类的实例和类的类型
30.Eq: Covariant equals() method defined for enum (EQ_DONT_DEFINE_EQUALS_FOR_ENUM)
This class defines an enumeration, and equality on enumerations are defined using object identity. Defining a covariant equals method for an enumeration value is exceptionally bad practice, since it would likely result in having two different enumeration values that compare as equals using the covariant enum method, and as not equal when compared normally. Don't do it.
31.Eq: equals() method defined that doesn't override equals(Object) (EQ_OTHER_NO_OBJECT)
类中定义的equals方法时不要覆写equals（Object）方法
32.Eq: equals() method defined that doesn't override Object.equals(Object) (EQ_OTHER_USE_OBJECT)
类中定义的equals方法时不要覆写Object中的equals（Object）方法
33.Eq: equals method overrides equals in superclass and may not be symmetric (EQ_OVERRIDING_EQUALS_NOT_SYMMETRIC)
34.Eq: Covariant equals() method defined, Object.equals(Object) inherited (EQ_SELF_USE_OBJECT)
类中定义了一组equals方法，但是都是继承的java.lang.Object class中的equals(Object)方法
35.FE: Doomed test for equality to NaN (FE_TEST_IF_EQUAL_TO_NOT_A_NUMBER)
This code checks to see if a floating point value is equal to the special Not A Number value (e.g., if (x == Double.NaN)). However, because of the special semantics of NaN, no value is equal to Nan, including NaN. Thus, x == Double.NaN always evaluates to false. To check to see if a value contained in x is the special Not A Number value, use Double.isNaN(x) (or Float.isNaN(x) if x is floating point precision).
36.FS: Format string placeholder incompatible with passed argument (VA_FORMAT_STRING_BAD_ARGUMENT)
错误使用参数类型来格式化字符串
37.FS: The type of a supplied argument doesn't match format specifier (VA_FORMAT_STRING_BAD_CONVERSION)
指定的格式字符串和参数类型不匹配，例如：String.format("%d", "1")
38.FS: MessageFormat supplied where printf style format expected (VA_FORMAT_STRING_EXPECTED_MESSAGE_FORMAT_SUPPLIED)
但用String的format方法时实际调用了ＭｅｓｓａｇｅＦｏｒｍａｔ中干的格式化方法而引起格式化结果出错。
39.FS: More arguments are passed than are actually used in the format string (VA_FORMAT_STRING_EXTRA_ARGUMENTS_PASSED)
使用String的format方法时有非法的参数也经过了格式化操作。
40.FS: Illegal format string (VA_FORMAT_STRING_ILLEGAL)
格式化String对象语句错误
41.FS: Format string references missing argument (VA_FORMAT_STRING_MISSING_ARGUMENT)
String的format操作缺少必要的参数。
42.FS: No previous argument for format string (VA_FORMAT_STRING_NO_PREVIOUS_ARGUMENT)
格式字符串定义错误，例如：formatter.format("%<s %s", "a", "b"); 抛出MissingFormatArgumentException异常
43.GC: No relationship between generic parameter and method argument (GC_UNRELATED_TYPES)
This call to a generic collection method contains an argument with an incompatible class from that of the collection's parameter (i.e., the type of the argument is neither a supertype nor a subtype of the corresponding generic type argument). Therefore, it is unlikely that the collection contains any objects that are equal to the method argument used here. Most likely, the wrong value is being passed to the method.
In general, instances of two unrelated classes are not equal. For example, if the Foo and Bar classes are not related by subtyping, then an instance of Foo should not be equal to an instance of Bar. Among other issues, doing so will likely result in an equals method that is not symmetrical. For example, if you define the Foo class so that a Foo can be equal to a String, your equals method isn't symmetrical since a String can only be equal to a String. 
In rare cases, people do define nonsymmetrical equals methods and still manage to make their code work. Although none of the APIs document or guarantee it, it is typically the case that if you check if a Collection<String> contains a Foo, the equals method of argument (e.g., the equals method of the Foo class) used to perform the equality checks. 
44.HE: Signature declares use of unhashable class in hashed construct (HE_SIGNATURE_DECLARES_HASHING_OF_UNHASHABLE_CLASS)
A method, field or class declares a generic signature where a non-hashable class is used in context where a hashable class is required. A class that declares an equals method but inherits a hashCode() method from Object is unhashable, since it doesn't fulfill the requirement that equal objects have equal hashCodes.
45.HE: Use of class without a hashCode() method in a hashed data structure (HE_USE_OF_UNHASHABLE_CLASS)
A class defines an equals(Object) method but not a hashCode() method, and thus doesn't fulfill the requirement that equal objects have equal hashCodes. An instance of this class is used in a hash data structure, making the need to fix this problem of highest importance.
46.ICAST: integral value cast to double and then passed to Math.ceil (ICAST_INT_CAST_TO_DOUBLE_PASSED_TO_CEIL)
integral的值转换为double后使用了Math.ceil方法
47.ICAST: int value cast to float and then passed to Math.round (ICAST_INT_CAST_TO_FLOAT_PASSED_TO_ROUND)
int 类型的值转换为float类型之后调用了Math.round方法
48.IJU: JUnit assertion in run method will not be noticed by JUnit (IJU_ASSERT_METHOD_INVOKED_FROM_RUN_METHOD)
在JUnit中的断言在run方法中不会被告知
49.IJU: TestCase declares a bad suite method (IJU_BAD_SUITE_METHOD)
在一个JUnit类中声明的一个suite()方法必须声明为
public static junit.framework.Test suite()
或者
public static junit.framework.TestSuite suite()的形式。
50.IL: A collection is added to itself (IL_CONTAINER_ADDED_TO_ITSELF)
集合本身作为add方法的参数，这样会引起内容溢出。
51.IL: An apparent infinite loop (IL_INFINITE_LOOP)
方法的自调用引起的死循环
52.IM: Integer multiply of result of integer remainder (IM_MULTIPLYING_RESULT_OF_IREM)
和整数余数进行乘法运算。例如：i % 60 * 1000 是进行(i % 60) * 1000运算而不是 i % (60 * 1000)
53.INT: Bad comparison of nonnegative value with negative constant (INT_BAD_COMPARISON_WITH_NONNEGATIVE_VALUE)
保证非负数和负数进行比较
54.INT: Bad comparison of signed byte (INT_BAD_COMPARISON_WITH_SIGNED_BYTE)
比较有符合数，要先把有符号数转换为无符合数再进行比较
55.IO: Doomed attempt to append to an object output stream (IO_APPENDING_TO_OBJECT_OUTPUT_STREAM)
宣布试图在对象的输出流处添加元素，如果你希望能够添加进一个对象的输出流中必须保证对象的输出流处于打开状态。
56.IP: A parameter is dead upon entry to a method but overwritten (IP_PARAMETER_IS_DEAD_BUT_OVERWRITTEN)
The initial value of this parameter is ignored, and the parameter is overwritten here. This often indicates a mistaken belief that the write to the parameter will be conveyed back to the caller.
传入参数的值被忽略，但是对传入值进行了修改，并返回给了调用者
57.MF: Class defines field that masks a superclass field (MF_CLASS_MASKS_FIELD)
子类中定义了和父类中同名的字段。在调用时会出错
58.MF: Method defines a variable that obscures a field (MF_METHOD_MASKS_FIELD)
在方法中定义的局部变量和类变量或者父类变量同名，从而引起字段混淆。
59.NP: Null pointer dereference (NP_ALWAYS_NULL)
对象赋为null值后 没有被重新赋值
60.NP: Null pointer dereference in method on exception path (NP_ALWAYS_NULL_EXCEPTION)
A pointer which is null on an exception path is dereferenced here.  This will lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
空指针引用上调用去除引用方法，将发生空指针异常
61.NP: Method does not check for null argument (NP_ARGUMENT_MIGHT_BE_NULL)
方法没有判断参数是否为空
62.NP: close() invoked on a value that is always null (NP_CLOSING_NULL)
一个为空的对象调用close方法
63.NP: Null value is guaranteed to be dereferenced (NP_GUARANTEED_DEREF)
There is a statement or branch that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).
在正常的null判断分支上，对象去除引用操作是受保护的不允许的
64.NP: Value is null and guaranteed to be dereferenced on exception path (NP_GUARANTEED_DEREF_ON_EXCEPTION_PATH)
There is a statement or branch on an exception path that if executed guarantees that a value is null at this point, and that value that is guaranteed to be dereferenced (except on forward paths involving runtime exceptions).
65.NP: Method call passes null to a nonnull parameter (NP_NONNULL_PARAM_VIOLATION)
方法中为null的参数没有被重新赋值
        void test(){
String ss = null;
sya(ss);
        }        
        public void sya(String ad){
ad.getBytes();
        }
66.NP: Method may return null, but is declared @NonNull (NP_NONNULL_RETURN_VIOLATION)
方法声明了返回值不能为空，但是方法中有可能返回null
67.NP: A known null value is checked to see if it is an instance of a type (NP_NULL_INSTANCEOF)
检查一个为null的值是否是想要的类型对象，而不是由于粗心或者逻辑错误引起的
68.NP: Possible null pointer dereference (NP_NULL_ON_SOME_PATH)
对象可能没有重新赋值
69.NP: Possible null pointer dereference in method on exception path (NP_NULL_ON_SOME_PATH_EXCEPTION)
A reference value which is null on some exception control path is dereferenced here.  This may lead to a NullPointerException when the code is executed.  Note that because FindBugs currently does not prune infeasible exception paths, this may be a false warning.
Also note that FindBugs considers the default case of a switch statement to be an exception path, since the default case is often infeasible.
在异常null值处理分支调用的方法上，可能存在对象去除引用操作
70.NP: Method call passes null for nonnull parameter (NP_NULL_PARAM_DEREF_ALL_TARGETS_DANGEROUS)
方法参数中声明为nonnull类型的参数为null
void test(){
String ss = null;
sya(ss);
        }        
        public void sya(@nonnull String ad){
ad.getBytes();
        }
71.NP: Store of null value into field annotated NonNull (NP_STORE_INTO_NONNULL_FIELD)
为一个已经声明为不能为null值的属性赋值为null。
72.Nm: Class defines equal(Object); should it be equals(Object)? (NM_BAD_EQUAL)
类中定义了一个equal方法但是却不是覆写的Object对象的equals方法
73.Nm: Class defines hashcode(); should it be hashCode()? (NM_LCASE_HASHCODE)
类中定义了一个hashCode方法但是却不是覆写的Object中的hashCode方法
74.Nm: Class defines tostring(); should it be toString()? (NM_LCASE_TOSTRING)
类中定义了一个toString方法但是却不是覆写的Object中的toString方法
75.Nm: Apparent method/constructor confusion (NM_METHOD_CONSTRUCTOR_CONFUSION)
构造方法定义混乱，保证一个标准的构造函数。        例如：
        SA(){        }
        void SA(){
        }
76.Nm: Very confusing method names (NM_VERY_CONFUSING)
混乱的方法命名，如getName和getname方法同时出现的时候
77.Nm: Method doesn't override method in superclass due to wrong package for parameter (NM_WRONG_PACKAGE)
方法因为取了不同包中的同名的对象而没有正确覆写父类中的同名方法
import alpha.Foo;
public class A {
  public int f(Foo x) { return 17; }
}

import beta.Foo;
public class B extends A {
  public int f(Foo x) { return 42; }
}
78.QBA: Method assigns boolean literal in boolean expression (QBA_QUESTIONABLE_BOOLEAN_ASSIGNMENT)
再if或者while表达式中使用boolean类型的值时应该使用==去判断，而不是采用=操作
79.RC: Suspicious reference comparison (RC_REF_COMPARISON)
比较两个对象值是否相等时应该采用equals方法，而不是==方法
80.RE: Invalid syntax for regular expression (RE_BAD_SYNTAX_FOR_REGULAR_EXPRESSION)
对正则表达式使用了错误的语法，会抛出未经检查的异常，表明正则表达式模式中的语法错误。
81.RE: File.separator used for regular expression (RE_CANT_USE_FILE_SEPARATOR_AS_REGULAR_EXPRESSION)
使用正则表达式使用了错误的文件分隔符，在windows系统中正则表达式不会匹配’\’而应该使用'\\'
82.RV: Random value from 0 to 1 is coerced to the integer 0 (RV_01_TO_INT)
从0到1随机值被强制为整数值0。在强制得到一个整数之前，你可能想得到多个随机值。或使用Random.nextInt（n）的方法。
83.RV: Bad attempt to compute absolute value of signed 32-bit hashcode (RV_ABSOLUTE_VALUE_OF_HASHCODE)
此代码生成一个哈希码，然后计算该哈希码的绝对值。如果哈希码是Integer.MIN_VALUE的，那么结果将是负数（因为Math.abs（Integer.MIN_VALUE的）== Integer.MIN_VALUE的）。
在2^ 32值之外字符串有一个Integer.MIN_VALUE的hashCode包括“polygenelubricants”，“GydZG_”和“，”DESIGNING WORKHOUSES “。
84.RV: Bad attempt to compute absolute value of signed 32-bit random integer (RV_ABSOLUTE_VALUE_OF_RANDOM_INT)
此代码生成一个随机的符号整数，然后计算该随机整数的绝对值。如果随机数生成数绝对值为Integer.MIN_VALUE的，那么结果将是负数（因为Math.abs（Integer.MIN_VALUE的）== Integer.MIN_VALUE的）。
85.RV: Exception created and dropped rather than thrown (RV_EXCEPTION_NOT_THROWN)
此代码创建一个异常（或错误）的对象，但不会用它做任何事情。例如：if (x < 0)
  new IllegalArgumentException("x must be nonnegative");
这可能是程序员的意图抛出创建的异常：
if (x < 0)
  throw new IllegalArgumentException("x must be nonnegative");
86.RV: Method ignores return value (RV_RETURN_VALUE_IGNORED)
该方法的返回值应该进行检查。这种警告通常出现在调用一个不可变对象的方法，认为它更新了对象的值。例如：String dateString = getHeaderField(name);
dateString.trim();
程序员似乎以为trim（）方法将更新dateString引用的字符串。但由于字符串是不可改变的，trim（）函数返回一个新字符串值，在这里它是被忽略了。该代码应更正：
String dateString = getHeaderField(name);
dateString = dateString.trim();
87.RpC: Repeated conditional tests (RpC_REPEATED_CONDITIONAL_TEST)
该代码包含对同一个条件试验了两次，两边完全一样例如：（如X == 0 | | x == 0）。可能第二次出现是打算判断别的不同条件（如X == 0 | | y== 0）。
88.SA: Double assignment of field (SA_FIELD_DOUBLE_ASSIGNMENT)
方法中的字段包含了双重任务，例如： 
 int x;
  public void foo() {
   x = x = 17;
  }
这种为变量赋值是无用的，并可能表明一个逻辑错误或拼写错误。
89.SA: Self assignment of field (SA_FIELD_SELF_ASSIGNMENT)
方法中包含自己对自己赋值的字段。例如：
int x;
  public void foo() {
    x = x;
  }
90.SA: Self comparison of field with itself (SA_FIELD_SELF_COMPARISON)
字段自己进行自比较可能表明错误或逻辑错误。
91.SA: Self comparison of value with itself (SA_LOCAL_SELF_COMPARISON)
方法中对一个局部变量自身进行比较运算，并可说明错误或逻辑错误。请确保您是比较正确的事情。
92.SA: Nonsensical self computation involving a variable (e.g., x & x) (SA_LOCAL_SELF_COMPUTATION)
此方法对同一变量执行了荒谬的计算（如x&x或x-x）操作。由于计算的性质，这一行动似乎没有意义，并可能表明错误或逻辑错误。
93.SF: Dead store due to switch statement fall through (SF_DEAD_STORE_DUE_TO_SWITCH_FALLTHROUGH)
在swtich中先前的case值因为swtich执行失败而被覆写，这就像是忘记使用break推出或者没有使用return语句放回先前的值一样。
94.SF: Dead store due to switch statement fall through to throw (SF_DEAD_STORE_DUE_TO_SWITCH_FALLTHROUGH_TO_THROW)
在swtich中因为出现异常而忽略了对case值的保存。
95.SIC: Deadly embrace of non-static inner class and thread local (SIC_THREADLOCAL_DEADLY_EMBRACE)
如果是一个静态内部类。实际上，在内部类和当前线程有死锁的可能。由于内部类不是静态的，它保留了对外部类的引用。如果线程包含对一个内部类实例的引用，那么内外实例的实例都可以被获取，这样就不具备垃圾会回收的资格。
96.SIO: Unnecessary type check done using instanceof operator (SIO_SUPERFLUOUS_INSTANCEOF)
在进行instanceof操作时进行没有必要的类型检查
97.STI: Unneeded use of currentThread() call, to call interrupted() (STI_INTERRUPTED_ON_CURRENTTHREAD)
此方法调用Thread.currentThread（）调用，只需调用interrupted（）方法。由于interrupted（）是一个静态方法， Thread.interrupted（）更简单易用。
98.STI: Static Thread.interrupted() method invoked on thread instance (STI_INTERRUPTED_ON_UNKNOWNTHREAD)
调用不是当前线程对象的Thread.interrupted()方法，由于interrupted（）方法是静态的，interrupted方法将会调用一个和作者原计划不同的对象。
99.Se: Method must be private in order for serialization to work (SE_METHOD_MUST_BE_PRIVATE)
这个类实现了Serializable接口，并定义自定义序列化的方法/反序列化。但由于这种方法不能声明为private，将被序列化/反序列化的API忽略掉。
100.Se: The readResolve method must not be declared as a static method. (SE_READ_RESOLVE_IS_STATIC)
为使readResolve方法得到序列化机制的识别，不能作为一个静态方法来声明。
101.UMAC: Uncallable method defined in anonymous class (UMAC_UNCALLABLE_METHOD_OF_ANONYMOUS_CLASS)
在匿名类中定义了一个既没有覆写超类中方法也不能直接调用的方法。因为在其他类的方法不能直接引用匿名类声明的方法，似乎这种方法不能被调用，这种方法可能只是没有任何作用的代码，但也可能覆写超类中声明。
102.UR: Uninitialized read of field in constructor (UR_UNINIT_READ)
此构造方法中使用了一个尚未赋值的字段或属性。
        String a;
        public SA() {
String abc = a;
System.out.println(abc);
        }
103.UR: Uninitialized read of field method called from constructor of superclass (UR_UNINIT_READ_CALLED_FROM_SUPER_CONSTRUCTOR)
方法被超类的构造函数调用时，在当前类中的字段或属性还没有被初始化。例如：
abstract class A {
  int hashCode;
  abstract Object getValue();
  A() {
    hashCode = getValue().hashCode();
    }
  }
class B extends A {
  Object value;
  B(Object v) {
    this.value = v;
    }
  Object getValue() {
    return value;
  }
  }
当B是创建时，A的构造函数将在B为value赋值之前触发，然而在A的初始化方法调用getValue方法时value这个变量还没有被初始化。
104.USELESS_STRING: Invocation of toString on an array (DMI_INVOKING_TOSTRING_ON_ANONYMOUS_ARRAY)
该代码调用上匿名数组的toString（）方法，产生的结果形如[@ 16f0472并没有实际的意义。考虑使用Arrays.toString方法来转换成可读的字符串，提供该数组的内容数组。例如：
String[] a = { "a" };
System.out.println(a.toString());
//正确的使用为
System.out.println(Arrays.toString(a));
105.USELESS_STRING: Invocation of toString on an array (DMI_INVOKING_TOSTRING_ON_ARRAY)
该代码调用上数组的toString（）方法，产生的结果形如[@ 16f0472并不能显示数组的真实内容。考虑使用Arrays.toString方法来转换成可读的字符串，提供该数组的内容数组
106.UwF: Field only ever set to null (UWF_NULL_FIELD)
字段的值总是为null值，所有读取该字段的值都为null。检查错误，如果它确实没有用就删除掉。
107.UwF: Unwritten field (UWF_UNWRITTEN_FIELD
此字段是永远不会写入值。所有读取将返回默认值。检查错误（如果它被初始化？），如果它确实没有用就删除掉。

### 五：Performance关于代码性能相关方面的
1.Bx: Primitive value is boxed and then immediately unboxed (BX_BOXING_IMMEDIATELY_UNBOXED)
对原始值进行装箱，然后立即取消装箱。这可能是在一个未要求装箱的地方进行了手动装箱，从而迫使编译器进行立即撤消装箱的操作
2.Bx: Primitive value is boxed then unboxed to perform primitive coercion (BX_BOXING_IMMEDIATELY_UNBOXED_TO_PERFORM_COERCION)
对原始值进行装箱然后立即把它强制转换为另外一种原始类型。例如：
new Double(d).intValue()应该直接进行强制转换例如：(int) d
3.Bx: Method allocates a boxed primitive just to call toString (DM_BOXED_PRIMITIVE_TOSTRING)
仅仅为了调用封装类的toString()而对原始类型进行封装操作。比这种方法更有效的是调用封装类的toString(…)方法例如：
new Integer(1).toString()    替换为   Integer.toString(1)
new Long(1).toString()    替换为   Long.toString(1) 
new Float(1.0).toString()    替换为   Float.toString(1.0) 
new Double(1.0).toString()    替换为   Double.toString(1.0) 
new Byte(1).toString()    替换为   Byte.toString(1) 
new Short(1).toString()    替换为   Short.toString(1) 
new Boolean(true).toString()    替换为   Boolean.toString(true)
4.Bx: Method invokes inefficient floating-point Number constructor; use static valueOf instead (DM_FP_NUMBER_CTOR)
使用new Double(double)方法总是会创建一个新的对象，然而使用Double.valueOf(double)方法可以把值保存在编辑器或者class library、JVM中。使用存储值的方式来避免对象的分配可以或得更好的代码性能
除非类必须符合Java 1.5以前的JVM，否则请使用自动装箱或valueOf（）方法创建Double和Float实例。
5.Bx: Method invokes inefficient Number constructor; use static valueOf instead (DM_NUMBER_CTOR)
使用new Integer(int)方法总是会创建一个新的对象，然而使用Integer.valueOf(int)方法可以把值保存在编辑器或者class library、JVM中。使用存储值的方式来避免对象的分配可以或得更好的代码性能
除非类必须符合Java 1.5以前的JVM，否则请使用自动装箱或valueOf（）方法创建Long, Integer, Short, Character, Byte实例。
6.Dm: The equals and hashCode methods of URL are blocking (DMI_BLOCKING_METHODS_ON_URL)
使用equals和hashCode方法来对url进行资源标识符解析时会引起堵塞。考虑使用java.net.URI来代替。
7.Dm: Maps and sets of URLs can be performance hogs (DMI_COLLECTION_OF_URLS)
方法或者字段使用url的map/set集合。因为equals方法或者hashCode方法来进行资源标识符解析时都会引起堵塞。考虑使用java.net.URI来代替。
8.Dm: Method invokes inefficient Boolean constructor; use Boolean.valueOf(...) instead (DM_BOOLEAN_CTOR)
使用new方法创建一个java.lang.Boolean类型能够的实例对象是浪费空间的，因为Boolean对象是不可变的而且只有两个有用的值。使用Boolean.valueOf()或者Java1.5中的自动装箱功能来创建一个Boolean实例。
9.Dm: Explicit garbage collection; extremely dubious except in benchmarking code (DM_GC)
在代码中显式的调用垃圾回收命名，这样做并不能起作用。在过去，有人在关闭操作或者finalize方法中调用垃圾回收方法导致了很多的性能浪费。这样大规模回收对象时会造成处理器运行缓慢。
10.Dm: Use the nextInt method of Random rather than nextDouble to generate a random integer (DM_NEXTINT_VIA_NEXTDOUBLE)
 如果r是一个java.util.Random对象，你可以使r.nextInt(n)生成一个0到n-1之前的随机数，而不是使用(int)(r.nextDouble() * n)
11.Dm: Method invokes inefficient new String(String) constructor (DM_STRING_CTOR)
使用java.lang.String(String)构造函数会浪费内存因为这种构造方式和String作为参数在功能上容易混乱。只是使用String直接作为参数的形式
12.Dm: Method invokes toString() method on a String (DM_STRING_TOSTRING)
调用String.toString()是多余的操作，只要使用String就可以了。
13.Dm: Method invokes inefficient new String() constructor (DM_STRING_VOID_CTOR)
使用没有参数的构造方法去创建新的String对象是浪费内存空间的，因为这样创建会和空字符串“”混淆。Java中保证完成相同的构造方法会产生描绘相同的String对象。所以你只要使用空字符串来创建就可以了。
14.ITA: Method uses toArray() with zero-length array argument (ITA_INEFFICIENT_TO_ARRAY)
当使用集合的toArray()方法时使用数组长度为0的数组作为参数。比这更有效的一种方法是
myCollection.toArray(new Foo[myCollection.size()])，如果数组的长度足够大就可以直接把集合中的内容包装到数组中直接返回从而避免了第二次创建一个新的数组来存放集合中值。
15.SBSC: Method concatenates strings using + in a loop (SBSC_USE_STRINGBUFFER_CONCATENATION)
在循环中构建一个String对象时从性能上讲使用StringBuffer来代替String对象
例如：
// This is bad
  String s = "";
  for (int i = 0; i < field.length; ++i) {
    s = s + field[i];
  }
  // This is better
  StringBuffer buf = new StringBuffer();
  for (int i = 0; i < field.length; ++i) {
    buf.append(field[i]);
  }
  String s = buf.toString();
16.SS: Unread field: should this field be static? (SS_SHOULD_BE_STATIC)
类中所包含的final属性字段在编译器中初始化为静态的值。考虑在定义时就把它定义为static类型的。
17.UM: Method calls static Math class method on a constant value (UM_UNNECESSARY_MATH)
在方法中使用了java.lang.Math的静态方法代替常量来使用，使用常量速度和准确度会更好。 以下为Math中的方法产生的值。
Method Parameter 
abs -any- 
acos 0.0 or 1.0 
asin 0.0 or 1.0 
atan 0.0 or 1.0 
atan2 0.0 cbrt 0.0 or 1.0 
ceil -any- 
cos 0.0 
cosh 0.0 
exp 0.0 or 1.0 
expm1 0.0 
floor -any- 
log 0.0 or 1.0 
log10 0.0 or 1.0 
rint -any- 
round -any- 
sin 0.0 
sinh 0.0 
sqrt 0.0 or 1.0 
tan 0.0 
tanh 0.0 
toDegrees 0.0 or 1.0 
toRadians 0.0
18.UPM: Private method is never called (UPM_UNCALLED_PRIVATE_METHOD)
定义为Private类型方法从未被调用，应该被删除。
19.UrF: Unread field (URF_UNREAD_FIELD)
类中定义的属性从未被调用，建议删除。
20.UuF: Unused field (UUF_UNUSED_FIELD)
类中定义的属性从未被使用，建议删除。
21.WMI: Inefficient use of keySet iterator instead of entrySet iterator (WMI_WRONG_MAP_ITERATOR)
当方法中接受一个Map类型的参数时，使用keySet的迭代器比使用entrySet的迭代器效率要高。
### 六：Internationalization关于代码国际化相关方面的
Dm: Consider using Locale parameterized version of invoked method (DM_CONVERT_CASE)
使用平台默认的编码格式对字符串进行大小写转换，这可能导致国际字符的转换不当。使用以下方式对字符进行转换
String.toUpperCase( Locale l )
String.toLowerCase( Locale l )

### 七：Multithreaded correctness关于代码多线程正确性相关方面的
1.DL: Synchronization on Boolean could lead to deadlock (DL_SYNCHRONIZATION_ON_BOOLEAN)
该代码同步一个封装的原始常量，例如一个Boolean类型。
private static Boolean inited = Boolean.FALSE;
...
  synchronized(inited) { 
    if (!inited) {
       init();
       inited = Boolean.TRUE;
       }
     }
...
由于通常只存在两个布尔对象，此代码可能是同步的其他无关的代码中相同的对象，这时会导致反应迟钝和可能死锁
2.DL: Synchronization on boxed primitive could lead to deadlock (DL_SYNCHRONIZATION_ON_BOXED_PRIMITIVE)
该代码同步一个封装的原始常量，例如一个Integer类型。
private static Integer count = 0;
...
  synchronized(count) { 
     count++;
     }
...
由于Integer对象可以共享和保存，此代码可能是同步的其他无关的代码中相同的对象，这时会导致反应迟钝和可能死锁
3.DL: Synchronization on interned String could lead to deadlock (DL_SYNCHRONIZATION_ON_SHARED_CONSTANT)
同步String类型的常量时，由于它被JVM中多个其他的对象所共有，这样在其他代码中会引起死锁。
4.DL: Synchronization on boxed primitive values (DL_SYNCHRONIZATION_ON_UNSHARED_BOXED_PRIMITIVE)
同步一个显然不是共有封装的原始值，例如一个Integer类型的对象。例如：
private static final Integer fileLock = new Integer(1);
...
  synchronized(fileLock) { 
     .. do something ..
     }
...
它最后被定义为以下方式来代替：private static final Object fileLock = new Object();
5.Dm: Monitor wait() called on Condition (DM_MONITOR_WAIT_ON_CONDITION)
方法中以java.util.concurrent.locks.Condition对象调用wait（）。等待一个条件发生时应该使用在Condition接口中定义的await()方法。
6.Dm: A thread was created using the default empty run method (DM_USELESS_THREAD)
这个方法没有通过run方法或者具体声明Thread类，也没有通过一个Runnable对象去定义一个线程，而这个线程出来浪费资源却什么也没有去做。
7.ESync: Empty synchronized block (ESync_EMPTY_SYNC)
该代码包含一个空的同步块：synchronized() {}
8.IS: Inconsistent synchronization (IS2_INCONSISTENT_SYNC)
不合理的同步
9.IS: Field not guarded against concurrent access (IS_FIELD_NOT_GUARDED)
域不是良好的同步访问---
此字段被标注为net.jcip.annotations.GuardedBy，但可以在某种程度上违反注释而去访问
10.JLM: Synchronization performed on Lock (JLM_JSR166_LOCK_MONITORENTER)
实现java.util.concurrent.locks.Lock的对象调用了同步的方法。应该这样处理，对象被锁定/解锁时使用acquire（）/ release（）方法而不是使用同步的方法。
11.LI: Incorrect lazy initialization of static field (LI_LAZY_INIT_STATIC)
静态域不正确的延迟初始化--
这种方法包含了一个不同步延迟初始化的非volatile静态字段。因为编译器或处理器可能会重新排列指令，如果该方法可以被多个线程调用，线程不能保证看到一个完全初始化的对象。你可以让字段可变来解决此问题
12.LI: Incorrect lazy initialization and update of static field (LI_LAZY_INIT_UPDATE_STATIC)
这种方法包含一个不同步延迟初始化的静态字段。之后为字段赋值，对象存储到该位置后进一步更新或访问。字段后尽快让其他线程能够访问。如果该方法的进一步访问该字段为初始化对象提供服务，然后你有一个非常严重的多线程bug，除非别的东西阻止任何其他线程访问存储的对象，直到它完全初始化。
即使你有信心，该方法是永远不会被多个线程调用时，在它的值还没有被充分初始化或移动，不把它设定为static字段时它可能会更好。
13.ML: Method synchronizes on an updated field (ML_SYNC_ON_UPDATED_FIELD)
对象获取一个可变字段时进行同步。这是没有意义的，因为不同的线程可以在不同的对象同步。
14.MSF: Mutable servlet field (MSF_MUTABLE_SERVLET_FIELD)
一个web服务一般只能创建一个servlet或者jsp的实例（例如：treates是一个单利类），它会被多个线程调用这个实例的方法服务于多个同时的请求。因此使用易变的字段属性产生竞争的情况。
15.MWN: Mismatched notify() (MWN_MISMATCHED_NOTIFY)
此方法调用Object.notify（）或Object.notifyAll（）而没有获取到该对象的对象锁。调用notify（）或notifyAll（）而没有持有该对象的对象锁，将导致IllegalMonitorStateException异常。
16.MWN: Mismatched wait() (MWN_MISMATCHED_WAIT)
此方法调用Object.wait()而没有获取到该对象的对象锁。调用wait（）而没有持有该对象的对象锁，将导致IllegalMonitorStateException异常。
17.NP: Synchronize and null check on the same field. (NP_SYNC_AND_NULL_CHECK_FIELD)
如果代码块是同步的，那么久不可能为空。如果是空，同步时就会抛出NullPointerException异常。最好是在另一个代码块中进行同步。
18.No: Using notify() rather than notifyAll() (NO_NOTIFY_NOT_NOTIFYALL)
调用notify（）而不是notifyAll（）方法。 Java的监控器通常用于多个条件。调用notify（）只唤醒一个线程，这意味着该线程被唤醒只是满足的当前的唯一条件。
19.RS: Class's readObject() method is synchronized (RS_READOBJECT_SYNC)
序列化类中定义了同步的readObject（）。通过定义，反序列化创建的对象只有一个线程可以访问，因此没有必要的readObject（）进行同步。如果的readObject（）方法本身造成对象对另一个线程可见，那么这本身就是不好的编码方式。
20.Ru: Invokes run on a thread (did you mean to start it instead?) (RU_INVOKE_RUN)
这种方法显式调用一个对象的run（）。一般来说，类是实现Runnable接口的，因为在一个新的线程他们将有自己的run（）方法，在这种情况下Thread.start（）方法调用是正确的。
21.SC: Constructor invokes Thread.start() (SC_START_IN_CTOR)
在构造函数中启动一个线程。如果类曾经被子类扩展过，那么这很可能是错的，因为线程将在子类构造之前开始启动。
22.SP: Method spins on field (SP_SPIN_ON_FIELD)
方法无限循环读取一个字段。编译器可合法悬挂宣读循环，变成一个无限循环的代码。这个类应该改变，所以使用适当的同步（包括等待和通知要求）
23.STCAL: Call to static Calendar (STCAL_INVOKE_ON_STATIC_CALENDAR_INSTANCE)
即使JavaDoc对此不包含暗示，而Calendars本身在多线程中使用就是不安全的。探测器发现当调用Calendars的实例时将会获得一个静态对象。
Calendar rightNow = Calendar.getInstance();
24.STCAL: Call to static DateFormat (STCAL_INVOKE_ON_STATIC_DATE_FORMAT_INSTANCE)
在官方的JavaDoc，DateFormats多线程使用本事就是不安全的。探测器发现调用一个DateFormat的实例将会获得一个静态对象。
myString = DateFormat.getDateInstance().format(myDate);
25.STCAL: Static Calendar (STCAL_STATIC_CALENDAR_INSTANCE)
Calendar在多线程中本身就是不安全的，如果在线程范围中共享一个Calendarde 实例而不使用一个同步的方法在应用中就会出现一些奇怪的行为。在sun.util.calendar.BaseCalendar.getCalendarDateFromFixedDate()中会抛出ArrayIndexOutOfBoundsExceptions or IndexOutOfBoundsExceptions异常。
26.STCAL: Static DateFormat (STCAL_STATIC_SIMPLE_DATE_FORMAT_INSTANCE)
DateFormat 在多线程中本身就是不安全的，如果在线程范围中共享一个DateFormat的实例而不使用一个同步的方法在应用中就会出现一些奇怪的行为。
27.SWL: Method calls Thread.sleep() with a lock held (SWL_SLEEP_WITH_LOCK_HELD)
当持有对象时调用Thread.sleep（）。这可能会导致很差的性能和可扩展性，或陷入死锁，因为其他线程可能正在等待获得锁。调用wait（）是一个更好的主意，释放对象的持有以允许其他线程运行。
28.UG: Unsynchronized get method, synchronized set method (UG_SYNC_SET_UNSYNC_GET)
这个类包含类似命名的get和set方法。在set方法是同步方法和get方法是非同步方法。这可能会导致在运行时的不正确行为，因为调用的get方法不一定返回对象一致状态。 GET方法应该同步。
29.UL: Method does not release lock on all paths (UL_UNRELEASED_LOCK)
方法获得了当前的对象所，但是在方法中始终没有释放它。一个正确的示例如下：
  Lock l = ...;
    l.lock();
    try {
        // do something
    } finally {
        l.unlock();
    }
30.UL: Method does not release lock on all exception paths (UL_UNRELEASED_LOCK_EXCEPTION_PATH)
方法获得了当前的对象所，但是在所有的异常处理中始终没有释放它。一个正确的示例如下：
  Lock l = ...;
    l.lock();
    try {
        // do something
    } finally {
        l.unlock();
    }
31.UW: Unconditional wait (UW_UNCOND_WAIT)
方法中包含调用java.lang.Object.wait（），而却没有放到条件流程控制中。该代码应确认条件尚未满足之前等待;先前任何通知将被忽略。
32.VO: A volatile reference to an array doesn't treat the array elements as volatile (VO_VOLATILE_REFERENCE_TO_ARRAY)
声明一个变量引用数组，这可能不是你想要的。如果一个变量引用数组，那么对引用数组的读和写都是不安全的，但是数组元素不是变量。取得数组的变量值你可以使用java.util.concurrent包中的数组的原子性特性
33.WL: Sychronization on getClass rather than class literal (WL_USING_GETCLASS_RATHER_THAN_CLASS_LITERAL)
实例的方法中同步this.getClass()，如果这个类有子类集合，那么子类集合中的对象将会在这个类的各个子类上进行同步，这不是我们想要的效果，我们只要同步当前的类对象而不包含它的所有子类，可以同步类名.getClass()。例如，java.awt.Label的代码：
     private static final String base = "label";
     private static int nameCounter = 0;
     String constructComponentName() {
        synchronized (getClass()) {
            return base + nameCounter++;
        }
     }
Label中的子类集合不可能在同一个子对象上进行同步，替换上面的方法为：
    private static final String base = "label";
     private static int nameCounter = 0;
     String constructComponentName() {
        synchronized (Label.class) {
            return base + nameCounter++;
        }
     }
34.WS: Class's writeObject() method is synchronized but nothing else is (WS_WRITEOBJECT_SYNC)
这个类有一个writeObject（）方法是同步的，但是这个类中没有其他的同步方法。
35.Wa: Condition.await() not in loop (WA_AWAIT_NOT_IN_LOOP)
方法没有在循环中调用java.util.concurrent.await()。如果对象是用于多种条件，打算调用wait()方法的条件可能不是实际发生的。
36.Wa: Wait not in loop (WA_NOT_IN_LOOP)
这种方法包含调用java.lang.Object.wait（），而这并不是一个循环。如果监视器用于多个条件，打算调用wait()方法的条件可能不是实际发生的。

### 八：Malicious codevulnerability关于恶意破坏代码相关方面的
1.EI: May expose internal representation by returning reference to mutable object (EI_EXPOSE_REP)
返回一个易变对象引用并把它保存在对象字段中时会暴露对象内部的字段描述，如果接受不守信任的代码访问或者没有检查就去改变易变对象的会涉及对象的安全和其他重要属性的安全。返回一个对象的新副本，在很多情况下更好的办法。
2.EI2: May expose internal representation by incorporating reference to mutable object (EI_EXPOSE_REP2)
此代码把外部可变对象引用存储到对象的内部表示。如果实例受到不信任的代码的访问和没有检查的变化危及对象和重要属性的安全。存储一个对象的副本，在很多情况下是更好的办法。
3.FI: Finalizer should be protected, not public (FI_PUBLIC_SHOULD_BE_PROTECTED)
一个类中的finalize（）方法必须声明为protected，而不能为public类型
4.MS: Public static method may expose internal representation by returning array (MS_EXPOSE_REP)
一个public类型的静态方法返回一个数组，可能引用内部属性的暴露。任何代码调用此方法都可以自由修改底层数组。一个解决办法是返回一个数组的副本。
5.MS: Field should be both final and package protected (MS_FINAL_PKGPROTECT)
一个静态字段可能被恶意代码或另外一个包所改变的。字段可以放到protected包中也可以定义为final类型的以避免此问题。
6.MS: Field is a mutable array (MS_MUTABLE_ARRAY)
一个定义为final类型的静态字段引用一个数组时它可以被恶意代码或在另其他包中所使用。这些代码可以自由修改数组的内容。
7.MS: Field is a mutable Hashtable (MS_MUTABLE_HASHTABLE)
一个定义为final类型的静态字段引用一个Hashtable时可以被恶意代码或者在其他包中被调用，这些方法可以修改Hashtable的值。
8.MS: Field should be moved out of an interface and made package protected (MS_OOI_PKGPROTECT)
将域尽量不要定义在接口中，并声明为包保护
在接口中定义了一个final类型的静态字段，如数组或哈希表等易变对象。这些对象可以被恶意代码或者在其他包中被调用，为了解决这个问题，需要把它定义到一个具体的实体类中并且声明为保护类型以避免这种错误。
9.MS: Field should be package protected (MS_PKGPROTECT)
一个静态字段是可以改变的恶意代码或其他的包访问修改。可以把这种类型的字段声明为final类型的以防止这种错误。

### 十：Dodgy关于代码运行期安全方面的
1.XSS: Servlet reflected cross site scripting vulnerability (XSS_REQUEST_PARAMETER_TO_SEND_ERROR)
在代码中在Servlet输出中直接写入一个HTTP参数，这会造成一个跨站点的脚本漏洞。
2.XSS: Servlet reflected cross site scripting vulnerability (XSS_REQUEST_PARAMETER_TO_SERVLET_WRITER)
代码直接写入参数的HTTP服务器错误页（使用HttpServletResponse.sendError）。表达了类似的不受信任的输入会引起跨站点脚本漏洞。
3.BC: Questionable cast to abstract collection (BC_BAD_CAST_TO_ABSTRACT_COLLECTION)
在代码投把一个集合强制类型转换为一个抽象的集合（如list，set或map）。保证该对象类型和将要转换的类型是一致的。如果你只是想要便利一个集合，那么你就不必将它转换为Set或List。
4.BC: Questionable cast to concrete collection (BC_BAD_CAST_TO_CONCRETE_COLLECTION)
代码把抽象的集合（如List，Set，或Collection）强制转换为具体落实类型（如一个ArrayList或HashSet）。这可能不正确，也可能使您的代码很脆弱，因为它使得难以在今后的切换指向其他具体实现。除非你有特别理由这样做，否则只需要使用抽象的集合类。
5.BC: Unchecked/unconfirmed cast (BC_UNCONFIRMED_CAST)
强制类型转换操作没有经过验证，而且不是所有的此种类型装换过的类都可以再强制类型转换为原类型。在代码中需要进行逻辑判断以保证可以进行这样的操作。
6.BC: instanceof will always return true (BC_VACUOUS_INSTANCEOF)
instanceof测试将始终返回真（除非被测试的值为空）。虽然这是安全，确保它是不是说明一些误解或其他一些逻辑错误。如果你真的想测试是空的价值，也许会更清楚这样做的更好空试验，而不是一个instanceof测试。
7.BSHIFT: Unsigned right shift cast to short/byte (ICAST_QUESTIONABLE_UNSIGNED_RIGHT_SHIFT)
无符号数右移后进行转换为short或者byte类型时可能会丢弃掉高位的值，这样的结果就是有符合数和无符号数无法区分（这取决于移位大小）
8.CI: Class is final but declares protected field (CI_CONFUSED_INHERITANCE)
这个类被声明为final的，而是字段属性却声明为保护类型的。由于是final类，它不能再被继承，而再声明为保护类型的很容易造成混淆。为了从外部能正确的使用它应该把它们声明为private或者public类型。
9.DB: Method uses the same code for two branches (DB_DUPLICATE_BRANCHES)
此方法使用相同的代码，以实现两个有条件的分支。检查以确保这是不是一个编码错误。
10.DB: Method uses the same code for two switch clauses (DB_DUPLICATE_SWITCH_CLAUSES)
他的方法使用相同的代码来实现两个switch的声明条款。这可能是重复代码的情况，但可能也显示出编码的错误。
11.DLS: Dead store to local variable (DLS_DEAD_LOCAL_STORE)
该指令为局部变量赋值，但在其后的没有对她做任何使用。通常，这表明一个错误，因为值从未使用过。
12.DLS: Useless assignment in return statement (DLS_DEAD_LOCAL_STORE_IN_RETURN)
本声明把一个局部变量放到方法的返回语句中。这对于方法中局部变量来说是没有意思的。
13.DLS: Dead store of null to local variable (DLS_DEAD_LOCAL_STORE_OF_NULL)
把一个本地变量赋值为null值，并且再也没有对这个变量做任何的操作。这样可能是为了垃圾回收，而是Java SE 6.0，这已不再需要。
14.DMI: Code contains a hard coded reference to an absolute pathname (DMI_HARDCODED_ABSOLUTE_FILENAME)
此代码包含文件对象为一个绝对路径名（例如，新的文件（“/ home/dannyc/workspace/j2ee/src/share/com/sun/enterprise/deployment”）;
15.DMI: Non serializable object written to ObjectOutput (DMI_NONSERIALIZABLE_OBJECT_WRITTEN)
代码中让一个非序列化的对象出现在ObjectOutput.writeObject()方法中，这样会引起一个错误。
16.DMI: Invocation of substring(0), which returns the original value (DMI_USELESS_SUBSTRING)
此代码调用了subString(0)方法，它将返回原来的值。
17.Eq: Class doesn't override equals in superclass (EQ_DOESNT_OVERRIDE_EQUALS)
子类定义了一个新的equals方法但是却不是覆写了父类本省的equals()方法。
18.FE: Test for floating point equality (FE_FLOATING_POINT_EQUALITY)
此操作比较两个浮点值是否相等。由于浮点运算可能会涉及到舍入，计算float和double值可能不准确。如果要求值必须准确，如货币值，可以考虑使用固定精度类型，如BigDecimal类型的值来比较
19.FS: Non-Boolean argument formatted using %b format specifier (VA_FORMAT_STRING_BAD_CONVERSION_TO_BOOLEAN)
使用%b去格式化Boolean类型的值不正确的但是它不会抛出异常，任何非空的值都会输出true，任何为空的值都会输出false
20.IC: Initialization circularity (IC_INIT_CIRCULARITY)
在引用两个相互调用为环状static方法去初始化一个实例时是错误的。
21.ICAST: integral division result cast to double or float (ICAST_IDIV_CAST_TO_DOUBLE)
整形数除法强制转换为double或者float类型。
int x = 2;
int y = 5;
// Wrong: yields result 0.0
double value1 =  x / y;
// Right: yields result 0.4
double value2 =  x / (double) y;
22.ICAST: Result of integer multiplication cast to long (ICAST_INTEGER_MULTIPLY_CAST_TO_LONG)
整形数做乘法运算结果转换为long值时如果采用
long convertDaysToMilliseconds(int days) { return 1000*3600*24*days; } 结果会因为超出整形的范围而出错。
如果使用：
long convertDaysToMilliseconds(int days) { return 1000L*3600*24*days; } 
或者：
static final long MILLISECONDS_PER_DAY = 24L*3600*1000;
        long convertDaysToMilliseconds(int days) { return days * MILLISECONDS_PER_DAY; } 
都可以避免此问题。
23.IM: Computation of average could overflow (IM_AVERAGE_COMPUTATION_COULD_OVERFLOW)
代码中使用x % 2 == 1的方法去验证运算是否存在余数的情况，但是如果出现负数的情况就不起作用了。使用x & 1 == 1, or x % 2 != 0来代替
24.INT: Vacuous comparison of integer value (INT_VACUOUS_COMPARISON)
整形数进行比较结果总是不变。例如：x <= Integer.MAX_VALUE
25.MTIA: Class extends Servlet class and uses instance variables (MTIA_SUSPECT_SERVLET_INSTANCE_FIELD)
这个类扩展从Servlet类，并使用实例的成员变量。由于只有一个Servlet类的实例，并在多线程方式使用，这种模式有可能存在问题。考虑只使用方法的局部变量。
26.MTIA: Class extends Struts Action class and uses instance variables (MTIA_SUSPECT_STRUTS_INSTANCE_FIELD)
类扩展自Struts的Action类并使用这个实例的成员变量，因为在Struts框架中只存在一个Action实例对象并且使用在多线程的情况下很可能会出现问题。
27.NP: Dereference of the result of readLine() without nullcheck (NP_DEREFERENCE_OF_READLINE_VALUE)
对readLine()的结果值没有进行判空操作就去重新赋值，这样的操作可以会抛出空指针异常。
28.NP: Immediate dereference of the result of readLine() (NP_IMMEDIATE_DEREFERENCE_OF_READLINE)
对readLine()的结果立即赋值，这样的操作可以会抛出空指针异常。
29.NP: Possible null pointer dereference due to return value of called method (NP_NULL_ON_SOME_PATH_FROM_RETURN_VALUE)
方法的返回值没有进行是否为空的检查就重新赋值，这样可能会出现空指针异常。
30.NP: Parameter must be nonnull but is marked as nullable (NP_PARAMETER_MUST_BE_NONNULL_BUT_MARKED_AS_NULLABLE)
参数值在任何情况下都不能为空，但是有明确的注释它可以为空。
31.NS: Potentially dangerous use of non-short-circuit logic (NS_DANGEROUS_NON_SHORT_CIRCUIT)
代码中使用（& or |）代替（&& or ||）操作，这会造成潜在的危险。
32.NS: Questionable use of non-short-circuit logic (NS_NON_SHORT_CIRCUIT)
代码中使用（& or |）代替（&& or ||）操作，会引起不安全的操作
33.PZLA: Consider returning a zero length array rather than null (PZLA_PREFER_ZERO_LENGTH_ARRAYS)
考虑返回一个零长度的数组，而不是null值
34.QF: Complicated, subtle or wrong increment in for-loop (QF_QUESTIONABLE_FOR_LOOP)
确定这个循环是正确的变量递增，看起来，另一个变量被初始化，检查的循环。这是由于for循环中太复杂的定义造成的。
35.RCN: Redundant comparison of non-null value to null (RCN_REDUNDANT_COMPARISON_OF_NULL_AND_NONNULL_VALUE)
方法中包含一个不能为空的赋值还包含一个可以为空的赋值。冗余比较非空值为空。
36.RCN: Redundant comparison of two null values (RCN_REDUNDANT_COMPARISON_TWO_NULL_VALUES)
方法中对两个null值进行比较
37.RCN: Redundant nullcheck of value known to be non-null (RCN_REDUNDANT_NULLCHECK_OF_NONNULL_VALUE)
方法中对不为空的值进行为空的判断。
38.REC: Exception is caught when Exception is not thrown (REC_CATCH_EXCEPTION)
在try/catch块中捕获异常，但是异常没有在try语句中抛出而RuntimeException又没有明确的被捕获
39.RI: Class implements same interface as superclass (RI_REDUNDANT_INTERFACES)
子类和父类都实现了同一个接口，这种定义是多余的。
40.RV: Method discards result of readLine after checking if it is nonnull (RV_DONT_JUST_NULL_CHECK_READLINE)
readLine方法的结果不为空时被抛弃
41.RV: Remainder of 32-bit signed random integer (RV_REM_OF_RANDOM_INT)
此代码生成一个随机的符号整数，然后计算另一个值的。由于随机数可以是负数，所以其余操作的结果也可以是负面的。考虑使用Random.nextInt（int）方法代替。
42.SA: Double assignment of local variable (SA_LOCAL_DOUBLE_ASSIGNMENT)
为一个局部变量两次赋值，这样是没有意义的。例如：
public void foo() {
    int x,y;
    x = x = 17;
  }
43.SA: Self assignment of local variable (SA_LOCAL_SELF_ASSIGNMENT)
局部变量使用自身给自己赋值
public void foo() {
    int x = 3;
    x = x;
  }
44.SF: Switch statement found where one case falls through to the next case (SF_SWITCH_FALLTHROUGH)
Switch语句中一个分支执行后又执行了下一个分支。通常case后面要跟break 或者return语句来跳出。
45.SF: Switch statement found where default case is missing (SF_SWITCH_NO_DEFAULT)
Switch没有默认情况下执行的case语句。
46.Se: private readResolve method not inherited by subclasses (SE_PRIVATE_READ_RESOLVE_NOT_INHERITED)
声明为private的序列化方法被子类继承
47.UCF: Useless control flow (UCF_USELESS_CONTROL_FLOW)
没有任何作用的条件语句。
if (argv.length == 0) {
        // TODO: handle this case
        }
48.UCF: Useless control flow to next line (UCF_USELESS_CONTROL_FLOW_NEXT_LINE)
无效的条件控制语句，注意if (argv.length == 1);以“;”结尾，下面的语句无论是否满足都会运行。
if (argv.length == 1);
        System.out.println("Hello, " + argv[0]);
49.UwF: Field not initialized in constructor (UWF_FIELD_NOT_INITIALIZED_IN_CONSTRUCTOR)
字段从来没有在任何构造函数初始化，对象被创建后值为空。如果该字段未被定义就重新赋值会产生一个空指针异常。
50.XFB: Method directly allocates a specific implementation of xml interfaces (XFB_XML_FACTORY_BYPASS)
方法自定义了一种XML接口的实现类。最好是使用官方提供的工厂类来创建这些对象，以便可以在运行期中改变。例如：
javax.xml.parsers.DocumentBuilderFactory
javax.xml.parsers.SAXParserFactory
javax.xml.transform.TransformerFactory
org.w3c.dom.Document.createXXXX





