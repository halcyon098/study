## 开发语言：
**HTML**(超文本标记语言)：用于构建网站的基础语言，是 Web 开发的基础。
**CSS**(层叠样式表)：用于控制网站的外观和布局。
**JavaScript**：用于在网站中添加交互性和动态效果。
**PHP**：一种服务器端的脚本语言，常用于开发动态网站和网站应用程序。
**Python**：一种流行的编程语言，用于各种应用程序开发，包括 Web 应用程序。
**Ruby**：一种动态、解释型的编程语言，常用于开发网站和 Web 应用程序。
```ruby
puts "Hello, World!"
```
**Java**：一种流行的面向对象编程语言，用于开发各种应用程序，包括 Web 应用程序。
**asp**: 这是 Microsoft Active Server Pages (ASP) 的简写，它是一种动态网页编程环境，由 Microsoft 开发。ASP 使用 VBScript 或 JScript 作为默认的脚本语言。
```bash
<%  
  Response.Write("Hello, World!")  
%>
```
**aspx**: 这是 ASP.NET 页面的文件扩展名，ASP.NET 是 Microsoft 的一个技术，用于创建动态网页。它支持多种语言，包括 C#, Visual Basic 和 JavaScript。
ASP和ASPX都使用了Response.Write方法来输出文本到Web浏览器。在ASP中，代码写在<% %>标签内，而在ASPX中，@ Page指令用于指定使用的编程语言（在这种情况下是C#）并告诉ASP.NET引擎该页面使用C#进行编译
```html
<%@ Page Language="C#" %>  
<!DOCTYPE html>  

<html>  
  <head>  
    <title>Hello World Example</title>  
  </head>  
  <body>  
    <%   
      Response.Write("Hello, World!");  
      %>  
  </body>  
</html>
```
**go**: Go 语言是由 Google 开发的开源编程语言，因其高效的性能和并发处理能力而受到广泛关注。"Hello, World!" 的 Go 语言版本可能是：
```go
package main  
import "fmt"  
func main() {  
    fmt.Println("Hello, World!")  
}
```
**C#**：一种面向对象的编程语言，常用于开发 Windows 应用程序和 Web 应用程序。以下是一个简单的 C# "Hello, World!" 程序：
```csharp
using System;  
  
class Program {  
    static void Main(string[] args) {  
        Console.WriteLine("Hello, World!");  
    }  
}
```
**C++**：一种流行的高级编程语言，用于开发各种应用程序，包括 Web 应用程序。
```cpp
#include <iostream>  
int main() {  
    std::cout << "Hello, World!";  
    return 0;  
}
```
**Swift**：一种用于开发 iOS 和 macOS 应用程序的编程语言，也可用于开发 Web 应用程序。
```swift
import Swift  
print("Hello, World!")
```
**Perl：**一种注释性语言，由Larry Wall 开发。Perl 常被推荐用于文本处理，它还融合了其他编程语言的大多数功能。加上Catalyst, Dancer 和 Mojolicious几个框架，以及工具包，Perl使得web开发和部署更简单。它的文本管理能力以及粘合系统的能力使其成为web开发中一个很棒的工具。
```perl
#!/usr/bin/perl -w  
use strict;  
use warnings;  
print "Hello, World!";
```
## 中间件
中间件（英语：Middleware）顾名思义是系统软件和用户应用软件之间连接的软件，以便于软件各部件之间的沟通，特别是应用软件对于系统软件的集中的逻辑，是一种独立的系统软件或服务程序，分布式应用软件借助这种软件在不同的技术之间共享资源。中间件在客户服务器的操作系统、网络和数据库之上，管理计算资源和网络通信。总的作用是为处于自己上层的应用软件提供运行与开发的环境，帮助用户灵活、高效地开发和集成复杂的应用软件。
也就是说，关于中间件，我们可以理解为：是一类能够为一种或多种应用程序合作互通、资源共享，同时还能够为该应用程序提供相关的服务的软件。中间件是一类软件统称，而非一种软件;中间件不仅仅实现互连，还要实现应用之间的互操作。
中间件与操作系统和数据库共同构成基础软件三大支柱，是一种应用于分布式系统的基础软件，位于应用与操作系统、数据库之间，为上层应用软件提供开发、运行和集成的平台。中间件解决了异构网络环境下软件互联和互操作等共性问题，并提供标准接口、协议，为应用软件间共享资源提供了可复用的“标准件”。
### 常用的中间件

- Tomcat：8080
- Weblogic：7001
- Jboss：8080
- Jetty：8080
- Webshere：9080
- Glassfish：8080
## 数据库
### 关系型数据库

- Oracle : 1521
- SQL Server : 1433
- MySQL : 3306
- pointbase : 9092
- DB2 : 5000
- Sybase : 5000
- PostgreSQL : 5432
### NOSQL数据库（全称为not only SQL，是一种非关系型的数据库）

- MongoDB : 27017
- Redis : 6379
- memcached : 11211

网上查找：
常用数据库及端口
数据库	默认端口

- Amazon Redshift	5439
- Apache Derby	1527
- Apache Cassandra	9042
- Apache Hive	10000 (Hive Server2) or 9083 (Hive Metastore)
- Azure SQL Database	1433
- ClickHouse	8123
- Couchbase Query Query Service	11210
- Exasol	8563
- Greenplum	5432
- H2	8082
- HSQLDB	9001
- IBM Db2 LUW	50000
- MariaDB	3306
- Microsoft SQL Server	1433 (TCP), 1434 (UDP might be required)
- MySQL	3306
- Oracle	1521
- PostgreSQL	5432
- Snowflake	443
- SQLite	None
- Sybase ASE	5000
- Vertica	5433



