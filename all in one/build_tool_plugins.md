### 建構工具插件

Spring Boot為Maven和Gradle提供建構工具插件。該插件提供各種各樣的特性，包括打包可執行jars。本節提供關於插件的更多詳情及用於擴展一個不支援的建構系統所需的幫助信息。如果你是剛剛開始，那可能需要先閱讀[Part III, “Using Spring Boot”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot)章節的[“Chapter 13, Build systems”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-build-systems)。

### Spring Boot Maven插件

[Spring Boot Maven插件](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-build-systems)為Maven提供Spring Boot支援，它允許你打包可執行jar或war存檔，然後就地運行應用。為了使用它，你需要使用Maven 3.2 （或更高版本）。

**注**：參考[Spring Boot Maven Plugin Site](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/)可以獲取全部的插件文件。

* 包含該插件

想要使用Spring Boot Maven插件隻需簡單地在你的pom.xml的`plugins`部分包含相應的XML：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- ... -->
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
</project>
```
該配置會在Maven生命周期的`package`階段重新打包一個jar或war。下面的範例顯示在`target`目錄下既有重新打包後的jar，也有原始的jar：
```shell
$ mvn package
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```
如果不包含像上麵那樣的`<execution/>`，你可以自己運行該插件（但隻有在package目標也被使用的情況）。例如：
```shell
$ mvn package spring-boot:repackage
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```
如果使用一個裡程碑或快照版本，你還需要添加正確的pluginRepository元素：
```xml
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
```
* 打包可執行jar和war文件

一旦`spring-boot-maven-plugin`被包含到你的pom.xml中，它就會自動嘗試使用`spring-boot:repackage`目標重寫存檔以使它們能夠執行。為了建構一個jar或war，你應該使用常規的packaging元素配置你的項目：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>jar</packaging>
    <!-- ... -->
</project>
```
生成的存檔在`package`階段會被Spring Boot增強。你想啟動的main類即可以通過指定一個配置選項，也可以通過為manifest添加一個`Main-Class`屬性這種常規的方式實現。如果你沒有指定一個main類，該插件會搜索帶有`public static void main(String[] args)`方法的類。

為了建構和運行一個項目的artifact，你可以輸入以下命令：
```shell
$ mvn package
$ java -jar target/mymodule-0.0.1-SNAPSHOT.jar
```
為了建構一個即是可執行的，又能部署到一個外部容器的war文件，你需要標記內嵌容器依賴為"provided"，例如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>war</packaging>
    <!-- ... -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <!-- ... -->
    </dependencies>
</project>
```
**注**：具體參考[“Section 74.1, “Create a deployable war file”” ](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-create-a-deployable-war-file)章節。 

在[插件信息頁麵](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/)有高級的配置選項和範例。

### Spring Boot Gradle插件

Spring Boot Gradle插件為Gradle提供Spring Boot支援，它允許你打包可執行jar或war存檔，運行Spring Boot應用，對於"神聖的"依賴可以在你的build.gradle文件中省略版本信息。

* 包含該插件

想要使用Spring Boot Gradle插件，你隻需簡單的包含一個`buildscript`依賴，並應用`spring-boot`插件：
```gradle
buildscript {
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.0.BUILD-SNAPSHOT")
    }
}
apply plugin: 'spring-boot'
```
如果想使用一個裡程碑或快照版本，你可以添加相應的repositories引用：
```gradle
buildscript {
    repositories {
        maven.url "http://repo.spring.io/snapshot"
        maven.url "http://repo.spring.io/milestone"
    }
    // ...
}
```
* 聲明不帶版本的依賴

`spring-boot`插件會為你的建構注冊一個自定義的Gradle `ResolutionStrategy`，它允許你在聲明對"神聖"的artifacts的依賴時獲取版本號。為了充分使用該功能，隻需要想通常那樣聲明依賴，但將版本號設置為空：
```gradle
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.thymeleaf:thymeleaf-spring4")
    compile("nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect")
}
```
**注**：你聲明的`spring-boot` Gradle插件的版本決定了"blessed"依賴的實際版本（確保可以重複建構）。你最好總是將`spring-boot` gradle插件版本設置為你想用的Spring Boot實際版本。提供的版本詳細信息可以在[附錄](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#appendix-dependency-versions)中找到。

`spring-boot`插件對於沒有指定版本的依賴隻會提供一個版本。如果不想使用插件提供的版本，你可以像平常那樣在聲明依賴的時候指定版本。例如：
```gradle
dependencies {
    compile("org.thymeleaf:thymeleaf-spring4:2.1.1.RELEASE")
}
```
* 自定義版本管理

如果你需要不同於Spring Boot的"blessed"依賴，有可能的話可以自定義`ResolutionStrategy`使用的版本。替代的版本元數據使用`versionManagement`配置。例如：
```gradle
dependencies {
    versionManagement("com.mycorp:mycorp-versions:1.0.0.RELEASE@properties")
    compile("org.springframework.data:spring-data-hadoop")
}
```
版本信息需要作為一個`.properties`文件發布到一個倉庫中。對於上麵的範例，`mycorp-versions.properties`文件可能包含以下內容：
```java
org.springframework.data\:spring-data-hadoop=2.0.0.RELEASE
```
屬性文件優先於Spring Boot默認設置，如果有必要的話可以覆蓋版本號。

* 默認排除規則

Gradle處理"exclude rules"的方式和Maven稍微有些不同，在使用starter POMs時這可能會引起無法預料的結果。特別地，當一個依賴可以通過不同的路徑訪問時，對該依賴聲明的exclusions將不會生效。例如，如果一個starter POM聲明以下內容：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.0.5.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.0.5.RELEASE</version>
    </dependency>
</dependencies>
```
`commons-logging` jar不會被Gradle排除，因為通過沒有`exclusion`元素的`spring-context`可以傳遞性的拉取到它（spring-context → spring-core → commons-logging）。

