### How-to指南

本章節將回答一些常見的"我該怎麼做"類型的問題，這些問題在我們使用Spring Boot時經常遇到。這絕不是一個詳盡的列表，但它覆蓋了很多方麵。

如果遇到一個特殊的我們沒有覆蓋的問題，你可能想去查看[stackoverflow.com](http://stackoverflow.com/tags/spring-boot)，看是否有人已經給出了答案;這也是一個很好的提新問題的地方（請使用`spring-boot`標簽）。

我們也樂意擴展本章節;如果想添加一個'how-to'，你可以給我們發一個[pull請求](http://github.com/spring-projects/spring-boot/tree/master)。

### Spring Boot應用

* 解決自動配置問題

Spring Boot自動配置總是嘗試盡最大努力去做正確的事，但有時候會失敗並且很難說出失敗原因。

在每個Spring Boot `ApplicationContext`中都存在一個相當有用的`ConditionEvaluationReport`。如果開啟`DEBUG`日誌輸出，你將會看到它。如果你使用`spring-boot-actuator`，則會有一個`autoconfig`的端點，它將以JSON形式渲染該報告。可以使用它調試應用程序，並能查看Spring Boot運行時都添加了哪些特性（及哪些沒添加）。

通過查看程式碼和javadoc可以獲取更多問題的答案。以下是一些經驗：

1. 查找名為`*AutoConfiguration`的類並閱讀程式碼，特別是`@Conditional*`註解，這可以幫你找出它們啟用哪些特性及何時啟用。
將`--debug`添加到命令列或添加系統屬性`-Ddebug`可以在控製台查看日誌，該日誌會記錄你的應用中所有自動配置的決策。在一個運行的Actuator app中，通過查看`autoconfig`端點（`/autoconfig`或等效的JMX）可以獲取相同訊息。
2. 查找是`@ConfigurationProperties`的類（比如[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)）並看下有哪些可用的外部配置選項。`@ConfigurationProperties`類有一個用於充當外部配置前綴的`name`屬性，因此`ServerProperties`的值為`prefix="server"`，它的配置屬性有`server.port`，`server.address`等。在運行的Actuator應用中可以查看`configprops`端點。
3. 查看使用`RelaxedEnvironment`明確地將配置從`Environment`曝露出去。它經常會使用一個前綴。
4. 查看`@Value`註解，它直接綁定到`Environment`。相比`RelaxedEnvironment`，這種方式稍微缺乏靈活性，但它也允許鬆散的綁定，特別是OS環境變量（所以`CAPITALS_AND_UNDERSCORES`是`period.separated`的同義詞）。
5. 查看`@ConditionalOnExpression`註解，它根據SpEL表達式的結果來開啟或關閉特性，通常使用解析自`Environment`的佔位符進行計算。

6. 啟動前自定義Environment或ApplicationContext

每個`SpringApplication`都有`ApplicationListeners`和`ApplicationContextInitializers`，用於自定義上下文（context）或環境(environment)。Spring Boot從`META-INF/spring.factories`下加載很多這樣的內部使用的自定義。有很多函式可以註冊其他的自定義：

1. 以程式方式為每個應用註冊自定義，通過在SpringApplication運行前調用它的`addListeners`和`addInitializers`函式來實現。
2. 以聲明方式為每個應用註冊自定義，通過設置`context.initializer.classes`或`context.listener.classes`來實現。
3. 以聲明方式為所有應用註冊自定義，通過添加一個`META-INF/spring.factories`並打包成一個jar文件（該應用將它作為一個函式庫）來實現。

`SpringApplication`會給監聽器（即使是在上下文被建立之前就存在的）發送一些特定的`ApplicationEvents`，然後也會註冊監聽`ApplicationContext`發佈的事件的監聽器。查看Spring Boot特性章節中的[Section 22.4, “Application events and listeners” ](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-application-events-and-listeners)可以獲取一個完整列表。

* 建構ApplicationContext層次結構（添加父或根上下文）

你可以使用`ApplicationBuilder`類建立父/根`ApplicationContext`層次結構。查看'Spring Boot特性'章節的[Section 22.3, “Fluent builder API” ](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-fluent-builder-api)獲取更多訊息。

* 建立一個非web（non-web）應用

不是所有的Spring應用都必須是web應用（或web服務）。如果你想在main函式中執行一些程式碼，但需要啟動一個Spring應用去設置需要的底層設施，那使用Spring Boot的`SpringApplication`特性可以很容易實現。`SpringApplication`會根據它是否需要一個web應用來改變它的`ApplicationContext`類。首先你需要做的是去掉servlet API依賴，如果不能這樣做（比如，基於相同的程式碼運行兩個應用），那你可以明確地調用`SpringApplication.setWebEnvironment(false)`或設置`applicationContextClass`屬性（通過Java API或使用外部配置）。你想運行的，作為業務邏輯的應用程式碼可以實現為一個`CommandLineRunner`，並將上下文降級為一個`@Bean`定義。

### 屬性&配置

* 外部化SpringApplication配置

SpringApplication已經被屬性化（主要是setters），所以你可以在建立應用時使用它的Java API修改它的行為。或者你可以使用properties文件中的`spring.main.*`來外部化（在應用程式碼外配置）這些配置。比如，在`application.properties`中可能會有以下內容：
```java
spring.main.web_environment=false
spring.main.show_banner=false
```
然後Spring Boot在啟動時將不會顯示banner，並且該應用也不是一個web應用。

* 改變應用程序外部配置文件的位置

預設情況下，來自不同源的屬性以一個定義好的順序添加到Spring的`Environment`中（查看'Sprin Boot特性'章節的[Chapter 23, Externalized Configuration](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config)獲取精確的順序）。

為應用程序源添加`@PropertySource`註解是一種很好的添加和修改源順序的函式。傳遞給`SpringApplication`靜態便利設施（convenience）函式的類和使用`setSources()`添加的類都會被檢查，以查看它們是否有`@PropertySources`，如果有，這些屬性會被盡可能早的添加到`Environment`裡，以確保`ApplicationContext`生命周期的所有階段都能使用。以這種方式添加的屬性優先於任何使用預設位置添加的屬性，但低於系統屬性，環境變量或命令列參數。

你也可以提供系統屬性（或環境變量）來改變該行為：

1. `spring.config.name`（`SPRING_CONFIG_NAME`）是根文件名，預設為`application`。
2. `spring.config.location`（`SPRING_CONFIG_LOCATION`）是要加載的文件（例如，一個classpath資源或一個URL）。Spring Boot為該文件設置一個單獨的`Environment`屬性，它可以被系統屬性，環境變量或命令列參數覆蓋。

不管你在environment設置什麼，Spring Boot都將加載上麵討論過的`application.properties`。如果使用YAML，那具有'.yml'擴展的文件預設也會被添加到該列表。

詳情參考[ConfigFileApplicationListener](http://github.com/spring-projects/spring-boot/tree/master/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java)

* 使用'short'命令列參數

有些人喜歡使用（例如）`--port=9000`代替`--server.port=9000`來設置命令列配置屬性。你可以通過在application.properties中使用佔位符來啟用該功能，比如：
```java
server.port=${port:8080}
```
**注**：如果你繼承自`spring-boot-starter-parent` POM，為了防止和Spring-style的佔位符產生衝突，`maven-resources-plugins`預設的過濾令牌（filter token）已經從`${*}`變為`@`（即`@maven.token@`代替了`${maven.token}`）。如果已經直接啟用maven對application.properties的過濾，你可能也想使用[其他的分隔符](http://maven.apache.org/plugins/maven-resources-plugin/resources-mojo.html#delimiters)替換預設的過濾令牌。

**注**：在這種特殊的情況下，端口綁定能夠在一個PaaS環境下工作，比如Heroku和Cloud Foundry，因為在這兩個平台中`PORT`環境變量是自動設置的，並且Spring能夠綁定`Environment`屬性的大寫同義詞。

* 使用YAML配置外部屬性

YAML是JSON的一個超集，可以非常方便的將外部配置以層次結構形式存儲起來。比如：
```json
spring:
    application:
        name: cruncher
    datasource:
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost/test
server:
    port: 9000
```
建立一個application.yml文件，將它放到classpath的根目錄下，並添加snakeyaml依賴（Maven坐標為`org.yaml:snakeyaml`，如果你使用`spring-boot-starter`那就已經被包含了）。一個YAML文件會被解析為一個Java `Map<String,Object>`（和一個JSON對象類似），Spring Boot會平伸該map，這樣它就隻有1級深度，並且有period-separated的keys，跟人們在Java中經常使用的Properties文件非常類似。
上麵的YAML範例對應於下面的application.properties文件：
```java
spring.application.name=cruncher
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost/test
server.port=9000
```
查看'Spring Boot特性'章節的[Section 23.6, “Using YAML instead of Properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-yaml)可以獲取更多關於YAML的訊息。

* 設置生效的Spring profiles

Spring `Environment`有一個API可以設置生效的profiles，但通常你會設置一個系統profile（`spring.profiles.active`）或一個OS環境變量（`SPRING_PROFILES_ACTIVE`）。比如，使用一個`-D`參數啟動應用程序（記著把它放到main類或jar文件之前）：
```shell
$ java -jar -Dspring.profiles.active=production demo-0.0.1-SNAPSHOT.jar
```
在Spring Boot中，你也可以在application.properties裡設置生效的profile，例如：
```java
spring.profiles.active=production
```
通過這種方式設置的值會被系統屬性或環境變量替換，但不會被`SpringApplicationBuilder.profiles()`函式替換。因此，後麵的Java API可用來在不改變預設設置的情況下增加profiles。

想要獲取更多訊息可查看'Spring Boot特性'章節的[Chapter 24, Profiles](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-profiles)。

* 根據環境改變配置

一個YAML文件實際上是一系列以`---`線分割的文件，每個文件都被單獨解析為一個平坦的（flattened）map。

如果一個YAML文件包含一個`spring.profiles`關鍵字，那profiles的值（以逗號分割的profiles列表）將被傳入Spring的`Environment.acceptsProfiles()`函式，並且如果這些profiles的任何一個被啟動，對應的文件被包含到最終的合並中（否則不會）。

範例：
```json
server:
    port: 9000
---

spring:
    profiles: development
server:
    port: 9001

---

spring:
    profiles: production
server:
    port: 0
```
在這個範例中，預設的端口是9000，但如果Spring profile 'development'生效則該端口是9001，如果'production'生效則它是0。

YAML文件以它們遇到的順序合並（所以後麵的值會覆蓋前麵的值）。

想要使用profiles文件完成同樣的操作，你可以使用`application-${profile}.properties`指定特殊的，profile相關的值。

* 發現外部屬性的內置選項

Spring Boot在運行時將來自application.properties（或.yml）的外部屬性綁定進一個應用中。在一個地方不可能存在詳盡的所有支援屬性的列表（技術上也是不可能的），因為你的classpath下的其他jar文件也能夠貢獻。

每個運行中且有Actuator特性的應用都會有一個`configprops`端點，它能夠展示所有邊界和可通過`@ConfigurationProperties`綁定的屬性。

附錄中包含一個[application.properties](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#common-application-properties)範例，它列舉了Spring Boot支援的大多數常用屬性。獲取權威列表可搜索`@ConfigurationProperties`和`@Value`的程式碼，還有不經常使用的`RelaxedEnvironment`。

### 內嵌的servlet容器

* 為應用添加Servlet，Filter或ServletContextListener

Servlet規範支援的Servlet，Filter，ServletContextListener和其他監聽器可以作為`@Bean`定義添加到你的應用中。需要格外小心的是，它們不會引起太多的其他beans的熱初始化，因為在應用生命周期的早期它們已經被安裝到容器裡了（比如，讓它們依賴你的DataSource或JPA配置就不是一個好主意）。你可以通過延遲初始化它們到第一次使用而不是初始化時來突破該限製。

在Filters和Servlets的情況下，你也可以通過添加一個`FilterRegistrationBean`或`ServletRegistrationBean`代替或以及底層的組件來添加映射（mappings）和初始化參數。

* 禁止註冊Servlet或Filter

正如[以上討論](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-add-a-servlet-filter-or-servletcontextlistener)的任何Servlet或Filter beans將被自動註冊到servlet容器中。為了禁止註冊一個特殊的Filter或Servlet bean，可以為它建立一個註冊bean，然後禁用該bean。例如：
```java
@Bean
public FilterRegistrationBean registration(MyFilter filter) {
    FilterRegistrationBean registration = new FilterRegistrationBean(filter);
    registration.setEnabled(false);
    return registration;
}
```

* 改變HTTP端口

在一個單獨的應用中，主HTTP端口預設為8080，但可以使用`server.port`設置（比如，在application.properties中或作為一個系統屬性）。由於`Environment`值的寬鬆綁定，你也可以使用`SERVER_PORT`（比如，作為一個OS環境變）。

為了完全關閉HTTP端點，但仍建立一個WebApplicationContext，你可以設置`server.port=-1`（測試時可能有用）。

想獲取更多詳情可查看'Spring Boot特性'章節的[Section 26.3.3, “Customizing embedded servlet containers”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-customizing-embedded-containers)，或[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)程式碼。

* 使用隨機未分配的HTTP端口

想掃描一個未使用的端口（為了防止衝突使用OS本地端口）可以使用`server.port=0`。

* 發現運行時的HTTP端口

你可以通過日誌輸出或它的EmbeddedServletContainer的EmbeddedWebApplicationContext獲取服務器正在運行的端口。獲取和確認服務器已經初始化的最好方式是添加一個`ApplicationListener<EmbeddedServletContainerInitializedEvent>`類型的`@Bean`，然後當事件發佈時將容器pull出來。

使用`@WebIntegrationTests`的一個有用實踐是設置`server.port=0`，然後使用`@Value`注入實際的（'local'）端口。例如：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = SampleDataJpaApplication.class)
@WebIntegrationTest("server.port:0")
public class CityRepositoryIntegrationTests {

    @Autowired
    EmbeddedWebApplicationContext server;

    @Value("${local.server.port}")
    int port;

    // ...

}
```
* 配置SSL

SSL能夠以聲明方式進行配置，一般通過在application.properties或application.yml設置各種各樣的`server.ssl.*`屬性。例如：
```json
server.port = 8443
server.ssl.key-store = classpath:keystore.jks
server.ssl.key-store-password = secret
server.ssl.key-password = another-secret
```
獲取所有支援的配置詳情可查看[Ssl](http://github.com/spring-projects/spring-boot/tree/master/spring-boot/src/main/java/org/springframework/boot/context/embedded/Ssl.java)。

**注**：Tomcat要求key存儲（如果你正在使用一個可信存儲）能夠直接在文件系統上訪問，即它不能從一個jar文件內讀取。Jetty和Undertow沒有該限製。

使用類似於以上範例的配置意味著該應用將不在支援端口為8080的普通HTTP連接。Spring Boot不支援通過application.properties同時配置HTTP連接器和HTTPS連接器。如果你兩個都想要，那就需要以程式的方式配置它們中的一個。推薦使用application.properties配置HTTPS，因為HTTP連接器是兩個中最容易以程式方式進行配置的。獲取範例可查看[spring-boot-sample-tomcat-multi-connectors](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors)範例項目。

* 配置Tomcat

通常你可以遵循[Section 63.7, “Discover built-in options for external properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-discover-build-in-options-for-external-properties)關於`@ConfigurationProperties`（這裡主要的是`ServerProperties`）的建議，但也看下`EmbeddedServletContainerCustomizer`和各種你可以添加的Tomcat-specific的`*Customizers`。

Tomcat APIs相當豐富，一旦獲取到`TomcatEmbeddedServletContainerFactory`，你就能夠以多種方式修改它。或核心選擇是添加你自己的`TomcatEmbeddedServletContainerFactory`。

* 啟用Tomcat的多連接器（Multiple Connectors）

你可以將一個`org.apache.catalina.connector.Connector`添加到`TomcatEmbeddedServletContainerFactory`，這就能夠允許多連接器，比如HTTP和HTTPS連接器：
```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
    tomcat.addAdditionalTomcatConnectors(createSslConnector());
    return tomcat;
}

private Connector createSslConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
    try {
        File keystore = new ClassPathResource("keystore").getFile();
        File truststore = new ClassPathResource("keystore").getFile();
        connector.setScheme("https");
        connector.setSecure(true);
        connector.setPort(8443);
        protocol.setSSLEnabled(true);
        protocol.setKeystoreFile(keystore.getAbsolutePath());
        protocol.setKeystorePass("changeit");
        protocol.setTruststoreFile(truststore.getAbsolutePath());
        protocol.setTruststorePass("changeit");
        protocol.setKeyAlias("apitester");
        return connector;
    }
    catch (IOException ex) {
        throw new IllegalStateException("can't access keystore: [" + "keystore"
                + "] or truststore: [" + "keystore" + "]", ex);
    }
}
```
* 在前端代理服務器後使用Tomcat

Spring Boot將自動配置Tomcat的`RemoteIpValve`，如果你啟用它的話。這允許你透明地使用標準的`x-forwarded-for`和`x-forwarded-proto`頭，很多前端代理服務器都會添加這些頭訊息（headers）。通過將這些屬性中的一個或全部設置為非空的內容來開啟該功能（它們是大多數代理約定的值，如果你隻設置其中的一個，則另一個也會被自動設置）。
```java
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto
```
如果你的代理使用不同的頭部（headers），你可以通過向application.properties添加一些條目來自定義該值的配置，比如：
```java
server.tomcat.remote_ip_header=x-your-remote-ip-header
server.tomcat.protocol_header=x-your-protocol-header
```
該值也可以配置為一個預設的，能夠匹配信任的內部代理的正則表達式。預設情況下，受信任的IP包括 10/8, 192.168/16, 169.254/16 和 127/8。可以通過向application.properties添加一個條目來自定義該值的配置，比如：
```java
server.tomcat.internal_proxies=192\\.168\\.\\d{1,3}\\.\\d{1,3}
```
**注**：隻有在你使用一個properties文件作為配置的時候才需要雙反斜杠。如果你使用YAML，單個反斜杠就足夠了，`192\.168\.\d{1,3}\.\d{1,3}`和上麵的等價。

另外，通過在一個`TomcatEmbeddedServletContainerFactory` bean中配置和添加`RemoteIpValve`，你就可以完全控製它的設置了。

* 使用Jetty替代Tomcat

Spring Boot starters（特別是spring-boot-starter-web）預設都是使用Tomcat作為內嵌容器的。你需要排除那些Tomcat的依賴並包含Jetty的依賴。為了讓這種處理盡可能簡單，Spring Boot將Tomcat和Jetty的依賴捆綁在一起，然後提供單獨的starters。

Maven範例：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```
Gradle範例：
```gradle
configurations {
    compile.exclude module: "spring-boot-starter-tomcat"
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.3.0.BUILD-SNAPSHOT")
    compile("org.springframework.boot:spring-boot-starter-jetty:1.3.0.BUILD-SNAPSHOT")
    // ...
}
```
* 配置Jetty

通常你可以遵循[Section 63.7, “Discover built-in options for external properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-discover-build-in-options-for-external-properties)關於`@ConfigurationProperties`（此處主要是ServerProperties）的建議，但也要看下`EmbeddedServletContainerCustomizer`。Jetty API相當豐富，一旦獲取到`JettyEmbeddedServletContainerFactory`，你就可以使用很多方式修改它。或更徹底地就是添加你自己的`JettyEmbeddedServletContainerFactory`。

* 使用Undertow替代Tomcat

使用Undertow替代Tomcat和[使用Jetty替代Tomcat](https://github.com/qibaoguang/Spring-Boot-Reference-Guide/edit/master/How-to_%20guides.md)非常類似。你需要排除Tomat依賴，並包含Undertow starter。

Maven範例：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```
Gradle範例：
```gradle
configurations {
    compile.exclude module: "spring-boot-starter-tomcat"
}

dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web:1.3.0.BUILD-SNAPSHOT")
    compile 'org.springframework.boot:spring-boot-starter-undertow:1.3.0.BUILD-SNAPSHOT")
    // ...
}

```
* 配置Undertow

通常你可以遵循[Section 63.7, “Discover built-in options for external properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-discover-build-in-options-for-external-properties)關於`@ConfigurationProperties`（此處主要是ServerProperties和ServerProperties.Undertow），但也要看下`EmbeddedServletContainerCustomizer`。一旦獲取到`UndertowEmbeddedServletContainerFactory`，你就可以使用一個`UndertowBuilderCustomizer`修改Undertow的配置以滿足你的需求。或更徹底地就是添加你自己的`UndertowEmbeddedServletContainerFactory`。

* 啟用Undertow的多監聽器（Multiple Listeners）

往`UndertowEmbeddedServletContainerFactory`添加一個`UndertowBuilderCustomizer`，然後添加一個監聽者到`Builder`：
```java
@Bean
public UndertowEmbeddedServletContainerFactory embeddedServletContainerFactory() {
    UndertowEmbeddedServletContainerFactory factory = new UndertowEmbeddedServletContainerFactory();
    factory.addBuilderCustomizers(new UndertowBuilderCustomizer() {

        @Override
        public void customize(Builder builder) {
            builder.addHttpListener(8080, "0.0.0.0");
        }

    });
    return factory;
}
```
* 使用Tomcat7

Tomcat7可用於Spring Boot，但預設使用的是Tomcat8。如果不能使用Tomcat8（例如，你使用的是Java1.6），你需要改變classpath去引用Tomcat7。

- 通過Maven使用Tomcat7

如果正在使用starter pom和parent，你只需要改變Tomcat的version屬性，比如，對於一個簡單的webapp或service：
```xml
<properties>
    <tomcat.version>7.0.59</tomcat.version>
</properties>
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    ...
</dependencies>
```
- 通過Gradle使用Tomcat7

你可以通過設置`tomcat.version`屬性改變Tomcat的版本：
```gradle
ext['tomcat.version'] = '7.0.59'
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web'
}
```
* 使用Jetty8

Jetty8可用於Spring Boot，但預設使用的是Jetty9。如果不能使用Jetty9（例如，因為你使用的是Java1.6），你只需改變classpath去引用Jetty8。你也需要排除Jetty的WebSocket相關的依賴。

- 通過Maven使用Jetty8

如果正在使用starter pom和parent，你只需添加Jetty starter，去掉WebSocket依賴，並改變version屬性，比如，對於一個簡單的webapp或service：
```xml
<properties>
    <jetty.version>8.1.15.v20140411</jetty.version>
    <jetty-jsp.version>2.2.0.v201112011158</jetty-jsp.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.eclipse.jetty.websocket</groupId>
                <artifactId>*</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```
- 通過Gradle使用Jetty8

你可以設置`jetty.version`屬性並排除相關的WebSocket依賴，比如對於一個簡單的webapp或service：
```gradle
ext['jetty.version'] = '8.1.15.v20140411'
dependencies {
    compile ('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    compile ('org.springframework.boot:spring-boot-starter-jetty') {
        exclude group: 'org.eclipse.jetty.websocket'
    }
}
```
* 使用@ServerEndpoint建立WebSocket端點

如果想在一個使用內嵌容器的Spring Boot應用中使用@ServerEndpoint，你需要聲明一個單獨的ServerEndpointExporter @Bean：
```java
@Bean
public ServerEndpointExporter serverEndpointExporter() {
    return new ServerEndpointExporter();
}
```
該bean將用底層的WebSocket容器註冊任何的被`@ServerEndpoint`註解的beans。當部署到一個單獨的servlet容器時，該角色將被一個servlet容器初始化函式履行，ServerEndpointExporter bean也就不是必需的了。

* 啟用HTTP響應壓縮

Spring Boot提供兩種啟用HTTP壓縮的機製;一種是Tomcat特有的，另一種是使用一個filter，可以配合Jetty，Tomcat和Undertow。

- 啟用Tomcat的HTTP響應壓縮

Tomcat對HTTP響應壓縮提供內建支援。預設是禁用的，但可以通過application.properties輕鬆的啟用：
```java
server.tomcat.compression: on
```
當設置為`on`時，Tomcat將壓縮響應的長度至少為2048字節。你可以配置一個整型值來設置該限製而不只是`on`，比如：
```java
server.tomcat.compression: 4096
```
預設情況下，Tomcat隻壓縮某些MIME類型的響應（text/html，text/xml和text/plain）。你可以使用`server.tomcat.compressableMimeTypes`屬性進行自定義，比如：
```java
server.tomcat.compressableMimeTypes=application/json,application/xml
```
- 使用GzipFilter開啟HTTP響應壓縮

如果你正在使用Jetty或Undertow，或想要更精確的控製HTTP響應壓縮，Spring Boot為Jetty的GzipFilter提供自動配置。雖然該過濾器是Jetty的一部分，但它也兼容Tomcat和Undertow。想要啟用該過濾器，只需簡單的為你的應用添加`org.eclipse.jetty:jetty-servlets`依賴。

GzipFilter可以使用`spring.http.gzip.*`屬性進行配置。具體參考[GzipFilterProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/GzipFilterProperties.java)。

### Spring MVC

* 編寫一個JSON REST服務

在Spring Boot應用中，任何Spring `@RestController`預設應該渲染為JSON響應，隻要classpath下存在Jackson2。例如：
```java
@RestController
public class MyController {

    @RequestMapping("/thing")
    public MyThing thing() {
            return new MyThing();
    }

}
```
隻要MyThing能夠通過Jackson2序列化（比如，一個標準的POJO或Groovy對象），[localhost:8080/thing](http://localhost:8080/thing)預設響應一個JSON表示。有時在一個瀏覽器中你可能看到XML響應因為瀏覽器傾向於發送XML
響應頭。

* 編寫一個XML REST服務

如果classpath下存在Jackson XML擴展（jackson-dataformat-xml），它會被用來渲染XML響應，範例和JSON的非常相似。想要使用它，只需為你的項目添加以下的依賴：
```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```
你可能也想添加對Woodstox的依賴。它比JDK提供的預設Stax實現快很多，並且支援良好的格式化輸出，提高了namespace處理能力：
```xml
<dependency>
    <groupId>org.codehaus.woodstox</groupId>
    <artifactId>woodstox-core-asl</artifactId>
</dependency>
```
如果Jackson的XML擴展不可用，Spring Boot將使用JAXB（JDK預設提供），不過你需要為MyThing添加額外的註解`@XmlRootElement`：
```java
@XmlRootElement
public class MyThing {
    private String name;
    // .. getters and setters
}
```
想要服務器渲染XML而不是JSON，你可能需要發送一個`Accept: text/xml`頭部（或使用瀏覽器）。

* 自定義Jackson ObjectMapper

在一個HTTP交互中，Spring MVC（客戶端和服務端）使用HttpMessageConverters協商內容轉換。如果classpath下存在Jackson，你就已經獲取到Jackson2ObjectMapperBuilder提供的預設轉換器。

建立的ObjectMapper（或用於Jackson XML轉換的XmlMapper）實例預設有以下自定義屬性：

- `MapperFeature.DEFAULT_VIEW_INCLUSION`禁用
- `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`禁用

Spring Boot也有一些簡化自定義該行為的特性。

你可以使用當前的environment配置ObjectMapper和XmlMapper實例。Jackson提供一個擴展套件，可以用來簡單的關閉或開啟一些特性，你可以用它們配置Jackson處理的不同方麵。這些特性在Jackson中使用5個枚舉進行描述的，並被映射到environment的屬性上：

|Jackson枚舉|Environment屬性|
|------|:-------|
|`com.fasterxml.jackson.databind.DeserializationFeature`|`spring.jackson.deserialization.<feature_name>=true|false`|
|`com.fasterxml.jackson.core.JsonGenerator.Feature`|`spring.jackson.generator.<feature_name>=true|false`|
|`com.fasterxml.jackson.databind.MapperFeature`|`spring.jackson.mapper.<feature_name>=true|false`|
|`com.fasterxml.jackson.core.JsonParser.Feature`|`spring.jackson.parser.<feature_name>=true|false`|
|`com.fasterxml.jackson.databind.SerializationFeature`|`spring.jackson.serialization.<feature_name>=true|false`|

例如，設置`spring.jackson.serialization.indent_output=true`可以開啟漂亮列印。注意，由於[鬆綁定](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-relaxed-binding)的使用，`indent_output`不必匹配對應的枚舉常量`INDENT_OUTPUT`。

如果想徹底替換預設的ObjectMapper，你需要定義一個該類型的`@Bean`並將它標記為`@Primary`。

定義一個Jackson2ObjectMapperBuilder類型的`@Bean`將允許你自定義預設的ObjectMapper和XmlMapper（分別用於MappingJackson2HttpMessageConverter和MappingJackson2XmlHttpMessageConverter）。

另一種自定義Jackson的函式是向你的上下文添加`com.fasterxml.jackson.databind.Module`類型的beans。它們會被註冊入每個ObjectMapper類型的bean，當為你的應用添加新特性時，這就提供了一種全局機製來貢獻自定義模塊。

最後，如果你提供任何MappingJackson2HttpMessageConverter類型的`@Beans`，那它們將替換MVC配置中的預設值。同時，也提供一個HttpMessageConverters類型的bean，它有一些有用的函式可以獲取預設的和用戶增強的message轉換器。

想要獲取更多細節可查看[Section 65.4, “Customize the @ResponseBody rendering”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-customize-the-responsebody-rendering)和[WebMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)程式碼。

* 自定義@ResponseBody渲染

Spring使用HttpMessageConverters渲染`@ResponseBody`（或來自`@RestController`的響應）。你可以通過在Spring Boot上下文中添加該類型的beans來貢獻其他的轉換器。如果你添加的bean類型預設已經包含了（像用於JSON轉換的MappingJackson2HttpMessageConverter），那它將替換預設的。Spring Boot提供一個方便的HttpMessageConverters類型的bean，它有一些有用的函式可以訪問預設的和用戶增強的message轉換器（有用，比如你想要手動將它們注入到一個自定義的`RestTemplate`）。

在通常的MVC用例中，任何你提供的WebMvcConfigurerAdapter beans通過覆蓋configureMessageConverters函式也能貢獻轉換器，但不同於通常的MVC，你可以隻提供你需要的轉換器（因為Spring Boot使用相同的機製來貢獻它預設的轉換器）。最終，如果你通過提供自己的` @EnableWebMvc`註解覆蓋Spring Boot預設的MVC配置，那你就可以完全控製，並使用來自WebMvcConfigurationSupport的getMessageConverters手動做任何事。

具體參考[WebMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)程式碼。

* 處理Multipart文件上傳

Spring Boot採用Servlet 3 `javax.servlet.http.Part` API來支援文件上傳。預設情況下，Spring Boot配置Spring MVC在單個請求中每個文件最大1Mb，最多10Mb的文件數據。你可以覆蓋那些值，也可以設置臨時文件存儲的位置（比如，存儲到`/tmp`文件夾下）及傳遞數據刷新到磁盤的閥值（通過使用MultipartProperties類曝露的屬性）。如果你需要設置文件不受限製，例如，可以設置`multipart.maxFileSize`屬性值為`-1`。

當你想要接收部分（multipart）編碼文件數據作為Spring MVC控製器（controller）處理函式中被`@RequestParam`註解的MultipartFile類型的參數時，multipart支援就非常有用了。

具體參考[MultipartAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/MultipartAutoConfiguration.java)程式碼。

* 關閉Spring MVC DispatcherServlet

Spring Boot想要服務來自應用程序root `/`下的所有內容。如果你想將自己的servlet映射到該目錄下也是可以的，但當然你可能失去一些Boot MVC特性。為了添加你自己的servlet，並將它映射到root資源，你只需聲明一個Servlet類型的`@Bean`，並給它特定的bean名稱`dispatcherServlet`（如果隻想關閉但不替換它，你可以使用該名稱建立不同類型的bean）。

* 關閉預設的MVC配置

完全控製MVC配置的最簡單方式是提供你自己的被`@EnableWebMvc`註解的`@Configuration`。這樣所有的MVC配置都逃不出你的掌心。

* 自定義ViewResolvers

ViewResolver是Spring MVC的核心組件，它負責轉換`@Controller`中的視圖名稱到實際的View實現。注意ViewResolvers主要用在UI應用中，而不是REST風格的服務（View不是用來渲染`@ResponseBody`的）。Spring有很多你可以選擇的ViewResolver實現，並且Spring自己對如何選擇相應實現也沒發表意見。另一方麵，Spring Boot會根據classpath上的依賴和應用上下文為你安裝一或兩個ViewResolver實現。DispatcherServlet使用所有在應用上下文中找到的解析器（resolvers），並依次嘗試每一個直到它獲取到結果，所以如果你正在添加自己的解析器，那就要小心順序和你的解析器添加的位置。

WebMvcAutoConfiguration將會為你的上下文添加以下ViewResolvers：

- bean id為`defaultViewResolver`的InternalResourceViewResolver。這個會定位可以使用DefaultServlet渲染的物理資源（比如，靜態資源和JSP頁面）。它在視圖（view name）上應用了一個前綴和後綴（預設都為空，但你可以通過`spring.view.prefix`和`spring.view.suffix`外部配置設置），然後查找在servlet上下文中具有該路徑的物理資源。可以通過提供相同類型的bean覆蓋它。
- id為`beanNameViewResolver`的BeanNameViewResolver。這是視圖解析器鏈的一個非常有用的成員，它可以在View被解析時收集任何具有相同名稱的beans。
- id為`viewResolver`的ContentNegotiatingViewResolver隻會在實際View類型的beans出現時添加。這是一個'主'解析器，它的職責會代理給其他解析器，它會嘗試找到客戶端發送的一個匹配'Accept'的HTTP頭部。這有一篇有用的，關於你需要更多了解的[ContentNegotiatingViewResolver](https://spring.io/blog/2013/06/03/content-negotiation-using-views)的博客，也要具體查看下程式碼。通過定義一個名叫'viewResolver'的bean，你可以關閉自動配置的ContentNegotiatingViewResolver。
- 如果使用Thymeleaf，你將有一個id為`thymeleafViewResolver`的ThymeleafViewResolver。它會通過加前綴和後綴的視圖名來查找資源（外部配置為`spring.thymeleaf.prefix`和`spring.thymeleaf.suffix`，對應的預設為'classpath:/templates/'和'.html'）。你可以通過提供相同名稱的bean來覆蓋它。
- 如果使用FreeMarker，你將有一個id為`freeMarkerViewResolver`的FreeMarkerViewResolver。它會使用加前綴和後綴（外部配置為`spring.freemarker.prefix`和`spring.freemarker.suffix`，對應的預設值為空和'.ftl'）的視圖名從加載路徑（外部配置為`spring.freemarker.templateLoaderPath`，預設為'classpath:/templates/'）下查找資源。你可以通過提供一個相同名稱的bean來覆蓋它。
- 如果使用Groovy模板（實際上隻要你把groovy-templates添加到classpath下），你將有一個id為`groovyTemplateViewResolver`的Groovy TemplateViewResolver。它會使用加前綴和後綴（外部屬性為`spring.groovy.template.prefix`和`spring.groovy.template.suffix`，對應的預設值為'classpath:/templates/'和'.tpl'）的視圖名從加載路徑下查找資源。你可以通過提供一個相同名稱的bean來覆蓋它。
- 如果使用Velocity，你將有一個id為`velocityViewResolver`的VelocityViewResolver。它會使用加前綴和後綴（外部屬性為`spring.velocity.prefix`和`spring.velocity.suffix`，對應的預設值為空和'.vm'）的視圖名從加載路徑（外部屬性為`spring.velocity.resourceLoaderPath`，預設為'classpath:/templates/'）下查找資源。你可以通過提供一個相同名稱的bean來覆蓋它。

具體參考：  [WebMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)，[ThymeleafAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)，[FreeMarkerAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)，[GroovyTemplateAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)，[VelocityAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)。

### 日誌

Spring Boot除了commons-logging  API外沒有其他強製性的日誌依賴，你有很多可選的日誌實現。想要使用[Logback](http://logback.qos.ch/)，你需要包含它，及一些對classpath下commons-logging的綁定。最簡單的方式是通過依賴`spring-boot-starter-logging`的starter pom。對於一個web應用程序，你只需添加`spring-boot-starter-web`依賴，因為它依賴於logging starter。例如，使用Maven：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
Spring Boot有一個LoggingSystem抽象，用於嘗試通過classpath上下文配置日誌系統。如果Logback可用，則首選它。如果你唯一需要做的就是設置不同日誌的級別，那可以通過在application.properties中使用`logging.level`前綴實現，比如：
```java
logging.level.org.springframework.web: DEBUG
logging.level.org.hibernate: ERROR
```
你也可以使用`logging.file`設置日誌文件的位置（除控製台之外，預設會輸出到控製台）。

想要對日誌系統進行更細粒度的配置，你需要使用正在說的LoggingSystem支援的原生配置格式。預設情況下，Spring Boot從系統的預設位置加載原生配置（比如對於Logback為`classpath:logback.xml`），但你可以使用`logging.config`屬性設置配置文件的位置。

* 配置Logback

如果你將一個logback.xml放到classpath根目錄下，那它將會被從這加載。Spring Boot提供一個預設的基本配置，如果你只是設置日誌級別，那你可以包含它，比如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <logger name="org.springframework.web" level="DEBUG"/>
</configuration>
```
如果查看spring-boot jar包中的預設logback.xml，你將會看到LoggingSystem為你建立的很多有用的系統屬性，比如：
- ${PID}，當前程序id
- ${LOG_FILE}，如果在Boot外部配置中設置了`logging.file`
- ${LOG_PATH}，如果設置了`logging.path`（表示日誌文件產生的目錄）

Spring Boot也提供使用自定義的Logback轉換器在控製台上輸出一些漂亮的彩色ANSI日誌訊息（不是日誌文件）。具體參考預設的`base.xml`配置。

如果Groovy在classpath下，你也可以使用logback.groovy配置Logback。

* 配置Log4j

Spring Boot也支援[Log4j](http://logging.apache.org/log4j/1.2)或[Log4j 2](http://logging.apache.org/log4j/2.x)作為日誌配置，但隻有在它們中的某個在classpath下存在的情況。如果你正在使用starter poms進行依賴裝配，這意味著你需要排除Logback，然後包含你選擇的Log4j版本。如果你不使用starter poms，那除了你選擇的Log4j版本外還要提供commons-logging（至少）。

最簡單的方式可能就是通過starter poms，儘管它需要排除一些依賴，比如，在Maven中：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j</artifactId>
</dependency>
```
想要使用Log4j 2，只需要依賴`spring-boot-starter-log4j2`而不是`spring-boot-starter-log4j`。

**注**：使用Log4j各版本的starters都會收集好依賴以滿足common logging的要求（比如，Tomcat中使用`java.util.logging`，但使用Log4j或 Log4j 2作為輸出）。具體查看Actuator Log4j或Log4j 2的範例，了解如何將它用於實戰。

* 使用YAML或JSON配置Log4j2

除了它的預設XML配置格式，Log4j 2也支援YAML和JSON配置文件。想要使用其他配置文件格式來配置Log4j 2，你需要添加合適的依賴到classpath。為了使用YAML，你需要添加`com.fasterxml.jackson.dataformat:jackson-dataformat-yaml`依賴，Log4j 2將查找名稱為`log4j2.yaml`或`log4j2.yml`的配置文件。為了使用JSON，你需要添加`com.fasterxml.jackson.core:jackson-databind`依賴，Log4j 2將查找名稱為`log4j2.json`或`log4j2.jsn`的配置文件

### 數據訪問

* 配置一個數據源

想要覆蓋預設的設置只需要定義一個你自己的DataSource類型的`@Bean`。Spring Boot提供一個工具建構類DataSourceBuilder，可用來建立一個標準的DataSource（如果它處於classpath下），或者僅建立你自己的DataSource，然後將它和在[Section 23.7.1, “Third-party configuration”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-3rd-party-configuration)解釋的一系列Environment屬性綁定。

比如：
```java
@Bean
@ConfigurationProperties(prefix="datasource.mine")
public DataSource dataSource() {
    return new FancyDataSource();
}
```
```java
datasource.mine.jdbcUrl=jdbc:h2:mem:mydb
datasource.mine.user=sa
datasource.mine.poolSize=30
```
具體參考'Spring Boot特性'章節中的[Section 28.1, “Configure a DataSource”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-configure-datasource)和[DataSourceAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.java)類程式碼。

* 配置兩個數據源

建立多個數據源和建立第一個工作都是一樣的。如果使用針對JDBC或JPA的預設自動配置，你可能想要將其中一個設置為`@Primary`（然後它就能被任何`@Autowired`注入獲取）。
```java
@Bean
@Primary
@ConfigurationProperties(prefix="datasource.primary")
public DataSource primaryDataSource() {
    return DataSourceBuilder.create().build();
}

@Bean
@ConfigurationProperties(prefix="datasource.secondary")
public DataSource secondaryDataSource() {
    return DataSourceBuilder.create().build();
}
```
* 使用Spring Data倉函式庫

Spring Data可以為你的`@Repository`接口建立各種風格的實現。Spring Boot會為你處理所有事情，隻要那些`@Repositories`接口跟你的`@EnableAutoConfiguration`類處於相同的包（或子包）。

對於很多應用來說，你需要做的就是將正確的Spring Data依賴添加到classpath下（對於JPA有一個`spring-boot-starter-data-jpa`，對於Mongodb有一個`spring-boot-starter-data-mongodb`），建立一些repository接口來處理`@Entity`對象。具體參考[JPA sample](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-data-jpa)或[Mongodb sample](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-data-mongodb)。

Spring Boot會基於它找到的`@EnableAutoConfiguration`來嘗試猜測你的`@Repository`定義的位置。想要獲取更多控製，可以使用`@EnableJpaRepositories`註解（來自Spring Data JPA）。

* 從Spring配置分離`@Entity`定義

Spring Boot會基於它找到的`@EnableAutoConfiguration`來嘗試猜測你的`@Entity`定義的位置。想要獲取更多控製，你可以使用`@EntityScan`註解，比如：
```java
@Configuration
@EnableAutoConfiguration
@EntityScan(basePackageClasses=City.class)
public class Application {

    //...

}
```
* 配置JPA屬性

Spring Data JPA已經提供了一些獨立的配置選項（比如，針對SQL日誌），並且Spring Boot會曝露它們，針對hibernate的外部配置屬性也更多些。最常見的選項如下：
```java
spring.jpa.hibernate.ddl-auto: create-drop
spring.jpa.hibernate.naming_strategy: org.hibernate.cfg.ImprovedNamingStrategy
spring.jpa.database: H2
spring.jpa.show-sql: true
```
（由於寬鬆的數據綁定策略，連字符或下劃線作為屬性keys作用應該是等效的）`ddl-auto`配置是個特殊情況，它有不同的預設設置，這取決於你是否使用一個內嵌數據函式庫（create-drop）。當本地EntityManagerFactory被建立時，所有`spring.jpa.properties.*`屬性都被作為正常的JPA屬性（去掉前綴）傳遞進去了。

具體參考[HibernateJpaAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.java)和[JpaBaseConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)。

* 使用自定義的EntityManagerFactory

為了完全控製EntityManagerFactory的配置，你需要添加一個名為`entityManagerFactory`的`@Bean`。Spring Boot自動配置會根據是否存在該類型的bean來關閉它的實體管理器（entity manager）。

* 使用兩個EntityManagers

即使預設的EntityManagerFactory工作的很好，你也需要定義一個新的EntityManagerFactory，因為一旦出現第二個該類型的bean，預設的將會被關閉。為了輕鬆的實現該操作，你可以使用Spring Boot提供的EntityManagerBuilder，或者如果你喜歡的話可以直接使用來自Spring ORM的LocalContainerEntityManagerFactoryBean。

範例：
```java
// add two data sources configured as above

@Bean
public LocalContainerEntityManagerFactoryBean customerEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(customerDataSource())
            .packages(Customer.class)
            .persistenceUnit("customers")
            .build();
}

@Bean
public LocalContainerEntityManagerFactoryBean orderEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(orderDataSource())
            .packages(Order.class)
            .persistenceUnit("orders")
            .build();
}
```
上麵的配置靠自己基本可以運行。想要完成作品你也需要為兩個EntityManagers配置TransactionManagers。其中的一個會被Spring Boot預設的JpaTransactionManager獲取，如果你將它標記為`@Primary`。另一個需要顯式注入到一個新實例。或你可以使用一個JTA事物管理器生成它兩個。

* 使用普通的persistence.xml

Spring不要求使用XML配置JPA提供者（provider），並且Spring Boot假定你想要充分利用該特性。如果你傾向於使用`persistence.xml`，那你需要定義你自己的id為'entityManagerFactory'的LocalEntityManagerFactoryBean類型的`@Bean`，並在那設置持久化單元的名稱。

預設設置可查看[JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)

* 使用Spring Data JPA和Mongo倉函式庫

Spring Data JPA和Spring Data Mongo都能自動為你建立Repository實現。如果它們同時出現在classpath下，你可能需要添加額外的配置來告訴Spring Boot你想要哪個（或兩個）為你建立倉函式庫。最明確地方式是使用標準的Spring Data `@Enable*Repositories`，然後告訴它你的Repository接口的位置（此處*即可以是Jpa，也可以是Mongo，或者兩者都是）。

這裡也有`spring.data.*.repositories.enabled`標誌，可用來在外部配置中開啟或關閉倉函式庫的自動配置。這在你想關閉Mongo倉函式庫，但仍舊使用自動配置的MongoTemplate時非常有用。

相同的障礙和特性也存在於其他自動配置的Spring Data倉函式庫類型（Elasticsearch, Solr）。只需要改變對應註解的名稱和標誌。

* 將Spring Data倉函式庫曝露為REST端點

Spring Data REST能夠將Repository的實現曝露為REST端點，隻要該應用啟用Spring MVC。

Spring Boot曝露一系列來自`spring.data.rest`命名空間的有用屬性來定製化[RepositoryRestConfiguration](http://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/core/config/RepositoryRestConfiguration.html)。如果需要提供其他定製，你可以建立一個繼承自SpringBootRepositoryRestMvcConfiguration的`@Configuration`類。該類功能和RepositoryRestMvcConfiguration相同，但允許你繼續使用`spring.data.rest.*`屬性。

### 數據函式庫初始化

一個數據函式庫可以使用不同的方式進行初始化，這取決於你的技術棧。或者你可以手動完成該任務，隻要數據函式庫是單獨的過程。

* 使用JPA初始化數據函式庫

JPA有個生成DDL的特性，這些可以設置為在數據函式庫啟動時運行。這可以通過兩個外部屬性進行控製：

- `spring.jpa.generate-ddl`（boolean）控製該特性的關閉和開啟，跟實現者沒關係
- `spring.jpa.hibernate.ddl-auto`（enum）是一個Hibernate特性，用於更細力度的控製該行為。更多詳情參考以下內容。

* 使用Hibernate初始化數據函式庫

你可以顯式設置`spring.jpa.hibernate.ddl-auto`，標準的Hibernate屬性值有`none`，`validate`，`update`，`create`，`create-drop`。Spring Boot根據你的數據函式庫是否為內嵌數據函式庫來選擇相應的預設值，如果是內嵌型的則預設值為`create-drop`，否則為`none`。通過查看Connection類型可以檢查是否為內嵌型數據函式庫，hsqldb，h2和derby是內嵌的，其他都不是。當從內存數據函式庫遷移到一個真正的數據函式庫時，你需要當心，在新的平台中不能對數據函式庫表和數據是否存在進行臆斷。你也需要顯式設置`ddl-auto`，或使用其他機製初始化數據函式庫。

此外，啟動時處於classpath根目錄下的import.sql文件會被執行。這在demos或測試時很有用，但在生產環境中你可能不期望這樣。這是Hibernate的特性，和Spring沒有一點關係。

* 使用Spring JDBC初始化數據函式庫

Spring JDBC有一個DataSource初始化特性。Spring Boot預設啟用了該特性，並從標準的位置schema.sql和data.sql（位於classpath根目錄）加載SQL。此外，Spring Boot將加載`schema-${platform}.sql`和`data-${platform}.sql`文件（如果存在），在這裡platform是`spring.datasource.platform`的值，比如，你可以將它設置為數據函式庫的供應商名稱（hsqldb, h2, oracle, mysql, postgresql等）。Spring Boot預設啟用Spring JDBC初始化快速失敗特性，所以如果腳本導致異常產生，那應用程序將啟動失敗。腳本的位置可以通過設置`spring.datasource.schema`和`spring.datasource.data`來改變，如果設置`spring.datasource.initialize=false`則哪個位置都不會被處理。

你可以設置`spring.datasource.continueOnError=true`禁用快速失敗特性。一旦應用程序成熟並被部署了很多次，那該設置就很有用，因為腳本可以充當"可憐人的遷移"-例如，插入失敗時意味著數據已經存在，也就沒必要阻止應用繼續運行。

如果你想要在一個JPA應用中使用schema.sql，那如果Hibernate試圖建立相同的表，`ddl-auto=create-drop`將導致錯誤產生。為了避免那些錯誤，可以將`ddl-auto`設置為“”（推薦）或“none”。不管是否使用`ddl-auto=create-drop`，你總可以使用data.sql初始化新數據。

* 初始化Spring Batch數據函式庫

如果你正在使用Spring Batch，那麼它會為大多數的流行數據函式庫平台預裝SQL初始化腳本。Spring Boot會檢測你的數據函式庫類型，並預設執行那些腳本，在這種情況下將關閉快速失敗特性（錯誤被記錄但不會阻止應用啟動）。這是因為那些腳本是可信任的，通常不會包含bugs，所以錯誤會被忽略掉，並且對錯誤的忽略可以讓腳本具有冪等性。你可以使用`spring.batch.initializer.enabled=false`顯式關閉初始化功能。

* 使用一個進階別的數據遷移工具

Spring Boot跟進階別的數據遷移工具[Flyway](http://flywaydb.org/)(基於SQL)和[Liquibase](http://www.liquibase.org/)(XML)工作的很好。通常我們傾向於Flyway，因為它一眼看去好像很容易，另外它通常不需要平台獨立：一般一個或至多需要兩個平台。

- 啟動時執行Flyway數據函式庫遷移

想要在啟動時自動運行Flyway數據函式庫遷移，需要將`org.flywaydb:flyway-core`添加到你的classpath下。

遷移是一些`V<VERSION>__<NAME>.sql`格式的腳本（`<VERSION>`是一個下劃線分割的版本號，比如'1'或'2_1'）。預設情況下，它們存放在一個`classpath:db/migration`的文件夾中，但你可以使用`flyway.locations`（一個列表）來改變它。詳情可參考flyway-core中的Flyway類，查看一些可用的配置，比如schemas。Spring Boot在[FlywayProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayProperties.java)中提供了一個小的屬性集，可用於禁止遷移，或關閉位置檢測。

預設情況下，Flyway將自動注入（`@Primary`）DataSource到你的上下文，並用它進行數據遷移。如果你想使用一個不同的DataSource，你可以建立一個，並將它標記為`@FlywayDataSource`的`@Bean`-如果你這樣做了，且想要兩個數據源，記得建立另一個並將它標記為`@Primary`。或者你可以通過在外部配置文件中設置`flyway.[url,user,password]`來使用Flyway的原生DataSource。

這是一個[Flyway範例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-flyway)，你可以作為參考。

- 啟動時執行Liquibase數據函式庫遷移

想要在啟動時自動運行Liquibase數據函式庫遷移，你需要將`org.liquibase:liquibase-core`添加到classpath下。

主改變日誌（master change log）預設從`db/changelog/db.changelog-master.yaml`讀取，但你可以使用`liquibase.change-log`進行設置。詳情查看[LiquibaseProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/liquibase/LiquibaseProperties.java)以獲取可用設置，比如上下文，預設的schema等。

這裡有個[Liquibase範例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-liquibase)可作為參考。

### 批處理應用

* 在啟動時執行Spring Batch作業

你可以在上下文的某個地方添加`@EnableBatchProcessing`來啟用Spring Batch的自動配置功能。

預設情況下，在啟動時它會執行應用的所有作業（Jobs），具體查看[JobLauncherCommandLineRunner](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/JobLauncherCommandLineRunner.java)。你可以通過指定`spring.batch.job.names`（多個作業名以逗號分割）來縮小到一個特定的作業或多個作業。

如果應用上下文包含一個JobRegistry，那麼處於`spring.batch.job.names`中的作業將會從registry中查找，而不是從上下文中自動裝配。這是複雜系統中常見的一個模式，在這些系統中多個作業被定義在子上下文和註冊中心。

具體參考[BatchAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java)和[@EnableBatchProcessing](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.java)。

### 執行器（Actuator）

* 改變HTTP端口或執行器端點的地址

在一個單獨的應用中，執行器的HTTP端口預設和主HTTP端口相同。想要讓應用監聽不同的端口，你可以設置外部屬性`management.port`。為了監聽一個完全不同的網絡地址（比如，你有一個用於管理的內部網絡和一個用於用戶應用程序的外部網絡），你可以將`management.address`設置為一個可用的IP地址，然後將服務器綁定到該地址。

查看[ManagementServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/autoconfigure/ManagementServerProperties.java)程式碼和'Production-ready特性'章節中的[Section 41.3, “Customizing the management server port”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready-customizing-management-server-port)來獲取更多詳情。

* 自定義'白標'（whitelabel，可以了解下相關理念）錯誤頁面

Spring Boot安裝了一個'whitelabel'錯誤頁面，如果你遇到一個服務器錯誤（機器客戶端消費的是JSON，其他媒體類型則會看到一個具有正確錯誤碼的合乎情理的響應），那就能在客戶端瀏覽器中看到該頁面。你可以設置`error.whitelabel.enabled=false`來關閉該功能，但通常你想要添加自己的錯誤頁面來取代whitelabel。確切地說，如何實現取決於你使用的模板技術。例如，你正在使用Thymeleaf，你將添加一個error.html模板。如果你正在使用FreeMarker，那你將添加一個error.ftl模板。通常，你需要的只是一個名稱為error的View，和/或一個處理`/error`路徑的`@Controller`。除非你替換了一些預設配置，否則你將在你的ApplicationContext中找到一個BeanNameViewResolver，所以一個id為error的`@Bean`可能是完成該操作的一個簡單方式。詳情參考[ErrorMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ErrorMvcAutoConfiguration.java)。

查看[Error Handling](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-error-handling)章節，了解下如何將處理器（handlers）註冊到servlet容器中。

### 安全

* 關閉Spring Boot安全配置

不管你在應用的什麼地方定義了一個使用`@EnableWebSecurity`註解的`@Configuration`，它將會關閉Spring Boot中的預設webapp安全設置。想要調整預設值，你可以嘗試設置`security.*`屬性（具體查看[SecurityProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/SecurityProperties.java)和[常見應用屬性](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#common-application-properties-security)的SECURITY章節）。

* 改變AuthenticationManager並添加用戶賬號

如果你提供了一個AuthenticationManager類型的`@Bean`，那麼預設的就不會被建立了，所以你可以獲得Spring Security可用的全部特性（比如，[不同的認證選項](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication)）。

Spring Security也提供了一個方便的AuthenticationManagerBuilder，可用於建構具有常見選項的AuthenticationManager。在一個webapp中，推薦將它注入到WebSecurityConfigurerAdapter的一個void函式中，比如：
```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
            auth.inMemoryAuthentication()
                .withUser("barry").password("password").roles("USER"); // ... etc.
    }

    // ... other stuff for application security
}
```
如果把它放到一個內部類或一個單獨的類中，你將得到最好的結果（也就是不跟很多其他`@Beans`混合在一起將允許你改變實例化的順序）。[secure web sample](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-secure)是一個有用的參考模板。

如果你遇到了實例化問題（比如，使用JDBC或JPA進行用戶詳細訊息的存儲），那將AuthenticationManagerBuilder回調提取到一個GlobalAuthenticationConfigurerAdapter（放到init()函式內以防其他地方也需要authentication manager）可能是個不錯的選擇，比如：
```java
@Configuration
public class AuthenticationManagerConfiguration extends

    GlobalAuthenticationConfigurerAdapter {
    @Override
    public void init(AuthenticationManagerBuilder auth) {
        auth.inMemoryAuthentication() // ... etc.
    }

}
```
* 當前端使用代理服務器時，啟用HTTPS

對於任何應用來說，確保所有的主端點（URL）都隻在HTTPS下可用是個重要的苦差事。如果你使用Tomcat作為servlet容器，那Spring Boot如果發現一些環境設置的話，它將自動添加Tomcat自己的RemoteIpValve，你也可以依賴於HttpServletRequest來報告是否請求是安全的（即使代理服務器的downstream處理真實的SSL終端）。這個標準行為取決於某些請求頭是否出現（`x-forwarded-for`和`x-forwarded-proto`），這些請求頭的名稱都是約定好的，所以對於大多數前端和代理都是有效的。

你可以向application.properties添加以下設置裡開啟該功能，比如：
```yml
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto
```
（這些屬性出現一個就會開啟該功能，或者你可以通過添加一個TomcatEmbeddedServletContainerFactory bean自己添加RemoteIpValve）

Spring Security也可以配置成針對所以或某些請求需要一個安全渠道（channel）。想要在一個Spring Boot應用中開啟它，你只需將application.properties中的`security.require_ssl`設置為`true`即可。

### 熱交換

* 重新加載靜態內容

Spring Boot有很多用於熱加載的選項。使用IDE開發是一個不錯的方式，特別是需要調試的時候（所有的現代IDEs都允許重新加載靜態資源，通常也支援對變更的Java類進行熱交換）。[Maven和Gradle插件](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins)也支援命令列下的靜態文件熱加載。如果你使用其他進階工具編寫css/js，並使用外部的css/js編譯器，那你就可以充分利用該功能。

* 在不重啟容器的情況下重新加載Thymeleaf模板

如果你正在使用Thymeleaf，那就將`spring.thymeleaf.cache`設置為false。查看[ThymeleafAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)可以獲取其他Thymeleaf自定義選項。

* 在不重啟容器的情況下重新加載FreeMarker模板

如果你正在使用FreeMarker，那就將`spring.freemarker.cache`設置為false。查看[FreeMarkerAutoConfiguration ](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java)可以獲取其他FreeMarker自定義選項。

* 在不重啟容器的情況下重新加載Groovy模板

如果你正在使用Groovy模板，那就將`spring.groovy.template.cache`設置為false。查看[GroovyTemplateAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java)可以獲取其他Groovy自定義選項。

* 在不重啟容器的情況下重新加載Velocity模板

如果你正在使用Velocity，那就將`spring.velocity.cache`設置為false。查看[VelocityAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/velocity/VelocityAutoConfiguration.java)可以獲取其他Velocity自定義選項。

* 在不重啟容器的情況下重新加載Java類

現代IDEs（Eclipse, IDEA等）都支援字節碼的熱交換，所以如果你做了一個沒有影響類或函式簽名的改變，它會利索地重新加載並沒有任何影響。

[Spring Loaded](https://github.com/spring-projects/spring-loaded)在這方麵走的更遠，它能夠重新加載函式簽名改變的類定義。如果對它進行一些自定義配置可以強製ApplicationContext刷新自己（但沒有通用的機製來確保這對一個運行中的應用總是安全的，所以它可能只是一個開發時間的技巧）。

- 使用Maven配置Spring Loaded

為了在Maven命令列下使用Spring Loaded，你只需將它作為一個依賴添加到Spring Boot插件聲明中即可，比如：
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.0.RELEASE</version>
        </dependency>
    </dependencies>
</plugin>
```
正常情況下，這在Eclipse和IntelliJ中工作的相當漂亮，隻要它們有相應的，和Maven預設一致的建構配置（Eclipse m2e對此支援的更好，開箱即用）。

