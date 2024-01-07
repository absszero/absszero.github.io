---
title: Develop in Docker
date: 2023-12-26T20:45:34+08:00
draft: false
keywords: ["Docker"]
---

## sDart

Docker 在產品容器化使用確實方便，但在思索怎麼用在開發環境遇到的第一個難題是太慢。Docker Desktop 啟動速度慢，每次開啟心裡就開始計時到底要多久。看著右上角的鯨魚 icon 不停的堆貨櫃、卸貨櫃、堆貨櫃...。不過這個問題在接觸到 [OrbStack](https://orbstack.dev/) 之後有很大的改善。一個可以替代 Docker Desktop 的輕量化容器執行工具。

有了最基本的工具之後，接下來就是選擇適當的 Laravel image。所幸 Laravel 官方在這一塊深耕許久， [Laravel Sail](https://laravel.com/docs/10.x/sail) 一個可以輕易選擇希望的 PHP 版本、Nodejs 版本，只需要修改 docker-compose.yaml 檔案即可。

```yaml
# 設定 PHP 版本
context: ./vendor/laravel/sail/runtimes/8.3

# 設定 nodejs 版本
build:
  args:
    NODE_VERSION: '18'
```

並且提供一個用 Shell Script 撰寫的 sail 指令腳本，可以簡化呼叫容器裡頭的 artisan、npm 等工具，讓你不用透過長長的 docker exec 指令。


```shell
# 執行 PHP 指令
sail php script.php
# composer 安裝套件
sail composer require laravel/sanctum
# 執行 artisan 指令
sail artisan queue:work
# 執行 npm 指令
sail npm run dev
```

## PHP7.4

由於 Laravel Sail 只支援 PHP 8.x，為了讓手邊的專案能夠順利進行。先在一個全新的 Laravel 專案上安裝 Sail 並執行 `sail artisan sail:publish` 取得 docker 目錄。該目錄包含了許多版本的 PHP 目錄： `8.0`, `8.1`, `8.2`, `8.3`。挑選最早期的 `8.0` 作為樣板開始打造 7.4 版的 PHP image。

主要是修改 `Dockerfile`，將 `8.0` 的字樣取代成 `7.4`，然後設定舊版本編號 `NODE_VERSION=16`，以支援當前專案的 Node 版本。接著需要回頭編輯 `docker-compose.yaml`。

```yaml
services:
    laravel.test:
        build:
            # context: ./vendor/laravel/sail/runtimes/8.3
            context: ./docker/7.4 # 使用自己打造的 7.4
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        #image: sail-8.2/app
        image: sail-7.4/app
```

接下來需要建立 image 透過 `sail build --no-cache`。 成功建立之後，可以透過 `sail composer install` 安裝相依套件。不過由於大部分的套件都相依 Github，需要本地端的個人權杖 `~/.composer/auth.json` 設定檔傳入：


```yaml
    volumes:
        - '.:/var/www/html'
        - "~/.composer/auth.json:/home/sail/.composer/auth.json:delegated"
```

操作到這邊， 7.4 版的運行環境算是解決了。


## 多專案架構

由於預計會有多個專案使用容器執行，Laravel Sail 的設計思維需要在每一個專案安裝 Laravel Sail，但最新版只支持 PHP 8.0 的專案，而且每個專案底下需要放置一個 docker-compose.yaml。並且每個 docker-compose.yaml 都長得十分相似。目前期望像是修改 apache 的 vhost 設定檔，可以統一在單個檔案裡頭完成所有專案的容器化設定。

思索過後，決定將 Sail 產生的 docker 目錄移動到跟專案同一個層的目錄底下，裡頭只放置一支 docker-compose.yaml。優點是易於管理，缺點就是專案越多，每一個都會啟動自己的 artisan serve，會佔用一些資源，後需可能需要透過 docker-compose 的 profiles 進行管理分類，只啟用特定的專案。

```ini
| - docker
|   |-- 7.4
|   |-- 8.0
|   |-- 8.1
|   |-- 8.2
|   |-- 8.3
|   |-- docker-compose.yaml
|
| - project_a
| - project_b
```

接著嘗試多專案的 docker-compose 設定。首先以原本的 laravel.test 為基底，建立一個 PHP74 的服務。然後利用 `extends` 指令繼承這個服務並增加相關的專案路徑 volume 掛載。需要特別注意的是 ports 的對應方面需要個別獨立不能重複。

```yaml
services:
  PHP74:
    build:
      context: ../docker/7.4
      dockerfile: Dockerfile
      args:
        WWWGROUP: "${WWWGROUP}"
    image: sail-7.4/app
    environment:
      WWWUSER: "${WWWUSER}"
    volumes:
      - "~/.composer/auth.json:/home/sail/.composer/auth.json:delegated"
    entrypoint: '/bin/true' # 由於是一個樣板，覆寫掉 Dockfile 的 ENTRYPOINT ["start-container"]
  project_a:
    container_name: "project_a"
    extends: PHP74
    entrypoint: "start-container"
    ports:
      - 8001:80

    volumes:
      - "/Volumes/data/www/project_a:/var/www/html"

  project_b:
    container_name: "project_b"
    extends: PHP74
    entrypoint: "start-container"
    ports:
      - 8002:80

    volumes:
      - "/Volumes/data/www/project_b:/var/www/html"
```

由於安裝完 Sail 之後，`./vendor/bin/sail` 指令也一並被安裝。為了方便其他沒有使用 Sail 的專案可以使用，將它移動到 `/usr/local/bin/sail` 。而該指令調用容器裡頭的程式時，需要提供 `APP_SERVICE` 作為識別目前專案所運行的容器名稱，所以每一個 service 都需要加上 `container_name=YOUR_SERVICE`，並且在專案的 `.env` 檔案需要添加 `APP_SERVICE=YOUR_SERVICE`。如此一來 Sail 才能調用相關的容器。

```ini
# .env
APP_SERVICE=project_a
....
....
```

完成之後可以在任意專案目錄底下執行 `sail up` 啟動容器。然後透過 http://localhost:8002 之類的 port 執行專案。

## 網域名稱

由於容器化的緣故，每一個專案都有自己的 Port 對應到 http://localhost。但事實上許多專案需要多個網域名稱，所以不能只透過 http://localhost 來連線。為了解決這個問題需要一個反向代理伺服器，將眾多網域跟每一個容器對應起來。

```goat
Domain 1 --.
           +                       .--- http://localhost:8001
Domain 2 --.---. Reverse Proxy .---+
           |                       .--- http://localhost:8002
Domain 3 --.

```

為了把整個開發環境容器化，所以這邊也使用跟容器整合良好的 [traefik](https://traefik.io/) 作為反向代理伺服器使用。直接在 docker-compose.yaml 添加該服務。

```yaml
services:
  TRAEFIK:
    image: "traefik:v2.10"
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
```

接著在專案服務上加上相對應的 `labels`：

```yaml
project_a:
  container_name: "project_a"
  extends: PHP74
  entrypoint: "start-container"
  ports:
    - 8001:80

  volumes:
    - "/Volumes/data/www/project_a:/var/www/html"
  labels:
      - "traefik.http.routers.project_a.rule=Host(`www.local`, `api.local`)"

project_b:
  container_name: "project_b"
  extends: PHP74
  entrypoint: "start-container"
  ports:
    - 8002:80

  volumes:
    - "/Volumes/data/www/project_b:/var/www/html"
  labels:
      - "traefik.http.routers.project_b.rule=Host(`demo.local`)"
```

當然，也別忘了更新本機 hosts 檔案：

```ini
127.0.0.1          www.local
127.0.0.1          api.local
127.0.0.1          demo.local
```

## 內部呼叫

雖然透過 traefik 反向代理之後，可以正常從網址指向到專案。但如果要從 project b 的程式呼叫 http://api.local，會遇到無法正確解析 DNS 的問題。由於 hosts 檔案是設定在本機上，只對本機使用有效。容器與容器之間的呼叫無法正確辨識網域名稱。目前的解決辦法是將 traefik 服務加上 aliases，包含所有透過它反向代理的網域名稱。

```yaml
TRAEFIK:
  image: "traefik:v2.10"
  command:
    - "--api.insecure=true"
    - "--providers.docker=true"
    - "--entrypoints.web.address=:80"
  ports:
    - "80:80"
    - "8080:8080"
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
  networks:
    default:
      aliases:
        - www.local
        - api.local
        - demo.local
```

### 其他問題

#### Hot Module Replacement

使用 vite 支援 HMR 的同時，它會啟動一個 server 端有自己的 port。大部分 port 使用 vite 預設的 `5173`。但在設定 docker compose  的時候，必經預先把 vite port 露出，這樣使用 HMR 的時候才能從本機連線到該 port。所以 port 不能重複的情況下，每一個專案要有自己的獨立 vite port。

```yaml
    ports:
      - 8001:80
      - 5174:5173
```

#### Monorepo

Sail 預設專案根目錄可以找到 node_modules 目錄，也以此為結構進行設計。所以套件指令都放在 node_modules/.bin 裡頭。但如果是 Monorepo 的專案結構，子專案可能不包含 node_modules，以某些原本好好的 node command 突然就無效了。

#### VSCode Extensions

某些套件可能會調用 node_modules 目錄裡頭的的指令，由於子專案可能不包含 node_modules，也就無法正常使用。