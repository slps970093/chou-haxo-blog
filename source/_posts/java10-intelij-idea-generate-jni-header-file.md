---
layout: blog
title: Java 10 在 Intelij IDEA 快速產生 JNI Header File
date: 2023-02-25 00:13:52
tags: [java, intelij, jni]
categories: java
---
使用命令提示字元產生JNI文件麻煩又容易出錯，可以使用 Intelij IDEA 提供的 Extenrnal Tools 來幫助我們快速產生 JNI Header File

## 操作說明

1. File --> Setting

![](/2023/02/25/java10-intelij-idea-generate-jni-header-file/001.png)

2. Tools --> Extenrnal Tools 點擊 + 號 新增

![](/2023/02/25/java10-intelij-idea-generate-jni-header-file/002.png)

3. 依序輸入 輸入完畢後按下 OK

Name: Generate Header File
**Tool Settings**<br />

```
Program: $JDKPath$/bin/javac
Arguments: -h ./native -sourcepath $OutputPath$ -d $OutputPath$ $FileDir$\$FileName$
Working directory: $ProjectFileDir$
```

![](/2023/02/25/java10-intelij-idea-generate-jni-header-file/003.png)

4. 最後 到你要產生JNI的*.JAVA檔案 按下滑鼠右鍵 找到 **Extenrnal Tools** 應該會找到一個 **Generate Header File** 這樣標頭檔案就會出現在專案目錄中的 **native** 資料夾囉

![](/2023/02/25/java10-intelij-idea-generate-jni-header-file/004.png)

## 參考資料

[IDEA一键快速生成JNI头文件（可直接复制使用） JDK8 以前可以用](https://juejin.cn/post/6997006784921600036)
[javah missing after JDK install](https://stackoverflow.com/questions/50352098/javah-missing-after-jdk-install)
[JEP 313: Remove the Native-header Generation Tool (javah)](https://openjdk.org/jeps/313)
