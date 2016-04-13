### 附錄B.1. 元數據格式

配置元數據位於jars文件中的`META-INF/spring-configuration-metadata.json`，它們使用一個具有"groups"或"properties"分類節點的簡單JSON格式：
```json
{"groups": [
    {
        "name": "server",
        "type": "org.springframework.boot.autoconfigure.web.ServerProperties",
        "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
    }
    ...
],"properties": [
    {
        "name": "server.port",
        "type": "java.lang.Integer",
        "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
    },
    {
        "name": "server.servlet-path",
        "type": "java.lang.String",
        "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
        "defaultValue": "/"
    }
    ...
]}
```
每個"property"是一個配置節點，用戶可以使用特定的值指定它。例如，`server.port`和`server.servlet-path`可能在`application.properties`中如以下定義：
```properties
server.port=9090
server.servlet-path=/home
```
"groups"是高級別的節點，它們本身不指定一個值，但為properties提供一個有上下文關聯的分組。例如，`server.port`和`server.servlet-path`屬性是`server`組的一部分。

**注**：不需要每個"property"都有一個"group"，一些屬性可以以自己的形式存在。







