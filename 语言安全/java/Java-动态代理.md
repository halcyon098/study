# Java-动态代理

## 代理模式

代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。简单的说就是，我们在访问实际对象时，是通过代理对象来访问的，代理模式就是在访问实际对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。

## 代理的实现

### 1.静态代理

上代码吧

接口类-IUser

```java
public interface IUser {
    public void show();
    public void updata();
    public void create();
}
```

实现类（委托类）-UserImpl

```java
ublic class UserImpl implements IUser {
    @Override
    public void show() {
        System.out.println("展示");
    }

    @Override
    public void updata() {
        System.out.println("更新");
    }

    @Override
    public void create() {
        System.out.println("创建");
    }
}
```

代理类-UserProxy

```java
public class UserProxy implements IUser {
    IUser user;
    public UserProxy(IUser user){
        this.user = user;
    }
    @Override
    public void show() {
        user.show();
        System.out.println("调用了show");

    }

    @Override
    public void updata() {
        user.updata();
        System.out.println("调用了updata");
    }

    @Override
    public void create() {
        user.create();
        System.out.println("调用了create");
    }
}
```



测试类-ProxyTest

```java
public class ProxyTest {
    public static void main(String[] args) {
        IUser user = new UserImpl();
//        直接调用实现类
//        user.show();
//        静态调用
        UserProxy userProxy = new UserProxy(user);
        userProxy.show();
            }
}
```

逻辑就是当需要调用IUser接口的实现类时调用代理类传入一个IUser的实现类对象，在代理类中调用实现类实现的方法，还可以额外添加其他功能（类似方法重写），代码中添加了类似方法调用日志功能。

### 2.动态代理

静态代理的缺陷是需要将接口中的抽象方法全部实现，当代码量大时会显得繁琐，工作量大。因此引入动态代理。

动态代理在jdk中已经为我们实现，避免繁琐的调用底层代码

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class ProxyTest {
    public static void main(String[] args) {
        IUser user = new UserImpl();
//        动态调用
//        获取class类方便后面传参
//        Proxy.newProxyInstance()Java自带的动态代理类方法，
//        newProxyInstance(ClassLoader loader,
//                Class<?>[] interfaces,
//                InvocationHandler h)
//          有三个参数：
//          1，类加载器：clz.getClassLoader()
//          2. 类接口数组，需要代理的接口
//          3. 调用处理器，对调用接口要应用的功能，需要自己写
        Class<?> clz = user.getClass();
        InvocationHandler in = new UserInvocationHandler(user);
        IUser user1 = (IUser) Proxy.newProxyInstance(
                clz.getClassLoader(),
                new Class[]{IUser.class},
//                等价于clz.getInterfaces(),
                in);
        user1.updata();
    }
}

```



Proxy.newProxyInstance()Java自带的动态代理类方法，
newProxyInstance(ClassLoader loader,
        Class<?>[] interfaces,
        InvocationHandler h)
  有三个参数：

1. 类加载器：clz.getClassLoader()
2. 类接口数组，需要代理的接口
3. 调用处理器，对调用接口要应用的功能，需要自己写

调用处理器-UserInvocationHandler

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class UserInvocationHandler implements InvocationHandler {
    IUser user;
    public UserInvocationHandler(){

    }
    public UserInvocationHandler(IUser user){
        this.user = user;
    }

//    public IUser getProxy(IUser u1){
//        IUser u = (IUser) Proxy.newProxyInstance(
//                u1.getClass().getClassLoader(),
//                new Class[]{IUser.class},
////                等价于clz.getInterfaces(),
//                new UserInvocationHandler(u1));
//        return u;
//    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("调用的方法名："+method.getName());
        method.invoke(user, args);

        return null;
    }
}
```

