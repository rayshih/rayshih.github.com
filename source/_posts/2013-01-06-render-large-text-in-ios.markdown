---
layout: post
title: "Render Large Text in iOS"
date: 2013-01-06 16:17
comments: true
categories: iOS
---

如何在 iOS 上 render 一大堆文字呢？很長很長的文章之類的。  
我一開始想說不管那麼多，先用 UILabel 直接 render 上去就對了。當然剛開始沒什麼問題，因為文章短嘛。結果到了文章變得異常的長的時候，UILabel 就直接不見了！？

<!-- more -->

## Why?

** iOS View 不允許太大的 View **

那所以到底多大才不能被接受？

> Note: In iOS 2.x, the maximum size of a UIView object is 1024 x 1024 points. In iOS 3.0 and later, views are no longer restricted to this maximum size but are still limited by the amount of memory they consume. It is in your best interests to keep view sizes as small as possible. Regardless of which version of iOS is running, you should consider tiling any content that is significantly larger than the dimensions the screen.

from [UIView Reference](http://developer.apple.com/library/ios/#documentation/uikit/reference/uiview_class/uiview/uiview.html)

簡單的說，就是有限制，但是官方沒寫到底多大不行。就我的實驗是在 simulator 上，高度超過 `8192 (= 2^13)` 就不行了。

## Solution！

**把文章切開！**

我自己的作法是用`\n`把文章切開，切開的每個部分都放到獨立的`UILabel`。最後再把這些`UILabel`放到`ScrollView`上面，我自己的作法是放到`UITableView`上，再用`- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath`這個Delegate方法去調整每個Cell的高度(當然`UILabel`的高度也需要調整啦)

---
_References:_

[iPhone SDK doesn't like long texts !](http://blog.tofodo.com/2009/03/iphone-sdk-doesnt-like-long-texts.html)  
[我在Stackoverflow上問的問題](http://stackoverflow.com/questions/14125563/uilabel-view-disappear-when-the-height-greater-than-8192)