- 使用Gradle和IntelliJ配置Spring Loaded

如果想將Spring Loaded和Gradle，IntelliJ結合起來，那你需要付出代價。預設情況下，IntelliJ將類編譯到一個跟Gradle不同的位置，這會導致Spring Loaded監控失敗。

為了正確配置IntelliJ，你可以使用`idea` Gradle插件：
```gradle
buildscript {
    repositories { jcenter() }
    dependencies {
        classpath "org.springframework.boot:spring-boot-gradle-plugin:1.3.0.BUILD-SNAPSHOT"
        classpath 'org.springframework:springloaded:1.2.0.RELEASE'
    }
}

apply plugin: 'idea'

idea {
    module {
        inheritOutputDirs = false
        outputDir = file("$buildDir/classes/main/")
    }
}

// ...
```
**注**：IntelliJ必須配置跟命令列Gradle任務相同的Java版本，並且springloaded必須作為一個buildscript依賴被包含進去。

此外，你也可以啟用Intellij內部的`Make Project Automatically`，這樣不管什麼時候隻要文件被保存都會自動編譯你的程式碼。

### 建構

* 使用Maven自定義依賴版本

如果你使用Maven進行一個直接或間接繼承`spring-boot-dependencies`（比如`spring-boot-starter-parent`）的建構，並想覆蓋一個特定的第三方依賴，那你可以添加合適的`<properties>`元素。瀏覽[spring-boot-dependencies](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-dependencies/pom.xml) POM可以獲取一個全麵的屬性列表。例如，想要選擇一個不同的slf4j版本，你可以添加以下內容：
```xml
<properties>
    <slf4j.version>1.7.5<slf4j.version>
</properties>
```
**注**：這隻在你的Maven項目繼承（直接或間接）自`spring-boot-dependencies`才有用。如果你使用`<scope>import</scope>`，將`spring-boot-dependencies`添加到自己的`dependencyManagement`片段，那你必須自己重新定義artifact而不是覆蓋屬性。

