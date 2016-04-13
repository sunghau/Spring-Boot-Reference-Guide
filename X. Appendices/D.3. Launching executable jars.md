### 附錄D.3. 啟動可執行jars

`org.springframework.boot.loader.Launcher`類是個特殊的啟動類，用於一個可執行jars的主要入口。它實際上就是你jar文件的`Main-Class`，並用來設置一個合適的`URLClassLoader`，最後調用你的`main()`方法。

這裡有３個啟動器子類，JarLauncher，WarLauncher和PropertiesLauncher。它們的作用是從嵌套的jar或war文件目錄中（相對於顯示的從classpath）加載資源（.class文件等）。在`[Jar|War]Launcher`情況下，嵌套路徑是固定的（`lib/*.jar`和war的`lib-provided/*.jar`），所以如果你需要很多其他jars隻需添加到那些位置即可。PropertiesLauncher默認查找你應用存檔的`lib/`目錄，但你可以通過設置環境變量`LOADER_PATH`或application.properties中的`loader.path`來添加其他的位置（逗號分割的目錄或存檔列表）。
