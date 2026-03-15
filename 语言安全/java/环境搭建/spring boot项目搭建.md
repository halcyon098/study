# spring boot项目搭建

idea有两种搭建方式：

1.新建项目，选择Spring Initializr

2.新建一个普通的Maven项目。配置pom.xml文件

下面主要使用第二种

## 添加依赖

pom.xml

```xml
<!-- Spring Boot的父POM，提供默认配置和依赖管理 -->  
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-parent</artifactId>  
    <version>2.6.0</version>  
</parent>  
  
<dependencies>  
    <!-- Spring Boot的Web起步依赖，用于构建Web应用 -->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency>  
      
    <!-- Spring Boot的测试起步依赖，包含常用的测试库 -->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-test</artifactId>  
        <!-- 注意：这里的版本通常不需要指定，因为父POM已管理 -->  
    </dependency>  
```

创建起来之后有可能没有src和test目录，需要自己新建，右键项目新建目录，idea自动给出提示，全选创建即可。或者手动创建目录，在右键目录，将目录添加到项目中。

1.在java文件夹下创建一个名称为com.example.ch01的包，在该包下创建启动类ch01Application

```java
package com.example.ch01; 
import org.springframework.boot.SpringApplication; // 导入SpringApplication类，用于启动Spring Boot应用  
import org.springframework.boot.autoconfigure.SpringBootApplication; // 导入SpringBootApplication注解  
  
// @SpringBootApplication注解是一个方便的注解，它包含了@Configuration、@EnableAutoConfiguration、@ComponentScan注解的功能  
// @Configuration：表明该类是一个配置类，Spring容器可以基于里面的注解（如@Bean）来生成bean  
// @EnableAutoConfiguration：告诉Spring Boot基于添加的jar依赖自动配置Spring应用。例如，如果classpath下有spring-boot-starter-web，Spring Boot会自动配置Tomcat和Spring MVC  
// @ComponentScan：告诉Spring在包和子包中查找其他组件、配置和服务，让@Component、@Service、@Repository等注解的类被Spring容器管理  
@SpringBootApplication  
public class ch01Application {  
    // main方法是Java程序的入口点。SpringApplication.run()方法会启动Spring应用  
    public static void main(String[] args) {  
        // SpringApplication.run()方法接收两个参数：第一个参数是应用的主配置类（带有@SpringBootApplication注解的类），第二个参数是传递给应用的命令行参数  
        SpringApplication.run(ch01Application.class,args);  
    }  
}
```

这段代码是一个典型的Spring Boot应用程序的启动类。Spring Boot旨在简化基于Spring的应用开发，通过自动配置（auto-configuration）和嵌入式服务器等技术，让开发者能够快速搭建并运行Spring应用。下面是对这段代码的详细注释分析，特别是关于`@SpringBootApplication`注解的部分。

 **关于`@SpringBootApplication`注解的进一步说明:**

- **@SpringBootApplication**：这是一个方便的注解，它结合了`@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`的功能。使用这个注解的类通常被称为主配置类（main configuration class），它用于启动Spring Boot应用。
- **@Configuration**：表明该类是一个配置类，Spring容器会基于该类中的注解（如`@Bean`）来生成bean。在Spring中，配置类通常用于定义bean的声明、导入其他配置类等。
- **@EnableAutoConfiguration**：告诉Spring Boot基于添加的jar依赖自动配置Spring应用。Spring Boot的自动配置特性能够极大地减少开发者需要编写的配置代码，因为它会智能地根据添加的依赖来推断并配置应用。
- **@ComponentScan**：告诉Spring在指定的包及其子包中查找其他组件、配置和服务，让带有`@Component`、`@Service`、`@Repository`等注解的类被Spring容器管理。默认情况下，`@SpringBootApplication`注解会扫描启动类所在的包及其子包。

综上所述，`@SpringBootApplication`注解是Spring Boot应用的核心，它简化了Spring应用的配置和启动过程。

2.在项目com.example.ch01包下创建名称为controller的包，在该包下创建控制器类HelloController

在您提供的`HelloController`类中，使用了Spring MVC框架中的两个关键注解：`@RestController`和`@RequestMapping`。下面是对这两个注解以及整个类的详细注释分析。

