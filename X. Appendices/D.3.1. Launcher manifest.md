### 附錄D.3.1 Launcher manifest

你需要指定一個合適的Launcher作為`META-INF/MANIFEST.MF`的`Main-Class`屬性。你實際想要啟動的類（也就是你編寫的包含main方法的類）需要在`Start-Class`屬性中定義。

例如，這裡有個典型的可執行jar文件的MANIFEST.MF：
```properties
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.mycompany.project.MyApplication
```
對於一個war文件，它可能是這樣的：
```properties
Main-Class: org.springframework.boot.loader.WarLauncher
Start-Class: com.mycompany.project.MyApplication
```
**注**：你不需要在manifest文件中指定`Class-Path`實體，classpath會從嵌套的jars中被推導出來。
