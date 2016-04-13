### 附錄D.2. Spring Boot的"JarFile"類

Spring Boot用於支援加載內嵌jars的核心類是`org.springframework.boot.loader.jar.JarFile`。它允許你從一個標準的jar文件或內嵌的子jar數據中加載jar內容。當首次加載的時候，每個JarEntry的位置被映射到一個偏移於外部jar的物理文件：
```java
myapp.jar
+---------+---------------------+
|         | /lib/mylib.jar      |
| A.class |+---------+---------+|
|         || B.class | B.class ||
|         |+---------+---------+|
+---------+---------------------+
^          ^          ^
0063       3452       3980
```
上麵的範例展示了如何在myapp.jar的0063處找到A.class。來自於內嵌jar的B.class實際可以在myapp.jar的3452處找到，B.class可以在3980處找到（圖有問題？）。

有了這些信息，我們就可以通過簡單的尋找外部jar的合適部分來加載指定的內嵌實體。我們不需要解壓存檔，也不需要將所有實體讀取到內存中。
