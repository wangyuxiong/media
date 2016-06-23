# java反射机制及动态代理

目前正在看Hadoop RPC框架的源码，在深入了解这些之前，有一些基础知识需要回顾下。

> 1. java反射机制及动态代理
> 2. java网络编程
> 3. java NIO

先总结下第一个，java反射机制及动态代理的相关知识点：

## java反射机制
在看与java反射机制相关的代码前，试着看看下面这几个问题：

- 什么是反射? 
- 为什么使用反射? 
- 使用它有哪些好处? 
- 哪些地方需要反射?

在程序运行时，允许改变程序结构和变量名称。如python ,ruby这类，我们称为是动态语言。
而对于java，C++，C# 这类则不能称为动态语言，我们一般称这些为静态语言。

```
int i = 1
i = “hello world ” 
```  

如上面这段代码，在静态语言中，在编译阶段编译器就会报错。而对于动态语言，是可以修改变量类型的，如下面：
   
```
i = 1
i = ‘hi’
```

### 1. 先尝试看看第一个问题，什么是反射机制？
  
  在运行时环境，动态获取类的信息以及动态调用对象的方法的功能，就是reflection机制。

### 2. 哪些地方需要反射？
   - 运行时判断任何一个对象所属的类
   - 运行时构造任何一个类的对象
   - 运行时判断任何一个类所具有的成员变量和方法
   - 运行时调用任何一个对象的方法

### 3. 反射的使用？
  先看看java reflection api， Class类是反射的入口点。有下面3种方式获取：
  
``` 
   1. Class.forName(“java.util.Data”)
   2. T.getClass()
   3. T.class
```

 一个Class对象实际表示一个类型，但这个类型不一定是一种类。比如说int不表示类，但是int.class是一个Class类型的对象。
注：数组类型，使用getName会返回一个很奇怪的名字，如：
`System.out.println(Double[].class.getName());  `
显示打印的值如下：
`[Ljava.lang.Double;`

创建一个类的实例newInstance方法：使用默认的构造函数，没有参数

```
T.class.newInstance();
Object objCopy = classType.getConstructor(new Class[]{}).newInstance(new Object[]{});
```

**反射分析类的能力：**
1. java.lang.reflect包下面有3个类，Field,Method,Constructor,分别用来描述属性，方法，构造函数
2. 还有一个修饰符的获取，Modifier

### 4. 一个反射的简单例子程序：

```
package com.lifeware.study.reflection;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
public class ReflectTetser {
	public Object copy(Object obj) throws IllegalArgumentException, SecurityException, InstantiationException, IllegalAccessException, InvocationTargetException, NoSuchMethodException{
		Class<?> classType = obj.getClass();
		System.out.println(classType.getName());
		Object objCopy = classType.getConstructor(new Class[]{}).newInstance(new Object[]{});
	
		Field[] fields = classType.getDeclaredFields();
		for(Field field:fields){
			System.out.println(field.getName());
			String firstLetter = field.getName().substring(0,1).toUpperCase();
			String getMethodName = “get” + firstLetter + field.getName().substring(1);
			String setMethodName = “set” + firstLetter + field.getName().substring(1);
			
			Method getMethod = classType.getMethod(getMethodName, new Class[]{});
			Method setMethod = classType.getMethod(setMethodName, new Class[]{field.getType()});
			
			Object value = getMethod.invoke(obj, new Object[]{});
			setMethod.invoke(objCopy, new Object[]{value});
		}
		return objCopy;
	}
	/**
	 * @param args
	 * @throws NoSuchMethodException 
	 * @throws InvocationTargetException 
	 * @throws IllegalAccessException 
	 * @throws InstantiationException 
	 * @throws SecurityException 
	 * @throws IllegalArgumentException 
	 */
	public static void main(String[] args) throws IllegalArgumentException, SecurityException, InstantiationException, IllegalAccessException, InvocationTargetException, NoSuchMethodException {
		// TODO Auto-generated method stub
		Customer cus = new Customer();
		cus.setId(new Long(100));
		cus.setAge(new Long(50));
		cus.setName(“zhangsan”);
		Customer cuscopy = (Customer) new ReflectTetser().copy(cus);
		System.out.println(cuscopy.getId() + “,” + cuscopy.getAge() + “,” + cuscopy.getName());
	}
}
class Customer{
	private Long id;
	private Long age;
	private String name;
	
	public Customer(){
		
	}
	
	public Long getId(){
		return id;
	}
	
	public Long getAge(){
		return age;
	}
	
	public String getName(){
		return name;
	}
	
	public void setId(Long id){
		this.id = id;
	}
	
	public void setAge(Long age){
		this.age = age;
	}
	
	public void setName(String name){
		this.name = name;
	}
}
```