```java
package com.example.ch01.controller; 
import org.springframework.web.bind.annotation.RequestMapping; // 导入RequestMapping注解  
import org.springframework.web.bind.annotation.RestController; // 导入RestController注解  
  
// @RestController注解是一个组合注解，它结合了@Controller和@ResponseBody的功能  
// @Controller：表明该类是一个Spring MVC控制器，Spring会自动检测并注册该类为Spring应用上下文中的bean  
// @ResponseBody：表示该控制器中的方法返回的对象会直接作为HTTP响应体返回给客户端，而不是解析为跳转路径或视图名  
// 因此，@RestController注解的控制器中的方法返回的字符串、对象等都会被自动转换为JSON或XML格式（取决于请求的Accept头部或配置），并返回给客户端  
@RestController  
public class HelloController {  
    // @RequestMapping注解用于将HTTP请求映射到特定的处理函数上  
    // 在这里，它将"/first"这个URL路径映射到index()方法上  
    // 当客户端发起对"/first"的GET请求时，Spring MVC会调用index()方法来处理该请求  
    @RequestMapping("/first")  
    public String index(){  
        // 这里只是简单地打印一条日志信息到控制台  
        // 在实际开发中，你可能会在这里执行更复杂的业务逻辑  
        System.out.println("HelloController is running--devtools");  
        // 方法返回一个字符串，由于该类被@RestController注解标记，所以这个字符串会被直接作为HTTP响应体返回给客户端  
        // 而不是被解析为跳转路径或视图名  
        return "welcome to spring boot application!--devtools";  
    }  
}
```

### 关于注解的进一步说明：

- **@RestController**：这是Spring 4.0中引入的一个注解，它是一个方便的注解，用于创建RESTful web服务。通过组合`@Controller`和`@ResponseBody`的功能，它简化了RESTful控制器的开发。使用`@RestController`注解的类中的所有方法默认都会将返回值写入HTTP响应体中，而不是解析为视图名。
- **@RequestMapping**：这个注解用于将HTTP请求映射到特定的处理器方法上。它可以定义在类级别或方法级别。在类级别使用时，它指定了一个基本路径，该路径会被该类中所有方法级别的`@RequestMapping`注解所继承。在方法级别使用时，它指定了处理请求的具体路径和HTTP方法（如GET、POST等，但默认是任何方法）。在您的例子中，它只被定义在方法级别，指定了处理"/first"路径的GET请求的方法。

## 单元测试

项目test目录下建com.example.ch01包，并编写测试用例类WebTest，在该类中启动Web环境，并使用随机端口作为Web服务器端口。在测试用例类上使用@AutoConfigureMockMvc注解对Web虚拟调用功能开启

这段代码是一个使用Spring Boot Test和MockMvc进行Web层测试的示例。下面是对这段代码的详细注释分析：

```java
// 使用@SpringBootTest注解来标记这个类是一个Spring Boot的测试类。  
// webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT 指定Spring Boot应用在测试时应该启动一个内嵌的Servlet容器，并且监听一个随机端口。  
// 这对于测试需要启动整个Spring Boot应用的场景非常有用，因为它允许你测试Web层的功能。  
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)  
  
// @AutoConfigureMockMvc注解用于自动配置MockMvc。  
// MockMvc是一个强大的工具，用于在Spring MVC测试环境中模拟HTTP请求。  
// 它允许你测试你的控制器而不需要启动整个服务器。  
@AutoConfigureMockMvc  
  
public class WebTest {  
  
    // @Autowired注解用于自动注入MockMvc实例。  
    // 由于@AutoConfigureMockMvc已经自动配置了MockMvc，因此你可以直接在这里注入并使用它。  
    @Test  
    void testWeb(@Autowired MockMvc mvc) throws Exception{  
        // 使用MockMvcRequestBuilders来构建一个MockHttpServletRequestBuilder实例。  
        // 这里通过MockMvcRequestBuilders.get("/first")来模拟一个对"/first"路径的GET请求。  
        MockHttpServletRequestBuilder builder = MockMvcRequestBuilders.get("/first");  
  
        // 使用MockMvc的perform方法来执行上面构建的请求。  
        // 这个方法会执行请求并返回一个MockMvcResultMatchers实例，用于对响应进行断言。  
        // 在这个例子中，我们没有添加任何断言，但通常你会在这里添加断言来验证响应的状态码、内容等。  
        mvc.perform(builder);  
    }  
}
```

**注意**：

- 这段代码虽然展示了如何使用MockMvc来模拟HTTP请求，但它并没有包含任何断言来验证响应。在实际测试中，你通常会使用`andExpect`方法来添加断言，例如`mvc.perform(builder).andExpect(status().isOk())`来验证响应状态码是否为200（OK）。
- `@SpringBootTest`注解启动了一个完整的Spring应用上下文，这对于测试需要加载完整应用配置的场景非常有用。然而，如果你只需要测试Spring MVC控制器，并且不想启动整个应用上下文，你可以考虑使用`@WebMvcTest`注解，它只会加载与Web层相关的bean，从而加快测试速度。
- `@AutoConfigureMockMvc`注解是Spring Boot Test提供的一个便利注解，用于自动配置MockMvc。在Spring Boot 2.2及以后版本中，如果你已经使用了`@SpringBootTest`或`@WebMvcTest`等注解，并且你的测试类在`src/test/java`目录下，那么通常不需要显式添加`@AutoConfigureMockMvc`，因为Spring Boot会自动为你配置MockMvc。

