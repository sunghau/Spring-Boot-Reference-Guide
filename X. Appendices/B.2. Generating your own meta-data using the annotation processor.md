### 附錄B.2. 使用註解處理器產生自己的元數據

通過使用`spring-boot-configuration-processor` jar， 你可以從被`@ConfigurationProperties`註解的節點輕鬆的產生自己的配置元數據文件。該jar包含一個在你的項目編譯時會被調用的Java註解處理器。想要使用該處理器，你只需簡單添加`spring-boot-configuration-processor`依賴，例如使用Maven你需要添加：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```
使用Gradle時，你可以使用[propdeps-plugin](https://github.com/spring-projects/gradle-plugins/tree/master/propdeps-plugin)並指定：
```gradle
dependencies {
		optional "org.springframework.boot:spring-boot-configuration-processor"
	}

	compileJava.dependsOn(processResources)
}
```
**注**：你需要將`compileJava.dependsOn(processResources)`添加到建構中，以確保資源在程式碼編譯之前處理。如果沒有該指令，任何`additional-spring-configuration-metadata.json`文件都不會被處理。

該處理器會處理被`@ConfigurationProperties`註解的類和函式，description屬性用於產生配置類字段值的Javadoc說明。

**注**：你應該使用簡單的文件來設置`@ConfigurationProperties`字段的Javadoc，因為在沒有被添加到JSON之前它們是不被處理的。


屬性是通過判斷是否存在標準的getters和setters來發現的，對於集合類型有特殊處理（即使隻出現一個getter）。該註解處理器也支援使用lombok的`@Data`, `@Getter`和`@Setter`註解。


