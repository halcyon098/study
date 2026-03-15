# 高版本绕过反射限制（jdk17+）

参考博客：https://zer0peach.github.io/2023/12/28/JDK17-%E7%BB%95%E8%BF%87%E5%8F%8D%E5%B0%84%E9%99%90%E5%88%B6/#:~:text=JDK17+%20%E5%8F%8D%E5%B0%84%E7%BB%95

https://pankas.top/2023/12/05/jdk17-%E5%8F%8D%E5%B0%84%E9%99%90%E5%88%B6%E7%BB%95%E8%BF%87/

官方文档对反射限制的描述：https://docs.oracle.com/en/java/javase/17/migrate/migrating-jdk-8-later-jdk-releases.html#GUID-7BB28E4D-99B3-4078-BDC4-FC24180CE82B

## 前言

springboot3.x使用的jdk版本至少是jdk17，在jdk17及之后无法反射 `java.*` 包下非`public` 修饰的属性和方法

根据 [Oracle的文档](https://docs.oracle.com/en/java/javase/17/migrate/migrating-jdk-8-later-jdk-releases.html#GUID-7BB28E4D-99B3-4078-BDC4-FC24180CE82B)，为了安全性，从JDK 17开始对java本身代码使用强封装，原文叫 **Strong Encapsulation**。任何对 `java.*` 代码中的**非public**变量和方法进行反射会抛出InaccessibleObjectException异常。

[JDK的文档](https://openjdk.org/jeps/403)解释了对java api进行封装的两个理由：

1. 对java代码进行反射是不安全的，比如可以调用ClassLoader的defineClass方法，这样在运行时候可以给程序注入任意代码。
2. java的这些非公开的api本身就是非标准的，让开发者依赖使用这个api会给JDK的维护带来负担。

所以从JDK 9开始就准备限制对java api的反射进行限制，直到JDK 17才正式禁用。jdk9到16仅会警告。



调用ClassLoader.defineClass加载字节码

```java
package ClassLoaderTest;

import java.io.IOException;
import java.io.Serializable;

public class Payload implements Serializable {
    public static void main(String[] args) throws IOException {
        System.out.println("success");
    }

}
```

```java
package ClassLoaderTest;

import java.lang.reflect.Method;
import java.nio.file.Files;
import java.nio.file.Paths;

//测试高版本jdk对反射的限制
public class demo3 {
    public static void main(String[] args) {
        try {

            byte[] bytes = Files.readAllBytes(Paths.get(
                    "D:\\IDEA\\IdeaProjects\\JavaSJ\\src\\ClassLoaderTest\\Payload.class"));
            Object o = ClassLoader.getSystemClassLoader();
            Class<?> clz = ClassLoader.class;
            Method defineClass = clz.getDeclaredMethod("defineClass", String.class, byte[].class, int.class, int.class);
            defineClass.setAccessible(true);
            Class evil = (Class) defineClass.invoke(o, "ClassLoaderTest.Payload", bytes, 0, bytes.length);
//           使用反射运行main函数
            Method mian = evil.getDeclaredMethod("main",String[].class);
            mian.invoke(null,(Object)new String[]{"a","b","c"});

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## JDK9 - JDK16（只有警告）

从JDK9开始，当我们用反射去获取 `java.*` 包下的非public变量和方法时只会警告，仍能够运行

```she
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.example.Main (file:/E:/test/test/target/classes/) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int)
WARNING: Please consider reporting this to the maintainers of org.example.Main
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```

##  JDK17+对反射的限制

报以下错误，无法运行

```java
Exception in thread "main" java.lang.reflect.InaccessibleObjectException: Unable to make protected final java.lang.Class java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int) throws java.lang.ClassFormatError accessible: module java.base does not "opens java.lang" to unnamed module @3b07d329

at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:354)
at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:297)
at java.base/java.lang.reflect.Method.checkCanSetAccessible(Method.java:199)
at java.base/java.lang.reflect.Method.setAccessible(Method.java:193)
at test.main(test.java:11)
```

## 绕过

官方描述：

请注意，`sun.misc`和`sun.reflect` 包可供所有 JDK 版本（包括 JDK 17）中的工具和库进行反射。

`java`启动器选项允许`--illegal-access` 反射到 JDK 9 到 JDK 16 中的 JDK 内部。

这为我们提供了思路，有两种绕过方法，一种是`sun.misc`和`sun.reflect`包下的我们是可以正常反射的，所以有个关键的类就可以拿来用来，就是 `Unsafe` 这个类。另一种就是修改Java启动选项，具体参数参考官方文档。

主要探讨Unsafe类。

###  Unsafe绕过反射限制

关于Unsafe类可以参考美团技术团队文档： https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html

同时注意 JDK17下Unsafe类下的 `defineClass` 和 `defineAnonymousClass` 已被移除，且从jdk9开始存在的另一个Unsafe类`jdk.internal.misc.Unsafe` 也是强封装的，和 `java.*` 包下的一样。

如何利用Unsafe来打破这个强封装module限制呢？

#### 跟进分析setAccessible函数

```java
 public void setAccessible(boolean flag) {
        AccessibleObject.checkPermission();
        if (flag) checkCanSetAccessible(Reflection.getCallerClass());
        setAccessible0(flag);
    }
```

当flag为true时，进入函数checkCanSetAccessible

```java
    void checkCanSetAccessible(Class<?> caller) {
        checkCanSetAccessible(caller, clazz);
    }
```

```java
    final void checkCanSetAccessible(Class<?> caller, Class<?> declaringClass) {
        checkCanSetAccessible(caller, declaringClass, true);
    }
