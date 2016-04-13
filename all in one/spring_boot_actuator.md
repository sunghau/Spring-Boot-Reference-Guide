###Spring Boot執行器：Production-ready特性

Spring Boot包含很多其他的特性，它們可以幫你監控和管理發布到生產環境的應用。你可以選擇使用HTTP端點，JMX或遠程shell（SSH或Telnet）來管理和監控應用。審計（Auditing），健康（health）和數據采集（metrics gathering）會自動應用到你的應用。

* 開啟production-ready特性

[spring-boot-actuator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator)模塊提供了Spring Boot所有的production-ready特性。啟用該特性的最簡單方式就是添加對spring-boot-starter-actuator ‘Starter POM’的依賴。

**執行器（Actuator）的定義**：執行器是一個製造業術語，指的是用於移動或控製東西的一個機械裝置。一個很小的改變就能讓執行器產生大量的運動。

基於Maven的項目想要添加執行器隻需添加下面的'starter'依賴：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```
對於Gradle，使用下面的聲明：
```java
dependencies {
    compile("org.springframework.boot:spring-boot-starter-actuator")
}
```
* 端點

執行器端點允許你監控應用及與應用進行交互。Spring Boot包含很多內置的端點，你也可以添加自己的。例如，health端點提供了應用的基本健康信息。

端點暴露的方式取決於你采用的技術類型。大部分應用選擇HTTP監控，端點的ID映射到一個URL。例如，默認情況下，health端點將被映射到/health。

下面的端點都是可用的：

| ID | 描述　|敏感（Sensitive）|
| ---- | :----- | :----- |
|autoconfig|顯示一個auto-configuration的報告，該報告展示所有auto-configuration候選者及它們被應用或未被應用的原因|true|
|beans|顯示一個應用中所有Spring Beans的完整列表|true|
|configprops|顯示一個所有@ConfigurationProperties的整理列表|true|
|dump|執行一個線程轉儲|true|
|env|暴露來自Spring　ConfigurableEnvironment的屬性|true|
|health|展示應用的健康信息（當使用一個未認證連接訪問時顯示一個簡單的'status'，使用認證連接訪問則顯示全部信息詳情）|false|
|info|顯示任意的應用信息|false|
|metrics|展示當前應用的'指標'信息|true|
|mappings|顯示一個所有@RequestMapping路徑的整理列表|true|
|shutdown|允許應用以優雅的方式關閉（默認情況下不啟用）|true|
|trace|顯示trace信息（默認為最新的一些HTTP請求）|true|

**注**：根據一個端點暴露的方式，sensitive參數可能會被用做一個安全提示。例如，在使用HTTP訪問sensitive端點時需要提供用戶名/密碼（如果沒有啟用web安全，可能會簡化為禁止訪問該端點）。

1. 自定義端點

使用Spring屬性可以自定義端點。你可以設置端點是否開啟（enabled），是否敏感（sensitive），甚至它的id。例如，下面的application.properties改變了敏感性和beans端點的id，也啟用了shutdown。
```java
endpoints.beans.id=springbeans
endpoints.beans.sensitive=false
endpoints.shutdown.enabled=true
```
**注**：前綴'endpoints + . + name'被用來唯一的標識被配置的端點。

默認情況下，除了shutdown外的所有端點都是啟用的。如果希望指定選擇端點的啟用，你可以使用endpoints.enabled屬性。例如，下面的配置禁用了除info外的所有端點：
```java
endpoints.enabled=false
endpoints.info.enabled=true
```
2. 健康信息

健康信息可以用來檢查應用的運行狀態。它經常被監控軟件用來提醒人們生產系統是否停止。health端點暴露的默認信息取決於端點是如何被訪問的。對於一個非安全，未認證的連接隻返回一個簡單的'status'信息。對於一個安全或認證過的連接其他詳細信息也會展示（具體參考[Section 41.6, “HTTP Health endpoint access restrictions” ]()）。

健康信息是從你的ApplicationContext中定義的所有[HealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) beans收集過來的。Spring Boot包含很多auto-configured的HealthIndicators，你也可以寫自己的。

3. 安全與HealthIndicators

HealthIndicators返回的信息常常性質上有點敏感。例如，你可能不想將數據庫服務器的詳情發布到外麵。因此，在使用一個未認證的HTTP連接時，默認隻會暴露健康狀態（health status）。如果想將所有的健康信息暴露出去，你可以把endpoints.health.sensitive設置為false。

為防止'拒絕服務'攻擊，Health響應會被緩存。你可以使用`endpoints.health.time-to-live`屬性改變默認的緩存時間（1000毫秒）。

- 自動配置的HealthIndicators

下面的HealthIndicators會被Spring Boot自動配置（在合適的時候）：

|名稱|描述|
|----|:-----|
|[DiskSpaceHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/DiskSpaceHealthIndicator.java)|低磁盤空間檢測|
|[DataSourceHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/DataSourceHealthIndicator.java)|檢查是否能從DataSource獲取連接|
|[MongoHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/MongoHealthIndicator.java)|檢查一個Mongo數據庫是否可用（up）|
|[RabbitHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/RabbitHealthIndicator.java)|檢查一個Rabbit服務器是否可用（up）|
|[RedisHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/RedisHealthIndicator.java)|檢查一個Redis服務器是否可用（up）|
|[SolrHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/SolrHealthIndicator.java)|檢查一個Solr服務器是否可用（up）|

- 編寫自定義HealthIndicators

想提供自定義健康信息，你可以注冊實現了[HealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java)接口的Spring beans。你需要提供一個health()方法的實現，並返回一個Health響應。Health響應需要包含一個status和可選的用於展示的詳情。
```java
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealth implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}
```
除了Spring Boot預定義的[Status](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java)類型，Health也可以返回一個代表新的系統狀態的自定義Status。在這種情況下，需要提供一個[HealthAggregator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthAggregator.java)接口的自定義實現，或使用management.health.status.order屬性配置默認的實現。

例如，假設一個新的，代碼為FATAL的Status被用於你的一個HealthIndicator實現中。為了配置嚴重程度，你需要將下面的配置添加到application屬性文件中：
```java
management.health.status.order: DOWN, OUT_OF_SERVICE, UNKNOWN, UP
```
如果使用HTTP訪問health端點，你可能想要注冊自定義的status，並使用HealthMvcEndpoint進行映射。例如，你可以將FATAL映射為HttpStatus.SERVICE_UNAVAILABLE。

4. 自定義應用info信息

通過設置Spring屬性info.*，你可以定義info端點暴露的數據。所有在info關鍵字下的Environment屬性都將被自動暴露。例如，你可以將下面的配置添加到application.properties：
```java
info.app.name=MyService
info.app.description=My awesome service
info.app.version=1.0.0
```
- 在建構時期自動擴展info屬性

你可以使用已經存在的建構配置自動擴展info屬性，而不是對在項目建構配置中存在的屬性進行硬編碼。這在Maven和Gradle都是可能的。

**使用Maven自動擴展屬性**

對於Maven項目，你可以使用資源過濾來自動擴展info屬性。如果使用spring-boot-starter-parent，你可以通過`@..@`佔位符引用Maven的'project properties'。
```java
project.artifactId=myproject
project.name=Demo
project.version=X.X.X.X
project.description=Demo project for info endpoint
info.build.artifact=@project.artifactId@
info.build.name=@project.name@
info.build.description=@project.description@
info.build.version=@project.version@
```
**注**：在上麵的範例中，我們使用project.*來設置一些值以防止由於某些原因Maven的資源過濾沒有開啟。Maven目標`spring-boot:run`直接將`src/main/resources`添加到classpath下（出於熱加載的目的）。這就繞過了資源過濾和自動擴展屬性的特性。你可以使用`exec:java`替換該目標或自定義插件的配置，具體參考[plugin usage page](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/usage.html)。

如果你不使用starter parent，在你的pom.xml你需要添加（處於<build/>元素內）：
```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```
和（處於<plugins/>內）：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <delimiters>
            <delimiter>@</delimiter>
        </delimiters>
    </configuration>
</plugin>
```
**使用Gradle自動擴展屬性**