**注**：每個Spring Boot發佈都是基於一些特定的第三方依賴集進行設計和測試的，覆蓋版本可能導致兼容性問題。

* 使用Maven建立可執行JAR

`spring-boot-maven-plugin`能夠用來建立可執行的'胖'JAR。如果你正在使用`spring-boot-starter-parent` POM，你可以簡單地聲明該插件，然後你的jar將被重新打包：
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
如果沒有使用parent POM，你仍舊可以使用該插件。不過，你需要另外添加一個`<executions>`片段：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>1.3.0.BUILD-SNAPSHOT</version>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
查看[插件文件](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/usage.html)獲取詳細的用例。

* 建立其他的可執行JAR

如果你想將自己的項目以library jar的形式被其他項目依賴，並且需要它是一個可執行版本（例如demo），你需要使用略微不同的方式來配置該建構。

對於Maven來說，正常的JAR插件和Spring Boot插件都有一個'classifier'，你可以添加它來建立另外的JAR。範例如下（使用Spring Boot Starter Parent管理插件版本，其他配置採用預設設置）：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </plugin>
    </plugins>
</build>
```
上述配置會產生兩個jars，預設的一個和使用帶有classifier 'exec'的Boot插件建構的可執行的一個。

對於Gradle用戶來說，步驟類似。範例如下：
```gradle
bootRepackage  {
    classifier = 'exec'
}
```
* 在可執行jar運行時提取特定的版本

在一個可執行jar中，為了運行，多數內嵌的函式庫不需要拆包（unpacked），然而有一些函式庫可能會遇到問題。例如，JRuby包含它自己的內嵌jar，它假定`jruby-complete.jar`本身總是能夠直接作為文件訪問的。

為了處理任何有問題的函式庫，你可以標記那些特定的內嵌jars，讓它們在可執行jar第一次運行時自動解壓到一個臨時文件夾中。例如，為了將JRuby標記為使用Maven插件拆包，你需要添加如下的配置：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <requiresUnpack>
                    <dependency>
                        <groupId>org.jruby</groupId>
                        <artifactId>jruby-complete</artifactId>
                    </dependency>
                </requiresUnpack>
            </configuration>
        </plugin>
    </plugins>
</build>
```
使用Gradle完全上述操作：
```gradle
springBoot  {
    requiresUnpack = ['org.jruby:jruby-complete']
}
```
* 使用排除建立不可執行的JAR

