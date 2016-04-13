### 附錄B.1.1. Group屬性

`groups`數組包含的JSON對象可以由以下屬性組成：

|名稱|類型|目的|
|----|:----|:----|
|name|String|group的全名，該屬性是強製性的|
|type|String|group數據類型的類名。例如，如果group是基於一個被`@ConfigurationProperties`注解的類，該屬性將包含該類的全限定名。如果基於一個`@Bean`方法，它將是該方法的返回類型。如果該類型未知，則該屬性將被忽略|
|description|String|一個簡短的group描述，用於展示給用戶。如果沒有可用描述，該屬性將被忽略。推薦使用一個簡短的段落描述，第一行提供一個簡潔的總結，最後一行以句號結尾|
|sourceType|String|貢獻該組的來源類名。例如，如果組基於一個被`@ConfigurationProperties`注解的`@Bean`方法，該屬性將包含`@Configuration`類的全限定名，該類包含此方法。如果來源類型未知，則該屬性將被忽略|
|sourceMethod|String|貢獻該組的方法的全名（包含括號及參數類型）。例如，被`@ConfigurationProperties`注解的`@Bean`方法名。如果源方法未知，該屬性將被忽略|
