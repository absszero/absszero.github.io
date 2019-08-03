---
title: 'ftp_login() : SSL/TLS handshake failed'
date: Tue, 17 Mar 2015 10:10:56 +0000
draft: false
tags: [IT 筆記, openssl, PHP]
---

目前猜測是 openssl 的問題，從原本的 {{< highlight bash >}} $ openssl version -a OpenSSL 0.9.8e-fips-rhel5 01 Jul 2008
{{< /highlight >}}
 安裝最新版 {{< highlight bash >}} $ /usr/local/ssl/bin/openssl version -a OpenSSL 1.0.2 22 Jan 2015
{{< /highlight >}}
 然後重新編譯 php 指定新的預設 openssl 位置 {{< highlight bash >}} ./configure --with-openssl=/usr/local/ssl
{{< /highlight >}}
 打完收工...