### 使用Spring Boot
本章節將會詳細介紹如何使用Spring Boot。它覆蓋了建構系統，自動配置和運行/部署選項等主題。我們也覆蓋了一些Spring Boot最佳實踐。盡管Spring Boot沒有什麼特別的（只是一個你能消費的庫），但仍有一些建議，如果你遵循的話將會讓你的開發進程更容易。

如果你剛接觸Spring Boot，那最好先讀下上一章節的[Getting Started](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started)指南。

### 建構系統

強烈建議你選擇一個支援依賴管理，能消費發布到Maven中央倉庫的artifacts的建構系統。我們推薦你選擇Maven或Gradle。選擇其他建構系統來使用Spring Boot也是可能的（比如Ant），但它們不會被很好的支援。

* Maven

Maven用戶可以繼承`spring-boot-starter-parent`項目來獲取合適的默認設置。該父項目提供以下特性：
- 默認編譯級別為Java 1.6
- 源碼編碼為UTF-8
- 一個依賴管理節點，允許你省略普通依賴的`<version>`標簽，繼承自`spring-boot-dependencies` POM。
- 合適的[資源過濾](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)
- 合適的插件配置（[exec插件](http://mojo.codehaus.org/exec-maven-plugin/)，[surefire](http://maven.apache.org/surefire/maven-surefire-plugin/)，[Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin)，[shade](http://maven.apache.org/plugins/maven-shade-plugin/)）
- 針對`application.properties`和`application.yml`的資源過濾

最後一點：由於默認配置文件接收Spring風格的佔位符（`${...}`），Maven filtering改用`@..@`佔位符（你可以使用Maven屬性`resource.delimiter`來覆蓋它）。

* 繼承starter parent

想配置你的項目繼承`spring-boot-starter-parent`隻需要簡單地設置`parent`為：
```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.3.0.BUILD-SNAPSHOT</version>
</parent>
```
**注**：你應該隻需要在該依賴上指定Spring Boot版本。如果導入其他的starters，你可以放心的省略版本號。

* 使用沒有父POM的Spring Boot

不是每個人都喜歡繼承`spring-boot-starter-parent` POM。你可能需要使用公司標準parent，或你可能傾向於顯式聲明所有Maven配置。

如果你不使用`spring-boot-starter-parent`，通過使用一個`scope=import`的依賴，你仍能獲取到依賴管理的好處：
```xml
<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.3.0.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
* 改變Java版本 

`spring-boot-starter-parent`選擇相當保守的Java兼容策略。如果你遵循我們的建議，使用最新的Java版本，你可以添加一個`java.version`屬性：
```xml
<properties>
    <java.version>1.8</java.version>
</properties>
```
* 使用Spring Boot Maven插件

Spring Boot包含一個[Maven插件](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-maven-plugin)，它可以將項目打包成一個可執行jar。如果想使用它，你可以將該插件添加到`<plugins>`節點處：
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
**注**：如果使用Spring Boot starter parent pom，你隻需要添加該插件而無需配置它，除非你想改變定義在partent中的設置。

* Gradle

Gradle用戶可以直接在它們的`dependencies`節點處導入”starter POMs“。跟Maven不同的是，這裡沒有用於導入共享配置的"超父"（super parent）。
```gradle
apply plugin: 'java'

repositories { jcenter() }
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.3.0.BUILD-SNAPSHOT")
}
```
[spring-boot-gradle-plugin](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-gradle-plugin)插件也是可以使用的，它提供創建可執行jar和從source運行項目的任務。它也添加了一個`ResolutionStrategy`用於讓你省略常用依賴的版本號：
```gradle
buildscript {
    repositories { jcenter() }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.0.BUILD-SNAPSHOT")
    }
}

apply plugin: 'java'
apply plugin: 'spring-boot'

