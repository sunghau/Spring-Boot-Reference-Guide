### 開始

如果你想從總體上對Spring Boot或Spring入門，本章節就是為你準備的！在這裡，我們將回答基本的"what?"，"how?"和"why?"問題。你會發現一個溫雅的Spring Boot介紹及安裝指南。然後我們建構第一個Spring Boot應用，並討論一些我們需要遵循的核心原則。

### Spring Boot介紹

Spring Boot使開發獨立的，產品級別的基於Spring的應用變得非常簡單，你隻需"just run"。
我們為Spring平台及第三方庫提供開箱即用的設置，這樣你就可以有條不紊地開始。多數Spring Boot應用需要很少的Spring配置。

你可以使用Spring Boot創建Java應用，並使用`java -jar`啟動它或采用傳統的war部署方式。我們也提供了一個運行"spring腳本"的命令行工具。

我們主要的目標是：

- 為所有的Spring開發提供一個從根本上更快的和廣泛使用的入門經驗。
- 開箱即用，但你可以通過不采用默認設置來擺脫這種方式。
- 提供一係列大型項目常用的非功能性特征（比如，內嵌服務器，安全，指標，健康檢測，外部化配置）。
- 絕對不需要代碼生成及XML配置。

### 系統要求

默認情況下，Spring Boot 1.3.0.BUILD-SNAPSHOT 需要Java７和Spring框架4.1.3或以上。你可以在Java6下使用Spring Boot，不過需要添加額外配置。具體參考[Section 73.9, “How to use Java 6” ](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-java-6)。建構環境明確支援的有Maven（3.2+）和Gradle（1.12+）。

**注**：盡管你可以在Java6或Java7環境下使用Spring Boot，通常我們建議你如果可能的話就使用Java8。

* Servlet容器

下列內嵌容器支援開箱即用（out of the box）：

|名稱|Servlet版本|Java版本|
|--------|:-------|:-------|
|Tomcat 8|3.1|Java 7+|
|Tomcat 7|3.0|Java 6+|
|Jetty 9|3.1|Java 7+|
|Jetty 8|3.0|Java 6+|
|Undertow 1.1|3.1|Java 7+|

你也可以將Spring Boot應用部署到任何兼容Servlet 3.0+的容器。

### Spring Boot安裝

