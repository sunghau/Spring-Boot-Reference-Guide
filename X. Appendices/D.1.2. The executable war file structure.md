### 附錄D.1.2. 可執行war文件結構

Spring Boot Loader兼容的war文件應該遵循以下結構：
```java
example.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-WEB-INF
    +-classes
    |  +-com
    |     +-mycompany
    |        +-project
    |           +-YouClasses.class
    +-lib
    |  +-dependency1.jar
    |  +-dependency2.jar
    +-lib-provided
       +-servlet-api.jar
       +-dependency3.jar
```
依賴需要放到內嵌的`WEB-INF/lib`目錄下。任何運行時需要但部署到普通web容器不需要的依賴應該放到`WEB-INF/lib-provided`目錄下。
