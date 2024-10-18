---
title: Set Bing Wallpaper After Login on Macos
date: 2024-10-18T22:22:08+08:00
draft: false
keywords: ["Bing", "Wallpaper", "Macos"]
---

每天上班都會看到同事的桌布更換，這也是 Windows 內建的功能。我很好奇那樣的景致是哪裡拍攝的，期望有一天排進我的旅遊清單。所以決定找一下有沒有類似的應用，可以幫我每日更換桌面。

看似簡單的需求相信已經有一堆應用可以實現。後來我確實找到 [Daily Wallpaper HD](https://apps.apple.com/us/app/daily-wallpaper-hd/id1507778016?mt=12) 能滿足我的需求。它會自動下載 Bing wallpaper 然後設置成桌布。但很快的我發現有兩個問題... 首先是解析度， Bing Wallpaper 預設的解析度偏低，放在 2K 螢幕有明顯的顆粒感。再來是這個應用需要登入的時候常駐，但對於每天只需要用一次的應用，常駐似乎有點多餘。

既然它是下載 Bing Wallpaper 也許我可以找到相對應的腳本，在登入的時候執行一次。後續我找了幾個開源的方案，包括：

- https://github.com/thejandroman/bing-wallpaper 已經太久沒有更新，無法更新桌布。
- https://github.com/scriptingosx/desktoppr 可以更新桌布，但載入需要透過 LaunchAgent，然後要自己提供 Bing 圖片連結。
- https://github.com/xiqishow/bing_wallpaper 腳本有說明如何下載 Bing 圖片，但設置桌布無效。


最終我採取 https://github.com/xiqishow/bing_wallpaper 配合 [Set Wallpaper with commands in Terminal possible? ](https://discussions.apple.com/thread/254859103?sortBy=rank) 這篇提到透過 Automator 生成一個 application。

步驟如下：

1. 透過 Automator 建立一個新的 application。
2. 新增 `Run Shell Script` action，Shell 選擇 /bin/bash，然後填入底下的腳本內容。
3. 新增 `Set the Desktop Picture` action 到 `Run Shell Script` 下方。
4. 最後回到 Macos 的 `System Settings` 搜尋 `Open at Login`，把剛才新增的 application 加進去。

下方的腳本主要進行的動作包括：

1. 下載每日的 bing 桌布描述內容的 json。
2. 從描述取得圖片網址，並替換成高解析度的網址，隨後檢查該圖片是否已經下載過了，有則結束執行。
3. 刪除原本的舊圖檔，下載新圖檔。
4. 輸出新圖檔的檔名位置，輸出內容會作為 `Set the Desktop Picture` 的輸入而設定該桌布。
5. 最後描述內容通常會說明該圖片的拍攝地點，把它當作系統訊息提示輸出。

```shell
#!/usr/bin/env bash

# 建立目錄
mkdir -p ~/bing-wallpapers/
cd ~/bing-wallpapers/

# 下載 images.json
curl --no-progress-meter --location --request GET 'https://bing.com/HPImageArchive.aspx?idx=0&n=1&format=js' --header 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36' -o images.json >> /dev/null

# 取得 wallpaper url, 確認是否已經下載過
url=`cat images.json | grep -o '"url":"[^"]*"'  | sed -e 's/"url":"/https:\/\/www.bing.com/' | sed -e 's/"//' | sed -e 's/1920x1080/UHD/g'`
filename="$(echo -n \"$url\" | md5).jpg"
if test -f $filename; then
  exit 0
fi

# 下載桌布
rm -f *.jpg
curl --no-progress-meter "$url" -o $filename >> /dev/null
if test -f $filename; then
  realpath $filename
  location=`cat images.json | grep -o '"copyright":"[^"]*' | sed -e 's/"copyright":"//'`
  osascript -e "display notification \"$location\" with title \"Wallpaper Location\""
fi
```