Spring Boot可以跟典型的Java開發工具一塊使用或安裝為一個命令行工具。不管怎樣，你將需要安裝[Java SDK v1.6 ](http://www.java.com/)或更高版本。在開始之前，你需要檢查下當前安裝的Java版本：
```shell
$ java -version
```
如果你是一個Java新手，或你只是想體驗一下Spring Boot，你可能想先嘗試[Spring Boot CLI](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-installing-the-cli)，否則繼續閱讀經典地安裝指南。

**注**：盡管Spring Boot兼容Java 1.6，如果可能的話，你應該考慮使用Java最新版本。

* 為Java開發者準備的安裝指南

你可以以和任何標準Java庫相同的方式使用Spring Boot。隻需要簡單地在你的classpath下包含正確的`spring-boot-*.jar`文件。Spring Boot不需要整合任何特殊的工具，所以你可以使用任何IDE或文本編輯器；Spring Boot應用也沒有什麼特殊之處，所以你可以像任何其他Java程序那樣運行和調試。

盡管你可以拷貝Spring Boot jars，不過，我們通常推薦你使用一個支援依賴管理的建構工具（比如Maven或Gradle）。

* Maven安裝

Spring Boot兼容Apache Maven 3.2或更高版本。如果沒有安裝Maven，你可以參考[maven.apache.org](http://maven.apache.org/)指南。

**注**：在很多操作系統上，你可以通過一個包管理器安裝Maven。如果你是一個OSX Homebrew用戶，可以嘗試`brew install maven`。Ubuntu用戶可以運行`sudo apt-get install maven`。

Spring Boot依賴的`groupId`為`org.springframework.boot`。通常你的Maven POM文件需要繼承`spring-boot-starter-parent`，然後聲明一個或多個[“Starter POMs”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-starter-poms)依賴。Spring Boot也提供了一個用於創建可執行jars的[Maven插件](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-maven-plugin)。

下面是一個典型的`pom.xml`文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.0.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <!-- Add Spring repositories -->
    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```
**注**：`spring-boot-starter-parent`是使用Spring Boot的一個不錯的方式，但它不總是合適的。有時你需要繼承一個不同的parent POM，或者你可能只是不喜歡我們的默認配置。查看[Section 13.1.2, “Using Spring Boot without the parent POM”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-maven-without-a-parent)獲取使用`import`的替代解決方案。

* Gradle安裝

Spring Boot兼容Gradle 1.12或更高版本。如果沒有安裝Gradle，你可以參考[www.gradle.org](http://www.gradle.org/)上的指南。

Spring Boot依賴可以使用`org.springframework.boot` `group`來聲明。通常，你的項目將聲明一個或多個[“Starter POMs”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-starter-poms)依賴。Spring Boot提供一個用於簡化依賴聲明和創建可執行jars的有用的[Gradle插件](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-gradle-plugin)。

**注**：當你需要建構一個項目時，Gradle Wrapper提供一個獲取Gradle的漂亮方式。它是一個伴隨你的代碼一塊提交的小腳本和庫，用於啟動建構進程。具體參考[Gradle Wrapper](www.gradle.org/docs/current/userguide/gradle_wrapper.html)。

下面是一個典型的`build.gradle`文件：
```gradle
buildscript {
    repositories {
        jcenter()
        maven { url "http://repo.spring.io/snapshot" }
        maven { url "http://repo.spring.io/milestone" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.0.BUILD-SNAPSHOT")
    }
}

apply plugin: 'java'
apply plugin: 'spring-boot'

jar {
    baseName = 'myproject'
    version =  '0.0.1-SNAPSHOT'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/snapshot" }
    maven { url "http://repo.spring.io/milestone" }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}

```
* Spring Boot CLI安裝

Spring Boot是一個命令行工具，用於使用Spring進行快速原型搭建。它允許你運行[Groovy](http://groovy.codehaus.org/)腳本，這意味著你可以使用類Java的語法，並且沒有那麼多的模板代碼。

你沒有必要為了使用Spring Boot而去用CLI，但它絕對是助力Spring應用的最快方式。

- 手動安裝

你可以從Spring軟件倉庫下載Spring CLI分發包：

1. [spring-boot-cli-1.3.0.BUILD-SNAPSHOT-bin.zip](http://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/1.3.0.BUILD-SNAPSHOT/spring-boot-cli-1.3.0.BUILD-SNAPSHOT-bin.zip)
2. [spring-boot-cli-1.3.0.BUILD-SNAPSHOT-bin.tar.gz](http://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/1.3.0.BUILD-SNAPSHOT/spring-boot-cli-1.3.0.BUILD-SNAPSHOT-bin.tar.gz)

不穩定的[snapshot分發包](http://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)也能獲取到。

下載完成後，遵循解壓後的存檔裡的[INSTALL.txt](http://raw.github.com/spring-projects/spring-boot/master/spring-boot-cli/src/main/content/INSTALL.txt)操作指南進行安裝。一般而言，在`.zip`文件的`bin/`目錄下存在一個`spring`腳本（Windows下是`spring.bat`），或者使用`java -jar`來運行一個`.jar`文件（該腳本會幫你確定classpath被正確設置）。

- 使用GVM安裝

GVM（Groovy環境管理器）可以用來管理多種不同版本的Groovy和Java二進製包，包括Groovy自身和Spring Boot CLI。可以從[gvmtool.net](http://gvmtool.net/)獲取`gvm`，並使用以下命令安裝Spring Boot：
```shell
$ gvm install springboot
$ spring --version
Spring Boot v1.3.0.BUILD-SNAPSHOT
```
如果你正在為CLI開發新的特性，並想輕鬆獲取你剛建構的版本，可以使用以下命令：
```shell
$ gvm install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-1.3.0.BUILD-SNAPSHOT-bin/spring-1.3.0.BUILD-SNAPSHOT/
$ gvm use springboot dev
$ spring --version
Spring CLI v1.3.0.BUILD-SNAPSHOT
```
這將會在你的gvm倉庫中安裝一個名叫`dev`的本地`spring`實例。它指向你的目標建構位置，所以每次你重新建構Spring Boot，`spring`將會是最新的。

你可以通過以下命令來驗證：
```shell
$ gvm ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 1.3.0.BUILD-SNAPSHOT

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```
- 使用OSX Homebrew進行安裝

如果你的環境是Mac，並使用[Homebrew](http://brew.sh/)，想要安裝Spring Boot CLI隻需如下操作：
```shell
$ brew tap pivotal/tap
$ brew install springboot
```
Homebrew將把`spring`安裝到`/usr/local/bin`下。

**注**：如果該方案不可用，可能是因為你的brew版本太老了。你隻需執行`brew update`並重試即可。

- 使用MacPorts進行安裝

如果你的環境是Mac，並使用[MacPorts](http://www.macports.org/)，想要安裝Spring Boot CLI隻需如下操作：
```shell
$ sudo port install spring-boot-cli
```
- 命令行實現

Spring Boot CLI啟動腳本為[BASH](http://en.wikipedia.org/wiki/Bash_%28Unix_shell%29)和[zsh](http://en.wikipedia.org/wiki/Zsh) shells提供完整的命令行實現。你可以在任何shell中`source`腳本（名稱也是`spring`），或將它放到你個人或系統範圍的bash實現初始化中。在一個Debian系統裡，系統範圍的腳本位於`/shell-completion/bash`下，當一個新的shell啟動時該目錄下的所有腳本都被執行。想要手動運行該腳本，例如，你已經使用`GVM`進行安裝了：
```shell
$ . ~/.gvm/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
  grab  help  jar  run  test  version
```

**注**：如果你使用Homebrew或MacPorts安裝Spring Boot CLI，命令行實現腳本會自動注冊到你的shell。

- Spring CLI範例快速入門

下面是一個相當簡單的web應用，你可以用它測試你的安裝是否成功。創建一個名叫`app.groovy`的文件：
```groovy
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```
然後簡單地從一個shell中運行它：
```shell
$ spring run app.groovy
```
**注**：當你首次運行該應用時將會花費一點時間，因為需要下載依賴。後續運行將會快很多。

在你最喜歡的瀏覽器中打開[localhost:8080](localhost:8080)，然後你應該看到以下輸出：
```java
Hello World!
```
### 從Spring Boot早期版本升級

如果你正在升級一個Spring Boot早期版本，查看下放在[project wiki](http://github.com/spring-projects/spring-boot/wiki)上的"release notes"。你會發現每次發布的更新指南和一個"new and noteworthy"特性列表。

想要升級一個已安裝的CLI，你需要使用合適的包管理命令（例如，`brew upgrade`），或如果你是手動安裝CLI，按照[standard instructions](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-manual-cli-installation)操作並記得更新你的`PATH`環境變量以移除任何老的引用。

### 開發你的第一個Spring Boot應用

讓我們使用Java開發一個簡單的"Hello World!" web應用，來強調下Spring Boot的一些關鍵特性。我們將使用Maven建構該項目，因為大多數IDEs都支援它。

**注**：[spring.io](http://spring.io/)網站包含很多使用Spring Boot的"入門"指南。如果你正在找特定問題的解決方案，可以先去那瞅瞅。

在開始前，你需要打開一個終端，檢查是否安裝可用的Java版本和Maven：
```shell
$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
```
```shell
$ mvn -v
Apache Maven 3.2.3 (33f8c3e1027c3ddde99d3cdebad2656a31e8fdf4; 2014-08-11T13:58:10-07:00)
Maven home: /Users/user/tools/apache-maven-3.1.1
Java version: 1.7.0_51, vendor: Oracle Corporation
```
**注**：該範例需要創建自己的文件夾。後續的操作假設你已創建一個合適的文件夾，並且它是你的“當前目錄”。

* 創建POM

我們需要以創建一個Maven `pom.xml`文件作為開始。該`pom.xml`是用來建構項目的處方。打開你最喜歡的文本編輯器，然後添加以下內容：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.0.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Additional lines to be added here... -->

    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```
這會給你一個可運轉的建構，你可以通過運行`mvn package`測試它（現在你可以忽略"jar將是空的-沒有包含任何內容！"的警告）。

**注**：目前你可以將該項目導入一個IDE（大多數現代的Java IDE都包含對Maven的內建支援）。簡單起見，我們將繼續使用普通的文本編輯器完成該範例。

* 添加classpath依賴

Spring Boot提供很多"Starter POMs"，這能夠讓你輕鬆的將jars添加到你的classpath下。我們的範例程序已經在POM的`partent`節點使用了`spring-boot-starter-parent`。`spring-boot-starter-parent`是一個特殊的starter，它提供了有用的Maven默認設置。同時，它也提供了一個`dependency-management`節點，這樣對於”blessed“依賴你可以省略`version`標記。

其他的”Starter POMs“簡單的提供依賴，這些依賴可能是你開發特定類型的應用時需要的。由於正在開發一個web應用，我們將添加一個`spring-boot-starter-web`依賴-但在此之前，讓我們看下目前所擁有的：
```shell
$ mvn dependency:tree
[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```
`mvn dependency:tree`命令以樹形表示來列印你的項目依賴。你可以看到`spring-boot-starter-parent`本身並沒有提供依賴。編輯我們的`pom.xml`，並在`parent`節點下添加`spring-boot-starter-web`依賴：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
如果再次運行`mvn dependency:tree`，你將看到現在有了一些其他依賴，包括Tomcat web服務器和Spring Boot自身。

* 編寫代碼

為了完成應用程序，我們需要創建一個單獨的Java文件。Maven默認會編譯`src/main/java`下的源碼，所以你需要創建那樣的文件結構，然後添加一個名為`src/main/java/Example.java`的文件：
```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```
盡管這裡沒有太多代碼，但很多事情正在發生。讓我們分步探討重要的部分。

* @RestController和@RequestMapping注解

我們的`Example`類上使用的第一個注解是`@RestController`。這被稱為一個構造型（stereotype）注解。它為閱讀代碼的人們提供建議。對於Spring，該類扮演了一個特殊角色。在本範例中，我們的類是一個web `@Controller`，所以當處理進來的web請求時，Spring會詢問它。

`@RequestMapping`注解提供路由信息。它告訴Spring任何來自"/"路徑的HTTP請求都應該被映射到`home`方法。`@RestController`注解告訴Spring以字符串的形式渲染結果，並直接返回給調用者。

**注**：`@RestController`和`@RequestMapping`注解是Spring MVC注解（它們不是Spring Boot的特定部分）。具體查看Spring參考文件的[MVC章節](http://docs.spring.io/spring/docs/4.1.5.RELEASE/spring-framework-reference/htmlsingle#mvc)。

* @EnableAutoConfiguration注解

第二個類級別的注解是`@EnableAutoConfiguration`。這個注解告訴Spring Boot根據添加的jar依賴猜測你想如何配置Spring。由於`spring-boot-starter-web`添加了Tomcat和Spring MVC，所以auto-configuration將假定你正在開發一個web應用並相應地對Spring進行設置。

**Starter POMs和Auto-Configuration**：設計auto-configuration的目的是更好的使用"Starter POMs"，但這兩個概念沒有直接的聯係。你可以自由地挑選starter POMs以外的jar依賴，並且Spring Boot將仍舊盡最大努力去自動配置你的應用。

* main方法

我們的應用程序最後部分是`main`方法。這只是一個標準的方法，它遵循Java對於一個應用程序入口點的約定。我們的main方法通過調用`run`，將業務委托給了Spring Boot的`SpringApplication`類。`SpringApplication`將引導我們的應用，啟動Spring，相應地啟動被自動配置的Tomcat web服務器。我們需要將`Example.class`作為參數傳遞給`run`方法來告訴`SpringApplication`誰是主要的Spring組件。為了暴露任何的命令行參數，`args`數組也會被傳遞過去。

### 運行範例

到此我們的應用應該可以工作了。由於使用了`spring-boot-starter-parent` POM，這樣我們就有了一個非常有用的`run`目標，我們可以用它啟動程序。在項目根目錄下輸入`mvn spring-boot:run`來啟動應用：
```shell
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.3.0.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```
如果使用一個瀏覽器打開[localhost:8080](http://localhost:8080)，你應該可以看到以下輸出：
```shell
Hello World!
```
點擊`ctrl-c`溫雅地關閉應用程序。

### 創建一個可執行jar

讓我們通過創建一個完全自包含的可執行jar文件來結束我們的範例，該jar文件可以在生產環境運行。可執行jars（有時候被成為胖jars "fat jars"）是包含你的編譯後的類和你的代碼運行所需的依賴jar的存檔。

**可執行jars和Java**：Java沒有提供任何標準的加載內嵌jar文件（即jar文件中還包含jar文件）的方法。如果你想發布一個自包含的應用這就是一個問題。為了解決該問題，很多開發者采用"共享的"jars。一個共享的jar簡單地將來自所有jars的類打包進一個單獨的“超級jar”。采用共享jar方式的問題是很難區分在你的應用程序中可以使用哪些庫。在多個jars中如果存在相同的文件名（但內容不一樣）也會是一個問題。Spring Boot采取一個[不同的途徑](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#executable-jar)，並允許你真正的內嵌jars。

為了創建可執行的jar，需要將`spring-boot-maven-plugin`添加到我們的`pom.xml`中。在`dependencies`節點下插入以下內容：
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
**注**：`spring-boot-starter-parent` POM包含用於綁定`repackage`目標的`<executions>`配置。如果你不使用parent POM，你將需要自己聲明該配置。具體參考[插件文件](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/usage.html)。

保存你的`pom.xml`，然後從命令行運行`mvn package`：
```shell
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:1.3.0.BUILD-SNAPSHOT:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
如果查看`target`目錄，你應該看到`myproject-0.0.1-SNAPSHOT.jar`。該文件應該有10Mb左右的大小。如果想偷看內部結構，你可以運行`jar tvf`：
```shell
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```
在`target`目錄下，你應該也能看到一個很小的名為`myproject-0.0.1-SNAPSHOT.jar.original`的文件。這是在Spring Boot重新打包前Maven創建的原始jar文件。

為了運行該應用程序，你可以使用`java -jar`命令：
```shell
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.3.0.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```
和以前一樣，點擊`ctrl-c`來溫柔地退出程序。
