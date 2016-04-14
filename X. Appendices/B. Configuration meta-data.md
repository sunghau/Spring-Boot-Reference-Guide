### 附錄B. 配置元數據

Spring Boot jars包含元數據文件，它們提供了所有支援的配置屬性詳情。這些文件設計用於讓IDE開發者能夠為使用application.properties或application.yml文件的用戶提供上下文幫助及程式碼完成功能。


主要的元數據文件是在編譯器通過處理所有被`@ConfigurationProperties`註解的節點來自動生成的。
