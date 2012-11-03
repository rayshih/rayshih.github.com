---
layout: post
title: "Insert jQuery to Any Page"
date: 2012-11-03 12:20
comments: true
categories: ["web", "javascript"]
---

最近在寫Crawler爬資料，用的是ruby的gem Nokogiri，但是每次改都要等他跑完才知道結果是不是對的，
如果是錯的要再回到browser來檢查html跟css跟自己寫的是不是一致，來來回回總是會覺得不太方便。
如果可以在browser裡面做完這件事情，效率一定快很多，但也不是所有的網站都用jQuery
，所以也無法隨意的開chrome console來隨意使用平常愛用的css selector
（當然也可以只用javascript，只是比較麻煩就是了）

<!-- more -->


所以我就自己寫了一個小小的javascript，如下:
{% gist 3985855 %}

只要把這個下面的連結拖到Bookmark，之後只要碰到任何沒有用jQuery的網頁，就直接點這個Bookmark就對了

<a href='javascript:(function(){script = document.createElement("script");script.src="http://ajax.googleapis.com/ajax/libs/jquery/1.8.0/jquery.min.js";document.body.appendChild(script);})();'>Insert jQuery</a>

-----
很可惜的Facebook這種有用https的似乎就不能用的樣子。
