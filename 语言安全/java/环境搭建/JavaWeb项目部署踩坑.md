# JavaWeb项目部署踩坑

1. 新建数据库后修改项目文件中数据库配置文件中的数据库名，账号，密码，引入数据库驱动

2. 引入各种库，点在jar包上右键，选择将其添加为库。

3. jsmartcom_zh_CN，jsp上传下载框架

4. jsp所需 jstl.jar和standard.jar两种包支持JSTL标签库

5. servlet包有2.x,3.x两个大版本，里面许多方法和引入的包不一样

6. 控制台中文乱码，tomcat的日志配置文件的编码需要修改，找到tomcat安装目录，找到conf下的logging.[properties](https://so.csdn.net/so/search?q=properties&spm=1001.2101.3001.7020)文件，将其中的encoding = UTF-8的部分全部修改为encoding = GBK，保存重启tomcat

7.  另一种情况，是涉及到在tomcat里运行的项目与后端交互的情况，这种情况较为复杂，可首先修改tomcat安装目录下的conf下的web.xml文件，在servlet标签组中加入：

   ```xml
    <init-param>
   
            <param-name>fileEncoding</param-name>
        
            <param-value>UTF-8</param-value>
   
     </init-param>
   ```

   重启tomcat

   ```
   
   ```