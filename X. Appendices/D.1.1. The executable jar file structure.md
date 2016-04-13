### 附錄D.1.1 可執行jar文件結構

Spring Boot Loader兼容的jar文件應該遵循以下結構：

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
 +-com
 |  +-mycompany
 |     + project
 |        +-YouClasses.class
 +-lib
    +-dependency1.jar
    +-dependency2.jar
```
依賴需要放到內部的lib目錄下。

