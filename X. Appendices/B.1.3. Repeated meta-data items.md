### 附錄B.1.3. 可重複的元數據節點

在同一個元數據文件中出現多次相同名稱的"property"和"group"對象是可以接受的。例如，Spring Boot將`spring.datasource`屬性綁定到Hikari，Tomcat和DBCP類，並且每個都潛在的提供了重複的屬性名。這些元數據的消費者需要確保他們支援這樣的場景。
