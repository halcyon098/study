# Java-反射

## 概述

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

要想解剖一个类,必须先要获取到该类的字节码文件对象。而解剖使用的就是Class类中的方法.所以先要获取到每一个字节码文件对应的Class类型的对象。

在Java中，使用类：

```java
Student st = new Student();//正
st.方法(参数);
```

使用反射实现相同的效果：

```java
Class clz = Class.forName("com.Student");//类的完整路径，包名+类名
Method method = clz.getMethod("printAge", int.class);//获取方法的 Method 对象，参数：方法名，形参的Class类型对象
Constructor constructor = clz.getConstructor();//根据 Class 对象实例获取 Constructor 对象
Object object = constructor.newInstance();//使用 Constructor 对象的 newInstance 方法获取反射类对象
method.invoke(object, 4);//利用  Method 对象的invoke 方法调用方法

```

> 有9个预先定义好的Class对象代表8个基本类型和void,它们被java虚拟机创建,和基本类型有相同的名字
>
> boolean, byte, char, short, int, long, float, and double；这8个基本类型的Class对象可以通过java.lang.Boolean.TYPE,java.lang.Integer.TYPE等来访问,同样可以通过int.class,boolean.class等来访问。
>
> 例如：int.class代表int类型的class类型对象，int.class与Integer.TYPE等价，int.class与Integer.class没关系，两个不同的class类实例

Student类：

```java
package test;

public class Student {
    public String name;
    private int age;
    public Student(String str) {
        System.out.println("默认的有参构造方法:"+str);
    }
    public Student(){
        System.out.println("调用了公有的无参构造方法");
    }
    public Student(String name, int num) {
        System.out.println("姓名"+name+"年龄"+num);
    }

    protected Student(int num){
        System.out.println("调用了受保护的无参构造方法");
    }
    private Student(boolean n){
        System.out.println("调用了私有的无参构造方法");
    }

    public void Call(String name){
        System.out.println("我是一个学生类");
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

分割函数，没什么用只是自己懒得都敲了

```java
 public static void Fg(String s){
        String g = "##############################";
        System.out.println(g+s+g);
    }
```



## 使用

### 1. 获取反射对象

在反射中，要获取一个类或调用一个类的方法，我们首先需要获取到该类的 Class 对象。

在 Java API 中，获取 Class 类对象有三种方法：

**第一种，使用 Class.forName 静态方法。**当你知道该类的全路径名时，你可以使用该方法获取 Class 类对象。

```java
try {
	Class stuClass3 = Class.forName("fanshe.Student");//注意此字符串必须是真实路径，就是带包名的类路径，包名.类名
	System.out.println(stuClass3 == stuClass2);//判断三种方式是否获取的是同一个Class对象
} catch (ClassNotFoundException e) {
	e.printStackTrace();
}
```



**第二种，使用 .class 方法。**

这种方法只适合在编译前就知道操作的 Class。

```java
Class stuClass2 = Student.class;
System.out.println(stuClass == stuClass2);//判断第一种方式获取的Class对象和第二种方式获取的是否是同一个

```



**第三种，使用类对象的 getClass() 方法。**

```java
Student stu1 = new Student();//这一new 产生一个Student对象，一个Class对象。
Class stuClass = stu1.getClass();//获取Class对象
System.out.println(stuClass.getName());
```



第二种需要导包，第三种已经有对象了，直接操作就行了，不需要反射，因此常用的就是第一种

```java
Class<?> clz = Class.forName("test.Student");
```

test.Student是需要操作类的完整名，包名+类名

### 2. 根据class实例创建Constructor对象

```java
Constructor sc = clz.getConstructor();
```

### 3.根据Constructor对象实例获取反射类对象

``` java
Object stuobj = sc.newInstance();
```

### 4.创建不同的类操作构造方法，属性，成员方法

//        绕过私有权限修饰符,暴力访问
       Constructor对象.setAccessible(true);

#### 4.1 构造方法

通过**Class对象**可以获取某个类中的：构造方法、成员变量、成员方法；并访问成员；

 1.获取构造方法：

​	1). 批量的方法：

​		public Constructor[] getConstructors()：所有"公有的"构造方法*

​		public Constructor[] getDeclaredConstructors()：获取所有的构造方法(包括私有、受保护、默认，公有)

​	2).获取单个的方法，并调用：

​		public Constructor getConstructor(Class... parameterTypes):获取单个的"公有的"构造方法

​		public Constructor getDeclaredConstructor(Class... parameterTypes):获取"某个构造方法"可以是私有的，或受保护、默认、公有；

​	调用构造方法：

​		Constructor-->newInstance(Object... initargs)

例子：

```java
//        构造方法
        Fg("全部公共的构造方法");
        Constructor[] conArr = clz.getConstructors();
        for (Constructor c :
                conArr) {
            System.out.println(c);
        }
        Fg("全部的构造方法");
        Constructor[] conArr1 = clz.getDeclaredConstructors();
        for (Constructor c :
                conArr1) {
            System.out.println(c);
        }
        Fg("调用构造方法");
        Constructor con = clz.getConstructor();
        Object o = con.newInstance();
