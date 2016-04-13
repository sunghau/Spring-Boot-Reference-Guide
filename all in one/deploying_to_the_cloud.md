### 部署到雲端

對於大多數流行雲PaaS（平台即服務）提供商，Spring Boot的可執行jars就是為它們準備的。這些提供商往往要求你帶上自己的容器；它們管理應用的進程（不特別針對Java應用程序），所以它們需要一些中間層來將你的應用適配到雲概念中的一個運行進程。

兩個流行的雲提供商，Heroku和Cloud Foundry，采取一個打包（'buildpack'）方法。為了啟動你的應用程序，不管需要什麼，buildpack都會將它們打包到你的部署代碼：它可能是一個JDK和一個java調用，也可能是一個內嵌的webserver，或者是一個成熟的應用服務器。buildpack是可插拔的，但你最好盡可能少的對它進行自定義設置。這可以減少不受你控製的功能範圍，最小化部署和生產環境的發散。

理想情況下，你的應用就像一個Spring Boot可執行jar，所有運行需要的東西都打包到它內部。

* Cloud Foundry

如果不指定其他打包方式，Cloud Foundry會啟用它提供的默認打包方式。Cloud Foundry的[Java buildpack](https://github.com/cloudfoundry/java-buildpack)對Spring應用有出色的支援，包括Spring Boot。你可以部署獨立的可執行jar應用，也可以部署傳統的.war形式的應用。

一旦你建構了應用（比如，使用`mvn clean package`）並[安裝](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html)了cf[命令行工具](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html)，你可以使用下面的`cf push`命令（將路徑指向你編譯後的.jar）來部署應用。在發布一個應用前，確保你已登陸cf命令行客戶端。
```shell
$ cf push acloudyspringtime -p target/demo-0.0.1-SNAPSHOT.jar
```
查看`cf push`[文件](http://docs.cloudfoundry.org/devguide/installcf/whats-new-v6.html#push)獲取更多可選項。如果相同目錄下存在[manifest.yml](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)，Cloud Foundry會使用它。

就此，cf開始上傳你的應用：
```java
Uploading acloudyspringtime... OK
Preparing to start acloudyspringtime... OK
-----> Downloaded app package (8.9M)
-----> Java Buildpack source: system
-----> Downloading Open JDK 1.7.0_51 from .../x86_64/openjdk-1.7.0_51.tar.gz (1.8s)
       Expanding Open JDK to .java-buildpack/open_jdk (1.2s)
-----> Downloading Spring Auto Reconfiguration from  0.8.7 .../auto-reconfiguration-0.8.7.jar (0.1s)
-----> Uploading droplet (44M)
Checking status of app 'acloudyspringtime'...
  0 of 1 instances running (1 starting)
  ...
  0 of 1 instances running (1 down)
  ...
  0 of 1 instances running (1 starting)
  ...
  1 of 1 instances running (1 running)

App started
```
恭喜！應用現在處於運行狀態！

檢驗部署應用的狀態是很簡單的：
```shell
$ cf apps
Getting applications in ...
OK

name                 requested state   instances   memory   disk   urls
...
acloudyspringtime    started           1/1         512M     1G     acloudyspringtime.cfapps.io
...
```
一旦Cloud Foundry意識到你的應用已經部署，你就可以點擊給定的應用URI，此處是[acloudyspringtime.cfapps.io/](http://acloudyspringtime.cfapps.io/)。

- 綁定服務

默認情況下，運行應用的元數據和服務連接信息被暴露為應用的環境變量（比如，$VCAP_SERVICES）。采用這種架構的原因是因為Cloud Foundry多語言特性（任何語言和平台都支援作為buildpack）。進程級別的環境變量是語言無關（language agnostic）的。

環境變量並不總是有利於設計最簡單的API，所以Spring Boot自動提取它們，然後將這些數據導入能夠通過Spring `Environment`抽象訪問的屬性裡：
```java
@Component
class MyBean implements EnvironmentAware {

    private String instanceId;

    @Override
    public void setEnvironment(Environment environment) {
        this.instanceId = environment.getProperty("vcap.application.instance_id");
    }

    // ...

}
```
所有的Cloud Foundry屬性都以vcap作為前綴。你可以使用vcap屬性獲取應用信息（比如應用的公共URL）和服務信息（比如數據庫證書）。具體參考VcapApplicationListener Javadoc。

**注**：[Spring Cloud Connectors](http://cloud.spring.io/spring-cloud-connectors/)項目很適合比如配置數據源的任務。Spring Boot提供自動配置支援和一個`spring-boot-starter-cloud-connectors` starter POM。

* Heroku

Heroku是另外一個流行的Paas平台。想要自定義Heroku的建構過程，你可以提供一個`Procfile`，它提供部署一個應用所需的指令。Heroku為Java應用分配一個端口，確保能夠路由到外部URI。

你必須配置你的應用監聽正確的端口。下面是用於我們的starter REST應用的Procfile：
```shell
web: java -Dserver.port=$PORT -jar target/demo-0.0.1-SNAPSHOT.jar
```
Spring Boot將`-D`參數作為屬性，通過一個Spring的Environment實例訪問。`server.port`配置屬性適合於內嵌的Tomcat，Jetty或Undertow實例啟用時使用。`$PORT`環境變量被分配給Heroku Paas使用。

Heroku默認使用Java 1.6。隻要你的Maven或Gradle建構時使用相同的版本就沒問題（Maven用戶可以設置`java.version`屬性）。如果你想使用JDK 1.7，在你的pom.xml和Procfile臨近處創建一個system.properties文件。在該文件中添加以下設置：
```java
java.runtime.version=1.7
```
這就是你需要做的一切。對於Heroku部署來說，經常做的工作就是使用`git push`將代碼推送到生產環境。
```shell
$ git push heroku master

Initializing repository, done.
Counting objects: 95, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (78/78), done.
Writing objects: 100% (95/95), 8.66 MiB | 606.00 KiB/s, done.
Total 95 (delta 31), reused 0 (delta 0)

-----> Java app detected
-----> Installing OpenJDK 1.7... done
-----> Installing Maven 3.2.3... done
-----> Installing settings.xml... done
-----> executing /app/tmp/cache/.maven/bin/mvn -B
       -Duser.home=/tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229
       -Dmaven.repo.local=/app/tmp/cache/.m2/repository
       -s /app/tmp/cache/.m2/settings.xml -DskipTests=true clean install

       [INFO] Scanning for projects...
       Downloading: http://repo.spring.io/...
       Downloaded: http://repo.spring.io/... (818 B at 1.8 KB/sec)
        ....
       Downloaded: http://s3pository.heroku.com/jvm/... (152 KB at 595.3 KB/sec)
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/target/...
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/pom.xml ...
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 59.358s
       [INFO] Finished at: Fri Mar 07 07:28:25 UTC 2014
       [INFO] Final Memory: 20M/493M
       [INFO] ------------------------------------------------------------------------

-----> Discovering process types
       Procfile declares types -> web

-----> Compressing... done, 70.4MB
-----> Launching... done, v6
       http://agile-sierra-1405.herokuapp.com/ deployed to Heroku

To git@heroku.com:agile-sierra-1405.git
 * [new branch]      master -> master

```
現在你的應用已經啟動並運行在Heroku。

* Openshift

[Openshift](https://www.openshift.com/)是RedHat公共（和企業）PaaS解決方案。和Heroku相似，它也是通過運行被git提交觸發的腳本來工作的，所以你可以使用任何你喜歡的方式編寫Spring Boot應用啟動腳本，隻要Java運行時環境可用（這是在Openshift上可以要求的一個標準特性）。為了實現這樣的效果，你可以使用[DIY Cartridge](https://www.openshift.com/developers/do-it-yourself)，並在`.openshift/action_scripts`下hooks你的倉庫：

基本模式如下：

1. 確保Java和建構工具已被遠程安裝，比如使用一個`pre_build` hook（默認會安裝Java和Maven，不會安裝Gradle）。
2. 使用一個`build` hook去建構你的jar（使用Maven或Gradle），比如
```shell
#!/bin/bash
cd $OPENSHIFT_REPO_DIR
mvn package -s .openshift/settings.xml -DskipTests=true
```
3. 添加一個調用`java -jar …​`的`start` hook
```shell
#!/bin/bash
cd $OPENSHIFT_REPO_DIR
nohup java -jar target/*.jar --server.port=${OPENSHIFT_DIY_PORT} --server.address=${OPENSHIFT_DIY_IP} &
```
4. 使用一個`stop` hook
```shell
#!/bin/bash
source $OPENSHIFT_CARTRIDGE_SDK_BASH
PID=$(ps -ef | grep java.*\.jar | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    client_result "Application is already stopped"
else
    kill $PID
fi
```
5. 將內嵌的服務綁定到平台提供的在application.properties定義的環境變量，比如
```shell
spring.datasource.url: jdbc:mysql://${OPENSHIFT_MYSQL_DB_HOST}:${OPENSHIFT_MYSQL_DB_PORT}/${OPENSHIFT_APP_NAME}
spring.datasource.username: ${OPENSHIFT_MYSQL_DB_USERNAME}
spring.datasource.password: ${OPENSHIFT_MYSQL_DB_PASSWORD}
```
在Openshift的網站上有一篇[running Gradle in Openshift](https://www.openshift.com/blogs/run-gradle-builds-on-openshift)博客，如果想使用gradle建構運行的應用可以參考它。由於一個[Gradle bug](http://issues.gradle.org/browse/GRADLE-2871)，你不能使用高於1.6版本的Gradle。

* Google App Engine

Google App Engine跟Servlet 2.5 API是有聯係的，所以在不修改的情況係你是不能部署一個Spring應用的。具體查看本指南的[Servlet 2.5章節](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-servlet-2-5)。
