---
layout: blog
title: 解決 docker image 在 build 階段遇到檔案異動的問題處理方式
date: 2023-02-06 10:42:23
tags: [docker,php]
categories: php
---

# 問題

當我建置好 ci/cd 環境時，有一天發現 build 階段發生錯誤，我看一下錯誤訊息看看可不可以在我電腦環境中發生一樣的錯誤

於是當我 docker build image 時，錯誤圖片如下

![](/2023/02/06/docker-image-php-file-modify/issue.jpg)

這個錯誤是因為 apt-get 時，由於 php 有更新所以導致的錯誤訊息，因為有異動到檔案導致 docker build 錯誤

# 解決方式

在 docker image 新增兩條指令，避免遇到 apt-get 時，更新依賴套件並異動設定檔案時發生錯誤導致 build 失敗

```
# 修正 upgrade 錯誤
# https://github.com/phusion/baseimage-docker/issues/542
RUN echo 'DPkg::options { "--force-confdef"; };' >> /etc/apt/apt.conf
RUN apt-get upgrade -y
```

# 參考資料

[Failed to run apt upgrade when building based on latest image](https://github.com/phusion/baseimage-docker/issues/542)