//调用有参构造方法
        Constructor con1 = clz.getDeclaredConstructor(boolean.class);
//        绕过私有权限修饰符,暴力访问
        con1.setAccessible(true);
//创建object类传入构造函数的参数
        Object o1 = con1.newInstance(true);
        Constructor con2 = clz.getConstructor(String.class);
        Object o2 = con2.newInstance("fanse");
```

#### 4.2 属性

获取成员变量并调用：

1.批量的

​	1).Field[] getFields():获取所有的"公有字段"

​	2).Field[] getDeclaredFields():获取所有字段，包括：私有、受保护、默认、公有；

2.获取单个的

​	1).public Field getField(String fieldName):获取某个"公有的"字段；

​	2).public Field getDeclaredField(String fieldName):获取某个字段(可以是私有的)

 设置字段的值：

​	Field --> public void set(Object obj,Object value):

​	参数说明：

 * 		obj:要设置的字段所在的对象；
 * 		value:要为字段设置的值；

例子：

```java
//        属性
        Fg("公有属性");
        Field[] fArr = clz.getFields();
        for (Field f :
                fArr) {
            System.out.println(f);
        }
        Fg("所有属性");
        Field[] fArr1 = clz.getDeclaredFields();
        for (Field f :
                fArr1) {
            System.out.println(f);
        }
        Fg("调用公有属性");
        Field f = clz.getField("name");
        System.out.println(f);
        Object o = clz.getConstructor().newInstance();
        f.set(o,"fanse");
        Student st = (Student) o;
        System.out.println(st.name);
        Fg("调用非公有属性");
//创建属性对应的Field实例对象
        Field f1 = clz.getDeclaredField("age");
        System.out.println(f1);
//绕过私有权限修饰符,暴力访问
        f1.setAccessible(true);
//使用set函数设置属性值
        f1.set(o,12);
//验证是否赋值
        System.out.println(st.getAge());
```



#### 4.3 方法

获取成员方法并调用：

1.批量的：

​	public Method[] getMethods():获取所有"公有方法"；（包含了父类的方法也包含Object类）

​	public Method[] getDeclaredMethods():获取所有的成员方法，包括私有的(不包括继承的)

2.获取单个的：

​	public Method getMethod(String name,Class<?>... parameterTypes):

​	参数：

- name : 方法名；
- Class ... : 形参的Class类型对象

public Method getDeclaredMethod(String name,Class<?>... parameterTypes)

​	调用方法：

​	Method --> public Object invoke(Object obj,Object... args):

​	参数说明：

 * obj : 要调用方法的对象；
 * args:调用方式时所传递的实参；

例子：

```java
//        方法
        Method[] m = clz.getMethods();
        for (Method e :
                m) {
            System.out.println(e);
        }
        Fg("非公");

        Method[] m1 = clz.getDeclaredMethods();
        for (Method e :
                m1) {
            System.out.println(e);
        }
        Fg("调用方法");
        Method m2 = clz.getMethod("show1", String.class);
//链式创建对象
        Object o = clz.getConstructor().newInstance();
        m2.invoke(o,"fnase");

        Method m3 = clz.getDeclaredMethod("show4",int.class);
        m3.setAccessible(true);
        m3.invoke(o,18);
