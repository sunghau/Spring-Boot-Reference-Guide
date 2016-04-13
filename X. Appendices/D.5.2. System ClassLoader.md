### 附錄D.5.2. 系統ClassLoader

啟動的應用在加載類時應該使用`Thread.getContextClassLoader()`（多數庫和框架都默認這樣做）。嘗試通過`ClassLoader.getSystemClassLoader()`加載嵌套的類將失敗。請注意`java.util.Logging`總是使用系統類加載器，由於這個原因你需要考慮一個不同的日誌實現。