repositories { jcenter() }
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```
* Ant

使用Apache Ant建構一個Spring Boot項目是完全可能的，然而，Spring Boot沒有為它提供特殊的支援或插件。Ant腳本可以使用Ivy依賴管理系統來導入starter POMs。

查看[Section 73.8, “Build an executable archive with Ant”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-build-an-executable-archive-with-ant)獲取更多指導。

* Starter POMs

Starter POMs是可以包含到應用中的一個方便的依賴關係描述符集合。你可以獲取所有Spring及相關技術的一站式服務，而不需要翻閱範例代碼，拷貝粘貼大量的依賴描述符。例如，如果你想使用Spring和JPA進行數據庫訪問，隻需要在你的項目中包含`spring-boot-starter-data-jpa`依賴，然後你就可以開始了。

該starters包含很多你搭建項目，快速運行所需的依賴，並提供一致的，管理的傳遞依賴集。

**名字有什麼含義**：所有的starters遵循一個相似的命名模式：`spring-boot-starter-*`，在這裡`*`是一種特殊類型的應用程序。該命名結構旨在幫你找到需要的starter。很多IDEs整合的Maven允許你通過名稱搜索依賴。例如，使用相應的Eclipse或STS插件，你可以簡單地在POM編輯器中點擊`ctrl-space`，然後輸入"spring-boot-starter"可以獲取一個完整列表。

下面的應用程序starters是Spring Boot在`org.springframework.boot`組下提供的：

**表 13.1. Spring Boot application starters** 

|名稱|描述|
|------|:-----|
|spring-boot-starter|核心Spring Boot starter，包括自動配置支援，日誌和YAML|
|spring-boot-starter-actuator|生產準備的特性，用於幫你監控和管理應用|
|spring-boot-starter-amqp|對"高級消息隊列協議"的支援，通過`spring-rabbit`實現|
|spring-boot-starter-aop|對麵向切麵程式的支援，包括`spring-aop`和AspectJ|
|spring-boot-starter-batch|對Spring Batch的支援，包括HSQLDB數據庫|
|spring-boot-starter-cloud-connectors|對Spring Cloud Connectors的支援，簡化在雲平台下（例如，Cloud Foundry 和Heroku）服務的連接|
|spring-boot-starter-data-elasticsearch|對Elasticsearch搜索和分析引擎的支援，包括`spring-data-elasticsearch`|
|spring-boot-starter-data-gemfire|對GemFire分布式數據存儲的支援，包括`spring-data-gemfire`|
|spring-boot-starter-data-jpa|對"Java持久化API"的支援，包括`spring-data-jpa`，`spring-orm`和Hibernate|
|spring-boot-starter-data-mongodb|對MongoDB NOSQL數據庫的支援，包括`spring-data-mongodb`|
|spring-boot-starter-data-rest|對通過REST暴露Spring Data倉庫的支援，通過`spring-data-rest-webmvc`實現|
|spring-boot-starter-data-solr|對Apache Solr搜索平台的支援，包括`spring-data-solr`|
|spring-boot-starter-freemarker|對FreeMarker模板引擎的支援|
|spring-boot-starter-groovy-templates|對Groovy模板引擎的支援|
|spring-boot-starter-hateoas|對基於HATEOAS的RESTful服務的支援，通過`spring-hateoas`實現|
|spring-boot-starter-hornetq|對"Java消息服務API"的支援，通過HornetQ實現|
|spring-boot-starter-integration|對普通`spring-integration`模塊的支援|
|spring-boot-starter-jdbc|對JDBC數據庫的支援|
|spring-boot-starter-jersey|對Jersey RESTful Web服務框架的支援|
|spring-boot-starter-jta-atomikos|對JTA分布式事務的支援，通過Atomikos實現|
|spring-boot-starter-jta-bitronix|對JTA分布式事務的支援，通過Bitronix實現|
|spring-boot-starter-mail|對`javax.mail`的支援|
|spring-boot-starter-mobile|對`spring-mobile`的支援|
|spring-boot-starter-mustache|對Mustache模板引擎的支援|
|spring-boot-starter-redis|對REDIS鍵值數據存儲的支援，包括`spring-redis`|
|spring-boot-starter-security|對`spring-security`的支援|
|spring-boot-starter-social-facebook|對`spring-social-facebook`的支援|
|spring-boot-starter-social-linkedin|對`spring-social-linkedin`的支援|
|spring-boot-starter-social-twitter|對`spring-social-twitter`的支援|
|spring-boot-starter-test|對常用測試依賴的支援，包括JUnit, Hamcrest和Mockito，還有`spring-test`模塊|
|spring-boot-starter-thymeleaf|對Thymeleaf模板引擎的支援，包括和Spring的整合|
|spring-boot-starter-velocity|對Velocity模板引擎的支援|
|spring-boot-starter-web|對全棧web開發的支援，包括Tomcat和`spring-webmvc`|
|spring-boot-starter-websocket|對WebSocket開發的支援|
|spring-boot-starter-ws|對Spring Web服務的支援|

除了應用程序的starters，下面的starters可以用於添加[生產準備](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready)的特性。

**表 13.2. Spring Boot生產準備的starters**

|名稱|描述|
|----|:----|
|spring-boot-starter-actuator|添加生產準備特性，比如指標和監控|
|spring-boot-starter-remote-shell|添加遠程`ssh` shell支援|

最後，Spring Boot包含一些可用於排除或交換具體技術方麵的starters。

**表 13.3. Spring Boot technical starters**

|名稱|描述|
|------|:------|
|spring-boot-starter-jetty|導入Jetty HTTP引擎（作為Tomcat的替代）|
|spring-boot-starter-log4j|對Log4J日誌系統的支援|
|spring-boot-starter-logging|導入Spring Boot的默認日誌系統（Logback）|
|spring-boot-starter-tomcat|導入Spring Boot的默認HTTP引擎（Tomcat）|
|spring-boot-starter-undertow|導入Undertow HTTP引擎（作為Tomcat的替代）|

**注**：查看GitHub上位於`spring-boot-starters`模塊內的[README文件](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/README.adoc)，可以獲取到一個社區貢獻的其他starter POMs列表。

### 組織你的代碼

Spring Boot不需要使用任何特殊的代碼結構，然而，這裡有一些有用的最佳實踐。

* 使用"default"包

當類沒有包含`package`聲明時，它被認為處於`default package`下。通常不推薦使用`default package`，並應該避免使用它。因為對於使用`@ComponentScan`，`@EntityScan`或`@SpringBootApplication`注解的Spring Boot應用來說，來自每個jar的類都會被讀取，這會造成一定的問題。

**注**：我們建議你遵循Java推薦的包命名規範，使用一個反轉的域名（例如`com.example.project`）。

* 定位main應用類

我們通常建議你將main應用類放在位於其他類上麵的根包（root package）中。通常使用`@EnableAutoConfiguration`注解你的main類，並且暗地裡為某些項定義了一個基礎“search package”。例如，如果你正在編寫一個JPA應用，被`@EnableAutoConfiguration`注解的類所在包將被用來搜索`@Entity`項。

使用根包允許你使用`@ComponentScan`注解而不需要定義一個`basePackage`屬性。如果main類位於根包中，你也可以使用`@SpringBootApplication`注解。

下面是一個典型的結構：
```shell
com
 +- example
     +- myproject
         +- Application.java
         |
         +- domain
         |   +- Customer.java
         |   +- CustomerRepository.java
         |
         +- service
         |   +- CustomerService.java
         |
         +- web
             +- CustomerController.java