```

```java
private boolean checkCanSetAccessible(Class<?> caller,
                                          Class<?> declaringClass,
                                          boolean throwExceptionIfDenied) {
        if (caller == MethodHandle.class) {
            throw new IllegalCallerException();   // should not happen
        }

        Module callerModule = caller.getModule();
        Module declaringModule = declaringClass.getModule();

        if (callerModule == declaringModule) return true;
        if (callerModule == Object.class.getModule()) return true;
        if (!declaringModule.isNamed()) return true;

        String pn = declaringClass.getPackageName();
        int modifiers;
        if (this instanceof Executable) {
            modifiers = ((Executable) this).getModifiers();
        } else {
            modifiers = ((Field) this).getModifiers();
        }

        // class is public and package is exported to caller
        boolean isClassPublic = Modifier.isPublic(declaringClass.getModifiers());
        if (isClassPublic && declaringModule.isExported(pn, callerModule)) {
            // member is public
            if (Modifier.isPublic(modifiers)) {
                return true;
            }

            // member is protected-static
            if (Modifier.isProtected(modifiers)
                && Modifier.isStatic(modifiers)
                && isSubclassOf(caller, declaringClass)) {
                return true;
            }
        }

        // package is open to caller
        if (declaringModule.isOpen(pn, callerModule)) {
            return true;
        }

        if (throwExceptionIfDenied) {
            // not accessible
            String msg = "Unable to make ";
            if (this instanceof Field)
                msg += "field ";
            msg += this + " accessible: " + declaringModule + " does not \"";
            if (isClassPublic && Modifier.isPublic(modifiers))
                msg += "exports";
            else
                msg += "opens";
            msg += " " + pn + "\" to " + callerModule;
            InaccessibleObjectException e = new InaccessibleObjectException(msg);
            if (printStackTraceWhenAccessFails()) {
                e.printStackTrace(System.err);
            }
            throw e;
        }
        return false;
    }
```

此时的关键代码

```java
		Module callerModule = caller.getModule();

        Module declaringModule = declaringClass.getModule();

        if (callerModule == declaringModule) return true;
```

判断我们调用类和目标类是不是一个`module`，如果**调用类的module和目标类的module一样**，就可以有修改权限

那我们可以尝试利用`Unsafe`来修改当前类的`module`属性和目标类(即 `java.*` 下类)的module属性一致来绕过

Unsafe类中有个 `getAndSetObject` 方法，其和反射赋值功能差不多，利用这个修改调用类的module

```java
    public final Object getAndSetObject(Object o, long offset, Object newValue) {
        return theInternalUnsafe.getAndSetReference(o, offset, newValue);
    }
```

Object o（操作的对象），, long offset（字段的偏移量）, Object newValue（要设置的新值）

由于我们要调用`ClassLoader`类，所以我们要修改当前类的`module`为`ClassLoader`的`module`

```java
System.out.println(ClassLoader.class.getModule());
System.out.println(Object.class.getModule());
System.out.println(Class.class.getModule());
```

```java
module java.base
module java.base
module java.base
```

接着找偏移

`Unsafe`提供两个方法来获取Field的偏移量

```java
staticFieldOffset(Field var1)`和`objectFieldOffset(Field var1)
```

```java
unsafe.objectFieldOffset(Class.class.getDeclaredField("module"));
```

这样参数都找完了

`getAndSetObject`可以用`putObject`来替代

之前的代码在jdk17以上是运行报错的

修改后的代码可以正常反射调用出来

```java
package ClassLoaderTest;

import com.sun.tools.javac.Main;
import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.nio.file.Files;
import java.nio.file.Paths;

//测试高版本jdk对反射的限制
public class demo3 {
    public static void main(String[] args) {
        try {

            byte[] bytes = Files.readAllBytes(Paths.get(
                    "D:\\IDEA\\IdeaProjects\\JavaSJ\\src\\ClassLoaderTest\\Payload.class"));
//获取Unsafe对象
            Class<?> unsafeClass = Class.forName("sun.misc.Unsafe");
            Field theUnsafe = unsafeClass.getDeclaredField("theUnsafe");
            theUnsafe.setAccessible(true);
            Unsafe unsafe = (Unsafe) theUnsafe.get(null);
//修改Module值
//            修改后的module值
            Module module = Object.class.getModule();
//            获取当前类的class
            Class<?> currentClass = demo3.class;
//            获取偏移量
            long addr = unsafe.objectFieldOffset(Class.class.getDeclaredField("module"));
//            修改当前的类的mudule与目标类(即 `java.*` 下类)的module属性一致，Object,Class,ClassLoader的module值一样
            unsafe.getAndSetObject(currentClass,addr,module);
            //        unsafe.putObject(currentClass,addr,baseModule);


            Class<?> clz = ClassLoader.class;
            Method defineClass = clz.getDeclaredMethod("defineClass", String.class, byte[].class, int.class, int.class);
            defineClass.setAccessible(true);
            Class evil = (Class) defineClass.invoke(ClassLoader.getSystemClassLoader(), "ClassLoaderTest.Payload", bytes, 0, bytes.length);
//           使用反射运行main函数
            Method mian = evil.getDeclaredMethod("main",String[].class);
            mian.invoke(null,(Object)new String[]{"a","b","c"});

//            System.out.println(ClassLoader.class.getModule());
//            System.out.println(Object.class.getModule());
//            System.out.println(Class.class.getModule());



        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


```

同样的，如果没有办法反射其他不在同一个module下的属性或方法，也可以利用这个办法来修改类的module来绕过，上面也可以修改`java.*` 下类的module和调用类的module一样，也是可以的，**但修改module后会可能会产生什么不可预知的后果。**

