---
title: "Prism 建立 API Server"
date: 2020-02-06T15:01:03+08:00
draft: false
tags: [Prism API OpenAPI Server]
---


## Why Prism

撰寫一份 API 文件有許多方式，其中一種是根據 `OpenAPI` 規範。而在完成 API 規劃後通常搭配如 `Swagger UI` 將規範文件變成容易閱讀 API 文件。

但前端人員在真實 API 拿到之前，為了不用等待後端人員，必須有一個假的 API Server 以方便開發串接。通常會用像是 `json-server` 之類的工具，根據 API 規範自行建立假 json 資料，並建立起一個假的 API Server。

該方式主要問題在於，開發人員需要將 API 規範中的範例擷取出來，收集成一個完整的 json 檔案餵給 json-server。但當 API 規範趨於複雜，花在假 API Sever 便越來越多。

`Prism`，一個透過 OpenAPI 規範產生假 API Server 的工具便油然而生。



## How to Install

安裝 Prism 可以透過 `npm` 的方式安裝，或使用官方提供的 Docker Image。

### 透過 npm

```bash
npm install -g @stoplight/prism-cli
```

### 透過 Docker

```bash
docker pull stoplight/prism:3
```



## How to Use

### 透過 `prism` 指令

如果是透過 npm 安裝 prism，可以藉由底下方式啟動 API Server。啟動一個監聽任何 IP 來源(`-h 0.0.0.0`)，以及 Port 為 4010(`-p 4010`)，並由 prism 動態產生假資料(`-d`) 的 API Server。

```bash
prism mock-h 0.0.0.0 -p 4010 -d https://raw.githack.com/OAI/OpenAPI-Specification/master/examples/v3.0/petstore-expanded.yaml

# 也可以給定本地端檔案
prism mock-h 0.0.0.0 -p 4010 -d /tmp/petstore.yaml
```

### 透過 Docker

```bash
# 假設 file.yaml 在當前的目錄位置，需要掛載當前目錄成為 Docker 的一個 volume
docker run --init --rm -it -v $(pwd):/tmp -P stoplight/prism:3 mock -h 0.0.0.0 "/tmp/file.yaml"
```



## Multiple API Servers

當擁有多個 API 規範之後，為了將每一個檔案搭建成 API Server。可以利用 `docker-compose` 搭配網站伺服器進行 `反向代理（reverse proxy）` 來實現。

假設目的是建立兩個 API Server，分別指向網址 http://api.test/api-1/ 與 網址 http://api.test/api-2/ 。

### docker-compose 產生 API Server

當前目錄結構假設為

```
/home/api-docs
└── api-1.yaml
└── api-2.yaml
```

在 `/home/api-docs`  新增檔案  `docker-compose.yaml` ，確保兩台 API Server 各使用不同的 Port。以這個例子分別為 `4010` 與 `4011`。

```yaml
version: '3'
services:
  prism_1:
    restart: unless-stopped
    image: stoplight/prism:3
    volumes:
    - /home/api-docs:/tmp
    command: >
      mock -p 4010 --host 0.0.0.0 "tmp/api-1.yaml"
    ports:
    - "4010:4010"
  prism_2:
    restart: unless-stopped
    image: stoplight/prism:3
    volumes:
    - /home/api-docs:/tmp
    command: >
      mock -p 4011 --host 0.0.0.0 "tmp/api-2.yaml"
    ports:
    - "4011:4011"
```

透過指令 `docker-compose up` 將 API Server 啟動。



### Apache 設定反向代理

假設目前已設定好一組 vhost 針對 http://api.test，先確保 ` mod_proxy ` 以及 ` mod_proxy_http ` 模組已載入。並根據底下情況設定 vhost

```apacheconf 
<VirtualHost *:80>
    ServerName api.test

    ProxyPreserveHost On
    # api-1 的反向代理
    ProxyPass /api-1 http://127.0.0.1:4010/
    ProxyPassReverse /api-1 http://127.0.0.1:4010/
    # api-2 的反向代理
    ProxyPass /api-2 http://127.0.0.1:4011/
    ProxyPassReverse /api-2 http://127.0.0.1:4011/
</VirtualHost>
```

重新啟動網站伺服器之後瀏覽網址 http://api.test/api-1/ 與 網址 http://api.test/api-2/ 。



## 參考

* https://github.com/guzzle/guzzle-services
* https://stoplight.io/p/docs/gh/stoplightio/prism/docs/guides/multiple-documents.md