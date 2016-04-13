### 附錄D.3.2. 暴露的存檔

一些PaaS實現可能選擇在運行前先解壓存檔。例如，Cloud Foundry就是這樣操作的。你可以運行一個解壓的存檔，隻需簡單的啟動合適的啟動器：
```shell
$ unzip -q myapp.jar
$ java org.springframework.boot.loader.JarLaunche
```

