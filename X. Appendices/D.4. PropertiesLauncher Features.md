### 附錄D.4. PropertiesLauncher特性

PropertiesLauncher有一些特殊的性質，它們可以通過外部屬性來啟用（系統屬性，環境變量，manifest實體或application.properties）。

|Key|作用|
|----|:-----|
|loader.path|逗號分割的classpath，比如`lib:${HOME}/app/lib`|
|loader.home|其他屬性文件的位置，比如[/opt/app](file:///opt/app)（默認為`${user.dir}`）|
|loader.args|main方法的默認參數（以空格分割）|
|loader.main|要啟動的main類名稱，比如`com.app.Application`|
|loader.config.name|屬性文件名，比如loader（默認為application）|
|loader.config.location|屬性文件路徑，比如`classpath:loader.properties`（默認為application.properties）|
|loader.system|布爾標識，表明所有的屬性都應該添加到系統屬性中（默認為false）|

Manifest實體keys通過大寫單詞首字母及將分隔符從"."改為"-"（比如`Loader-Path`）來進行格式化。`loader.main`是個特例，它是通過查找manifest的`Start-Class`，這樣也兼容JarLauncher。

環境變量可以大寫字母並且用下劃線代替句號。

* `loader.home`是其他屬性文件（覆蓋默認）的目錄位置，隻要沒有指定`loader.config.location`。
* `loader.path`可以包含目錄（遞歸地掃描jar和zip文件），存檔路徑或通配符模式（針對默認的JVM行為）。
* 佔位符在使用前會被系統和環境變量加上屬性文件本身的值替換掉。