踩坑：记得将@Test注解写到方法上面才可以启动测试方法

## 业务组件测试

在项目中的com.example.ch01包下创建名称为service的包，在该包下创建类HelloService，定义测试用例类。在项目ch01测试文件夹下的com.example.ch01包下编写测试用例类ServiceTest

当然，下面是对你提供的`HelloService`类及其`getById`方法的注释分析：

```java
// @Service注解标记这个类为服务层组件。  
// 在Spring框架中，服务层组件通常用于封装业务逻辑，它们可以被控制器层或其他服务层组件调用。  
// 使用@Service注解后，Spring容器会自动检测并管理这个类的实例，包括其生命周期和依赖注入。  
@Service  
public class HelloService {  
  
    // 这是一个公开的方法，用于根据ID获取某些信息（尽管在这个例子中它实际上只是打印了一个消息）。  
    // 方法名getById暗示了这是一个通过ID查询数据的操作，但在实际实现中，它并没有返回任何数据，只是打印了一个包含ID的字符串。  
    // 方法的参数是一个Integer类型的id，它代表要查询的数据的标识符。  
    // 需要注意的是，这个方法没有返回值（void），这在实际的业务逻辑中可能不是最佳实践，除非这个方法的目的仅仅是为了执行某些操作（如日志记录、状态更新等）而不需要返回结果。  
    public void getById(Integer id){  
        System.out.println("Service get id:"+id);  
    }  
}
```

**改进建议**：

1. **返回值**：如果`getById`方法旨在从数据库中检索数据，那么它应该返回一个包含所请求数据的对象或对象的集合。如果出于某种原因（如日志记录、验证等）而不需要返回数据，那么可以考虑将方法名更改为更能反映其目的的名称，例如`logById`或`validateById`。
2. **异常处理**：在业务逻辑中，如果根据ID查询数据失败（例如，ID不存在），那么可能需要抛出或处理异常。在这个例子中，由于方法只是打印了一个消息，所以没有异常处理，但在实际应用中，异常处理是非常重要的。
3. **参数验证**：虽然在这个简单的例子中可能不需要，但在实际业务逻辑中，对方法参数进行验证（如检查ID是否为`null`或空）是一个好习惯，可以防止程序在无效输入上崩溃。
4. **日志记录**：虽然这个例子中使用了`System.out.println`来打印消息，但在生产环境中，你应该使用日志框架（如SLF4J、Log4j等）来记录日志。这样做的好处是你可以更灵活地控制日志的级别、格式和输出位置。

当然，以下是对你提供的`ServiceTest`类及其`testService`方法的注释分析：

```java
// @SpringBootTest注解用于标记这个类是一个Spring Boot的测试类。  
// webEnvironment = SpringBootTest.WebEnvironment.NONE 指定在测试时不需要启动内嵌的Servlet容器。  
// 这意味着这个测试类不会测试与Web层相关的功能，而是专注于服务层或更低层次的测试。  
// 由于没有启动Web环境，因此这个测试类中的测试方法不会受到Web层配置（如Spring MVC、Spring WebFlux等）的影响。  
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)  
  
public class ServiceTest {  
  
    // 使用@Autowired注解自动注入HelloService的实例。  
    // 由于@SpringBootTest注解已经启动了Spring应用上下文，因此Spring会负责创建HelloService的实例，并将其注入到这个测试类中。  
    // 注意，这里假设HelloService是一个由Spring管理的bean，通常是通过@Service、@Component等注解标记的。  
    @Autowired  
    private HelloService helloService;  
  
    // 这是一个JUnit测试方法，用于测试HelloService的getById方法。  
    // 由于这个方法使用了@Test注解，因此JUnit测试框架会识别并运行它。  
    // 在这个测试方法中，我们调用了helloService的getById方法，并传递了一个整数15作为参数。  
    // 根据HelloService的实现，这个方法应该会打印出"Service get id:15"到控制台。  
    // 然而，这个测试方法本身并没有包含任何断言来验证helloService的行为是否符合预期。  
    // 在实际测试中，你应该添加断言来验证服务层的行为，例如检查返回值、验证数据库交互、检查日志输出等。  
    @Test  
    void testService(){  
        helloService.getById(15);  
    }  
  
}
```

**改进建议**：