```
`Application.java`文件將聲明`main`方法，還有基本的`@Configuration`。
```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
### 配置類

Spring Boot提倡基於Java的配置。盡管你可以使用一個XML源來調用`SpringApplication.run()`，我們通常建議你使用`@Configuration`類作為主要源。一般定義`main`方法的類也是主要`@Configuration`的一個很好候選。

**注**：很多使用XML配置的Spring配置範例已經被發布到網絡上。你應該總是盡可能的使用基於Java的配置。搜索查看`enable*`注解就是一個好的開端。

* 導入其他配置類

你不需要將所有的`@Configuration`放進一個單獨的類。`@Import`注解可以用來導入其他配置類。另外，你也可以使用`@ComponentScan`注解自動收集所有的Spring組件，包括`@Configuration`類。

* 導入XML配置

如果你絕對需要使用基於XML的配置，我們建議你仍舊從一個`@Configuration`類開始。你可以使用附加的`@ImportResource`注解加載XML配置文件。

### 自動配置

Spring Boot自動配置（auto-configuration）嘗試根據你添加的jar依賴自動配置你的Spring應用。例如，如果你的classpath下存在`HSQLDB`，並且你沒有手動配置任何數據庫連接beans，那麼我們將自動配置一個內存型（in-memory）數據庫。

你可以通過將`@EnableAutoConfiguration`或`@SpringBootApplication`注解添加到一個`@Configuration`類上來選擇自動配置。

**注**：你隻需要添加一個`@EnableAutoConfiguration`注解。我們建議你將它添加到主`@Configuration`類上。

* 逐步替換自動配置

自動配置是非侵占性的，任何時候你都可以定義自己的配置類來替換自動配置的特定部分。例如，如果你添加自己的`DataSource`  bean，默認的內嵌數據庫支援將不被考慮。

如果需要找出當前應用了哪些自動配置及應用的原因，你可以使用`--debug`開關啟動應用。這將會記錄一個自動配置的報告並輸出到控製台。

* 禁用特定的自動配置

