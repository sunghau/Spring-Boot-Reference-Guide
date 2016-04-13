### Spring Boot CLI

Spring Boot CLI是一個命令行工具，如果想使用Spring進行快速開發可以使用它。它允許你運行Groovy腳本，這意味著你可以使用熟悉的類Java語法，並且沒有那麼多的模板代碼。你也可以啟動一個新的工程或為Spring Boot CLI編寫自己的命令。

### 安裝CLI

你可以手動安裝Spring Boot CLI，也可以使用GVM（Groovy環境管理工具）或Homebrew，MacPorts（如果你是一個OSX用戶）。參考"Getting started"的[Section 10.2, “Installing the Spring Boot CLI” ](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-installing-the-cli)可以看到全麵的安裝指令。

### 使用CLI

一旦安裝好CLI，你可以輸入`spring`來運行它。如果你不使用任何參數運行`spring`，將會展現一個簡單的幫助界麵：
```shell
$ spring
usage: spring [--help] [--version]
       <command> [<args>]

Available commands are:

  run [options] <files> [--] [args]
    Run a spring groovy script

  ... more command help is shown here
```
你可以使用`help`獲取任何支援命令的詳細信息。例如：
```shell
$ spring help run
spring run - Run a spring groovy script

usage: spring run [options] <files> [--] [args]

Option                     Description
------                     -----------
--autoconfigure [Boolean]  Add autoconfigure compiler
                             transformations (default: true)
--classpath, -cp           Additional classpath entries
-e, --edit                 Open the file with the default system
                             editor
--no-guess-dependencies    Do not attempt to guess dependencies
--no-guess-imports         Do not attempt to guess imports
-q, --quiet                Quiet logging
-v, --verbose              Verbose logging of dependency
                             resolution
--watch                    Watch the specified file for changes
```
`version`命令提供一個檢查你正在使用的Spring Boot版本的快速方式：
```shell
$ spring version
Spring CLI v1.3.0.BUILD-SNAPSHOT
```
* 使用CLI運行應用

你可以使用`run`命令編譯和運行Groovy源代碼。Spring Boot CLI完全自包含，以致於你不需要安裝任何外部的Groovy。

下面是一個使用Groovy編寫的"hello world" web應用：
hello.grooy
```java
@RestController
class WebApplication {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```
想要編譯和運行應用，輸入：
```shell
$ spring run hello.groovy
```
想要給應用傳遞命令行參數，你需要使用一個`--`來將它們和"spring"命令參數區分開來。例如：
```shell
$ spring run hello.groovy -- --server.port=9000
```
想要設置JVM命令行參數，你可以使用`JAVA_OPTS`環境變量，例如：
```shell
$ JAVA_OPTS=-Xmx1024m spring run hello.groovy
```
- 推斷"grab"依賴

標準的Groovy包含一個`@Grab`注解，它允許你聲明對第三方庫的依賴。這項有用的技術允許Groovy以和Maven或Gradle相同的方式下載jars，但不需要使用建構工具。

Spring Boot進一步延伸了該技術，它會基於你的代碼嘗試推導你"grab"哪個庫。例如，由於WebApplication代碼上使用了`@RestController`注解，"Tomcat"和"Spring MVC"將被獲取（grabbed）。

下面items被用作"grab hints"：

|items|Grabs|
|-----|:-----|
|JdbcTemplate,NamedParameterJdbcTemplate,DataSource|JDBC應用|
|@EnableJms|JMS應用|
|@EnableCaching|Caching abstraction|
|@Test|JUnit|
|@EnableRabbit|RabbitMQ|
|@EnableReactor|Project Reactor|
|繼承Specification|Spock test|
|@EnableBatchProcessing|Spring Batch|
|@MessageEndpoint,@EnableIntegrationPatterns|Spring Integration|
|@EnableDeviceResolver|Spring Mobile|
|@Controller,@RestController,@EnableWebMvc|Spring MVC + Embedded Tomcat|
|@EnableWebSecurity|Spring Security|
|@EnableTransactionManagement|Spring Transaction Management|

**注**：想要理解自定義是如何生效，可以查看Spring Boot CLI源碼中的[CompilerAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-cli/src/main/java/org/springframework/boot/cli/compiler/CompilerAutoConfiguration.java)子類。

