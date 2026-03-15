# java中修改final修饰的值

## 低版本

jdk17及之后无法反射 `java.*` 包下非`public` 修饰的属性和方法

使用反射，步骤：

获取字段-》修改修饰符-》修改值

```java
public class Test {
    private final String passw="未修改";
    public Test(){

    }


    public String getPassw() {
        return passw;
    }

    @Override
    public String toString() {
        return "Test{" +
                "passw=" + passw +
                '}';
    }
}
```



```java
public class FinalTest {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException, NoSuchMethodException, InvocationTargetException, InstantiationException {
        Class<?> clz = Class.forName("FinalTest.Test");
        Field pas = clz.getDeclaredField("passw");
        pas.setAccessible(true);


        Test t = new Test();

        pas.set(t,"已修改");
        System.out.println(t.getPassw());
        System.out.println(pas.get(t));

    }
}
```

运行结果：

未修改

已修改

### 原因

java编译器对final修饰属性进行的内联优化 即编译时将final的值直接放到了引用他的地方,即使通过反射修改了该属性 也没啥用

java会对如下final修饰的类型进行优化

> byte short int long float double boolean char LiteralString(直接双引号括起来的字符串)

**反射是可以修改`final`变量的，但是如果是基本数据类型或者`String`类型的时候，无法通过对象获取修改后的值，因为`JVM`对其进行了内联优化。**

那有没有办法获取修改后的值呢？

有，可以通过反射中的`Field.get(Object obj)`方法获取

```java
//获取field对应的变量在user对象中的值
System.out.println("修改后"+field.get(user));
```

