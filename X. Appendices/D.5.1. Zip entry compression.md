### 附錄D.5.1 Zip實體壓縮

對於一個嵌套jar的ZipEntry必須使用`ZipEntry.STORED`函式保存。這是需要的，這樣我們可以直接查找嵌套jar中的個別內容。嵌套jar的內容本身可以仍舊被壓縮，正如外部jar的其他任何實體。
