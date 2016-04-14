### 附錄 B.2.1. 內嵌屬性

該註解處理器自動將內部類當做內嵌屬性處理。例如，下面的類：
```java
@ConfigurationProperties(prefix="server")
public class ServerProperties {

    private String name;

    private Host host;

    // ... getter and setters

    private static class Host {

        private String ip;

        private int port;

        // ... getter and setters

    }

}
```
