---
layout: post
title: "Before and After Hook of Rspec"
date: 2012-10-29 15:56
comments: true
categories: ["Rspec", "Rails"]
---

最近因為比較有空閒，所以就開始強迫自己一定要寫spec。
寫spec幾乎一定會用到`before :all`等hook，而這次又在這個地方，
花了不少時間。在這邊記錄一下以前到現在用before, after hook 
碰到的問題及要注意的地方。

<!-- more -->

1. Before, After Hook的執行順序
-------------------------------

所有的`before :all` 一定都在所有的`before :each`之前執行

例如：

``` ruby
describe "Outter" do
	before(:all) { puts "Before all" }
	after(:all) { puts "After all" }
	describe "Inner" do
		before(:each) { puts "Before each" }
		after(:each) { puts "After each" }
		it("should work") {puts "Test 1"}
		it("should work") {puts "Test 2"}
	end
end
```

``` bash Output
Before all
Before each
Test 1
After each
.Before each
Test 2
After each
.After all
```

這是正常使用狀況下，所有的`before :all`, `after :all` 都在所有
`before :each`, `after :each`的上層。也就是說，如果有`before :each`在
`before :all`的上層或同一層，會發生什麼事情呢？

``` ruby
describe "Outter" do
	before(:all) { puts "Before all" }
	after(:all) { puts "After all" }
	before(:each) { puts "Outter before each" }
	after(:each) { puts "Outter after each" }
	describe "Inner" do
		before(:all) { puts "Inner before all" }
		after(:all) { puts "Inner after all" }
		before(:each) { puts "Before each" }
		after(:each) { puts "After each" }
		it("should work") {puts "Test 1"}
		it("should work") {puts "Test 2"}
	end
end
```

``` bash Output
Before all
Inner before all 		# 所有的before all在這行前結束
Outter before each
Before each
Test 1
After each
Outter after each
.Outter before each
Before each
Test 2
After each
Outter after each
.Inner after all		# 所有的after all從這行開始
After all
```

可以看到所有的`before :all`, `after :all`都在最外面執行了。
也就是不管層級關係，`before :all` 一定都出現在 `before :each` 之前。
而所有的`after :all`一定都出現在`after :each`之後。

**所以在內層使用`before :all`, `after :all`時要小心，最好不要這樣用**

_待續_

---

心得：  
_Octopress好好用_  
_太久沒寫東西，文筆變超級差_  
_晚點再來寫下一篇_