```

##### 调用main方法

```java
/**
 * 获取Student类的main方法、不要与当前的main方法搞混了
 */
public class Main {
	
	public static void main(String[] args) {
		try {
			//1、获取Student对象的字节码
			Class clazz = Class.forName("fanshe.main.Student");
			
			//2、获取main方法
			 Method methodMain = clazz.getMethod("main", String[].class);//第一个参数：方法名称，第二个参数：方法形参的类型，
			//3、调用main方法
			// methodMain.invoke(null, new String[]{"a","b","c"});
			 //第一个参数，对象类型，因为方法是static静态的，所以为null可以，第二个参数是String数组，这里要注意在jdk1.4时是数组，jdk1.5之后是可变参数
			 //这里拆的时候将  new String[]{"a","b","c"} 拆成3个对象。。。所以需要将它强转。
			 methodMain.invoke(null, (Object)new String[]{"a","b","c"});//方式一
			// methodMain.invoke(null, new Object[]{new String[]{"a","b","c"}});//方式二
			
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		
	}
}

```

### 5、反射方法的其它使用之---通过反射运行配置文件内容

student类：



```java
public class Student {

	public void show(){
		System.out.println("is show()");
	}
}
```

配置文件以txt文件为例子（pro.txt）：

```java
className = cn.fanshe.Student
methodName = show
```

测试类：



```java
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.lang.reflect.Method;
import java.util.Properties;
/*
 * 我们利用反射和配置文件，可以使：应用程序更新时，对源码无需进行任何修改
 * 我们只需要将新类发送给客户端，并修改配置文件即可
 */
public class Demo {

	public static void main(String[] args) throws Exception {
		//通过反射获取Class对象
		Class stuClass = Class.forName(getValue("className"));//"cn.fanshe.Student"
        
		//2获取show()方法
		Method m = stuClass.getMethod(getValue("methodName"));//show
        
		//3.调用show()方法
		m.invoke(stuClass.getConstructor().newInstance());

	}

	//此方法接收一个key，在配置文件中获取相应的value

	public static String getValue(String key) throws IOException{

		Properties pro = new Properties();//获取配置文件的对象
		FileReader in = new FileReader("pro.txt");//获取输入流
		pro.load(in);//将流加载到配置文件对象中
		in.close();
		return pro.getProperty(key);//返回根据key获取的value值
	}
}
```

控制台输出：

is show()

**需求：**
当我们升级这个系统时，不要Student类，而需要新写一个Student2的类时，这时只需要更改pro.txt的文件内容就可以了。代码就一点不用改动

要替换的student2类：

```java
public class Student2 {
	public void show2(){
		System.out.println("is show2()");
	}
}
```

配置文件更改为：

```java
className = cn.fanshe.Student2
methodName = show2
```

控制台输出：

is show2();



### 6、反射方法的其它使用之---通过反射越过泛型检

泛型用在编译期，编译过后泛型擦除（消失掉）。所以是可以通过反射越过泛型检查的

测试类：

```java
import java.lang.reflect.Method;
import java.util.ArrayList;
/*
 * 通过反射越过泛型检查
 * 
 * 例如：有一个String泛型的集合，怎样能向这个集合中添加一个Integer类型的值？
 */
public class Demo {
	public static void main(String[] args) throws Exception{
		ArrayList<String> strList = new ArrayList<>();
		strList.add("aaa");
		strList.add("bbb");
	//	strList.add(100);
		//获取ArrayList的Class对象，反向的调用add()方法，添加数据
		Class listClass = strList.getClass(); //得到 strList 对象的字节码 对象
		//获取add()方法
		Method m = listClass.getMethod("add", Object.class);
		//调用add()方法
		m.invoke(strList, 100);
		//遍历集合
		for(Object obj : strList){
			System.out.println(obj);
		}
	}

}
```

控制台输出：

aaa
	bbb
	100

## 内部类

反射调用内部类的时候需要使用`$`来代替`.`,如`com.anbai.Test`类有一个叫做`Hello`的内部类，那么调用的时候就应该将类名写成：`com.anbai.Test$Hello`。