如果發現應用了你不想要的特定自動配置類，你可以使用`@EnableAutoConfiguration`注解的排除屬性來禁用它們。
```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

### Spring Beans和依賴注入

你可以自由地使用任何標準的Spring框架技術去定義beans和它們注入的依賴。簡單起見，我們經常使用`@ComponentScan`注解搜索beans，並結合`@Autowired`構造器注入。

如果使用上麵建議的結構組織代碼（將應用類放到根包下），你可以添加`@ComponentScan`注解而不需要任何參數。你的所有應用程序組件（`@Component`, `@Service`, `@Repository`, `@Controller`等）將被自動注冊為Spring Beans。

下面是一個`@Service` Bean的範例，它使用建構器注入獲取一個需要的`RiskAssessor` bean。
```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...
}
```
**注**：注意如何使用建構器注入來允許`riskAssessor`字段被標記為`final`，這意味著`riskAssessor`後續是不能改變的。

### 使用@SpringBootApplication注解

很多Spring Boot開發者總是使用`@Configuration`，`@EnableAutoConfiguration`和`@ComponentScan`注解他們的main類。由於這些注解被如此頻繁地一塊使用（特別是你遵循以上[最佳實踐](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-structuring-your-code)時），Spring Boot提供一個方便的`@SpringBootApplication`選擇。

該`@SpringBootApplication`注解等價於以默認屬性使用`@Configuration`，`@EnableAutoConfiguration`和`@ComponentScan`。
```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
### 運行應用程序

將應用打包成jar並使用一個內嵌HTTP服務器的一個最大好處是，你可以像其他方式那樣運行你的應用程序。調試Spring Boot應用也很簡單；你不需要任何特殊IDE或擴展。

**注**：本章節隻覆蓋基於jar的打包，如果選擇將應用打包成war文件，你最好參考一下服務器和IDE文件。

* 從IDE中運行

你可以從IDE中運行Spring Boot應用，就像一個簡單的Java應用，但是，你首先需要導入項目。導入步驟跟你的IDE和建構系統有關。大多數IDEs能夠直接導入Maven項目，例如Eclipse用戶可以選擇`File`菜單的`Import…​` --> `Existing Maven Projects`。

如果不能直接將項目導入IDE，你可以需要使用建構系統生成IDE元數據。Maven有針對[Eclipse](http://maven.apache.org/plugins/maven-eclipse-plugin/)和[IDEA](http://maven.apache.org/plugins/maven-idea-plugin/)的插件；Gradle為[各種IDEs](http://www.gradle.org/docs/current/userguide/ide_support.html)提供插件。

**注**：如果意外地運行一個web應用兩次，你將看到一個"端口已在使用中"錯誤。為了確保任何存在的實例是關閉的，STS用戶可以使用`Relaunch`按鈕而不是`Run`按鈕。

* 作為一個打包後的應用運行

如果使用Spring Boot Maven或Gradle插件創建一個可執行jar，你可以使用`java -jar`運行你的應用。例如：
```shell
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar
```
運行一個打包的程序並開啟遠程調試支援是可能的，這允許你將調試器附加到打包的應用程序上：
```shell
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myproject-0.0.1-SNAPSHOT.jar
```
* 使用Maven插件運行

Spring Boot Maven插件包含一個`run`目標，它可以用來快速編譯和運行應用程序。應用程序以一種暴露的方式運行，由於即時"熱"加載，你可以編輯資源。
```shell
$ mvn spring-boot:run
```
你可能想使用有用的操作系統環境變量：
```shell
$ export MAVEN_OPTS=-Xmx1024m -XX:MaxPermSize=128M -Djava.security.egd=file:/dev/./urandom
```
("egd"設置是通過為Tomcat提供一個更快的會話keys熵源來加速Tomcat的。)

* 使用Gradle插件運行

Spring Boot Gradle插件也包含一個`run`目標，它可以用來以暴露的方式運行你的應用程序。不管你什麼時候導入`spring-boot-plugin`，`bootRun`任務總是被添加進去。
```shell
$ gradle bootRun
```
你可能想使用那些有用的操作系統環境變量：
```shell
$ export JAVA_OPTS=-Xmx1024m -XX:MaxPermSize=128M -Djava.security.egd=file:/dev/./urandom
```
* 熱交換

由於Spring Boot應用程序只是普通的Java應用，那JVM熱交換（hot-swapping）應該能出色的工作。JVM熱交換在它能替換的字節碼上有些限製，更全麵的解決方案可以使用[Spring Loaded](https://github.com/spring-projects/spring-loaded)項目或[JRebel](http://zeroturnaround.com/software/jrebel/)。

關於熱交換可以參考[“How-to”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-hotswapping)章節。

### 打包用於生產的應用程序

可執行jars可用於生產部署。由於它們是自包含的，非常適合基於雲的部署。關於其他“生產準備”的特性，比如健康監控，審計和指標REST，或JMX端點，可以考慮添加`spring-boot-actuator`。具體參考[Part V, “Spring Boot Actuator: Production-ready features”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready)。