通過配置Java插件的processResources任務，你也可以自動使用來自Gradle項目的屬性擴展info屬性。
```java
processResources {
    expand(project.properties)
}
```
然後你可以通過佔位符引用Gradle項目的屬性：
```java
info.build.name=${name}
info.build.description=${description}
info.build.version=${version}
```
- Git提交信息

info端點的另一個有用特性是，當項目建構完成後，它可以發布關於你的git源碼倉庫狀態的信息。如果在你的jar中包含一個git.properties文件，git.branch和git.commit屬性將被加載。

對於Maven用戶，`spring-boot-starter-parent` POM包含一個能夠產生git.properties文件的預配置插件。隻需要簡單的將下面的聲明添加到你的POM中：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
對於Gradle用戶可以使用一個相似的插件[gradle-git](https://github.com/ajoberstar/gradle-git)，盡管為了產生屬性文件可能需要稍微多點工作。

### 基於HTTP的監控和管理

如果你正在開發一個Spring MVC應用，Spring Boot執行器自動將所有啟用的端點通過HTTP暴露出去。默認約定使用端點的id作為URL路徑，例如，health暴露為/health。

* 保護敏感端點

如果你的項目中添加的有Spring Security，所有通過HTTP暴露的敏感端點都會受到保護。默認情況下會使用基本認證（basic authentication，用戶名為user，密碼為應用啟動時在控製台列印的密碼）。

你可以使用Spring屬性改變用戶名，密碼和訪問端點需要的安全角色。例如，你可能會在application.properties中添加下列配置：
```java
security.user.name=admin
security.user.password=secret
management.security.role=SUPERUSER
```

**注**：如果你不使用Spring Security，那你的HTTP端點就被公開暴露，你應該慎重考慮啟用哪些端點。具體參考[Section 40.1, “Customizing endpoints”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready-customizing-endpoints)。

* 自定義管理服務器的上下文路徑

有時候將所有的管理端口劃分到一個路徑下是有用的。例如，你的應用可能已經將`/info`作為他用。你可以用`management.contextPath`屬性為管理端口設置一個前綴：
```java
management.context-path=/manage
```
上麵的application.properties範例將把端口從`/{id}`改為`/manage/{id}`（比如，/manage/info）。

* 自定義管理服務器的端口

對於基於雲的部署，使用默認的HTTP端口暴露管理端點（endpoints）是明智的選擇。然而，如果你的應用是在自己的數據中心運行，那你可能傾向於使用一個不同的HTTP端口來暴露端點。

`management.port`屬性可以用來改變HTTP端口：
```java
management.port=8081
```
由於你的管理端口經常被防火牆保護，不對外暴露也就不需要保護管理端點，即使你的主要應用是安全的。在這種情況下，classpath下會存在Spring Security庫，你可以設置下面的屬性來禁用安全管理策略（management security）：
```java
management.security.enabled=false
```
（如果classpath下不存在Spring Security，那也就不需要顯示的以這種方式來禁用安全管理策略，它甚至可能會破壞應用程序。）

* 自定義管理服務器的地址

你可以通過設置`management.address`屬性來定義管理端點可以使用的地址。這在你隻想監聽內部或麵向生產環境的網絡，或隻監聽來自localhost的連接時非常有用。

下面的application.properties範例不允許遠程管理連接：
```java
management.port=8081
management.address=127.0.0.1
```

* 禁用HTTP端點

如果不想使用HTTP暴露端點，你可以將管理端口設置為-1：
`management.port=-1`

* HTTP Health端點訪問限製

通過health端點暴露的信息根據是否為匿名訪問而不同。默認情況下，當匿名訪問時，任何有關服務器的健康詳情都被隱藏了，該端點隻簡單的指示服務器是運行（up）還是停止（down）。此外，當匿名訪問時，響應會被緩存一個可配置的時間段以防止端點被用於'拒絕服務'攻擊。`endpoints.health.time-to-live`屬性被用來配置緩存時間（單位為毫秒），默認為1000毫秒，也就是1秒。

上述的限製可以被禁止，從而允許匿名用戶完全訪問health端點。想達到這個效果，可以將`endpoints.health.sensitive`設為`false`。

### 基於JMX的監控和管理

Java管理擴展（JMX）提供了一種標準的監控和管理應用的機製。默認情況下，Spring Boot在`org.springframework.boot`域下將管理端點暴露為JMX MBeans。

* 自定義MBean名稱

MBean的名稱通常產生於端點的id。例如，health端點被暴露為`org.springframework.boot/Endpoint/HealthEndpoint`。

如果你的應用包含多個Spring ApplicationContext，你會發現存在名稱衝突。為了解決這個問題，你可以將`endpoints.jmx.uniqueNames`設置為true，這樣MBean的名稱總是唯一的。

你也可以自定義JMX域，所有的端點都在該域下暴露。這裡有個application.properties範例：
```java
endpoints.jmx.domain=myapp
endpoints.jmx.uniqueNames=true
```
* 禁用JMX端點

如果不想通過JMX暴露端點，你可以將`spring.jmx.enabled`屬性設置為false：
```java
spring.jmx.enabled=false
```

* 使用Jolokia通過HTTP實現JMX遠程管理

Jolokia是一個JMX-HTTP橋，它提供了一種訪問JMX beans的替代方法。想要使用Jolokia，隻需添加`org.jolokia:jolokia-core`的依賴。例如，使用Maven需要添加下面的配置：
```xml
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
 </dependency>
```
在你的管理HTTP服務器上可以通過`/jolokia`訪問Jolokia。

- 自定義Jolokia

Jolokia有很多配置，傳統上一般使用servlet參數進行設置。使用Spring Boot，你可以在application.properties中通過把參數加上`jolokia.config.`前綴來設置：
```java
jolokia.config.debug=true
```
- 禁用Jolokia

如果你正在使用Jolokia，但不想讓Spring Boot配置它，隻需要簡單的將`endpoints.jolokia.enabled`屬性設置為false：
```java
endpoints.jolokia.enabled=false
```   

### 使用遠程shell來進行監控和管理

Spring Boot支援整合一個稱為'CRaSH'的Java shell。你可以在CRaSH中使用ssh或telnet命令連接到運行的應用。為了啟用遠程shell支援，你隻需添加`spring-boot-starter-remote-shell`的依賴：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-remote-shell</artifactId>
 </dependency>
```
**注**：如果想使用telnet訪問，你還需添加對`org.crsh:crsh.shell.telnet`的依賴。

* 連接遠程shell

默認情況下，遠程shell監聽端口2000以等待連接。默認用戶名為`user`，密碼為隨機生成的，並且在輸出日誌中會顯示。如果應用使用Spring Security，該shell默認使用[相同的配置](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-security)。如果不是，將使用一個簡單的認證策略，你可能會看到類似這樣的信息：
```java
Using default password for shell access: ec03e16c-4cf4-49ee-b745-7c8255c1dd7e
```
Linux和OSX用戶可以使用`ssh`連接遠程shell，Windows用戶可以下載並安裝[PuTTY](http://www.putty.org/)。
```shell
$ ssh -p 2000 user@localhost

user@localhost's password:
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.3.0.BUILD-SNAPSHOT) on myhost
```
輸入help可以獲取一係列命令的幫助。Spring boot提供`metrics`，`beans`，`autoconfig`和`endpoint`命令。

- 遠程shell證書

你可以使用`shell.auth.simple.user.name`和`shell.auth.simple.user.password`屬性配置自定義的連接證書。也可以使用Spring Security的AuthenticationManager處理登錄職責。具體參考Javadoc[CrshAutoConfiguration](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/actuate/autoconfigure/CrshAutoConfiguration.html)和[ShellProperties](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/actuate/autoconfigure/ShellProperties.html)。

* 擴展遠程shell

有很多有趣的方式可以用來擴展遠程shell。

- 遠程shell命令

你可以使用Groovy或Java編寫其他的shell命令（具體參考CRaSH文件）。默認情況下，Spring Boot會搜索以下路徑的命令：
1. `classpath*:/commands/**`
2. `classpath*:/crash/commands/**`

**注**：可以通過`shell.commandPathPatterns`屬性改變搜索路徑。

下面是一個從`src/main/resources/commands/hello.groovy`加載的'hello world'命令：
```java
package commands

import org.crsh.cli.Usage
import org.crsh.cli.Command

class hello {

    @Usage("Say Hello")
    @Command
    def main(InvocationContext context) {
        return "Hello"
    }

}
```
Spring Boot將一些額外屬性添加到了InvocationContext，你可以在命令中訪問它們：

|屬性名稱|描述|
|------|:------|
|spring.boot.version|Spring Boot的版本|
|spring.version|Spring框架的核心版本|
|spring.beanfactory|獲取Spring的BeanFactory|
|spring.environment|獲取Spring的Environment|

- 遠程shell插件

除了創建新命令，也可以擴展CRaSH shell的其他特性。所有繼承`org.crsh.plugin.CRaSHPlugin`的Spring Beans將自動注冊到shell。

具體查看[CRaSH參考文件](http://www.crashub.org/)。

### 度量指標（Metrics）

Spring Boot執行器包括一個支援'gauge'和'counter'級別的度量指標服務。'gauge'記錄一個單一值；'counter'記錄一個增量（增加或減少）。同時，Spring Boot提供一個[PublicMetrics](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/endpoint/PublicMetrics.java)接口，你可以實現它，從而暴露以上兩種機製不能記錄的指標。具體參考[SystemPublicMetrics](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/endpoint/SystemPublicMetrics.java)。

所有HTTP請求的指標都被自動記錄，所以如果點擊`metrics`端點，你可能會看到類似以下的響應：
```javascript
{
    "counter.status.200.root": 20,
    "counter.status.200.metrics": 3,
    "counter.status.200.star-star": 5,
    "counter.status.401.root": 4,
    "gauge.response.star-star": 6,
    "gauge.response.root": 2,
    "gauge.response.metrics": 3,
    "classes": 5808,
    "classes.loaded": 5808,
    "classes.unloaded": 0,
    "heap": 3728384,
    "heap.committed": 986624,
    "heap.init": 262144,
    "heap.used": 52765,
    "mem": 986624,
    "mem.free": 933858,
    "processors": 8,
    "threads": 15,
    "threads.daemon": 11,
    "threads.peak": 15,
    "uptime": 494836,
    "instance.uptime": 489782,
    "datasource.primary.active": 5,
    "datasource.primary.usage": 0.25
}
```
此處我們可以看到基本的`memory`，`heap`，`class loading`，`processor`和`thread pool`信息，連同一些HTTP指標。在該實例中，`root`('/')，`/metrics` URLs分別返回20次，3次`HTTP 200`響應。同時可以看到`root` URL返回了4次`HTTP 401`（unauthorized）響應。雙asterix（star-star）來自於被Spring MVC `/**`匹配到的一個請求（通常為一個靜態資源）。

`gauge`級別展示了一個請求的最後響應時間。所以，`root`的最後請求被響應耗時2毫秒，`/metrics`耗時3毫秒。

* 系統指標

Spring Boot暴露以下系統指標：
- 系統內存總量（mem），單位:Kb
- 空閒內存數量（mem.free），單位:Kb
- 處理器數量（processors）
- 系統正常運行時間（uptime），單位:毫秒
- 應用上下文（就是一個應用實例）正常運行時間（instance.uptime），單位:毫秒
- 系統平均負載（systemload.average）
- 堆信息（heap，heap.committed，heap.init，heap.used），單位:Kb
- 線程信息（threads，thread.peak，thead.daemon）
- 類加載信息（classes，classes.loaded，classes.unloaded）
- 垃圾收集信息（gc.xxx.count, gc.xxx.time）

* 數據源指標

Spring Boot會為你應用中定義的支援的DataSource暴露以下指標：
- 最大連接數（datasource.xxx.max）
- 最小連接數（datasource.xxx.min）
- 活動連接數（datasource.xxx.active）
- 連接池的使用情況（datasource.xxx.usage）

所有的數據源指標共用`datasoure.`前綴。該前綴對每個數據源都非常合適：
- 如果是主數據源（唯一可用的數據源或存在的數據源中被@Primary標記的）前綴為datasource.primary
- 如果數據源bean名稱以dataSource結尾，那前綴就是bean的名稱去掉dataSource的部分（例如，batchDataSource的前綴是datasource.batch）
- 其他情況使用bean的名稱作為前綴

通過注冊一個自定義版本的DataSourcePublicMetrics bean，你可以覆蓋部分或全部的默認行為。默認情況下，Spring Boot提供支援所有數據源的元數據；如果你喜歡的數據源恰好不被支援，你可以添加另外的DataSourcePoolMetadataProvider beans。具體參考DataSourcePoolMetadataProvidersConfiguration。

* Tomcat session指標

如果你使用Tomcat作為內嵌的servlet容器，session指標將被自動暴露出去。`httpsessions.active`和`httpsessions.max`提供了活動的和最大的session數量。

* 記錄自己的指標

想要記錄你自己的指標，隻需將[CounterService](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/CounterService.java)或[GaugeService](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/GaugeService.java)注入到你的bean中。CounterService暴露increment，decrement和reset方法；GaugeService提供一個submit方法。

下面是一個簡單的範例，它記錄了方法調用的次數：
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.metrics.CounterService;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final CounterService counterService;

    @Autowired
    public MyService(CounterService counterService) {
        this.counterService = counterService;
    }

    public void exampleMethod() {
        this.counterService.increment("services.system.myservice.invoked");
    }

}
```
**注**：你可以將任何的字符串用作指標的名稱，但最好遵循所選存儲或圖技術的指南。[Matt Aimonetti’s Blog](http://matt.aimonetti.net/posts/2013/06/26/practical-guide-to-graphite-monitoring/)中有一些好的關於圖（Graphite）的指南。

* 添加你自己的公共指標

想要添加額外的，每次指標端點被調用時都會重新計算的度量指標，隻需簡單的注冊其他的PublicMetrics實現bean(s)。默認情況下，端點會聚合所有這樣的beans，通過定義自己的MetricsEndpoint可以輕易改變這種情況。

* 指標倉庫

通過綁定一個[MetricRepository](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/repository/MetricRepository.java)來實現指標服務。`MetricRepository`負責存儲和追溯指標信息。Spring Boot提供一個`InMemoryMetricRepository`和一個`RedisMetricRepository`（默認使用in-memory倉庫），不過你可以編寫自己的`MetricRepository`。`MetricRepository`接口實際是`MetricReader`接口和`MetricWriter`接口的上層組合。具體參考[Javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/actuate/metrics/repository/MetricRepository.html)

沒有什麼能阻止你直接將`MetricRepository`的數據導入應用中的後端存儲，但我們建議你使用默認的`InMemoryMetricRepository`（如果擔心堆使用情況，你可以使用自定義的Map實例），然後通過一個scheduled export job填充後端倉庫（意思是先將數據保存到內存中，然後通過異步job將數據持久化到數據庫，可以提高系統性能）。通過這種方式，你可以將指標數據緩存到內存中，然後通過低頻率或批量導出來減少網絡擁堵。Spring Boot提供一個`Exporter`接口及一些幫你開始的基本實現。

* Dropwizard指標

[Dropwizard ‘Metrics’庫](https://dropwizard.github.io/metrics/)的用戶會發現Spring Boot指標被發布到了`com.codahale.metrics.MetricRegistry`。當你聲明對`io.dropwizard.metrics:metrics-core`庫的依賴時會創建一個默認的`com.codahale.metrics.MetricRegistry` Spring bean；如果需要自定義，你可以注冊自己的@Bean實例。來自於`MetricRegistry`的指標也是自動通過`/metrics`端點暴露的。

用戶可以通過使用合適類型的指標名稱作為前綴來創建Dropwizard指標（比如，`histogram.*`, `meter.*`）。

* 訊息渠道整合

如果你的classpath下存在'Spring Messaging' jar，一個名為`metricsChannel`的`MessageChannel`將被自動創建（除非已經存在一個）。此外，所有的指標更新事件作為'messages'發布到該渠道上。訂閱該渠道的客戶端可以進行額外的分析或行動。

### 審計

Spring Boot執行器具有一個靈活的審計框架，一旦Spring Security處於活動狀態（默認拋出'authentication success'，'failure'和'access denied'異常），它就會發布事件。這對於報告非常有用，同時可以基於認證失敗實現一個鎖定策略。

你也可以使用審計服務處理自己的業務事件。為此，你可以將存在的`AuditEventRepository`注入到自己的組件，並直接使用它，或者只是簡單地通過Spring `ApplicationEventPublisher`發布`AuditApplicationEvent`（使用`ApplicationEventPublisherAware`）。

### 追蹤（Tracing）

對於所有的HTTP請求Spring Boot自動啟用追蹤。你可以查看`trace`端點，並獲取最近一些請求的基本信息：
```javascript
[{
    "timestamp": 1394343677415,
    "info": {
        "method": "GET",
        "path": "/trace",
        "headers": {
            "request": {
                "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
                "Connection": "keep-alive",
                "Accept-Encoding": "gzip, deflate",
                "User-Agent": "Mozilla/5.0 Gecko/Firefox",
                "Accept-Language": "en-US,en;q=0.5",
                "Cookie": "_ga=GA1.1.827067509.1390890128; ..."
                "Authorization": "Basic ...",
                "Host": "localhost:8080"
            },
            "response": {
                "Strict-Transport-Security": "max-age=31536000 ; includeSubDomains",
                "X-Application-Context": "application:8080",
                "Content-Type": "application/json;charset=UTF-8",
                "status": "200"
            }
        }
    }
},{
    "timestamp": 1394343684465,
    ...
}]
```
- 自定義追蹤

如果需要追蹤其他的事件，你可以將一個[TraceRepository](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/trace/TraceRepository.java)注入到你的Spring Beans中。`add`方法接收一個將被轉化為JSON的`Map`結構，該數據將被記錄下來。

默認情況下，使用的`InMemoryTraceRepository`將存儲最新的100個事件。如果需要擴展該容量，你可以定義自己的`InMemoryTraceRepository`實例。如果需要，你可以創建自己的替代`TraceRepository`實現。

### 進程監控

在Spring Boot執行器中，你可以找到幾個創建有利於進程監控的文件的類：
- `ApplicationPidFileWriter`創建一個包含應用PID的文件（默認位於應用目錄，文件名為application.pid）
- `EmbeddedServerPortFileWriter`創建一個或多個包含內嵌服務器端口的文件（默認位於應用目錄，文件名為application.port）

默認情況下，這些writers沒有被啟動，但你可以使用下面描述的任何方式來啟用它們。

* 擴展屬性

你需要啟動`META-INF/spring.factories`文件裡的listener(s)：
```java
org.springframework.context.ApplicationListener=\
org.springframework.boot.actuate.system.ApplicationPidFileWriter,
org.springframework.boot.actuate.system.EmbeddedServerPortFileWriter
```

* 以程式方式

你也可以通過調用`SpringApplication.addListeners(…)`方法來啟動一個監聽器，並傳遞相應的`Writer`對象。該方法允許你通過`Writer`構造器自定義文件名和路徑。
