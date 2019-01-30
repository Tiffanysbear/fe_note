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
## 一些有用的属性
### 不让 Android 手机识别邮箱
```css
<meta content="email=no" name="format-detection" />
```
### 禁止 iOS 识别长串数字为电话
```css
<meta content="telephone=no" name="format-detection" />
```
### 禁止 iOS 弹出各种操作窗口
```css
-webkit-touch-callout:none
```
### 禁止用户选中文字
```css
-webkit-user-select:none
```

## translate
动画效果中，使用translate比使用定位性能高，且性能更好。

## 使用setTimeout

### 问题描述
使用下述的语句，会使得code立即执行

```javascript
   setTimeout(function(){
     //.….code
   },0);

```
原理：[JavaScript下的setTimeout(fn,0)意味着什么？](http://www.cnblogs.com/silin6/p/4333999.html)

## Safari浏览器input输入框
### 问题描述
在safari下，input输入框，点击时会有一个默认的小人出现，点击后会自动补充联系人的信息
### 解决办法
只有将其隐藏

```css
input::-webkit-contacts-auto-fill-button {
  visibility: hidden;
  display: none !important;
  pointer-events: none;
  position: absolute;
  right: 0;
}
```

## 移动端input文字输入-文字输入限制
### 问题描述
当用户进行中文输入时，input 事件会截断非直接输入，什么是非直接输入呢，在我们输入汉字的时候，比如说「开心」，中间过程中会输入拼音，每次输入一个字母都会触发 input 事件，然而在没有点选候选字或者点击「选定」按钮前，都属于非直接输入。

### 解决办法
此时，input事件需要结合compositionstart和compositionend 这两个事件。
<br>
compositionstart: 开始非直接输入开始时触发
<br>
compositionend：非直接输入结束时触发。
```javascript
var inputLock = false;
function do(inputElement) {
    var regex = /[^1-9a-zA-Z]/g;
    inputElement.value = inputElement.value.replace(regex, '');
}

inputElement.addEventListener('compositionstart', function() {
  inputLock = true;
});


inputElement.addEventListener('compositionend', function(event) {
  inputLock = false;
  do(event.target);
})


inputElement.addEventListener('input', function(event) {
  if (!inputLock) {
    do(event.target);
    event.returnValue = false;
  }
});
```

## 移动端input文字输入-emoji表情输入
### 问题描述
当输入emoji表情的时候，js中判断emoji表情的length为2，因此emoji正常应该最多只能输入8个，但是ios端却把emoji的length算为1。
### 解决办法
限制字数，当超过字数限制的时候，把前16个字截断显示出来。


## textarea置底展示问题
### 问题描述
ios中的输入唤起键盘后，整个页面会被键盘压缩，也就是说页面的高度变小，并且所有的fixed全部变为了absolute。键盘会将页面顶上去。那么如果希望可以将输入框和键盘完全贴合，我们可以使用div模拟一个假的输入框，使用定位将真正的输入框隐藏掉，当点击假的输入框的时候，将真正的输入框定位到键盘上方，并且手动获取输入框焦点。
<br>
### 解决办法
在实现过程中需要注意下面几个问题：
<br>
1、真正的输入框的位置计算：
<br>
首先记录无键盘时的window.innerHeight，当键盘弹出后再获取当前的window.innerHeight，两者的差值即为键盘的高度，那么定位真输入框自然就很容易了
<br>
2、在ios下手动获取焦点不可以用click事件，需要使用tap事件才可以手动触发
```javascript
    $('#fake-input').on($.os.ios?'tap' : 'click', function() {
        initHeight = window.innerHeight;
        $('#input').focus();
    });
 ```
3、当键盘收起的时候我们需要将真输入框再次隐藏掉，除了使用失去焦点（blur）方法，还有什么方法可以判断键盘是否收起呢？
<br>
这里可以使用setInterval监听，当当前window.innerHeight和整屏高度相等的时候判断为键盘收起。
<br>
注意：键盘弹起需要一点时间，所以计算当前屏幕高度也需要使用setInterval
<br>
4、因为textarea中的文字不能置底显示，当输入超过一行textarea需要自动调整高度，因此将scrollHeight赋值给textarea的height。当删除文字的时候需要height也有变化，因此每次input都先将height置0，然后再赋值。
<br>

```javascript
    $('#textarea').css('height', 0);
    $('#textarea').css('height', $('#textarea')[0].scrollHeight);
  ```