為了確保正確的排除被實際應用，Spring Boot Gradle插件將自動添加排除規則。所有排除被定義在`spring-boot-dependencies` POM，並且針對"starter" POMs的隱式規則也會被添加。

如果不想自動應用排除規則，你可以使用以下配置：
```gradle
springBoot {
    applyExcludeRules=false
}
```
* 打包可執行jar和war文件

一旦`spring-boot`插件被應用到你的項目，它將使用`bootRepackage`任務自動嘗試重寫存檔以使它們能夠執行。為了建構一個jar或war，你需要按通常的方式配置項目。

你想啟動的main類既可以通過一個配置選項指定，也可以通過向manifest添加一個`Main-Class`屬性。如果你沒有指定main類，該插件會搜索帶有`public static void main(String[] args)`方法的類。

為了建構和運行一個項目artifact，你可以輸入以下內容：
```shell
$ gradle build
$ java -jar build/libs/mymodule-0.0.1-SNAPSHOT.jar
```
為了建構一個即能執行也可以部署到外部容器的war包，你需要將內嵌容器依賴標記為"providedRuntime"，比如：
```gradle
...
apply plugin: 'war'

war {
    baseName = 'myapp'
    version =  '0.5.0'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/libs-snapshot" }
}

configurations {
    providedRuntime
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")
    ...
}
```
**注**：具體參考[“Section 74.1, “Create a deployable war file””](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-create-a-deployable-war-file)。

* 就地（in-place）運行項目

為了在不先建構jar的情況下運行項目，你可以使用"bootRun"任務：
```shell
$ gradle bootRun
```
默認情況下，以這種方式運行項目可以讓你的靜態classpath資源（比如，默認位於`src/main/resources`下）在應用運行期間被重新加載。使靜態資源可以重新加載意味著`bootRun`任務不會使用`processResources`任務的輸出，比如，當調用`bootRun`時，你的應用將以資源未處理的形式來使用它們。

你可以禁止直接使用靜態classpath資源。這意味著資源不再是可重新加載的，但`processResources`任務的輸出將會被使用。想要這樣做，隻需將`bootRun`任務的`addResources`設為false：
```gradle
bootRun {
    addResources = false
}
```
* Spring Boot插件配置

Gradle插件自動擴展你的建構腳本DSL，它為腳本添加一個`springBoot`元素以此作為Boot插件的全局配置。你可以像配置其他Gradle擴展那樣為`springBoot`設置相應的屬性（下面有配置選項列表）。
```gradle
springBoot {
    backupSource = false
}
```
* Repackage配置

該插件添加了一個`bootRepackage`任務，你可以直接配置它，比如：
```gradle
bootRepackage {
    mainClass = 'demo.Application'
}
```
下面是可用的配置選項：

