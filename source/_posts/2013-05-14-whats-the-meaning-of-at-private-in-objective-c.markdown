---
layout: post
title: "What's the meaning of @private in Objective C"
date: 2013-05-14 14:39
comments: true
categories: Objective-C
---

我們有時候會看到別人寫的code裡面包含了這樣的東西：

{% codeblock Person.h lang:objc %}
@interface Person : NSObject {
@public
    NSString *_name;
@private
    NSInteger _age;
}

@end
{% endcodeblock %}

所以 @private 跟 @public 到底是什麼意思？

<!-- more -->

## 先複習 Property 是什麼
一般 Objective C 要達成封裝的作法目前都是這樣

{% codeblock Person.h lang:objc %}
@interface Person: NSObject
@property (strong, nonatomic) NSString \*name;
@end
{% endcodeblock %}

{% codeblock Person.m lang:objc %}
@interface Person()
@property (nonatomic) NSInteger age;
@end
{% endcodeblock %}

也就是把 public 的 property 放在 header file ，private 的 property 放在 implementation file 裡面。
而其實 property 都是 compiler 幫你生好的 getter/setter method 而已。

有用過比較舊版的 objective c 的人，應該有經驗除了要加 property 以外，還要在 interface 下面打一串記憶體宣告，
然後再利用 @synthesize 的語法把這一串 getter/setter 的宣告 mapping 到記憶體宣告上，對吧？

其實 Objective C 跟 C++ 或 Java 對於 variable 的存取設計很像，也就是其實可以直接像 C++ 一樣用 `->` operator 存取。

像這樣，請搭配最上面的 `Person.h` 一起服用：

{% codeblock main.m lang: objc %}
int main(int argc, const char * argv[])
{

    @autoreleasepool {
        
        Person *person = [[Person alloc] init];
        person->_name = @"Ray Shih";
        // person->_age = 24; // Will raise compile error.
        
        NSString *resultName = person->_name;
        NSLog(@"My name is %@.", resultName);
    }
    return 0;
}
{% endcodeblock %}

## 所以 @private 跟 @public 到底是什麼？

其實就跟 C++, Java 等的 public, private 類似，只是在 Objective C 裡面不常見而已。

如同上面的 `main.m` ，可以看到因為 \_age 是宣告在 @private 下面，所以就沒有辦法從外界直接存取了，取而代之的就應該要用 getter/setter method。

---

實際上這個作法現在應該越來越少看到，但有時看一些舊的 open source code 還是會看到這的寫法。

P.S. 本來以為 Struct 可以 Cast 成 Objective C Object，但似乎 memory layout 不一樣，而且 NSObject 還會再塞一些東西。總之，失敗了，如果有人成功的話，告訴我一聲吧～
