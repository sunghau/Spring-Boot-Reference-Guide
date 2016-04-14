Spring Boot特性
===============
### SpringApplication
SpringApplication類提供了一種從main()函式啟動Spring應用的便捷方式。在很多情況下，你只需委托給SpringApplication.run這個靜態函式：
```java
public static void main(String[] args){
    SpringApplication.run(MySpringConfiguration.class, args);
}
```
* 自定義Banner
  
通過在classpath下添加一個banner.txt或設置banner.location來指定相應的文件可以改變啟動過程中列印的banner。如果這個文件有特殊的編碼，你可以使用banner.encoding設置它（預設為UTF-8）。

在banner.txt中可以使用如下的變量：

| 變量        | 描述     | 
| ----------- | :--------|
|${application.version}|MANIFEST.MF中聲明的應用版本號，例如1.0|
|${application.formatted-version}|MANIFEST.MF中聲明的被格式化後的應用版本號（被括號包裹且以v作為前綴），用於顯示，例如(v1.0)|
|${spring-boot.version}|正在使用的Spring Boot版本號，例如1.2.2.BUILD-SNAPSHOT|
|${spring-boot.formatted-version}|正在使用的Spring Boot被格式化後的版本號（被括號包裹且以v作為前綴）,  用於顯示，例如(v1.2.2.BUILD-SNAPSHOT)|

**注**：如果想以程式的方式產生一個banner，可以使用SpringBootApplication.setBanner(…)函式。使用org.springframework.boot.Banner接口，實現你自己的printBanner()函式。

* 自定義SpringApplication

如果預設的SpringApplication不符合你的口味，你可以建立一個本地的實例並自定義它。例如，關閉banner你可以這樣寫：
```java
public static void main(String[] args){
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setShowBanner(false);
    app.run(args);
}
```
**注**：傳遞給SpringApplication的構造器參數是spring beans的配置源。在大多數情況下，這些將是@Configuration類的引用，但它們也可能是XML配置或要掃描包的引用。

