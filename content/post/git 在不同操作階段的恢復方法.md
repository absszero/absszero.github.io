---
title: 'git 在不同操作階段的恢復方法'
date: Sat, 27 Jun 2015 03:28:21 +0000
draft: false
tags: [git, IT 筆記]
---

初學 git 透過 GUI 添加檔案跟提交都很便利，但是要如何恢復卻常常卡住。 主要是不熟悉指令端實際的運作情況，所以不確定 GUI 該如何選擇來恢復檔案。 下面把自己常常遇到要恢復的情況做個紀錄。 **未加到 stage(index)** 表示還沒透過 git add 指令把修改過的檔案加到 stage，可以這樣恢復：
{{< highlight bash >}}
git chekcout your_file
{{< /highlight >}}
 **已經加到 stage(index)** 表示還已經利用 git add 指令把修改過的檔案加到 stage，可以這樣恢復到未加到 stage(index)：
 {{< highlight bash >}}
 git reset HEAD your_file
{{< /highlight >}}
 結束之後，當然也可以再用 **未加到 stage(index)**的方式還原檔案。 **已經提交(commit)** 表示已經利用 git commit 指令把 stage 的檔案提交了，此時有兩種恢復方式， 一種是再做一次提交，但是提交的內容是把它改回原本的模樣。這個時候使用 **revert** 指令。
 {{< highlight bash >}}
 git revert HEAD
{{< /highlight >}}
 另外一種方式是回到某個提交狀態(例如上一個提交狀態，還沒錯誤提交的狀態)
 {{< highlight bash >}}
 git reset --hard HEAD~1
{{< /highlight >}}
 **HEAD~1** 表示當前(HEAD)提交狀態的上一個狀態 **--hard** 參數表示目前的目錄狀態，也強制回到某個提交的狀態。 也就是如果錯誤提交之後，又修改了某些檔案，該參數也強制把修改的檔案也改回來。 至於何時適合做 revert，而何時又適合做 reset 呢？ 我猜想可以用是否已經做 push 做為考量。 如果已經 push 了，使用 revert 產生一個新的提交，說明上一個提交是錯誤的，藉此恢復。 如果還未 push，可以考慮使用 reset，直接減少一個提交紀錄。