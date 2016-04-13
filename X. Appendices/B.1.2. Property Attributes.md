### 附錄B.1.2. Property屬性

`properties`數組中包含的JSON對象可由以下屬性構成：

|名稱|類型|目的|
|----|:----|:----|
|name|String|property的全名，格式為小寫虛線分割的形式（比如`server.servlet-path`）。該屬性是強製性的|
|type|String|property數據類型的類名。例如`java.lang.String`。該屬性可以用來指導用戶他們可以輸入值的類型。為了保持一致，原生類型使用它們的包裝類代替，比如`boolean`變成了`java.lang.Boolean`。注意，這個類可能是個從一個字符串轉換而來的複雜類型。如果類型未知則該屬性會被忽略|
|description|String|一個簡短的組的描述，用於展示給用戶。如果沒有描述可用則該屬性會被忽略。推薦使用一個簡短的段落描述，開頭提供一個簡潔的總結，最後一行以句號結束|
|sourceType|String|貢獻property的來源類名。例如，如果property來自一個被`@ConfigurationProperties`注解的類，該屬性將包括該類的全限定名。如果來源類型未知則該屬性會被忽略|
|defaultValue|Object|當property沒有定義時使用的默認值。如果property類型是個數組則該屬性也可以是個數組。如果默認值未知則該屬性會被忽略|
|deprecated|boolean|指定該property是否過期。如果該字段沒有過期或該信息未知則該屬性會被忽略|
