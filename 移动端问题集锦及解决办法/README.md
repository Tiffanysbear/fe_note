## 背景
> 对开发移动端时遇到的问题进行收集、记录和总结，对于已经有解决方案，附上解决方案。

## tap击穿问题 
### 问题描述
绑定tap方法的dom元素，触发该方法时，该dom元素下方同一位置的dom元素会触发click事件或者有浏览器认为可以被点击交互的dom元素（input的focus事件），成为tap击穿现象。
### 产生原因
1、click事件在移动端会有300ms的延迟，因为需要检测双击事件。[移动端300ms延迟原因](https://www.telerik.com/blogs/what-exactly-is.....-the-300ms-click-delay)

2、zepto的tap事件是绑定在document.body上的，tap事件执行（冒泡之后）之前，click事件已经被"执行"，只是被延迟了而已，所以在tap事件用preventDefault也没用

3、
### 解决方案