### 5. 当然，反射写通用的数组代码时，还需要用到：java.lang.reflect.Array

```
public static Object goodArrayGrow(Object a){
		Class<?> c1 = a.getClass();
		if(!c1.isArray()){
			return null;
		}
		Class<?> componentType = c1.getComponentType();
		int length = Array.getLength(a);
		int newLength = length * 11/10 + 10;
		Object newArray = Array.newInstance(componentType, newLength);
		System.arraycopy(a, 0, newArray, 0, length);
		
		return newArray;
	}
```

## 动态代理
*动态代理部分：想清楚下面四个问题*

> 1. 什么是动态代理? 
>     一种用于转发请求，进行特殊处理的机制，“动态”应该指的是“运行期”。 
> 2. 为什么使用动态代理? 
>     可以对请求进行任何处理(如事务，日志等，这都是网上说的，我当然可以做任何处理) 
> 3. 使用它有哪些好处? 
>     如上 
> 4. 哪些地方需要动态代理? 
>     不允许直接访问某些类；对访问要做特殊处理等，我只能想到这些。
  
**下面具体再看看动态代理相关的内容：**

### 1. 和动态代理有关的有两个类 

- interface InvocationHandler 
  只这一个方法， Object invoke(Object proxy, Method method, Object[] args) 
          
- class Proxy 真正表示动态代理的类，提供两个静态方法: 
  -  `Class<?> getProxyClass(ClassLoader loader, Class<?>[] interface) `:
用来产生代理类，参数要提供interface数组，它会生成这些interface的“虚拟实现”,用来冒充真实的对象。 
  - `Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) `: 产生代理对象，多了InvocationHandler参数(只是InvocationHandler接口的实现类),它与代理对象关联，当请求分发到代理对象后，会自动执行h.invoke(…)方法.

### 2. 动态机制的实现步骤：
>  /**
>    * 1. 实现InvocationHandler接口创建自己的调用处理器
>    *    InvocationHandler handler = new InvocationHandlerImpl(server);
>    * 2. 通过Proxy指定ClassLoader对象和一组interface创建动态代理类
>    *    Class clazz = Proxy.getProxyClass(classLoader,new class[]{…})
>    * 3. 通过反射机制获取动态代理类的构造函数，其参数类型是调用处理器接口类型:
>    *    Constructor constructor  = clazz.getConstructor(new Class[]{InvocationHandler.class})
>    * 4. 通过构造函数创建动态代理类实例，将调用处理器对象作为参数被传入
>    *    Interface proxy = constructor.newInstance(new Object[]{handler})
>    *    
>    * Proxy中newProxyInstance方法已经封装了步骤2~4，实例如下：
> */


### 3. 一个简单实用的例子：

```
package com.lifeware.study.reflection;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
public class DynamicProxyTest {
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		CalculatorProtocol server = new Server();
		InvocationHandler handler = new CalculatorHandler(server);
		CalculatorProtocol client = (CalculatorProtocol)Proxy.newProxyInstance(server.getClass().getClassLoader(), 
				server.getClass().getInterfaces(), handler);
		int result = client.add(3, 2);
		System.out.println(“3+2=” + result);
		result = client.subtract(5, 2);
		System.out.println(“5-2=” + result);
	}
}
//定义一个接口协议
interface CalculatorProtocol{
	public int add(int a,int b);
	public int subtract(int a,int b);
}
//实现接口协议
class Server implements CalculatorProtocol{
	public int add(int a,int b){
		return a+b;
	}
	
	public int subtract(int a,int b){
		return a-b;
	}
}
class CalculatorHandler implements InvocationHandler{
	private Object objOriginal;
	public CalculatorHandler(Object obj){
		this.objOriginal = obj;
	}
	
	public Object invoke(Object proxy,Method method,Object[] args) throws Throwable{
		//可加入预处理
		Object result = method.invoke(this.objOriginal, args);
		return result;
				
	}
	
}
```



