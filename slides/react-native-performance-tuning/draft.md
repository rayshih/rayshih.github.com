# React Native Performance Tuning

## Abstract

React Native is a very hot topic recently. You can iterate your app faster by using react native. But there are also costs to use it. One of them is that performance tuning is more complicated. The speaker will share his experience of performance tuning for React Native on Android.

## Outline

- Self Introduction
- What is React Native
- Why React Native
- Our Architecture
- Performance Issues of React Native
    - Communication effort between JS and Java
    - `runAfterInteractions`
    - `LayoutAnimation`
    - `requestAnimationFrame`
    - `setNativeProps`
    - `shouldComponentUpdate`

- Performance Issues of Reactive System: Rx
- Performance Issues of React
    - VDom diff
- Single Source of Truth
    - The tradeoff of global single state
- Conclusion


## Draft

TL;DR;

- React Native is fast, kind of magic.
- "Dr. Stephen Strange: Study and practice. Years of it."
- Doesn't mean you don't need to know android framework

What is React Native

- intro to react
- React Native is just React without with DOM, but native view

Why react native?

- don't need to parse JSON! yeah! (well...with risk)
- easier to scale (in terms of architecture)
	- declarative way to write view (example)
	- SSOT is easy to understand
- shorter iteration, faster development especially for views themselves

Measure Performance

- systrace
- React addons Perf (skip if no time)
- just console.log

Performance problem of Java realm

- the video problem

Performance problem of JS(React) realm

- react is fast, because of vdom
	- manipulate actual view is slow
	- list of data updated -> remove list view and rerender
- react is slow, because of vdom (what?)
	- because vdom diff still take times
	- `shouldComponentUpdate`
		- child component

So we choose fun-react

- react -> reactive -> rx
- no manually write `shouldComponentUpdate` anymore
	- the `createView`
	- the `component`
- and simpler event handler

Rx may be the cause

- `createView`: shallow compare props
- because we still need `interaction`
	- layout
- SSOT doesn't mean single global state
	- local state, like form
- `component`: `props.get`, `props.select` and `distinctUntilChanged`
- hotfix: `throttle`

Performance problem of React Native

- bridge is slow
- avoid across bridge
	- LayoutAnimation
	- fallback to pure native solution

- avoid VDOM diff: `setNativeProps`

Different Navigation

- replace or animated
	- don't need to trigger render views are not on the screen

Animation problem

- `runAfterInteraction`
- upgrade, upgrade and upgrade

Conclusion

- React Native may be fast or slow, depends on how well you understand your tool
- Don't guess or hotfix too early, please do find the root cause


## To check

- Profiling First
	- do experiment

- ListView
	- rowHasChanged

- Animations
	- LayoutAnimation


## References

- <https://facebook.github.io/react-native/docs/performance.html>
	- official
	- common sources of performance problems

- <https://facebook.github.io/react-native/docs/android-ui-performance.html>
	- systrace

Blogs

- [携程是如何做React Native优化的](https://mp.weixin.qq.com/s?__biz=MzA3ODg4MDk0Ng%3D%3D&mid=2651112846&idx=1&sn=840bfcfe4b10c834ff7f7e982da0c0b4)
	- bundle loading time, crash and listview

- [React Native痛点解析之性能调优](http://www.infoq.com/cn/articles/react-native-performance-tuning)

- [Performance Limitations of React Native and How to Overcome Them](https://medium.com/@talkol/performance-limitations-of-react-native-and-how-to-overcome-them-947630d7f440#.lc5kwi8e7)

> The performance bottleneck often occurs when we move from one realm to the other. In order to architect performant React Native apps, we must keep passes over the bridge to a minimum.