如果你建構的產物既有可執行的jar和非可執行的jar，那你常常需要為可執行的版本添加額外的配置文件，而這些文件在一個library jar中是不需要的。比如，application.yml配置文件可能需要從非可執行的JAR中排除。

下面是如何在Maven中實現：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-jar-plugin</artifactId>
            <executions>
                <execution>
                    <id>exec</id>
                    <phase>package</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                    <configuration>
                        <classifier>exec</classifier>
                    </configuration>
                </execution>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                    <configuration>
                        <!-- Need this to ensure application.yml is excluded -->
                        <forceCreation>true</forceCreation>
                        <excludes>
                            <exclude>application.yml</exclude>
                        </excludes>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
在Gradle中，你可以使用標準任務的DSL（領域特定語言）特性建立一個新的JAR存檔，然後在bootRepackage任務中使用withJarTask屬性添加對它的依賴：
```gradle
jar {
    baseName = 'spring-boot-sample-profile'
    version =  '0.0.0'
    excludes = ['**/application.yml']
}

task('execJar', type:Jar, dependsOn: 'jar') {
    baseName = 'spring-boot-sample-profile'
    version =  '0.0.0'
    classifier = 'exec'
    from sourceSets.main.output
}

bootRepackage  {
    withJarTask = tasks['execJar']
}
```
* 遠程調試一個使用Maven啟動的Spring Boot項目