- 推斷"grab"坐標

Spring Boot擴展Groovy標準"@Grab"注解使其能夠允許你指定一個沒有group或version的依賴，例如`@Grab('freemarker')`。
artifact’s的組和版本是通過查看Spring Boot的依賴元數據推斷出來的。注意默認的元數據是和你使用的CLI版本綁定的－隻有在你遷移到一個CLI新版本時它才會改變，這樣當你的依賴改變時你就可以控製了。在[附錄](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#appendix-dependency-versions)的表格中可以查看默認元數據包含的依賴和它們的版本。

- 默認import語句

為了幫助你減少Groovy代碼量，一些`import`語句被自動包含進來了。注意上麵的範例中引用`@Component`，`@RestController`和`@RequestMapping`而沒有使用全限定名或`import`語句。

**注**：很多Spring注解在不使用`import`語句的情況下可以正常工作。嘗試運行你的應用，看一下在添加imports之前哪些會失敗。

- 自動創建main方法

跟等效的Java應用不同，你不需要在Groovy腳本中添加一個`public static void main(String[] args)`方法。Spring Boot 會使用你編譯後的代碼自動創建一個SpringApplication。

- 自定義"grab"元數據

Spring Boot提供一個新的`@GrabMetadata`注解，你可以使用它提供自定義的依賴元數據，以覆蓋Spring Boot的默認配置。該元數據通過使用提供一個或多個配置文件坐標的注解來指定（使用一個屬性標識符"type"部署到Maven倉庫）.。配置文件中的每個實體必須遵循`group:module=version`的格式。

例如，下面的聲明：
```java
`@GrabMetadata("com.example.custom-versions:1.0.0")`
```
將會加載Maven倉庫處於`com/example/custom-versions/1.0.0/`下的`custom-versions-1.0.0.properties`文件。

可以通過注解指定多個屬性文件，它們會以聲明的順序被使用。例如：
```java
`@GrabMetadata(["com.example.custom-versions:1.0.0",
        "com.example.more-versions:1.0.0"])`
```
意味著位於`more-versions`的屬性將覆蓋位於`custom-versions`的屬性。

你可以在任何能夠使用`@Grab`的地方使用`@GrabMetadata`，然而，為了確保元數據的順序一致，你在應用程序中最多隻能使用一次`@GrabMetadata`。[Spring IO Platform](http://platform.spring.io/)是一個非常有用的依賴元數據源(Spring Boot的超集)，例如：
```java
@GrabMetadata('io.spring.platform:platform-versions:1.0.4.RELEASE')
```
* 測試你的代碼

`test`命令允許你編譯和運行應用程序的測試用例。常規使用方式如下：
```shell
$ spring test app.groovy tests.groovy
Total: 1, Success: 1, : Failures: 0
Passed? true
```
在這個範例中，`test.groovy`包含JUnit `@Test`方法或Spock `Specification`類。所有的普通框架注解和靜態方法在不使用import導入的情況下，仍舊可以使用。

下面是我們使用的`test.groovy`文件（含有一個JUnit測試）：
```java
class ApplicationTests {

    @Test
    void homeSaysHello() {
        assertEquals("Hello World!", new WebApplication().home())
    }

}
```
**注**：如果有多個測試源文件，你可以傾向於使用一個test目錄來組織它們。

* 多源文件應用

你可以在所有接收文件輸入的命令中使用shell通配符。這允許你輕鬆處理來自一個目錄下的多個文件，例如：
```shell
$ spring run *.groovy
```
如果你想將'test'或'spec'代碼從主應用代碼中分離，這項技術就十分有用了：
```shell
$ spring test app/*.groovy test/*.groovy
```
* 應用打包

你可以使用`jar`命令打包應用程序為一個可執行的jar文件。例如：
```shell
$ spring jar my-app.jar *.groovy
```
最終的jar包括編譯應用產生的類和所有依賴，這樣你就可以使用`java -jar`來執行它了。該jar文件也包括來自應用classpath的實體。你可以使用`--include`和`--exclude`添加明確的路徑（兩者都是用逗號分割，同樣都接收值為'+'和'-'的前綴，'-'意味著它們將從默認設置中移除）。默認包含（includes）：
```shell
public/**, resources/**, static/**, templates/**, META-INF/**, *
```
默認排除(excludes)：
```shell
.*, repository/**, build/**, target/**, **/*.jar, **/*.groovy
```
查看`spring help jar`可以獲得更多信息。

* 初始化新工程

`init`命令允許你使用[start.spring.io](https://start.spring.io/)在不離開shell的情況下創建一個新的項目。例如：
```shell
$ spring init --dependencies=web,data-jpa my-project
Using service at https://start.spring.io
Project extracted to '/Users/developer/example/my-project'
```
這創建了一個`my-project`目錄，它是一個基本Maven且依賴`spring-boot-starter-web`和`spring-boot-starter-data-jpa`的項目。你可以使用`--list`參數列出該服務的能力。
```shell
$ spring init --list
=======================================
Capabilities of https://start.spring.io
=======================================

Available dependencies:
-----------------------
actuator - Actuator: Production ready features to help you monitor and manage your application
...
web - Web: Support for full-stack web development, including Tomcat and spring-webmvc
websocket - Websocket: Support for WebSocket development
ws - WS: Support for Spring Web Services

Available project types:
------------------------
gradle-build -  Gradle Config [format:build, build:gradle]
gradle-project -  Gradle Project [format:project, build:gradle]
maven-build -  Maven POM [format:build, build:maven]
maven-project -  Maven Project [format:project, build:maven] (default)

...
```
`init`命令支援很多選項，查看`help`輸出可以獲得更多詳情。例如，下面的命令創建一個使用Java8和war打包的gradle項目：
```shell
$ spring init --build=gradle --java-version=1.8 --dependencies=websocket --packaging=war sample-app.zip
Using service at https://start.spring.io
Content saved to 'sample-app.zip'
```
* 使用內嵌shell

Spring Boot包括完整的BASH和zsh shells的命令行腳本。如果你不使用它們中的任何一個（可能你是一個Window用戶），那你可以使用`shell`命令啟用一個整合shell。
```shell
$ spring shell
Spring Boot (v1.3.0.BUILD-SNAPSHOT)
Hit TAB to complete. Type \'help' and hit RETURN for help, and \'exit' to quit.
```
從內嵌shell中可以直接運行其他命令：
```shell
$ version
Spring CLI v1.3.0.BUILD-SNAPSHOT
```
內嵌shell支援ANSI顏色輸出和tab補全。如果需要運行一個原生命令，你可以使用`$`前綴。點擊ctrl-c將退出內嵌shell。

* 為CLI添加擴展

使用`install`命令可以為CLI添加擴展。該命令接收一個或多個格式為`group:artifact:version`的artifact坐標集。例如：
```shell
$ spring install com.example:spring-boot-cli-extension:1.0.0.RELEASE
```
除了安裝你提供坐標的artifacts標識外，所有依賴也會被安裝。使用`uninstall`可以卸載一個依賴。和`install`命令一樣，它接收一個或多個格式為`group:artifact:version`的artifact坐標集。例如：
```shell
$ spring uninstall com.example:spring-boot-cli-extension:1.0.0.RELEASE
```
它會通過你提供的坐標卸載相應的artifacts標識和它們的依賴。

為了卸載所有附加依賴，你可以使用`--all`選項。例如：
```shell
$ spring uninstall --all
```
* 使用Groovy beans DSL開發應用

Spring框架4.0版本對beans{} DSL（借鑒自[Grails](http://grails.org/)）提供原生支援，你可以使用相同的格式在你的Groovy應用程序腳本中嵌入bean定義。有時候這是一個包括外部特性的很好的方式，比如中間件聲明。例如：
```java
@Configuration
class Application implements CommandLineRunner {

    @Autowired
    SharedService service

    @Override
    void run(String... args) {
        println service.message
    }

}

import my.company.SharedService

beans {
    service(SharedService) {
        message = "Hello World"
    }
}
```
你可以使用beans{}混合位於相同文件的類聲明，隻要它們都處於頂級，或如果你喜歡的話，可以將beans DSL放到一個單獨的文件中。


