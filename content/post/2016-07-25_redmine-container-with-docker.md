---
title: 'Redmine container with Docker'
date: Mon, 25 Jul 2016 14:34:41 +0800
draft: false
tags: [IT 筆記]
---

除了官方的 Docker image 之外，[sameersbn/redmine](https://hub.docker.com/r/sameersbn/redmine/) 算是最火紅的 image。安裝說明也十分詳細，不過目前搭配外部的 MySQL Sever，並且該 Server 架設在與 Docker 相同的主機上。需要注意的地方有兩點： 第一個是 MySQL 的 LISTEN IP，預設可能是 127.0.0.1，這邊將它設定成 0.0.0.0。 這部份從 /etc/mysql/my.cnf 修改 bind-address = 127.0.0.1 改為 bind-address = 0.0.0.0 第二是要考慮容器連接到外部的 MySQL 所走的路由閘道位置，預設應該是 docker0 這張虛擬網卡的 IP。假設是 172.17.42.1 最後使用底下指令啟動容器： {{< highlight shell >}} docker run \
--name=redmine -d -p 40000:80 \
--env='DB_ADAPTER=mysql2' \
--env='DB_HOST=172.17.42.1' \
--env='DB_USER=redmine' \
--env='DB_PASS=password' \
--volume=/volume1/redmine:/home/redmine/data \
sameersbn/redmine:3.3.3-1
{{< /highlight >}}