你也可以使用application.properties文件來配置SpringApplication。具體參考[Externalized 配置](#Externalized 配置)。查看配置選項的完整列表，可參考[SpringApplication Javadoc](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/SpringApplication.html).
    
* 流暢的建構API

如果你需要建立一個分層的ApplicationContext（多個具有父子關係的上下文），或你只是喜歡使用流暢的建構API，你可以使用SpringApplicationBuilder。SpringApplicationBuilder允許你以鏈式方式調用多個函式，包括可以建立層次結構的parent和child函式。
```java
new SpringApplicationBuilder()
    .showBanner(false)
    .sources(Parent.class)
    .child(Application.class)
    .run(args);
```
**注**：建立ApplicationContext層次時有些限製，比如，Web組件(components)必須包含在子上下文(child context)中，且相同的Environment即用於父上下文也用於子上下文中。具體參考[SpringApplicationBuilder javadoc](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/builder/SpringApplicationBuilder.html)

* Application事件和監聽器

除了常見的Spring框架事件，比如[ContextRefreshedEvent](http://docs.spring.io/spring/docs/4.1.4.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)，一個SpringApplication也發送一些額外的應用事件。一些事件實際上是在ApplicationContext被建立前觸發的。

你可以使用多種方式註冊事件監聽器，最普通的是使用SpringApplication.addListeners(…)函式。在你的應用運行時，應用事件會以下面的次序發送：

1. 在運行開始，但除了監聽器註冊和初始化以外的任何處理之前，會發送一個ApplicationStartedEvent。
2. 在Environment將被用於已知的上下文，但在上下文被建立前，會發送一個ApplicationEnvironmentPreparedEvent。
3. 在refresh開始前，但在bean定義已被加載後，會發送一個ApplicationPreparedEvent。
4. 啟動過程中如果出現異常，會發送一個ApplicationFailedEvent。

**注**：你通常不需要使用應用程序事件，但知道它們的存在會很方便（在某些場合可能會使用到）。在Spring內部，Spring Boot使用事件處理各種各樣的任務。

* Web環境

一個SpringApplication將嘗試為你建立正確類型的ApplicationContext。在預設情況下，使用AnnotationConfigApplicationContext或AnnotationConfigEmbeddedWebApplicationContext取決於你正在開發的是否是web應用。

用於確定一個web環境的算法相當簡單（基於是否存在某些類）。如果需要覆蓋預設行為，你可以使用setWebEnvironment(boolean webEnvironment)。通過調用setApplicationContextClass(…)，你可以完全控製ApplicationContext的類型。

**注**：當JUnit測試裡使用SpringApplication時，調用setWebEnvironment(false)是可取的。

* 命令列啟動器

如果你想獲取原始的命令列參數，或一旦SpringApplication啟動，你需要運行一些特定的程式碼，你可以實現CommandLineRunner接口。在所有實現該接口的Spring beans上將調用run(String… args)函式。
```java
import org.springframework.boot.*
import org.springframework.stereotype.*

@Component
public class MyBean implements CommandLineRunner {
    public void run(String... args) {
        // Do something...
    }
}
```
如果一些CommandLineRunner beans被定義必須以特定的次序調用，你可以額外實現org.springframework.core.Ordered接口或使用org.springframework.core.annotation.Order註解。

* Application退出

每個SpringApplication在退出時為了確保ApplicationContext被優雅的關閉，將會註冊一個JVM的shutdown鉤子。所有標準的Spring生命周期回調（比如，DisposableBean接口或@PreDestroy註解）都能使用。

此外，如果beans想在應用結束時返回一個特定的退出碼（exit code），可以實現org.springframework.boot.ExitCodeGenerator接口。

### 外化配置

Spring Boot允許外化（externalize）你的配置，這樣你能夠在不同的環境下使用相同的程式碼。你可以使用properties文件，YAML文件，環境變量和命令列參數來外化配置。使用@Value註解，可以直接將屬性值注入到你的beans中，並通過Spring的Environment抽象或綁定到結構化對象來訪問。

Spring Boot使用一個非常特別的PropertySource次序來允許對值進行合理的覆蓋，需要以下面的次序考慮屬性：

1. 命令列參數
2. 來自於java:comp/env的JNDI屬性
3. Java系統屬性（System.getProperties()）
4. 操作系統環境變量
5. 隻有在random.*裡包含的屬性會產生一個RandomValuePropertySource
6. 在打包的jar外的應用程序配置文件（application.properties，包含YAML和profile變量）
7. 在打包的jar內的應用程序配置文件（application.properties，包含YAML和profile變量）
8. 在@Configuration類上的@PropertySource註解
9. 預設屬性（使用SpringApplication.setDefaultProperties指定）

下面是一個具體的範例（假設你開發一個使用name屬性的@Component）：
```java
import org.springframework.stereotype.*
import org.springframework.beans.factory.annotation.*

@Component
public class MyBean {
    @Value("${name}")
    private String name;
    // ...
}
```
你可以將一個application.properties文件捆綁到jar內，用來提供一個合理的預設name屬性值。當運行在生產環境時，可以在jar外提供一個application.properties文件來覆蓋name屬性。對於一次性的測試，你可以使用特定的命令列開關啟動（比如，java -jar app.jar --name="Spring"）。

RandomValuePropertySource在注入隨機值（比如，密鑰或測試用例）時很有用。它能產生整數，longs或字符串，比如：
```java
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```
random.int*語法是OPEN value (,max) CLOSE，此處OPEN，CLOSE可以是任何字符，並且value，max是整數。如果提供max，那麼value是最小的值，max是最大的值（不包含在內）。

* 訪問命令列屬性

預設情況下，SpringApplication將任何可選的命令列參數（以'--'開頭，比如，--server.port=9000）轉化為property，並將其添加到Spring Environment中。如上所述，命令列屬性總是優先於其他屬性源。

如果你不想將命令列屬性添加到Environment裡，你可以使用SpringApplication.setAddCommandLineProperties(false)來禁止它們。

* Application屬性文件

SpringApplication將從以下位置加載application.properties文件，並把它們添加到Spring Environment中：

1. 當前目錄下的一個/config子目錄
2. 當前目錄
3. 一個classpath下的/config包
4. classpath根路徑（root）

這個列表是按優先級排序的（列表中位置高的將覆蓋位置低的）。

**注**：你可以使用YAML（'.yml'）文件替代'.properties'。

如果不喜歡將application.properties作為配置文件名，你可以通過指定spring.config.name環境屬性來切換其他的名稱。你也可以使用spring.config.location環境屬性來引用一個明確的路徑（目錄位置或文件路徑列表以逗號分割）。
```shell
$ java -jar myproject.jar --spring.config.name=myproject
//or
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```
如果spring.config.location包含目錄（相對於文件），那它們應該以/結尾（在加載前，spring.config.name產生的名稱將被追加到後麵）。不管spring.config.location是什麼值，預設的搜索路徑classpath:,classpath:/config,file:,file:config/總會被使用。以這種方式，你可以在application.properties中為應用設置預設值，然後在運行的時候使用不同的文件覆蓋它，同時保留預設配置。

**注**：如果你使用環境變量而不是系統配置，大多數操作系統不允許以句號分割（period-separated）的key名稱，但你可以使用下劃線（underscores）代替（比如，使用SPRING_CONFIG_NAME代替spring.config.name）。如果你的應用運行在一個容器中，那麼JNDI屬性（java:comp/env）或servlet上下文初始化參數可以用來取代環境變量或系統屬性，當然也可以使用環境變量或系統屬性。

* 特定的Profile屬性

除了application.properties文件，特定配置屬性也能通過命令慣例application-{profile}.properties來定義。特定Profile屬性從跟標準application.properties相同的路徑加載，並且特定profile文件會覆蓋預設的配置。

* 屬性佔位符

當application.properties裡的值被使用時，它們會被存在的Environment過濾，所以你能夠引用先前定義的值（比如，系統屬性）。
```java
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```
**注**：你也能使用相應的技巧為存在的Spring Boot屬性建立'短'變量，具體參考[Section 63.3, “Use ‘short’ command line arguments”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-short-command-line-arguments)。

* 使用YAML代替Properties

[YAML](http://yaml.org/)是JSON的一個超集，也是一種方便的定義層次配置數據的格式。無論你何時將[SnakeYAML ](http://code.google.com/p/snakeyaml/)函式庫放到classpath下，SpringApplication類都會自動支援YAML作為properties的替換。

**注**：如果你使用'starter POMs'，spring-boot-starter會自動提供SnakeYAML。

* 加載YAML

Spring框架提供兩個便利的類用於加載YAML文件，YamlPropertiesFactoryBean會將YAML作為Properties來加載，YamlMapFactoryBean會將YAML作為Map來加載。

範例：
```json
environments:
    dev:
        url: http://dev.bar.com
        name: Developer Setup
    prod:
        url: http://foo.bar.com
        name: My Cool App
```
上麵的YAML文件會被轉化到下面的屬性中：
```java
environments.dev.url=http://dev.bar.com
environments.dev.name=Developer Setup
environments.prod.url=http://foo.bar.com
environments.prod.name=My Cool App
```
YAML列表被表示成使用[index]間接引用作為屬性keys的形式，例如下面的YAML：
```json
my:
   servers:
       - dev.bar.com
       - foo.bar.com
```
將會轉化到下面的屬性中:
```java
my.servers[0]=dev.bar.com
my.servers[1]=foo.bar.com
```
使用Spring DataBinder工具綁定那樣的屬性（這是@ConfigurationProperties做的事），你需要確定目標bean中有個java.util.List或Set類型的屬性，並且需要提供一個setter或使用可變的值初始化它，比如，下面的程式碼將綁定上麵的屬性：
```java
@ConfigurationProperties(prefix="my")
public class Config {
    private List<String> servers = new ArrayList<String>();
    public List<String> getServers() {
        return this.servers;
    }
}
```
* 在Spring環境中使用YAML曝露屬性

YamlPropertySourceLoader類能夠用於將YAML作為一個PropertySource導出到Sprig Environment。這允許你使用熟悉的@Value註解和佔位符語法訪問YAML屬性。

* Multi-profile YAML文件

你可以在單個文件中定義多個特定配置（profile-specific）的YAML文件，並通過一個spring.profiles key標示應用的文件。例如：
```json
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production
server:
    address: 192.168.1.120
```
在上麵的例子中，如果development配置被啟動，那server.address屬性將是127.0.0.1。如果development和production配置（profiles）沒有啟用，則該屬性的值將是192.168.1.100。

* YAML缺點

YAML文件不能通過@PropertySource註解加載。所以，在這種情況下，如果需要使用@PropertySource註解的方式加載值，那就要使用properties文件。

* 類型安全的配置屬性

使用@Value("${property}")註解注入配置屬性有時可能比較笨重，特別是需要使用多個properties或你的數據本身有層次結構。為了控製和校驗你的應用配置，Spring Boot提供一個允許強類型beans的替代函式來使用properties。

範例：
```java
@Component
@ConfigurationProperties(prefix="connection")
public class ConnectionSettings {
    private String username;
    private InetAddress remoteAddress;
    // ... getters and setters
}
```
當@EnableConfigurationProperties註解應用到你的@Configuration時，任何被@ConfigurationProperties註解的beans將自動被Environment屬性配置。這種風格的配置特別適合與SpringApplication的外部YAML配置進行配合使用。
```json
# application.yml
connection:
    username: admin
    remoteAddress: 192.168.1.1
# additional configuration as required
```
為了使用@ConfigurationProperties beans，你可以使用與其他任何bean相同的方式注入它們。
```java
@Service
public class MyService {
    @Autowired
    private ConnectionSettings connection;
     //...
    @PostConstruct
    public void openConnection() {
        Server server = new Server();
        this.connection.configure(server);
    }
}
```
你可以通過在@EnableConfigurationProperties註解中直接簡單的列出屬性類來快捷的註冊@ConfigurationProperties bean的定義。
```java
@Configuration
@EnableConfigurationProperties(ConnectionSettings.class)
public class MyConfiguration {
}
```
**注**：使用@ConfigurationProperties能夠產生可被IDEs使用的元數據文件。具體參考[Appendix B, Configuration meta-data](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#configuration-metadata)。

* 第3方配置

正如使用@ConfigurationProperties註解一個類，你也可以在@Bean函式上使用它。當你需要綁定屬性到不受你控製的第三方組件時，這種方式非常有用。

為了從Environment屬性配置一個bean，將@ConfigurationProperties添加到它的bean註冊過程：
```java
@ConfigurationProperties(prefix = "foo")
@Bean
public FooComponent fooComponent() {
    ...
}
```
和上麵ConnectionSettings的範例方式相同，任何以foo為前綴的屬性定義都會被映射到FooComponent上。

* 鬆散的綁定（Relaxed binding）

Spring Boot使用一些寬鬆的規則用於綁定Environment屬性到@ConfigurationProperties beans，所以Environment屬性名和bean屬性名不需要精確匹配。常見的範例中有用的包括虛線分割（比如，context--path綁定到contextPath）和將環境屬性轉為大寫字母（比如，PORT綁定port）。

範例：
```java
@Component
@ConfigurationProperties(prefix="person")
public class ConnectionSettings {
    private String firstName;
}
```
下面的屬性名都能用於上麵的@ConfigurationProperties類：

| 屬性        | 說明   |
| --------    | :----- |
|person.firstName|標準駝峰規則|
|person.first-name|虛線表示，推薦用於.properties和.yml文件中|
|PERSON_FIRST_NAME|大寫形式，使用系統環境變量時推薦|

Spring會嘗試強製外部的應用屬性在綁定到@ConfigurationProperties beans時類型是正確的。如果需要自定義類型轉換，你可以提供一個ConversionService bean（bean id為conversionService）或自定義屬性編輯器（通過一個CustomEditorConfigurer bean）。

* @ConfigurationProperties校驗 

Spring Boot將嘗試校驗外部的配置，預設使用JSR-303（如果在classpath路徑中）。你可以輕鬆的為你的@ConfigurationProperties類添加JSR-303 javax.validation約束註解：
```java
@Component
@ConfigurationProperties(prefix="connection")
public class ConnectionSettings {
    @NotNull
    private InetAddress remoteAddress;
    // ... getters and setters
}
```
你也可以通過建立一個叫做configurationPropertiesValidator的bean來添加自定義的Spring Validator。

**注**：spring-boot-actuator模塊包含一個曝露所有@ConfigurationProperties beans的端點。簡單地將你的web瀏覽器指向/configprops或使用等效的JMX端點。具體參考[Production ready features](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready-endpoints)。

### Profiles
Spring Profiles提供了一種隔離應用程序配置的方式，並讓這些配置隻能在特定的環境下生效。任何@Component或@Configuration都能被@Profile標記，從而限製加載它的時機。
```java
@Configuration
@Profile("production")
public class ProductionConfiguration {
    // ...
}
```
以正常的Spring方式，你可以使用一個spring.profiles.active的Environment屬性來指定哪個配置生效。你可以使用平常的任何方式來指定該屬性，例如，可以將它包含到你的application.properties中：
```java
spring.profiles.active=dev,hsqldb
```
或使用命令列開關：
```shell
--spring.profiles.active=dev,hsqldb
```
* 添加啟動的配置(profiles)

spring.profiles.active屬性和其他屬性一樣都遵循相同的排列規則，最高的PropertySource獲勝。也就是說，你可以在application.properties中指定生效的配置，然後使用命令列開關替換它們。

有時，將特定的配置屬性添加到生效的配置中而不是替換它們是有用的。spring.profiles.include屬性可以用來無條件的添加生效的配置。SpringApplication的入口點也提供了一個用於設置額外配置的Java API（比如，在那些通過spring.profiles.active屬性生效的配置之上）：參考setAdditionalProfiles()函式。

範例：當一個應用使用下面的屬性，並用`--spring.profiles.active=prod`開關運行，那proddb和prodmq配置也會生效：
```java
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include: proddb,prodmq
```
**注**：spring.profiles屬性可以定義到一個YAML文件中，用於決定什麼時候該文件被包含進配置中。具體參考[Section 63.6, “Change configuration depending on the environment”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-change-configuration-depending-on-the-environment)

* 以程式方式設置profiles

在應用運行前，你可以通過調用SpringApplication.setAdditionalProfiles(…)函式，以程式的方式設置生效的配置。使用Spring的ConfigurableEnvironment接口激動配置也是可行的。

* Profile特定配置文件

application.properties（或application.yml）和通過@ConfigurationProperties引用的文件這兩種配置特定變種都被當作文件來加載的，具體參考[Section 23.3, “Profile specific properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-profile-specific-properties)。

### 日誌
Spring Boot內部日誌系統使用的是[Commons Logging](http://commons.apache.org/logging)，但開放底層的日誌實現。預設為會[Java Util Logging](http://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html), [Log4J](http://logging.apache.org/log4j/), [Log4J2](http://logging.apache.org/log4j/2.x/)和[Logback](http://logback.qos.ch/)提供配置。每種情況下都會預先配置使用控製台輸出，也可以使用可選的文件輸出。

預設情況下，如果你使用'Starter POMs'，那麼就會使用Logback記錄日誌。為了確保那些使用Java Util Logging, Commons Logging, Log4J或SLF4J的依賴函式庫能夠正常工作，正確的Logback路由也被包含進來。

**注**：如果上麵的列表看起來令人困惑，不要擔心，Java有很多可用的日誌框架。通常，你不需要改變日誌依賴，Spring Boot預設的就能很好的工作。

* 日誌格式

Spring Boot預設的日誌輸出格式如下：
```java
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```
輸出的節點（items）如下：

1. 日期和時間 - 精確到毫秒，且易於排序。
2. 日誌級別 - ERROR, WARN, INFO, DEBUG 或 TRACE。
3. Process ID。
4. 一個用於區分實際日誌訊息開頭的---分隔符。
5. 線程名 - 包括在方括號中（控製台輸出可能會被截斷）。
6. 日誌名 - 通常是源class的類名（縮寫）。
7. 日誌訊息。

* 控製台輸出

預設的日誌配置會在寫日誌訊息時將它們回顯到控製台。預設，ERROR, WARN和INFO級別的訊息會被記錄。可以在啟動應用時，通過`--debug`標識開啟控製台的DEBUG級別日誌記錄。
```shell
$ java -jar myapp.jar --debug
```
如果你的終端支援ANSI，為了增加可讀性將會使用彩色的日誌輸出。你可以設置`spring.output.ansi.enabled`為一個[支援的值](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html)來覆蓋自動檢測。

* 文件輸出

預設情況下，Spring Boot隻會將日誌記錄到控製台而不會寫進日誌文件。如果除了輸出到控製台你還想寫入到日誌文件，那你需要設置`logging.file`或`logging.path`屬性（例如在你的application.properties中）。

下表顯示如何組合使用`logging.*`：

|logging.file|logging.path| 範例 | 描述  |
| --------   | :-----  | :-----  | :-----|
|  (none)    | (none)  |         | 隻記錄到控製台 |
|Specific file|(none)|my.log|寫到特定的日誌文件裡，名稱可以是一個精確的位置或相對於當前目錄|
|(none)|Specific folder|/var/log|寫到特定文件夾下的spring.log裡，名稱可以是一個精確的位置或相對於當前目錄|

日誌文件每達到10M就會被輪換（分割），和控製台一樣，預設記錄ERROR, WARN和INFO級別的訊息。

* 日誌級別

所有支援的日誌系統在Spring的Environment（例如在application.properties裡）都有通過'logging.level.*=LEVEL'（'LEVEL'是TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF中的一個）設置的日誌級別。

範例：application.properties
```java
logging.level.org.springframework.web: DEBUG
logging.level.org.hibernate: ERROR
```
* 自定義日誌配置

通過將適當的函式庫添加到classpath，可以啟動各種日誌系統。然後在classpath的根目錄(root)或通過Spring Environment的`logging.config`屬性指定的位置提供一個合適的配置文件來達到進一步的定製（注意由於日誌是在ApplicationContext被建立之前初始化的，所以不可能在Spring的@Configuration文件中，通過@PropertySources控製日誌。系統屬性和平常的Spring Boot外部配置文件能正常工作）。

根據你的日誌系統，下面的文件會被加載：

| 日誌系統        | 定製   |
| --------   | :-----:  | 
|Logback|logback.xml|
|Log4j|log4j.properties或log4j.xml|
|Log4j2|log4j2.xml|
|JDK (Java Util Logging)|logging.properties|

為了幫助定製一些其他的屬性，從Spring的Envrionment轉換到系統屬性：

| Spring Environment| System Property| 評價 |
| --------   | :-----:  | :----:  |
|logging.file|LOG_FILE|如果定義，在預設的日誌配置中使用|
|logging.path|LOG_PATH|如果定義，在預設的日誌配置中使用|
|PID|PID|當前的處理程序(process)ID（如果能夠被發現且還沒有作為操作系統環境變量被定義）|

所有支援的日誌系統在解析它們的配置文件時都能查詢系統屬性。具體可以參考spring-boot.jar中的預設配置。

**注**：在運行可執行的jar時，Java Util Logging有類加載問題，我們建議你盡可能避免使用它。

### 開發Web應用
Spring Boot非常適合開發web應用程序。你可以使用內嵌的Tomcat，Jetty或Undertow輕輕鬆鬆地建立一個HTTP服務器。大多數的web應用都使用spring-boot-starter-web模塊進行快速搭建和運行。

* Spring Web MVC框架

Spring Web MVC框架（通常簡稱為"Spring MVC"）是一個富"模型，視圖，控製器"的web框架。
Spring MVC允許你建立特定的@Controller或@RestController beans來處理傳入的HTTP請求。
使用@RequestMapping註解可以將控製器中的函式映射到相應的HTTP請求。

範例：
```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
        // ...
    }
}
```
* Spring MVC自動配置

Spring Boot為Spring MVC提供適用於多數應用的自動配置功能。在Spring預設基礎上，自動配置添加了以下特性：

1. 引入ContentNegotiatingViewResolver和BeanNameViewResolver beans。
2. 對靜態資源的支援，包括對WebJars的支援。
3. 自動註冊Converter，GenericConverter，Formatter beans。
4. 對HttpMessageConverters的支援。
5. 自動註冊MessageCodeResolver。
6. 對靜態index.html的支援。
7. 對自定義Favicon的支援。

如果想全麵控製Spring MVC，你可以添加自己的@Configuration，並使用@EnableWebMvc對其註解。如果想保留Spring Boot MVC的特性，並只是添加其他的[MVC配置](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle#mvc)(攔截器，formatters，視圖控製器等)，你可以添加自己的WebMvcConfigurerAdapter類型的@Bean（不使用@EnableWebMvc註解）。

* HttpMessageConverters

Spring MVC使用HttpMessageConverter接口轉換HTTP請求和響應。合理的缺省值被包含的恰到好處（out of the box），例如對象可以自動轉換為JSON（使用Jackson函式庫）或XML（如果Jackson XML擴展可用則使用它，否則使用JAXB）。字符串預設使用UTF-8編碼。

如果需要添加或自定義轉換器，你可以使用Spring Boot的HttpMessageConverters類：
```java
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration
public class MyConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }
}
```
任何在上下文中出現的HttpMessageConverter bean將會添加到converters列表，你可以通過這種方式覆蓋預設的轉換器（converters）。

* MessageCodesResolver

Spring MVC有一個策略，用於從綁定的errors產生用來渲染錯誤訊息的錯誤碼：MessageCodesResolver。如果設置`spring.mvc.message-codes-resolver.format`屬性為`PREFIX_ERROR_CODE`或`POSTFIX_ERROR_CODE`（具體查看`DefaultMessageCodesResolver.Format`枚舉值），Spring Boot會為你建立一個MessageCodesResolver。

* 靜態內容

預設情況下，Spring Boot從classpath下一個叫/static（/public，/resources或/META-INF/resources）的文件夾或從ServletContext根目錄提供靜態內容。這使用了Spring MVC的ResourceHttpRequestHandler，所以你可以通過添加自己的WebMvcConfigurerAdapter並覆寫addResourceHandlers函式來改變這個行為（加載靜態文件）。

在一個單獨的web應用中，容器預設的servlet是開啟的，如果Spring決定不處理某些請求，預設的servlet作為一個回退（降級）將從ServletContext根目錄加載內容。大多數時候，這不會發生（除非你修改預設的MVC配置），因為Spring總能夠通過DispatcherServlet處理請求。

此外，上述標準的靜態資源位置有個例外情況是[Webjars內容](http://www.webjars.org/)。任何在/webjars/**路徑下的資源都將從jar文件中提供，隻要它們以Webjars的格式打包。

**注**：如果你的應用將被打包成jar，那就不要使用src/main/webapp文件夾。儘管該文件夾是一個共同的標準，但它僅在打包成war的情況下起作用，並且如果產生一個jar，多數建構工具都會靜悄悄的忽略它。

* 模板引擎

正如REST web服務，你也可以使用Spring MVC提供動態HTML內容。Spring MVC支援各種各樣的模板技術，包括Velocity, FreeMarker和JSPs。很多其他的模板引擎也提供它們自己的Spring MVC整合。

Spring Boot為以下的模板引擎提供自動配置支援：

1. [FreeMarker](http://freemarker.org/docs/)
2. [Groovy](http://beta.groovy-lang.org/docs/groovy-2.3.0/html/documentation/markup-template-engine.html)
3. [Thymeleaf](http://www.thymeleaf.org/)
4. [Velocity](http://velocity.apache.org/)

**注**：如果可能的話，應該忽略JSPs，因為在內嵌的servlet容器使用它們時存在一些[已知的限製](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-jsp-limitations)。

當你使用這些引擎的任何一種，並採用預設的配置，你的模板將會從src/main/resources/templates目錄下自動加載。

**注**：IntelliJ IDEA根據你運行應用的方式會對classpath進行不同的整理。在IDE裡通過main函式運行你的應用跟從Maven或Gradle或打包好的jar中運行相比會導致不同的順序。這可能導致Spring Boot不能從classpath下成功地找到模板。如果遇到這個問題，你可以在IDE裡重新對classpath進行排序，將模塊的類和資源放到第一位。或者，你可以配置模塊的前綴為classpath*:/templates/，這樣會查找classpath下的所有模板目錄。

* 錯誤處理

Spring Boot預設提供一個/error映射用來以合適的方式處理所有的錯誤，並且它在servlet容器中註冊了一個全局的
錯誤頁面。對於機器客戶端（相對於瀏覽器而言，瀏覽器偏重於人的行為），它會產生一個具有詳細錯誤，HTTP狀態，異常訊息的JSON響應。對於瀏覽器客戶端，它會產生一個白色標簽樣式（whitelabel）的錯誤視圖，該視圖將以HTML格式顯示同樣的數據（可以添加一個解析為erro的View來自定義它）。為了完全替換預設的行為，你可以實現ErrorController，並註冊一個該類型的bean定義，或簡單地添加一個ErrorAttributes類型的bean以使用現存的機製，只是替換顯示的內容。

如果在某些條件下需要比較多的錯誤頁面，內嵌的servlet容器提供了一個統一的Java DSL（領域特定語言）來自定義錯誤處理。
範例：
```java
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer(){
    return new MyCustomizer();
}

// ...
private static class MyCustomizer implements EmbeddedServletContainerCustomizer {
    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }
}
```
你也可以使用常規的Spring MVC特性來處理錯誤，比如[@ExceptionHandler函式](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#mvc-exceptionhandlers)和[@ControllerAdvice](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-controller-advice)。ErrorController將會撿起任何沒有處理的異常。

N.B. 如果你為一個路徑註冊一個ErrorPage，最終被一個過濾器（Filter）處理（對於一些非Spring web框架，像Jersey和Wicket這很常見），然後過濾器需要顯式註冊為一個ERROR分發器（dispatcher）。
```java
@Bean
public FilterRegistrationBean myFilter() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new MyFilter());
    ...
    registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
    return registration;
}
```
**注**：預設的FilterRegistrationBean沒有包含ERROR分發器類型。

* Spring HATEOAS

如果你正在開發一個使用超媒體的RESTful API，Spring Boot將為Spring HATEOAS提供自動配置，這在多數應用中都工作良好。自動配置替換了對使用@EnableHypermediaSupport的需求，並註冊一定數量的beans來簡化建構基於超媒體的應用，這些beans包括一個LinkDiscoverer和配置好的用於將響應正確編排為想要的表示的ObjectMapper。ObjectMapper可以根據spring.jackson.*屬性或一個存在的Jackson2ObjectMapperBuilder bean進行自定義。

通過使用@EnableHypermediaSupport，你可以控製Spring HATEOAS的配置。注意這會禁用上述的對ObjectMapper的自定義。

* JAX-RS和Jersey

如果喜歡JAX-RS為REST端點提供的程式模型，你可以使用可用的實現替代Spring MVC。如果在你的應用上下文中將Jersey 1.x和Apache Celtix的Servlet或Filter註冊為一個@Bean，那它們工作的相當好。Jersey 2.x有一些原生的Spring支援，所以我們會在Spring Boot為它提供自動配置支援，連同一個啟動器（starter）。

想要開始使用Jersey 2.x只需要加入spring-boot-starter-jersey依賴，然後你需要一個ResourceConfig類型的@Bean，用於註冊所有的端點（endpoints）。
```java
@Component
public class JerseyConfig extends ResourceConfig {
    public JerseyConfig() {
        register(Endpoint.class);
    }
}
```
所有註冊的端點都應該被@Components和HTTP資源annotations（比如@GET）註解。
```java
@Component
@Path("/hello")
public class Endpoint {
    @GET
    public String message() {
        return "Hello";
    }
}
```
由於Endpoint是一個Spring組件（@Component），所以它的生命周期受Spring管理，並且你可以使用@Autowired添加依賴及使用@Value注入外部配置。Jersey servlet將被註冊，並預設映射到/*。你可以將@ApplicationPath添加到ResourceConfig來改變該映射。

預設情況下，Jersey將在一個ServletRegistrationBean類型的@Bean中被設置成名稱為jerseyServletRegistration的Servlet。通過建立自己的相同名稱的bean，你可以禁止或覆蓋這個bean。你也可以通過設置`spring.jersey.type=filter`來使用一個Filter代替Servlet（在這種情況下，被覆蓋或替換的@Bean是jerseyFilterRegistration）。該servlet有@Order屬性，你可以通過`spring.jersey.filter.order`進行設置。不管是Servlet還是Filter註冊都可以使用spring.jersey.init.*定義一個屬性集合作為初始化參數傳遞過去。

這裡有一個[Jersey範例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-jersey)，你可以查看如何設置相關事項。

* 內嵌servlet容器支援

Spring Boot支援內嵌的Tomcat, Jetty和Undertow服務器。多數開發者只需要使用合適的'Starter POM'來獲取一個完全配置好的實例即可。預設情況下，內嵌的服務器會在8080端口監聽HTTP請求。

**1.Servlets和Filters**

當使用內嵌的servlet容器時，你可以直接將servlet和filter註冊為Spring的beans。在配置期間，如果你想引用來自application.properties的值，這是非常方便的。預設情況下，如果上下文隻包含單一的Servlet，那它將被映射到根路徑（/）。在多Servlet beans的情況下，bean的名稱將被用作路徑的前綴。過濾器會被映射到/*。
    
如果基於約定（convention-based）的映射不夠靈活，你可以使用ServletRegistrationBean和FilterRegistrationBean類實現完全的控製。如果你的bean實現了ServletContextInitializer接口，也可以直接註冊它們。

**2.EmbeddedWebApplicationContext**

Spring Boot底層使用了一個新的ApplicationContext類型，用於對內嵌servlet容器的支援。EmbeddedWebApplicationContext是一個特殊類型的WebApplicationContext，它通過搜索一個單一的EmbeddedServletContainerFactory bean來啟動自己。通常，TomcatEmbeddedServletContainerFactory，JettyEmbeddedServletContainerFactory或UndertowEmbeddedServletContainerFactory將被自動配置。

**注**：你通常不需要知道這些實現類。大多數應用將被自動配置，並根據你的行為建立合適的ApplicationContext和EmbeddedServletContainerFactory。

**3.自定義內嵌servlet容器**

常見的Servlet容器設置可以通過Spring Environment屬性進行配置。通常，你會把這些屬性定義到application.properties文件中。
常見的服務器設置包括：

1. server.port - 進來的HTTP請求的監聽端口號
2. server.address - 綁定的接口地址
3. server.sessionTimeout - session超時時間

具體參考[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)。

**程式方式的自定義**

如果需要以程式的方式配置內嵌的servlet容器，你可以註冊一個實現EmbeddedServletContainerCustomizer接口的Spring bean。EmbeddedServletContainerCustomizer提供對ConfigurableEmbeddedServletContainer的訪問，ConfigurableEmbeddedServletContainer包含很多自定義的setter函式。
```java
import org.springframework.boot.context.embedded.*;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements EmbeddedServletContainerCustomizer {
    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.setPort(9000);
    }
}
```
**直接自定義ConfigurableEmbeddedServletContainer**

如果上麵的自定義手法過於受限，你可以自己註冊TomcatEmbeddedServletContainerFactory，JettyEmbeddedServletContainerFactory或UndertowEmbeddedServletContainerFactory。
```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory factory = new TomcatEmbeddedServletContainerFactory();
    factory.setPort(9000);
    factory.setSessionTimeout(10, TimeUnit.MINUTES);
    factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html");
    return factory;
}
```
很多可選的配置都提供了setter函式，也提供了一些受保護的鉤子函式以滿足你的某些特殊需求。具體參考相關文件。

**4.JSP的限製**

在內嵌的servlet容器中運行一個Spring Boot應用時（並打包成一個可執行的存檔archive），容器對JSP的支援有一些限製。

1. tomcat隻支援war的打包方式，不支援可執行的jar。
2. 內嵌的Jetty目前不支援JSPs。
3. Undertow不支援JSPs。

這裡有個[JSP範例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-jsp)，你可以查看如何設置相關事項。

### 安全
如果Spring Security在classpath下，那麼web應用預設對所有的HTTP路徑（也稱為終點，端點，表示API的具體網址）使用'basic'認證。為了給web應用添加函式級別的保護，你可以添加@EnableGlobalMethodSecurity並使用想要的設置。其他訊息參考[Spring Security Reference](http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle#jc-method)。

預設的AuthenticationManager有一個單一的user（'user'的用戶名和隨機密碼會在應用啟動時以INFO日誌級別列印出來）。如下：
```java
Using default security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```
**注**：如果你對日誌配置進行微調，確保`org.springframework.boot.autoconfigure.security`類別能記錄INFO訊息，否則預設的密碼不會被列印。

你可以通過提供`security.user.password`改變預設的密碼。這些和其他有用的屬性通過[SecurityProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/SecurityProperties.java)（以security為前綴的屬性）被外部化了。

預設的安全配置（security configuration）是在SecurityAutoConfiguration和導入的類中實現的（SpringBootWebSecurityConfiguration用於web安全，AuthenticationManagerConfiguration用於與非web應用也相關的認證配置）。你可以添加一個@EnableWebSecurity bean來徹底關掉Spring Boot的預設配置。為了對它進行自定義，你需要使用外部的屬性配置和WebSecurityConfigurerAdapter類型的beans（比如，添加基於表單的登陸）。在[Spring Boot範例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/)裡有一些安全相關的應用可以帶你體驗常見的用例。

在一個web應用中你能得到的基本特性如下：

1. 一個使用內存存儲的AuthenticationManager bean和唯一的user（查看SecurityProperties.User獲取user的屬性）。
2. 忽略（不保護）常見的靜態資源路徑（`/css/**, /js/**, /images/**`和 `**/favicon.ico`）。
3. 對其他的路徑實施HTTP Basic安全保護。
4. 安全相關的事件會發佈到Spring的ApplicationEventPublisher（成功和失敗的認證，拒絕訪問）。
5. Spring Security提供的常見底層特性（HSTS, XSS, CSRF, 緩存）預設都被開啟。

上述所有特性都能打開和關閉，或使用外部的配置進行修改（security.*）。為了覆蓋訪問規則（access rules）而不改變其他自動配置的特性，你可以添加一個使用@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)註解的WebSecurityConfigurerAdapter類型的@Bean。

如果Actuator也在使用，你會發現：

1. 即使應用路徑不受保護，被管理的路徑也會受到保護。
2. 安全相關的事件被轉換為AuditEvents（審計事件），並發佈給AuditService。
3. 預設的用戶有ADMIN和USER的角色。

使用外部屬性能夠修改Actuator（執行器）的安全特性（management.security.*）。為了覆蓋應用程序的訪問規則，你可以添加一個WebSecurityConfigurerAdapter類型的@Bean。同時，如果不想覆蓋執行器的訪問規則，你可以使用@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)註解該bean，否則使用@Order(ManagementServerProperties.ACCESS_OVERRIDE_ORDER)註解該bean。

### 使用SQL數據函式庫
Spring框架為使用SQL數據函式庫提供了廣泛的支援。從使用JdbcTemplate直接訪問JDBC到完全的對象關係映射技術，比如Hibernate。Spring Data提供一個額外的功能，直接從接口建立Repository實現，並使用約定從你的函式名生成查詢。

* 配置DataSource

Java的javax.sql.DataSource接口提供了一個標準的使用數據函式庫連接的函式。傳統做法是，一個DataSource使用一個URL連同相應的證書去初始化一個數據函式庫連接。

**1.對內嵌數據函式庫的支援**

開發應用時使用內存數據函式庫是很實用的。顯而易見地，內存數據函式庫不需要提供持久化存儲。你不需要在應用啟動時填充數據函式庫，也不需要在應用結束時丟棄數據。

Spring Boot可以自動配置的內嵌數據函式庫包括[H2](http://www.h2database.com/), [HSQL](http://hsqldb.org/)和[Derby](http://db.apache.org/derby/)。你不需要提供任何連接URLs，只需要簡單的添加你想使用的內嵌數據函式庫依賴。

範例：典型的POM依賴如下：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
```
**注**：對於自動配置的內嵌數據函式庫，你需要依賴spring-jdbc。在範例中，它通過`spring-boot-starter-data-jpa`被傳遞地拉過來了。

**2.連接到一個生產環境數據函式庫**

在生產環境中，數據函式庫連接可以使用DataSource池進行自動配置。下面是選取一個特定實現的算法：

- 由於Tomcat數據源連接池的性能和並發，在tomcat可用時，我們總是優先使用它。
- 如果HikariCP可用，我們將使用它。
- 如果Commons DBCP可用，我們將使用它，但在生產環境不推薦使用它。
- 最後，如果Commons DBCP2可用，我們將使用它。

如果你使用spring-boot-starter-jdbc或spring-boot-starter-data-jpa 'starter POMs'，你將會自動獲取對tomcat-jdbc的依賴。

**注**：其他的連接池可以手動配置。如果你定義自己的DataSource bean，自動配置不會發生。

DataSource配置通過外部配置文件的spring.datasource.*屬性控製。範例中，你可能會在application.properties中聲明下面的片段：
```java
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```
其他可選的配置可以查看[DataSourceProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java)。同時注意你可以通過spring.datasource.*配置任何DataSource實現相關的特定屬性：具體參考你使用的連接池實現的文件。

**注**：既然Spring Boot能夠從大多數數據函式庫的url上推斷出driver-class-name，那麼你就不需要再指定它了。對於一個將要建立的DataSource連接池，我們需要能夠驗證Driver是否可用，所以我們會在做任何事情之前檢查它。比如，如果你設置spring.datasource.driverClassName=com.mysql.jdbc.Driver，然後這個類就會被加載。

**3.連接到一個JNDI數據函式庫**

如果正在將Spring Boot應用部署到一個應用服務器，你可能想要用應用服務器內建的特性來配置和管理你的DataSource，並使用JNDI訪問它。

spring.datasource.jndi-name屬性可以用來替代spring.datasource.url，spring.datasource.username和spring.datasource.password去從一個特定的JNDI路徑訪問DataSource。比如，下面application.properties中的片段展示了如何獲取JBoss定義的DataSource：
```java
spring.datasource.jndi-name=java:jboss/datasources/customers
```
* 使用JdbcTemplate

Spring的JdbcTemplate和NamedParameterJdbcTemplate類是被自動配置的，你可以在自己的beans中通過@Autowire直接注入它們。
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    // ...
}
```
* JPA和Spring Data

Java持久化API是一個允許你將對象映射為關係數據函式庫的標準技術。spring-boot-starter-data-jpa POM提供了一種快速上手的方式。它提供下列關鍵的依賴：

- Hibernate - 一個非常流行的JPA實現。
- Spring Data JPA - 讓實現基於JPA的repositories更容易。
- Spring ORMs - Spring框架的核心ORM支援。

**注**：我們不想在這涉及太多關於JPA或Spring Data的細節。你可以參考來自[spring.io](http://spring.io/)的指南[使用JPA獲取數據](http://spring.io/guides/gs/accessing-data-jpa/)，並閱讀[Spring Data JPA](http://projects.spring.io/spring-data-jpa/)和[Hibernate](http://hibernate.org/orm/documentation/)的參考文件。

**1.實體類**

傳統上，JPA實體類被定義到一個persistence.xml文件中。在Spring Boot中，這個文件不是必需的，並被'實體掃描'替代。預設情況下，在你主（main）配置類（被@EnableAutoConfiguration或@SpringBootApplication註解的類）下的所有包都將被查找。

任何被@Entity，@Embeddable或@MappedSuperclass註解的類都將被考慮。一個普通的實體類看起來像下面這樣：
```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String state;

    // ... additional members, often include @OneToMany mappings

    protected City() {
        // no-args constructor required by JPA spec
        // this one is protected since it shouldn't be used directly
    }

    public City(String name, String state) {
        this.name = name;
        this.country = country;
    }

    public String getName() {
        return this.name;
    }

    public String getState() {
        return this.state;
    }
    // ... etc
}
```
**注**：你可以使用@EntityScan註解自定義實體掃描路徑。具體參考[Section 67.4, “Separate @Entity definitions from Spring configuration”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-separate-entity-definitions-from-spring-configuration)。

**2.Spring Data JPA倉函式庫**

Spring Data JPA倉函式庫（repositories）是用來定義訪問數據的接口。根據你的函式名，JPA查詢會被自動建立。比如，一個CityRepository接口可能聲明一個findAllByState(String state)函式，用來查找給定狀態的所有城市。

對於比較複雜的查詢，你可以使用Spring Data的[Query](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html)來註解你的函式。

Spring Data倉函式庫通常繼承自[Repository](http://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html)或[CrudRepository](http://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)接口。如果你使用自動配置，包括在你的主配置類（被@EnableAutoConfiguration或@SpringBootApplication註解的類）的包下的倉函式庫將會被搜索。
 
下面是一個傳統的Spring Data倉函式庫：
```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndCountryAllIgnoringCase(String name, String country);
}
```
**注**：我們僅僅觸及了Spring Data JPA的表麵。具體查看它的[參考指南](http://projects.spring.io/spring-data-jpa/)。

**3.建立和刪除JPA數據函式庫**

預設情況下，隻有在你使用內嵌數據函式庫（H2, HSQL或Derby）時，JPA數據函式庫才會被自動建立。你可以使用spring.jpa.*屬性顯示的設置JPA。比如，為了建立和刪除表你可以將下面的配置添加到application.properties中：
```java
spring.jpa.hibernate.ddl-auto=create-drop
```
**注**：Hibernate自己內部對建立，刪除表支援（如果你恰好記得這回事更好）的屬性是hibernate.hbm2ddl.auto。使用spring.jpa.properties.*（前綴在被添加到實體管理器之前會被剝離掉），你可以設置Hibernate本身的屬性，比如hibernate.hbm2ddl.auto。範例：`spring.jpa.properties.hibernate.globally_quoted_identifiers=true`將傳遞hibernate.globally_quoted_identifiers到Hibernate實體管理器。

預設情況下，DDL執行（或驗證）被延遲到ApplicationContext啟動。這也有一個spring.jpa.generate-ddl標識，如果Hibernate自動配置被啟動，那該標識就不會被使用，因為ddl-auto設置粒度更細。

### 使用NoSQL技術
Spring Data提供其他項目，用來幫你使用各種各樣的NoSQL技術，包括[MongoDB](http://projects.spring.io/spring-data-mongodb/), [Neo4J](http://projects.spring.io/spring-data-neo4j/), [Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch/), [Solr](http://projects.spring.io/spring-data-solr/), [Redis](http://projects.spring.io/spring-data-redis/), [Gemfire](http://projects.spring.io/spring-data-gemfire/), [Couchbase](http://projects.spring.io/spring-data-couchbase/)和[Cassandra](http://projects.spring.io/spring-data-cassandra/)。Spring Boot為Redis, MongoDB, Elasticsearch, Solr和Gemfire提供自動配置。你可以充分利用其他項目，但你需要自己配置它們。具體查看[projects.spring.io/spring-data.](http://projects.spring.io/spring-data/)中合適的參考文件。

* Redis

[Redis](http://redis.io/)是一個緩存，訊息中間件及具有豐富特性的鍵值存儲系統。Spring Boot為[Jedis](https://github.com/xetorthio/jedis/)客戶端函式庫和由[Spring Data Redis](https://github.com/spring-projects/spring-data-redis)提供的基於Jedis客戶端的抽象提供自動配置。`spring-boot-starter-redis`'Starter POM'為收集依賴提供一種便利的方式。

1. 連接Redis

你可以注入一個自動配置的RedisConnectionFactory，StringRedisTemplate或普通的跟其他Spring Bean相同的RedisTemplate實例。預設情況下，這個實例將嘗試使用localhost:6379連接Redis服務器。
```java
@Component
public class MyBean {

    private StringRedisTemplate template;

    @Autowired
    public MyBean(StringRedisTemplate template) {
        this.template = template;
    }
    // ...
}
```
如果你添加一個你自己的任何自動配置類型的@Bean，它將替換預設的（除了RedisTemplate的情況，它是根據bean的名稱'redisTemplate'而不是它的類型進行排除的）。如果在classpath路徑下存在commons-pool2，預設你會獲得一個連接池工廠。

* MongoDB

[MongoDB](http://www.mongodb.com/)是一個開源的NoSQL文件數據函式庫，它使用一個JSON格式的模式（schema）替換了傳統的基於表的關係數據。Spring Boot為使用MongoDB提供了很多便利，包括`spring-boot-starter-data-mongodb`'Starter POM'。

**1. 連接MongoDB數據函式庫**

你可以注入一個自動配置的`org.springframework.data.mongodb.MongoDbFactory`來訪問Mongo數據函式庫。預設情況下，該實例將嘗試使用URL：`mongodb://localhost/test`連接一個MongoDB服務器。
```java
import org.springframework.data.mongodb.MongoDbFactory;
import com.mongodb.DB;

@Component
public class MyBean {

    private final MongoDbFactory mongo;

    @Autowired
    public MyBean(MongoDbFactory mongo) {
        this.mongo = mongo;
    }

    // ...
    public void example() {
        DB db = mongo.getDb();
        // ...
    }
}
```
你可以通過設置`spring.data.mongodb.uri`來改變該url，或指定一個host/port。比如，你可能會在你的application.properties中設置如下的屬性：
```java
spring.data.mongodb.host=mongoserver
spring.data.mongodb.port=27017
```
**注**：如果沒有指定`spring.data.mongodb.port`，那將使用預設的端口27017。你可以簡單的從上麵的範例中刪除這一行。如果不使用Spring Data Mongo，你可以注入com.mongodb.Mongo beans而不是使用MongoDbFactory。

如果想全麵控製MongoDB連接的建立，你也可以聲明自己的MongoDbFactory或Mongo，@Beans。

**2. MongoDBTemplate**

Spring Data Mongo提供了一個[MongoTemplate](http://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html)類，它的設計和Spring的JdbcTemplate很相似。正如JdbcTemplate一樣，Spring Boot會為你自動配置一個bean，你只需簡單的注入它即可：
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final MongoTemplate mongoTemplate;

    @Autowired
    public MyBean(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }
    // ...
}
```
具體參考MongoOperations Javadoc。

**3. Spring Data MongoDB倉函式庫**

Spring Data的倉函式庫包括對MongoDB的支援。正如上麵討論的JPA倉函式庫，基本的原則是查詢會自動基於你的函式名建立。

實際上，不管是Spring Data JPA還是Spring Data MongoDB都共享相同的基礎設施。所以你可以使用上麵的JPA範例，並假設那個City現在是一個Mongo數據類而不是JPA　@Entity，它將以同樣的方式工作。
```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndCountryAllIgnoringCase(String name, String country);

}
```
* Gemfire

[Spring Data Gemfire](https://github.com/spring-projects/spring-data-gemfire)為使用[Pivotal Gemfire](http://www.pivotal.io/big-data/pivotal-gemfire#details)數據管理平台提供了方便的，Spring友好的工具。Spring Boot提供了一個用於聚集依賴的`spring-boot-starter-data-gemfire`'Starter POM'。目前不支援Gemfire的自動配置，但你可以使用一個[單一的註解](https://github.com/spring-projects/spring-data-gemfire/blob/master/src/main/java/org/springframework/data/gemfire/repository/config/EnableGemfireRepositories.java)使Spring Data倉函式庫支援它。

* Solr

[Apache Solr](http://lucene.apache.org/solr/)是一個搜索引擎。Spring Boot為solr客戶端函式庫及[Spring Data Solr](https://github.com/spring-projects/spring-data-solr)提供的基於solr客戶端函式庫的抽象提供了基本的配置。Spring Boot提供了一個用於聚集依賴的`spring-boot-starter-data-solr`'Starter POM'。
 
**1. 連接Solr**

你可以像其他Spring beans一樣注入一個自動配置的SolrServer實例。預設情況下，該實例將嘗試使用`localhost:8983/solr`連接一個服務器。
```java
@Component
public class MyBean {

    private SolrServer solr;

    @Autowired
    public MyBean(SolrServer solr) {
        this.solr = solr;
    }
    // ...
}
```
如果你添加一個自己的SolrServer類型的@Bean，它將會替換預設的。

**2. Spring Data Solr倉函式庫**

Spring Data的倉函式庫包括了對Apache Solr的支援。正如上麵討論的JPA倉函式庫，基本的原則是查詢會自動基於你的函式名建立。

實際上，不管是Spring Data JPA還是Spring Data Solr都共享相同的基礎設施。所以你可以使用上麵的JPA範例，並假設那個City現在是一個@SolrDocument類而不是JPA　@Entity，它將以同樣的方式工作。

**注**：具體參考[Spring Data Solr文件](http://projects.spring.io/spring-data-solr/)。

* Elasticsearch

[Elastic Search](http://www.elasticsearch.org/)是一個開源的，分布式，實時搜索和分析引擎。Spring Boot為Elasticsearch及[Spring Data Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch)提供的基於它的抽象提供了基本的配置。Spring Boot提供了一個用於聚集依賴的`spring-boot-starter-data-elasticsearch`'Starter POM'。
  
**1. 連接Elasticsearch**

你可以像其他Spring beans那樣注入一個自動配置的ElasticsearchTemplate或Elasticsearch客戶端實例。預設情況下，該實例將嘗試連接到一個本地內存服務器（在Elasticsearch項目中的一個NodeClient），但你可以通過設置`spring.data.elasticsearch.clusterNodes`為一個以逗號分割的host:port列表來將其切換到一個遠程服務器（比如，TransportClient）。
```java
@Component
public class MyBean {

    private ElasticsearchTemplate template;

    @Autowired
    public MyBean(ElasticsearchTemplate template) {
        this.template = template;
    }
    // ...
}
```
如果你添加一個你自己的ElasticsearchTemplate類型的@Bean，它將替換預設的。

**2. Spring Data Elasticseach倉函式庫**

Spring Data的倉函式庫包括了對Elasticsearch的支援。正如上麵討論的JPA倉函式庫，基本的原則是查詢會自動基於你的函式名建立。

實際上，不管是Spring Data JPA還是Spring Data　Elasticsearch都共享相同的基礎設施。所以你可以使用上麵的JPA範例，並假設那個City現在是一個Elasticsearch @Document類而不是JPA　@Entity，它將以同樣的方式工作。

**注**：具體參考[Spring Data Elasticsearch文件](http://docs.spring.io/spring-data/elasticsearch/docs/)。
  
### 訊息

Spring Framework框架為整合訊息系統提供了擴展（extensive）支援：從使用JmsTemplate簡化JMS　API，到實現一個完整異步訊息接收的底層設施。Spring AMQP提供一個相似的用於'進階訊息隊列協議'的特徵集，並且Spring Boot也為RabbitTemplate和RabbitMQ提供了自動配置選項。Spring Websocket提供原生的STOMP訊息支援，並且Spring Boot通過starters和一些自動配置也提供了對它的支援。

* JMS

javax.jms.ConnectionFactory接口提供了一個標準的用於建立一個javax.jms.Connection的函式，javax.jms.Connection用於和JMS代理（broker）交互。儘管為了使用JMS，Spring需要一個ConnectionFactory，但通常你不需要直接使用它，而是依賴於上層訊息抽象（具體參考Spring框架的[相關章節](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#jms)）。Spring Boot也會自動配置發送和接收訊息需要的設施（infrastructure）。
 
 1. HornetQ支援

如果在classpath下發現HornetQ，Spring Boot會自動配置ConnectionFactory。如果需要代理，將會開啟一個內嵌的，已經自動配置好的代理（除非顯式設置mode屬性）。支援的modes有：embedded（顯式聲明使用一個內嵌的代理，如果該代理在classpath下不可用將導致一個錯誤），native（使用netty傳輸協議連接代理）。當後者被配置，Spring Boot配置一個連接到一個代理的ConnectionFactory，該代理運行在使用預設配置的本地機器上。

**注**：如果使用spring-boot-starter-hornetq，連接到一個已存在的HornetQ實例所需的依賴都會被提供，同時還有用於整合JMS的Spring基礎設施。將org.hornetq:hornetq-jms-server添加到你的應用中，你就可以使用embedded模式。

HornetQ配置被spring.hornetq.*中的外部配置屬性所控製。例如，你可能在application.properties聲明以下片段：
```java
spring.hornetq.mode=native
spring.hornetq.host=192.168.1.210
spring.hornetq.port=9876
```
當內嵌代理時，你可以選擇是否啟用持久化，並且列表中的目標都應該是可用的。這些可以通過一個以逗號分割的列表來指定一些預設的配置項，或定義org.hornetq.jms.server.config.JMSQueueConfiguration或org.hornetq.jms.server.config.TopicConfiguration類型的bean(s)來配置更進階的隊列和主題。具體參考[HornetQProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/hornetq/HornetQProperties.java)。

沒有涉及JNDI查找，目標是通過名字解析的，名字即可以使用HornetQ配置中的name屬性，也可以是配置中提供的names。

  2. ActiveQ支援

如果發現ActiveMQ在classpath下可用，Spring Boot會配置一個ConnectionFactory。如果需要代理，將會開啟一個內嵌的，已經自動配置好的代理（隻要配置中沒有指定代理URL）。

ActiveMQ配置是通過spring.activemq.*中的外部配置來控製的。例如，你可能在application.properties中聲明下面的片段：
```java
spring.activemq.broker-url=tcp://192.168.1.210:9876
spring.activemq.user=admin
spring.activemq.password=secret
```
具體參考[ActiveMQProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java)。

預設情況下，如果目標還不存在，ActiveMQ將建立一個，所以目標是通過它們提供的名稱解析出來的。

  3. 使用JNDI ConnectionFactory

如果你在一個應用服務器中運行你的應用，Spring Boot將嘗試使用JNDI定位一個JMS ConnectionFactory。預設情況會檢查java:/JmsXA和java:/
XAConnectionFactory。如果需要的話，你可以使用spring.jms.jndi-name屬性來指定一個替代位置。
```java
spring.jms.jndi-name=java:/MyConnectionFactory
```
  4. 發送訊息

Spring的JmsTemplate會被自動配置，你可以將它直接注入到你自己的beans中：
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;
@Component
public class MyBean {
private final JmsTemplate jmsTemplate;
@Autowired
public MyBean(JmsTemplate jmsTemplate) {
this.jmsTemplate = jmsTemplate;
}
// ...
}
```

**注**：[JmsMessagingTemplate](http://docs.spring.io/spring/docs/4.1.4.RELEASE/javadoc-api/org/springframework/jms/core/JmsMessagingTemplate.html)(Spring4.1新增的)也可以使用相同的方式注入。

  5. 接收訊息

當JMS基礎設施能夠使用時，任何bean都能夠被@JmsListener註解，以建立一個監聽者端點。如果沒有定義JmsListenerContainerFactory，一個預設的將會被自動配置。下面的組件在someQueue目標上建立一個監聽者端點。
```java
@Component
public class MyBean {
@JmsListener(destination = "someQueue")
public void processMessage(String content) {
// ...
}
}
```
具體查看[@EnableJms javadoc](http://docs.spring.io/spring/docs/4.1.4.RELEASE/javadoc-api/org/springframework/jms/annotation/EnableJms.html)。

### 發送郵件

Spring框架使用JavaMailSender接口為發送郵件提供了一個簡單的抽象，並且Spring Boot也為它提供了自動配置和一個starter模塊。
具體查看[JavaMailSender參考文件](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#mail)。

如果spring.mail.host和相關的函式庫（通過spring-boot-starter-mail定義）都存在，一個預設的JavaMailSender將被建立。該sender可以通過spring.mail命名空間下的配置項進一步自定義，具體參考[MailProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mail/MailProperties.java)。

### 使用JTA處理分布式事務

Spring Boot使用一個[Atomkos](http://www.atomikos.com/)或[Bitronix](http://docs.codehaus.org/display/BTM/Home)的內嵌事務管理器來支援跨多個XA資源的分布式JTA事務。當部署到一個恰當的J2EE應用服務器時也會支援JTA事務。

當發現一個JTA環境時，Spring Boot將使用Spring的JtaTransactionManager來管理事務。自動配置的JMS，DataSource和JPA　beans將被升級以支援XA事務。你可以使用標準的Spring idioms，比如@Transactional，來參與到一個分布式事務中。如果你處於JTA環境裡，但仍舊想使用本地事務，你可以將spring.jta.enabled屬性設置為false來禁用JTA自動配置功能。

* 使用一個Atomikos事務管理器

Atomikos是一個非常流行的開源事務管理器，它可以嵌入到你的Spring Boot應用中。你可以使用`spring-boot-starter-jta-atomikos`Starter POM去獲取正確的Atomikos函式庫。Spring Boot會自動配置Atomikos，並將合適的depends-on應用到你的Spring Beans上，確保它們以正確的順序啟動和關閉。

預設情況下，Atomikos事務日誌將被記錄在應用home目錄（你的應用jar文件放置的目錄）下的transaction-logs文件夾中。你可以在application.properties文件中通過設置spring.jta.log-dir屬性來自定義該目錄。以spring.jta.開頭的屬性能用來自定義Atomikos的UserTransactionServiceIml實現。具體參考[AtomikosProperties javadoc](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/jta/atomikos/AtomikosProperties.html)。

**注**：為了確保多個事務管理器能夠安全地和相應的資源管理器配合，每個Atomikos實例必須設置一個唯一的ID。預設情況下，該ID是Atomikos實例運行的機器上的IP地址。為了確保生產環境中該ID的唯一性，你需要為應用的每個實例設置不同的spring.jta.transaction-manager-id屬性值。

* 使用一個Bitronix事務管理器

Bitronix是另一個流行的開源JTA事務管理器實現。你可以使用`spring-boot-starter-jta-bitronix`starter POM為項目添加合適的Birtronix依賴。和Atomikos類似，Spring Boot將自動配置Bitronix，並對beans進行後處理（post-process）以確保它們以正確的順序啟動和關閉。

預設情況下，Bitronix事務日誌將被記錄到應用home目錄下的transaction-logs文件夾中。通過設置spring.jta.log-dir屬性，你可以自定義該目錄。以spring.jta.開頭的屬性將被綁定到bitronix.tm.Configuration　bean，你可以通過這完成進一步的自定義。具體參考[Bitronix文件](http://btm.codehaus.org/api/2.0.1/bitronix/tm/Configuration.html)。

**注**：為了確保多個事務管理器能夠安全地和相應的資源管理器配合，每個Bitronix實例必須設置一個唯一的ID。預設情況下，該ID是Bitronix實例運行的機器上的IP地址。為了確保生產環境中該ID的唯一性，你需要為應用的每個實例設置不同的spring.jta.transaction-manager-id屬性值。

* 使用一個J2EE管理的事務管理器

如果你將Spring Boot應用打包為一個war或ear文件，並將它部署到一個J2EE的應用服務器中，那你就能使用應用服務器內建的事務管理器。Spring Boot將嘗試通過查找常見的JNDI路徑（java:comp/UserTransaction, java:comp/TransactionManager等）來自動配置一個事務管理器。如果使用應用服務器提供的事務服務，你通常需要確保所有的資源都被應用服務器管理，並通過JNDI曝露出去。Spring Boot通過查找JNDI路徑java:/JmsXA或java:/XAConnectionFactory獲取一個ConnectionFactory來自動配置JMS，並且你可以使用spring.datasource.jndi-name屬性配置你的DataSource。

* 混合XA和non-XA的JMS連接

當使用JTA時，主要的JMS ConnectionFactory bean將是XA　aware，並參與到分布式事務中。有些情況下，你可能需要使用non-XA的ConnectionFactory去處理一些JMS訊息。例如，你的JMS處理邏輯可能比XA超時時間長。

如果想使用一個non-XA的ConnectionFactory，你可以注入nonXaJmsConnectionFactory　bean而不是@Primary jmsConnectionFactory　bean。為了保持一致，jmsConnectionFactory　bean將以別名xaJmsConnectionFactor來被使用。

範例如下：
```java
// Inject the primary (XA aware) ConnectionFactory
@Autowired
private ConnectionFactory defaultConnectionFactory;
// Inject the XA aware ConnectionFactory (uses the alias and injects the same as above)
@Autowired
@Qualifier("xaJmsConnectionFactory")
private ConnectionFactory xaConnectionFactory;
// Inject the non-XA aware ConnectionFactory
@Autowired
@Qualifier("nonXaJmsConnectionFactory")
private ConnectionFactory nonXaConnectionFactory;
```
* 支援可替代的內嵌事務管理器

[XAConnectionFactoryWrapper](http://github.com/spring-projects/spring-boot/tree/master/spring-boot/src/main/java/org/springframework/boot/jta/XAConnectionFactoryWrapper.java)和[XADataSourceWrapper](http://github.com/spring-projects/spring-boot/tree/master/spring-boot/src/main/java/org/springframework/boot/jta/XADataSourceWrapper.java)接口用於支援可替換的內嵌事務管理器。該接口用於包裝XAConnectionFactory和XADataSource　beans，並將它們曝露為普通的ConnectionFactory和DataSource　beans，這樣在分布式事務中可以透明使用。

### Spring整合

Spring整合提供基於訊息和其他協議的，比如HTTP，TCP等的抽象。如果Spring整合在classpath下可用，它將會通過@EnableIntegration註解被初始化。如果classpath下'spring-integration-jmx'可用，則訊息處理統計分析將被通過JMX發佈出去。具體參考[IntegrationAutoConfiguration類](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationAutoConfiguration.java)。

### 基於JMX的監控和管理

Java管理擴展（JMX）提供了一個標準的用於監控和管理應用的機製。預設情況下，Spring Boot將建立一個id為‘mbeanServer’的MBeanServer，並導出任何被Spring JMX註解（@ManagedResource,@ManagedAttribute,@ManagedOperation）的beans。具體參考[JmxAutoConfiguration類](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.java)。

### 測試

Spring Boot提供很多有用的測試應用的工具。spring-boot-starter-test POM提供Spring Test，JUnit，Hamcrest和Mockito的依賴。在spring-boot核心模塊org.springframework.boot.test包下也有很多有用的測試工具。

* 測試作用域依賴

如果使用spring-boot-starter-test ‘Starter POM’（在test作用域內），你將發現下列被提供的函式庫：
- Spring Test - 對Spring應用的整合測試支援
- JUnit - 事實上的（de-facto）標準，用於Java應用的單元測試。
- Hamcrest - 一個匹配對象的函式庫（也稱為約束或前置條件），它允許assertThat等JUnit類型的斷言。
- Mockito - 一個Java模擬框架。

這也有一些我們寫測試用例時經常用到的函式庫。如果它們不能滿足你的要求，你可以隨意添加其他的測試用的依賴函式庫。

* 測試Spring應用

依賴注入最大的優點就是它能夠讓你的程式碼更容易進行單元測試。你只需簡單的通過new操作符實例化對象，而不需要涉及Spring。你也可以使用模擬對象替換真正的依賴。

你常常需要在進行單元測試後，開始整合測試（在這個過程中只需要涉及到Spring的ApplicationContext）。在執行整合測試時，不需要部署應用或連接到其他基礎設施是非常有用的。

Spring框架包含一個dedicated測試模塊，用於這樣的整合測試。你可以直接聲明對org.springframework:spring-test的依賴，或使用spring-boot-starter-test ‘Starter POM’以透明的方式拉取它。

如果你以前沒有使用過spring-test模塊，可以查看Spring框架參考文件中的[相關章節](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#testing)。

* 測試Spring Boot應用

一個Spring Boot應用只是一個Spring ApplicationContext，所以在測試它時除了正常情況下處理一個vanilla Spring　context外不需要做其他特別事情。唯一需要注意的是，如果你使用SpringApplication建立上下文，外部配置，日誌和Spring Boot的其他特性隻會在預設的上下文中起作用。

Spring Boot提供一個@SpringApplicationConfiguration註解用來替換標準的spring-test　@ContextConfiguration註解。如果使用@SpringApplicationConfiguration來設置你的測試中使用的ApplicationContext，它最終將通過SpringApplication建立，並且你將獲取到Spring Boot的其他特性。

範例如下：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = SampleDataJpaApplication.class)
public class CityRepositoryIntegrationTests {
@Autowired
CityRepository repository;
// ...
}	
```
**提示**：上下文加載器會通過查找@WebIntegrationTest或@WebAppConfiguration註解來猜測你想測試的是否是web應用（例如，是否使用MockMVC，MockMVC和@WebAppConfiguration是spring-test的一部分）。

如果想讓一個web應用啟動，並監聽它的正常的端口，你可以使用HTTP來測試它（比如，使用RestTemplate），並使用@WebIntegrationTest註解你的測試類（或它的一個父類）。這很有用，因為它意味著你可以對你的應用進行全棧測試，但在一次HTTP交互後，你需要在你的測試類中注入相應的組件並使用它們斷言應用的內部狀態。

範例：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = SampleDataJpaApplication.class)
@WebIntegrationTest
public class CityRepositoryIntegrationTests {
@Autowired
CityRepository repository;
RestTemplate restTemplate = new TestRestTemplate();
// ... interact with the running server
}
```
**注**：Spring測試框架在每次測試時會緩存應用上下文。因此，隻要你的測試共享相同的配置，不管你實際運行多少測試，開啟和停止服務器隻會發生一次。

你可以為@WebIntegrationTest添加環境變量屬性來改變應用服務器端口號，比如@WebIntegrationTest("server.port:9000")。此外，你可以將server.port和management.port屬性設置為０來讓你的整合測試使用隨機的端口號，例如：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MyApplication.class)
@WebIntegrationTest({"server.port=0", "management.port=0"})
public class SomeIntegrationTests {
// ...
}
```
可以查看[Section 64.4, “Discover the HTTP port at runtime”]()，它描述了如何在測試期間發現分配的實際端口。

1. 使用Spock測試Spring Boot應用

如果期望使用Spock測試一個Spring Boot應用，你應該將Spock的spock-spring模塊依賴添加到應用的建構中。spock-spring將Spring的測試框架整合到了Spock裡。

注意你不能使用上述提到的@SpringApplicationConfiguration註解，因為[Spock找不到@ContextConfiguration元註解](https://code.google.com/p/spock/issues/detail?id=349)。為了繞過該限製，你應該直接使用@ContextConfiguration註解，並使用Spring Boot特定的上下文加載器來配置它。
```java
@ContextConfiguration(loader = SpringApplicationContextLoader.class)
class ExampleSpec extends Specification {
// ...
}
```
**注**：上麵描述的註解在Spock中可以使用，比如，你可以使用@WebIntegrationTest註解你的Specification以滿足測試需要。

* 測試工具

打包進spring-boot的一些有用的測試工具類。

1. ConfigFileApplicationContextInitializer

ConfigFileApplicationContextInitializer是一個ApplicationContextInitializer，可以用來測試加載Spring Boot的application.properties文件。當不需要使用@SpringApplicationConfiguration提供的全部特性時，你可以使用它。

```java
@ContextConfiguration(classes = Config.class,initializers = ConfigFileApplicationContextInitializer.class)
```　　
2. EnvironmentTestUtils

EnvironmentTestUtils允許你快速添加屬性到一個ConfigurableEnvironment或ConfigurableApplicationContext。只需簡單的使用key=value字符串調用它：
```java
EnvironmentTestUtils.addEnvironment(env, "org=Spring", "name=Boot");
```
3. OutputCapture

OutputCapture是一個JUnit Rule，用於捕獲System.out和System.err輸出。只需簡單的將捕獲聲明為一個@Rule，並使用toString()斷言：
```java
import org.junit.Rule;
import org.junit.Test;
import org.springframework.boot.test.OutputCapture;
import static org.hamcrest.Matchers.*;
import static org.junit.Assert.*;

public class MyTest {
@Rule
public OutputCapture capture = new OutputCapture();
@Test
public void testName() throws Exception {
System.out.println("Hello World!");
assertThat(capture.toString(), containsString("World"));
}
}
```

4. TestRestTemplate

TestRestTemplate是一個方便進行整合測試的Spring RestTemplate子類。你會獲取到一個普通的模板或一個發送基本HTTP認證（使用用戶名和密碼）的模板。在任何情況下，這些模板都表現出對測試友好：不允許重定向（這樣你可以對響應地址進行斷言），忽略cookies（這樣模板就是無狀態的），對於服務端錯誤不會拋出異常。推薦使用Apache HTTP Client(4.3.2或更好的版本)，但不強製這樣做。如果在classpath下存在Apache HTTP Client，TestRestTemplate將以正確配置的client進行響應。
```java
public class MyTest {
RestTemplate template = new TestRestTemplate();
@Test
public void testRequest() throws Exception {
HttpHeaders headers = template.getForEntity("http://myhost.com", String.class).getHeaders();
assertThat(headers.getLocation().toString(), containsString("myotherhost"));
}
}
```

### 開發自動配置和使用條件

如果你在一個開發者共享函式庫的公司工作，或你在從事一個開源或商業型的函式庫，你可能想要開發自己的auto-configuration。Auto-configuration類能夠在外部的jars中綁定，並仍能被Spring Boot發現。

* 理解auto-configured beans

從底層來講，auto-configured是使用標準的@Configuration實現的類，另外的@Conditional註解用來約束在什麼情況下使用auto-configuration。通常auto-configuration類使用@ConditionalOnClass和@ConditionalOnMissingBean註解。這是為了確保隻有在相關的類被發現，和你沒有聲明自己的@Configuration時才應用auto-configuration。

你可以瀏覽spring-boot-autoconfigure的程式碼，查看我們提供的@Configuration類（查看META-INF/spring.factories文件）。

* 定位auto-configuration候選者

Spring Boot會檢查你發佈的jar中是否存在META-INF/spring.factories文件。該文件應該列出以EnableAutoConfiguration為key的配置類：
```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```
如果配置需要應用特定的順序，你可以使用[@AutoConfigureAfter](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.java)或[@AutoConfigureBefore](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.java)註解。例如，你想提供web-specific配置，你的類就需要應用在WebMvcAutoConfiguration後麵。

* Condition註解

你幾乎總是需要在你的auto-configuration類裡添加一個或更多的@Condition註解。@ConditionalOnMissingBean註解是一個常見的範例，它經常用於允許開發者覆蓋auto-configuration，如果他們不喜歡你提供的預設行為。

Spring Boot包含很多@Conditional註解，你可以在自己的程式碼中通過註解@Configuration類或單獨的@Bean函式來重用它們。

1. Class條件

@ConditionalOnClass和@ConditionalOnMissingClass註解允許根據特定類是否出現來跳過配置。由於註解元數據是使用[ASM](http://asm.ow2.org/)來解析的，你實際上可以使用value屬性來引用真正的類，即使該類可能實際上並沒有出現在運行應用的classpath下。如果你傾向於使用一個String值來指定類名，你也可以使用name屬性。

2. Bean條件

@ConditionalOnBean和@ConditionalOnMissingBean註解允許根據特定beans是否出現來跳過配置。你可以使用value屬性來指定beans（by type），也可以使用name來指定beans（by name）。search屬性允許你限製搜索beans時需要考慮的ApplicationContext的層次。

**注**：當@Configuration類被解析時@Conditional註解會被處理。Auto-configure @Configuration總是最後被解析（在所有用戶定義beans後麵），然而，如果你將那些註解用到常規的@Configuration類，需要注意不能引用那些還沒有建立好的bean定義。

3. Property條件

@ConditionalOnProperty註解允許根據一個Spring Environment屬性來決定是否包含配置。可以使用prefix和name屬性指定要檢查的配置屬性。預設情況下，任何存在的隻要不是false的屬性都會匹配。你也可以使用havingValue和matchIfMissing屬性建立更進階的檢測。

4. Resource條件

@ConditionalOnResource註解允許隻有在特定資源出現時配置才會被包含。資源可以使用常見的Spring約定命名，例如file:/home/user/test.dat。

5. Web Application條件

@ConditionalOnWebApplication和@ConditionalOnNotWebApplication註解允許根據應用是否為一個'web應用'來決定是否包含配置。一個web應用是任何使用Spring WebApplicationContext，定義一個session作用域或有一個StandardServletEnvironment的應用。

6. SpEL表達式條件

@ConditionalOnExpression註解允許根據[SpEL表達式](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#expressions)結果來跳過配置。

### WebSockets

Spring Boot為內嵌的Tomcat(8和7)，Jetty 9和Undertow提供WebSockets自動配置。如果你正在將一個war包部署到一個單獨的容器，Spring Boot會假設該容器會對它的WebSocket支援相關的配置負責。

Spring框架提供[豐富的WebSocket支援](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#websocket)，通過spring-boot-starter-websocket模塊可以輕易獲取到。