想要為使用Maven啟動的Spring Boot應用添加一個遠程調試器，你可以使用[mave插件](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/)的jvmArguments屬性。詳情參考[範例](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/examples/run-debug.html)。

* 遠程調試一個使用Gradle啟動的Spring Boot項目

想要為使用Gradle啟動的Spring Boot應用添加一個遠程調試器，你可以使用build.gradle的applicationDefaultJvmArgs屬性或`--debug-jvm`命令列選項。

build.gradle：
```gradle
applicationDefaultJvmArgs = [
    "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005"
]
```
命令列：
```shell
$ gradle run --debug-jvm
```
詳情查看[Gradle應用插件](http://www.gradle.org/docs/current/userguide/application_plugin.html)。

* 使用Ant建構可執行存檔（archive）

想要使用Ant進行建構，你需要抓取依賴，編譯，然後像通常那樣建立一個jar或war存檔。為了讓它可以執行：

1. 使用合適的啟動器配置`Main-Class`，比如對於jar文件使用JarLauncher，然後將其他需要的屬性以manifest實體指定，主要是一個`Start-Class`。
2. 將運行時依賴添加到一個內嵌的'lib'目錄（對於jar），`provided`（內嵌容器）依賴添加到一個內嵌的`lib-provided`目錄。記住***不要***壓縮存檔中的實體。
3. 在存檔的根目錄添加`spring-boot-loader`類（這樣`Main-Class`就可用了）。

範例：
```xml
<target name="build" depends="compile">
    <copy todir="target/classes/lib">
        <fileset dir="lib/runtime" />
    </copy>
    <jar destfile="target/spring-boot-sample-actuator-${spring-boot.version}.jar" compress="false">
        <fileset dir="target/classes" />
        <fileset dir="src/main/resources" />
        <zipfileset src="lib/loader/spring-boot-loader-jar-${spring-boot.version}.jar" />
        <manifest>
            <attribute name="Main-Class" value="org.springframework.boot.loader.JarLauncher" />
            <attribute name="Start-Class" value="${start-class}" />
        </manifest>
    </jar>
</target>
```
該Actuator範例中有一個build.xml文件，可以使用以下命令來運行：
```shell
$ ant -lib <path_to>/ivy-2.2.jar
```
在上述操作之後，你可以使用以下命令運行該應用：
```shell
$ java -jar target/*.jar
```
* 如何使用Java6

如果想在Java6環境中使用Spring Boot，你需要改變一些配置。具體的變化取決於你應用的功能。

- 內嵌Servlet容器兼容性

如果你在使用Boot的內嵌Servlet容器，你需要使用一個兼容Java6的容器。Tomcat 7和Jetty 8都是Java 6兼容的。具體參考[Section 63.15, “Use Tomcat 7”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-tomcat-7)和[Section 63.16, “Use Jetty 8”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-jetty-8)。

- JTA API兼容性

Java事務API自身並不要求Java 7，而是官方的API jar包含的已建構類要求Java 7。如果你正在使用JTA，那麼你需要使用能夠在Java 6工作的建構版本替換官方的JTA 1.2 API jar。為了完成該操作，你需要排除任何對`javax.transaction:javax.transaction-api`的傳遞依賴，並使用`org.jboss.spec.javax.transaction:jboss-transaction-api_1.2_spec:1.0.0.Final`依賴替換它們。


### 傳統部署

* 建立一個可部署的war文件

產生一個可部署war包的第一步是提供一個SpringBootServletInitializer子類，並覆蓋它的configure函式。這充分利用了Spring框架對Servlet 3.0的支援，並允許你在應用通過servlet容器啟動時配置它。通常，你只需把應用的主類改為繼承SpringBootServletInitializer即可：
```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
```
下一步是更新你的建構配置，這樣你的項目將產生一個war包而不是jar包。如果你使用Maven，並使用`spring-boot-starter-parent`（為了配置Maven的war插件），所有你需要做的就是更改pom.xml的packaging為war：
```xml
<packaging>war</packaging>
```
如果你使用Gradle，你需要修改build.gradle來將war插件應用到項目上：
```gradle
apply plugin: 'war'
```
該過程最後的一步是確保內嵌的servlet容器不能幹擾war包將部署的servlet容器。為了達到這個目的，你需要將內嵌容器的依賴標記為provided。

如果使用Maven：
```xml
<dependencies>
    <!-- … -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- … -->
</dependencies>
```
如果使用Gradle：
```gradle
dependencies {
    // …
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
    // …
}
```
如果你使用[Spring Boot建構工具](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins)，將內嵌容器依賴標記為provided將產生一個可執行war包，在`lib-provided`目錄有該war包的provided依賴。這意味著，除了部署到servlet容器，你還可以通過使用命令列`java -jar`命令來運行應用。

**注**：查看Spring Boot基於以上配置的一個[Maven範例應用](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-traditional/pom.xml)。

* 為老的servlet容器建立一個可部署的war文件

老的Servlet容器不支援在Servlet 3.0中使用的ServletContextInitializer啟動處理。你仍舊可以在這些容器使用Spring和Spring Boot，但你需要為應用添加一個web.xml，並將它配置為通過一個DispatcherServlet加載一個ApplicationContext。

* 將現有的應用轉換為Spring Boot

對於一個非web項目，轉換為Spring Boot應用很容易（拋棄建立ApplicationContext的程式碼，取而代之的是調用SpringApplication或SpringApplicationBuilder）。Spring MVC web應用通常先建立一個可部署的war應用，然後將它遷移為一個可執行的war或jar。建議閱讀[Getting Started Guide on Converting a jar to a war.](http://spring.io/guides/gs/convert-jar-to-war/)。

通過繼承SpringBootServletInitializer建立一個可執行war（比如，在一個名為Application的類中），然後添加Spring Boot的`@EnableAutoConfiguration`註解。範例：
```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        // Customize the application or call application.sources(...) to add sources
        // Since our example is itself a @Configuration class we actually don't
        // need to override this method.
        return application;
    }

}
```
記住不管你往sources放什麼東西，它僅是一個Spring ApplicationContext，正常情況下，任何生效的在這裡也會起作用。有一些beans你可以先移除，然後讓Spring Boot提供它的預設實現，不過有可能需要先完成一些事情。

靜態資源可以移到classpath根目錄下的`/public`（或`/static`，`/resources`，`/META-INF/resources`）。同樣的方式也適合於`messages.properties`（Spring Boot在classpath根目錄下自動發現這些配置）。

美妙的（Vanilla usage of）Spring DispatcherServlet和Spring Security不需要改變。如果你的應用有其他特性，比如使用其他servlets或filters，那你可能需要添加一些配置到你的Application上下文中，按以下操作替換web.xml的那些元素：

- 在容器中安裝一個Servlet或ServletRegistrationBean類型的`@Bean`，就好像web.xml中的`<servlet/>`和`<servlet-mapping/>`。
- 同樣的添加一個Filter或FilterRegistrationBean類型的`@Bean`（類似於`<filter/>`和`<filter-mapping/>`）。
- 在XML文件中的ApplicationContext可以通過`@Import`添加到你的Application中。簡單的情況下，大量使用註解配置可以在幾行內定義`@Bean`定義。

一旦war可以使用，我們就通過添加一個main函式到Application來讓它可以執行，比如：
```java
public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
}
```
應用可以劃分為多個類別：

- 沒有web.xml的Servlet 3.0+應用
- 有web.xml的應用
- 有上下文層次的應用
- 沒有上下文層次的應用

所有這些都可以進行適當的轉化，但每個可能需要稍微不同的技巧。

Servlet 3.0+的應用轉化的相當簡單，如果它們已經使用Spring Servlet 3.0+初始化器輔助類。通常所有來自一個存在的WebApplicationInitializer的程式碼可以移到一個SpringBootServletInitializer中。如果一個存在的應用有多個ApplicationContext（比如，如果它使用AbstractDispatcherServletInitializer），那你可以將所有上下文源放進一個單一的SpringApplication。你遇到的主要難題可能是如果那樣不能工作，那你就要維護上下文層次。參考範例[entry on building a hierarchy](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-build-an-application-context-hierarchy)。一個存在的包含web相關特性的父上下文通常需要分解，這樣所有的ServletContextAware組件都處於子上下文中。

對於還不是Spring應用的應用來說，上麵的指南有助於你把應用轉換為一個Spring Boot應用，但你也可以選擇其他方式。

* 部署WAR到Weblogic

想要將Spring Boot應用部署到Weblogic，你需要確保你的servlet初始化器直接實現WebApplicationInitializer（即使你繼承的基類已經實現了它）。

一個傳統的Weblogic初始化器可能如下所示：
```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.web.SpringBootServletInitializer;
import org.springframework.web.WebApplicationInitializer;

@SpringBootApplication
public class MyApplication extends SpringBootServletInitializer implements WebApplicationInitializer {

}
```
如果使用logback，你需要告訴Weblogic你傾向使用的打包版本而不是服務器預裝的版本。你可以通過添加一個具有如下內容的`WEB-INF/weblogic.xml`實現該操作：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wls:weblogic-web-app
	xmlns:wls="http://xmlns.oracle.com/weblogic/weblogic-web-app"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		http://java.sun.com/xml/ns/javaee/ejb-jar_3_0.xsd
		http://xmlns.oracle.com/weblogic/weblogic-web-app
		http://xmlns.oracle.com/weblogic/weblogic-web-app/1.4/weblogic-web-app.xsd">
	<wls:container-descriptor>
		<wls:prefer-application-packages>
			<wls:package-name>org.slf4j</wls:package-name>
		</wls:prefer-application-packages>
	</wls:container-descriptor>
</wls:weblogic-web-app>
```
* 部署WAR到老的(Servlet2.5)容器

Spring Boot使用 Servlet 3.0 APIs初始化ServletContext（註冊Servlets等），所以你不能在一個Servlet 2.5的容器中原封不動的使用同樣的應用。使用一些特定的工具也是可以在一個老的容器中運行Spring Boot應用的。如果添加了`org.springframework.boot:spring-boot-legacy`依賴，你只需要建立一個web.xml，聲明一個用於建立應用上下文的上下文監聽器，過濾器和servlets。上下文監聽器是專用於Spring Boot的，其他的都是一個Servlet 2.5的Spring應用所具有的。範例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>demo.Application</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.boot.legacy.context.web.SpringBootContextLoaderListener</listener-class>
    </listener>

    <filter>
        <filter-name>metricFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>metricFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextAttribute</param-name>
            <param-value>org.springframework.web.context.WebApplicationContext.ROOT</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```
在該範例中，我們使用一個單一的應用上下文（通過上下文監聽器建立的），然後使用一個init參數將它附加到DispatcherServlet。這在一個Spring Boot應用中是很正常的（你通常隻有一個應用上下文）。
