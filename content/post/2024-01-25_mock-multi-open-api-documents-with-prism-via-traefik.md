---
title: Mock multi openAPI documents with Prism via Traefik
date: 2024-01-25T12:41:15+08:00
keywords: ["openAPI", "Prism", "Traefik"]
---

Prism 可以藉由 openAPI 文件生成 mock server，官方也有針對這一塊說明如何處理多個檔案的情況： [Serving Multiple OpenAPI Documents](https://docs.stoplight.io/docs/prism/4bf78efbf9dd2-serving-multiple-open-api-documents)。該文章說明主要是透過 [Caddy](https://caddyserver.com/) 作為反向代理伺服器，透過編修 `Caddyfile` 設定反向代理伺服器。

而在近期接觸到 docker-compose 友好的  `Traefik` 之後，便心想透過 Traefik 作為代理伺服器。

## 目錄結構

```ini
| - specs
|   |-- petstore.yaml
    |-- demo.yaml
|
|-- docker-compose.yaml
```

## docker-compose.yaml

```yaml
services:
  traefik:
    image: 'traefik:v2.10'
    command:
      - '--api.insecure=true'
      - '--providers.docker=true'
      - '--entrypoints.web.address=:80'
    ports:
      - '80:80'
      - '8080:8080'
    volumes:
      - '/var/run/docker.sock:/var/run/docker.sock:ro'
  petstore:
    restart: unless-stopped
    image: 'stoplight/prism:5'
    volumes:
      - './specs:/tmp'
    command: mock -p 4010 --host 0.0.0.0 /tmp/petstore.yaml
    labels:
      - traefik.http.services.petstore.loadbalancer.server.port=4010
      - traefik.http.routers.petstore.rule=PathPrefix(`/petstore`)
      - traefik.http.routers.petstore.middlewares=strip-petstore
      - traefik.http.middlewares.strip-petstore.stripprefix.prefixes=/petstore

  demo:
    restart: unless-stopped
    image: 'stoplight/prism:5'
    volumes:
      - './specs:/tmp'
    command: mock -p 4010 --host 0.0.0.0 /tmp/demo.yaml
    labels:
      - traefik.http.services.demo.loadbalancer.server.port=4010
      - traefik.http.routers.demo.rule=PathPrefix(`/demo`)
      - traefik.http.routers.demo.middlewares=strip-demo
      - traefik.http.middlewares.strip-demo.stripprefix.prefixes=/demo

```

traefik 跟 prism 使用可參照官方說明，沒遇到什麼問題。當時實作遇到的癥結點是 labels 的設定。

 - prism 指定了 4010 port 作為服務 port。

    ```- traefik.http.services.petstore.loadbalancer.server.port=4010```

 - 為了區別不同的 API 文件，使用路由下掛的方式處理。 `PathPrefix` 會匹配所有符合的路由到指定的 prism 容器。

    ```- traefik.http.routers.petstore.rule=PathPrefix(`/petstore`)```

 -  由於 traefik 預設會將整個匹配路徑帶給 prism 容器。所以 `/petstore/pet`， prism 容器也會收到 `/petstore/pet`。需要將他去掉變成 `/pet`。

    ```yaml
    # 指定 petstore router 的 middlewares 名稱為 petstore
    - traefik.http.routers.petstore.middlewares=strip-petstore
    - traefik.http.middlewares.strip-petstore.stripprefix.prefixes=/petstore
    ```

掛多個 API 文件的技巧類似，只是需要特別注意 `labels` 裡頭的命名。

## 自動生成 prism 容器？

由於 specs 目錄下後續會放置多個 API 文件，原本希望在每次提交新的文件之後，透過 CI/CD 部署自動生成 prism。但由於 yaml 不支持 foreach 的功能。多半需要第三方的樣版引擎更新該文件。但也考量到文件越多 prism 使用越肥，而某些文件可能根本不需要 mock server。最後覺得有需要再添加到 docker-compose.yaml 可能是更理想的做法。