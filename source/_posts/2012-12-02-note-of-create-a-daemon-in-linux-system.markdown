---
layout: post
title: "撰寫 Linux Daemon 筆記"
date: 2012-12-02 17:07
comments: true
categories: linux
---

本學期專題要撰寫一個需要變成Daemon的程式，因此花了不少時間研究（正事都沒做orz）。
也順便證明自己當初「系統程式」這門課上課不認真。還債阿orz

如何讓你的程式變成Daemon
------------------------

在Ruby下有個Gem叫做Daemons很好用，看看範例非常簡單就做好了。  
(雖然其實還是要學習如何Handle SIGTERM)

<!-- more -->

在Python下面就麻煩了，雖然有個官方Library叫做Python-Daemon，
但是他只幫你處理開啟Daemon的部份，其他要自己解決，
所以基本上還是要先理解Daemon開啟流程，以及他的原理。  
（其他的Library還要再研究看看）


開啟Daemon的步驟與原理
----------------------

1. fork child process 1，並在child process 1 產生新的session, group
    * 這次fork主要是為了確保產生新的session
    * 1. session跟tty是綁在一起的
    * 2. 因為parent本來就是session leader，所以不fork的話沒辦法產生新的session

2. fork child process 2
    * 因為child process 1雖然產生了新的session，但是這個session還是有可能
    * 取得新的tty（因為是session leader），所以必須fork第二次，確保不會與tty連接

3. 記錄process id



關閉Daemon步驟
--------------

1. 檔案讀取process id
2. 發出Kill Signal 給Daemon
3. Daemon 處理 Kill Signal

參考資料
--------

1. <http://code.activestate.com/recipes/278731/>
    * 開啟Daemon Python範例，配合超詳細解說

2. <http://www.jejik.com/articles/2007/02/a_simple_unix_linux_daemon_in_python/>
    * 完整Daemon開啟與關閉Python Code，可以直接拿來用

3. <http://stackoverflow.com/questions/881388/what-is-the-reason-for-performing-a-double-fork-when-creating-a-daemon>
    * 基本上看前兩個範例後，我個人還是不知道為什麼要double fork
    * 直到看了這篇stackflow中的兩則最高rating Answer，才得以解惑

