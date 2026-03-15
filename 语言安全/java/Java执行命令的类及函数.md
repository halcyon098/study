# Java执行命令的类及函数

反弹计算器为例

```java
public class demo1 {
    //    谈计算器
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException, InstantiationException {
//            Runtime.getRuntime().exec("calc");
//            Class<?> clz = Class.forName("java.lang.Runtime");
//            Method getRuntime = clz.getDeclaredMethod("getRuntime");
//            Runtime o1 = (Runtime) getRuntime.invoke(clz);
//
//            Method exec = clz.getDeclaredMethod("exec", String.class);
//            exec.invoke(o1,"calc");


//            new ProcessBuilder("calc").start();
//        Class<?> clz = Class.forName("java.lang.ProcessBuilder");
//        Constructor pBC =  clz.getDeclaredConstructor(String[].class);
//        ProcessBuilder processBuilder = (ProcessBuilder) pBC.newInstance(new String[][]{{"calc"}});
//        Method pM = clz.getDeclaredMethod("start");
//        pM.invoke(processBuilder,null);

//            ProcessImpl.start();
        Class<?> clz = Class.forName("java.lang.ProcessImpl");
        Method start = clz.getDeclaredMethod("start", String[].class, Map.class, String.class, ProcessBuilder.Redirect[].class, boolean.class);
        start.setAccessible(true);
        start.invoke(clz, new String[]{"calc"},null,null,null,false);


    }


}
```

跟随查看一下调用链：exec—> ProcessBuilder --> ProcessImpl.start --> ProcessImpl.create

ProcessImpl类底层调用C语言执行系统命令，过程中的三个类都可以实现命令执行

如果要输出命令结果，需要使用输出流：

```java
//        InputStream inputStream = new ProcessBuilder("whoami").start().getInputStream();
        InputStream inputStream = Runtime.getRuntime().exec("whoami").getInputStream();
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        byte[] bytes = new byte[1024];
        int a = -1;
        while ((a=inputStream.read(bytes))!=-1){
            byteArrayOutputStream.write(bytes,0,a);
        }
        System.out.println(byteArrayOutputStream.toString());
```

