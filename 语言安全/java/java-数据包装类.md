# java-数据包装类

## 基本数据类型及包装类：

**int -- Integer**

**char->Character**

fload -- Fload

short.->Short

long->Long

float->Float

double->Double

boolean -Boolean

这些类都在java.lang包

### 包装类存在意义：

1. 让基本数据类到有面向对象的特征

2. 封装了字符串转化成基本数据类型的方法

## 使用

java对数据封装类支持自动打包和解包,在使用时与基本数据类型没区别

```java
public class Shuju {
    public static void main(String[] args) {
        Integer i = 10;//自动打包
        System.out.println(i+20);
        int j = i;//自动解包
        System.out.println(j);

    }
}
```