|名稱|描述|
|-------|:------|
|enabled|布爾值，用於控製repackager的開關（如果你隻想要Boot的其他特性而不是這個，那它就派上用場了）|
|mainClass|要運行的main類。如果沒有指定，則使用project屬性`mainClassName`。如果沒有定義`mainClassName` id，則搜索存檔以尋找一個合適的類。"合適"意味著一個唯一的，具有良好格式的`main()`方法的類（如果找到多個則建構會失敗）。你也可以通過"run"任務（`main`屬性）指定main類的名稱，和/或將"startScripts"（`mainClassName`屬性）作為"springBoot"配置的替代。|
|classifier|添加到存檔的一個文件名字段（在擴展之前），這樣最初保存的存檔仍舊存放在最初的位置。在存檔被重新打包（repackage）的情況下，該屬性默認為null。默認值適用於多數情況，但如果你想在另一個項目中使用原jar作為依賴，最好使用一個擴展來定義該可執行jar|
|withJarTask|`Jar`任務的名稱或值，用於定位要被repackage的存檔|
|customConfiguration|自定義配置的名稱，用於填充內嵌的lib目錄（不指定該屬性，你將獲取所有編譯和運行時依賴）|

* 使用Gradle自定義配置進行Repackage

有時候不打包解析自`compile`，`runtime`和`provided`作用域的默認依賴可能更合適些。如果創建的可執行jar被原樣運行，你需要將所有的依賴內嵌進該jar中；然而，如果目的是explode一個jar文件，並手動運行main類，你可能在`CLASSPATH`下已經有一些可用的庫了。在這種情況下，你可以使用不同的依賴集重新打包（repackage）你的jar。