1. **添加断言**：虽然这个测试方法调用了服务层的方法，但它没有验证任何结果。你应该添加断言来确保服务层的行为符合预期。例如，如果`getById`方法应该返回一个对象，你可以检查该对象是否非空且包含正确的数据。在这个例子中，由于`getById`方法只是打印了一个消息，你可能需要验证日志输出或使用Mockito等框架来模拟依赖项。
2. **模拟依赖项**：如果`HelloService`依赖于其他组件（如数据库访问层、其他服务等），你可能需要使用Mockito等框架来模拟这些依赖项，以便在隔离的环境中测试`HelloService`。
3. **测试多个场景**：编写多个测试方法来覆盖不同的场景，例如传递有效的ID、传递无效的ID（如果服务层需要处理这种情况）、传递`null`等。
4. **使用日志断言**：如果你想要验证日志输出，你可以使用像`slf4j-test`这样的库来捕获和断言日志消息。然而，请注意，直接测试日志输出可能不是最佳实践，因为它会使测试与日志实现的细节紧密耦合。更好的做法可能是验证服务层的行为（如数据库交互、返回值等），这些行为间接地验证了日志输出。

## 热部署的配置

（1）在pom.xml文件中添加依赖

```xml-dtd
        <!-- Spring Boot的开发工具，提供开发时的一些便利功能，如自动重启 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
```

（2）设置启动热部署

（1）在IDEA的菜单栏中依次选择“File”→“Settings”，进入IDEA的设置对话框，然后选择“Build，Execution，Deployment”的“Compiler”选项。

在右侧勾选“Build project automatically”选项将项目设置为自动编译，然后单击“Apply”→“OK”按钮保存设置。

（2）IDEA2019的，按住快捷键 ctrl+shift+alt +/选择Registry，选中compiler.automake.allow.when.app.running

踩坑：高版本没有这个包了，解决办法：[Settings](https://so.csdn.net/so/search?q=Settings&spm=1001.2101.3001.7020)--> Advanced Settings -->Compiler，将里面的两个选项全部勾选上

## 项目打包和运行

### 打包为可执行的JAR包

1）添加Maven打包插件。SpringBoot程序是基于Maven创建的，在对Spring Boot项目进行打包前，需要在项目pom.xml文件中加入Maven打包插件

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

2）使用IDEA进行打包，点击进入右侧栏的maven窗口，在生命周期下面双击“package”进行打包

 3）如果打包成功会在项目的target文件夹下创建项目对应的可执行JAR包,进入该目录在CMD中使用java -jar jar包名，执行。

## 打包为WAR包并运行

1）在项目的pom.xml文件中声明当前项目的打包方式为war。

```xml
   <groupId>org.example</groupId>
    <artifactId>JavaEE-SJ</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
```

2）在pom.xml文件中排除内置的Tomcat

```xml
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
<!--                排除tomcat依赖-->
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```



3）排除内置的Tomcat后，需要在pom.xml文件中手动添加Tomcat的依赖，以便在后续开发中使用对应的API

```xml
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
<!--            仅在编译和测试阶段使用，不会被打包-->
            <scope>provided</scope>
        </dependency>
```

4）在项目的pom.xml文件中定义打包插件，以及项目打包后包的名称

```xml
 <build>
        <finalName>springboot-war</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

5）修改Spring Boot启动类，使用外置Tomcat时，默认启动类需要继承SpringBootServletInitializer类，并重写configure()方法。修改后的Spring Boot启动类代码如下所示。

```java
@SpringBootApplication

//启动基于注解的方式的servlet组件扫描
@ServletComponentScan
public class ch01Application extends SpringBootServletInitializer {
    // main方法是Java程序的入口点。SpringApplication.run()方法会启动Spring应用
    public static void main(String[] args) {
        // SpringApplication.run()方法接收两个参数：第一个参数是应用的主配置类（带有@SpringBootApplication注解的类），第二个参数是传递给应用的命令行参数
        SpringApplication.run(ch01Application.class,args);
    }

//    使用外置Tomcat时，默认启动类需要继承SpringBootServletInitializer类，并重写configure()方法。

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(ch01Application.class);
    }
}
```

```java
/**  
 * 配置SpringApplicationBuilder的方法。  
 * 这个方法用于指定Spring Boot应用程序的主配置类。  
 *   
 * @param builder SpringApplicationBuilder实例，它是用于构建Spring应用程序的。  
 * @return 配置后的SpringApplicationBuilder实例，允许链式调用。  
 */  
protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {  
    // 使用builder的sources方法，并传递ch01Application.class作为参数。  
    // 这告诉SpringApplicationBuilder，当构建Spring应用程序时，应该使用ch01Application类作为主配置类。  
    // ch01Application类通常包含@SpringBootApplication注解，这是Spring Boot的入口注解。  
    return builder.sources(ch01Application.class);  
}
```

6）打包，和jar包打一样

7）将打包好的WAR包拷贝到本地Tomcat安装目录下的webapps文件夹下

8）在CMD窗口中执行Tomcat安装目录下bin目录中的startup.bat命令启动Tomcat。在浏览器中访问http://localhost:8080/springboot-war/first