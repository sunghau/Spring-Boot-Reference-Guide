### 附錄D.2.1 對標準Java "JarFile"的兼容性

Spring Boot Loader努力保持對已有代碼和庫的兼容。`org.springframework.boot.loader.jar.JarFile`繼承自`java.util.jar.JarFile`，可以作為降級替換。