使用自定義的配置將自動禁用來自`compile`，`runtime`和`provided`作用域的依賴解析。自定義配置即可以定義為全局的（處於`springBoot`部分內），也可以定義為任務級的。
```gradle
task clientJar(type: Jar) {
    appendix = 'client'
    from sourceSets.main.output
    exclude('**/*Something*')
}

task clientBoot(type: BootRepackage, dependsOn: clientJar) {
    withJarTask = clientJar
    customConfiguration = "mycustomconfiguration"
}
```
在以上範例中，我們創建了一個新的`clientJar` Jar任務從你編譯後的源中打包一個自定義文件集。然後我們創建一個新的`clientBoot` BootRepackage任務，並讓它使用`clientJar`任務和`mycustomconfiguration`。
```gradle
configurations {
    mycustomconfiguration.exclude group: 'log4j'
}

dependencies {
    mycustomconfiguration configurations.runtime
}
```
在`BootRepackage`中引用的配置是一個正常的[Gradle配置](http://www.gradle.org/docs/current/dsl/org.gradle.api.artifacts.Configuration.html)。在上麵的範例中，我們創建了一個新的名叫`mycustomconfiguration`的配置，指示它來自一個`runtime`，並排除對`log4j`的依賴。如果`clientBoot`任務被執行，重新打包的jar將含有所有來自`runtime`作用域的依賴，除了`log4j` jars。

* 配置選項

可用的配置選項如下：

|名稱|描述|
|-------|:--------|
|mainClass|可執行jar運行的main類|
|providedConfiguration|provided配置的名稱（默認為providedRuntime）|
|backupSource|在重新打包之前，原先的存檔是否備份（默認為true）|
|customConfiguration|自定義配置的名稱|
|layout|存檔類型，對應於內部依賴是如何製定的（默認基於存檔類型進行推測）|
|requiresUnpack|一個依賴列表（格式為"groupId:artifactId"，為了運行，它們需要從fat jars中解壓出來。）所有節點被打包進胖jar，但運行的時候它們將被自動解壓|

* 理解Gradle插件是如何工作的

當`spring-boot`被應用到你的Gradle項目，一個默認的名叫`bootRepackage`的任務被自動創建。`bootRepackage`任務依賴於Gradle `assemble`任務，當執行時，它會嘗試找到所有限定符為空的jar artifacts（也就是說，tests和sources jars被自動跳過）。

由於`bootRepackage`查找'所有'創建jar artifacts的事實，Gradle任務執行的順序就非常重要了。多數項目隻創建一個單一的jar文件，所以通常這不是一個問題。然而，如果你正打算創建一個更複雜的，使用自定義`jar`和`BootRepackage`任務的項目setup，有幾個方麵需要考慮。

如果'僅僅'從項目創建自定義jar文件，你可以簡單地禁用默認的`jar`和`bootRepackage`任務：
```gradle
jar.enabled = false
bootRepackage.enabled = false
```
另一個選項是指示默認的`bootRepackage`任務隻能使用一個默認的`jar`任務：
```gradle
bootRepackage.withJarTask = jar
```
如果你有一個默認的項目setup，在該項目中，主（main）jar文件被創建和重新打包。並且，你仍舊想創建額外的自定義jars，你可以將自定義的repackage任務結合起來，然後使用`dependsOn`，這樣`bootJars`任務就會在默認的`bootRepackage`任務執行以後運行：
```gradle
task bootJars
bootJars.dependsOn = [clientBoot1,clientBoot2,clientBoot3]
build.dependsOn(bootJars)
```
上麵所有方麵經常用於避免一個已經創建的boot jar又被重新打包的情況。重新打包一個存在的boot jar不是什麼大問題，但你可能會發現它包含不必要的依賴。

* 使用Gradle將artifacts發布到一個Maven倉庫

如果你聲明依賴但沒有指定版本，且你想要將artifacts發布到一個Maven倉庫，那你需要使用詳細的Spring Boot依賴管理來配置Maven發布。通過配置它發布繼承自`spring-boot-starter-parent`的poms或引入來自`spring-boot-dependencies`的依賴管理可以實現該需求。這種配置的具體細節取決於你如何使用Gradle及如何發布該artifacts的。

- 自定義Gradle，用於產生一個繼承依賴管理的pom

下面範例展示了如何配置Gradle去產生一個繼承自`spring-boot-starter-parent`的pom。請參考[Gradle用戶指南](http://gradle.org/docs/current/userguide/userguide.html)獲取更多信息。
```gradle
uploadArchives {
    repositories {
        mavenDeployer {
            pom {
                project {
                    parent {
                        groupId "org.springframework.boot"
                        artifactId "spring-boot-starter-parent"
                        version "1.3.0.BUILD-SNAPSHOT"
                    }
                }
            }
        }
    }
}
```
- 自定義Gradle，用於產生一個導入依賴管理的pom

以下範例展示了如何配置Gradle去產生一個導入`spring-boot-dependencies`提供的依賴管理的pom。請參考[Gradle用戶指南](http://gradle.org/docs/current/userguide/userguide.html)獲取更多信息。
```gradle
uploadArchives {
    repositories {
        mavenDeployer {
            pom {
                project {
                    dependencyManagement {
                        dependencies {
                            dependency {
                                groupId "org.springframework.boot"
                                artifactId "spring-boot-dependencies"
                                version "1.3.0.BUILD-SNAPSHOT"
                                type "pom"
                                scope "import"
                            }
                        }
                    }
                }
            }
        }
    }
}
```
### 對其他建構系統的支援

如果想使用除了Maven和Gradle之外的建構工具，你可能需要開發自己的插件。可執行jars需要遵循一個特定格式，並且一些實體需要以不壓縮的方式寫入（詳情查看附錄中的[可執行jar格式](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#executable-jar)章節）。

Spring Boot Maven和Gradle插件都利用`spring-boot-loader-tools`來實際地產生jars。如果需要，你也可以自由地直接使用該庫。

* 重新打包存檔

使用`org.springframework.boot.loader.tools.Repackager`可以將一個存在的存檔重新打包，這樣它就變成一個自包含的可執行存檔。`Repackager`類需要提供單一的構造器參數，它引用一個存在的jar或war包。使用兩個可用的`repackage()`方法中的一個來替換原始的文件或寫入一個新的目標。在repackager運行前還可以設置各種配置。

* 內嵌的庫

當重新打包一個存檔時，你可以使用`org.springframework.boot.loader.tools.Libraries`接口來包含對依賴文件的引用。在這裡我們不提供任何該Libraries接口的具體實現，因為它們通常跟具體的建構系統相關。

如果你的存檔已經包含libraries，你可以使用`Libraries.NONE`。

* 查找main類

如果你沒有使用`Repackager.setMainClass()`指定一個main類，該repackager將使用[ASM](http://asm.ow2.org/)去讀取class文件，然後嘗試查找一個合適的，具有`public static void main(String[] args)`方法的類。如果發現多個候選者，將會拋出異常。

* repackage實現範例

這裡是一個傳統的repackage範例：
```java
Repackager repackager = new Repackager(sourceJarFile);
repackager.setBackupSource(false);
repackager.repackage(new Libraries() {
            @Override
            public void doWithLibraries(LibraryCallback callback) throws IOException {
                // Build system specific implementation, callback for each dependency
                // callback.library(new Library(nestedFile, LibraryScope.COMPILE));
            }
        });
```
