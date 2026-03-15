# Java类加载

继承关系：

ClassLoader->SecureClassLoader->URLClassLoader->AppClassLoader

调用关系：

loadClass->findClass（重写的方法）->defineClass（从字节码中加载类）

```java
public class Payload implements Serializable {
    static {
        try {
            System.out.println("静态代码块");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    {
        System.out.println("构造代码块");
    }

    public Payload(){
        System.out.println("构造方法");
    }

}

```



```java
public class demo2 {
    public static void main(String[] args) {
        try{
//            获取系统类加载器
            ClassLoader cl = ClassLoader.getSystemClassLoader();
//            获取类，并进行初始化，触发静态代码块
            Class<?> clz = Class.forName("ClassLoaderTest.Payload");
//            重载方法，可以指定不进行初始化
//            Class<?> clz = Class.forName("ClassLoaderTest.Payload", false,cl);
//            创建实例，当于 new Payload()，触发构造方法和构造代码块
//            clz.newInstance();
//            使用加载类方法获取对象，不进行初始化
//            Class<?> c =  cl.loadClass("ClassLoaderTest.Payload");
//            初始化并创建实例
//            c.newInstance();

        }catch(Exception e){
            e.printStackTrace();
        }


    }

}
```

**loadClass不会初始化**

编译一个反弹计算器的测试类

```java
public class Test {
    static {
        try {
            Runtime.getRuntime().exec("calc");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

编译后改变class文件位置，使用URLClassLoader和file协议本地加载字节码

```java
URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{new URL("file:///D:\\JavaDS\\temp\\")});
Class<?> c =  urlClassLoader.loadClass("Test");
c.newInstance();
```

成功弹出计算器，**类似的使用http协议请求加载远程代码**

在D:\\JavaDS\\temp\\目录下开一个本地http服务

python -m http.server 9999

```java
   URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{new URL("http://localhost:9999/")});
   Class<?> c =  urlClassLoader.loadClass("Test");
   c.newInstance();
```

成功弹起计算器。

还可以使用jar协议加载jar包