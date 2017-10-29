## 背景
> 对开发移动端时遇到的问题进行收集、记录和总结，对于已经有解决方案，附上解决方案。

## tap击穿问题 
### 问题描述
绑定tap方法的dom元素，触发该方法时，该dom元素下方同一位置的dom元素会触发click事件或者有浏览器认为可以被点击交互的dom元素（input的focus事件），称为tap击穿现象。
### 产生原因
1、click事件在移动端会有300ms的延迟，因为需要检测双击事件。[移动端300ms延迟原因](https://www.telerik.com/blogs/what-exactly-is.....-the-300ms-click-delay)

2、zepto的tap事件是绑定在document.body上的，tap事件执行（冒泡之后）之前，click事件已经被"执行"，只是被延迟了而已，所以在tap事件用preventDefault也没用 

### 解决方案
1、上下元素使用同样的事件，同样适用tap或者click事件

2、使用fastclick库，会把click的300ms延迟干掉

3、自己封装tap事件，使用tuochstart、touchend、touchmove事件，e.preventEvent()将事件阻止

```javascript
//封装一个tap事件
function tap(ele, callback) {
    var tag = 0;
    // 手机端和移动端不同处理
    if (isMobile()) {
        $(ele).on('tuochstart', function(event) {
            event.preventDefault();
            tag = 0;
        // 如果移动了，则不计入tap事件
        }).on('touchmove', function() {
            tag = 1;
        }).on('touchend', function(event) {
            if (tag == 0) {
                callback(event);
            }
        })
    } else {
        $(ele).on('click', function(event) {
            callback(event);
        })
    }
}

```

## position: fixed + input输入框问题

### 问题描述
IOS下，当input输入框获取焦点focus，弹起虚拟键盘之后，页面上position: fixed的元素的位置会错乱。
### 解决方案
1、当input元素focus时，改成position: absolute，blur的时候再将定位改为 position: fixed

2、使用iscroll库

3、使用div内滚动

## 消除 transition 闪屏

```css
-webkit-transform-style: preserve-3d;
/*设置内嵌的元素在 3D 空间如何呈现：保留 3D*/
-webkit-backface-visibility: hidden;
/*（设置进行转换的元素的背面在面对用户时是否可见：隐藏）*/

```
## IOS字体大小重置

### 问题描述

iOS 与 OS X 端字体的优化(横竖屏会出现字体加粗不一致等) iOS 浏览器横屏时会重置字体大小，设置 text-size-adjust 为 none 可以解决 iOS 上的问题，但桌面版 Safari 的字体缩放功能会失效，因此最佳方案是将 text-size-adjust 为 100% 。

### 解决方案

```css
-webkit-text-size-adjust: 100%;
-ms-text-size-adjust: 100%;
text-size-adjust: 100%;
	
```
















