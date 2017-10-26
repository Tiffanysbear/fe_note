## 需求背景
> 经常会有这样的需求，当页面滚动到某一个位置时，需要某个页面元素固定在屏幕顶部，并且有时需要连续滚动吸顶。在PC端主要的实现是通过 CSS 的 position: fixed 属性，但是在移动端，尤其是在安卓端，存在诸多的兼容性问题。 

## 问题
position：fixed给移动端带来的问题： 

* IOS8在页面滚动时，吸顶不连续；页面滑动时，不见吸顶，页面滚动停止后，吸顶缓慢出现
* 滚动到顶部之后，会出现两个一样的吸顶, 过一会才恢复正常。
* footer底部输入框 focus 状态，footer 底部输入框被居中，而不是吸附在软键盘上部。
* iPhone 4s&5 / iOS 6&7 / Safari 下，页面底部footer输入框失去焦点时，header定位出错。当页面有滚动动作时，header定位恢复正常。
* iPhone 4 / iOS 5 / Safari下，当页面发生跳转，再退回时，fixed区域消失，当内容获得焦点时，fixed区域才显示。
* 安卓低版本／自带浏览器，不支持fixed属性，iOS4 也是不支持 fixed 的。
* 三星i9100(S2) / 自带浏览器，在滚屏过程中，fixed定位异常，touchend之后恢复正常。
*  部分低版本Android对支持不好，video poster属性设置的封面图会遮挡fixed元素。
*  QQ、UC浏览器滚动页面时footer定位错误，会往上偏移，是由于地址栏收起的缘故。
*  <font color="red">*remind：不要在 fixed 元素中使用 input / textarea 元素。</font>

## 解决方案

分别处理各个问题：
### IOS
在IOS端，使用``` position： sticky ``` 这个属性，使用类似于``` position: relative``` 和 ``` position: absolute``` 的结合体。在目标区域在屏幕中可见时，它的行为就像```position:relative```; 而当页面滚动超出目标区域时，它的表现就像```position:fixed```，它会固定在目标位置。

 使用时，需要加上私有前缀

```javacript
	position: -webkit-sticky;
	position: -moz-sticky;
	position: -ms-sticky;
	position: sticky; 
```
	
对于``` position：sticky ``` 的	使用，需要注意很多的细节，sticky满足以下条件才能生效：

1、具有sticky属性的元素，其父级高度必须大于sticky元素的高度。

2、sticky元素的底部，不能和父级底部重叠。（这条不好表述，文后详细说明）

3、sticky元素的父级不能含有overflow:hidden 和 overflow:auto 属性

4、必须具有top，或 bottom 属性。

同时要注意，sticky元素仅在他父级容器内有效，超出容器范围则不再生效了。

### 安卓

滚动距离超过某位置时，js动态设置样式；为了防止惯性滚动引起的fix不及时的情况，在``` touchstart```、 ```touchmove``` 、``` touchend ```事件都进行监听。

```javacript
	     // 注意处理遮罩层的位置
        var scrollHandler = function () {
            if (topLength < me.getScrollTop()) {
                target.css('position', 'fixed');
                me.replaceEle.show();
            }
            else {
                target.css('position', 'relative');
                me.replaceEle.hide();
            }
        };
        // 安卓情况下，防止惯性滚动引起的fix不及时的情况
        if (/(Android)/i.test(navigator.userAgent)) {

            $(window).on('scroll', scrollHandler);

            $(document.body).on('touchstart', scrollHandler);
            $(document.body).on('touchmove', scrollHandler);
            $(document.body).on('touchend', function () {
                scrollHandler();
                setTimeout(scrollHandler, 1000);
            });
        }
```
### 不支持sticky

如果浏览器不支持position:sticky，那么就使用js动态的在节点在fixed定位于static定位中切换，但是需要对切换过程做一些优化。
1、使用函数节流防抖减少dom操作频繁粗发，但是保证在规定时间内必须执行一次。
2、使用window.requestAnimationFrame 方法在下一帧前触发浏览器的强制同步布局，是对dom的操作能及时渲染到页面上。
3、减少对dom的读写操作，或者把dom操作把读、写操作分开，可以减少渲染次数。

*** 
参照 [原文代码](https://segmentfault.com/a/1190000008004300)

```javacript
		(function() {
		    function Sticky(){
		        this.init.apply(this, arguments);
		    }
		
		    /**
		     * 滚动fixed组件初始化
		     * @param {object}         setting                allocate传进来的参数
		     * @param {object}         setting.stickyNode     需要设置position:sticky的节点，通常是最外层
		     * @param {object}         setting.fixedNode      当滚动一定距离时需要fixed在顶部的节点
		     * @param {int}            setting.top            fixed之后距离顶部的top值
		     * @param {int}            setting.zIndex         fixed之后的z-index值
		     * @param {string}         setting.fixedClazz     fixed时给fixedNode添加的类
		     * @param {function}     setting.runInScrollFn  滚动期间额外执行的函数
		     * @return {void}  
		     */
		    Sticky.setting = {
		        stickyNode: null,
		        fixedNode: null,
		        top: 0,
		        zIndex: 100,
		        fixedClazz: '',
		        runInScrollFn: null
		    };
		    var sPro = Sticky.prototype;
		    var g = window;
		
		    /**
		     * 初始化
		     * @param  {object} options 设置
		     * @return {void}         
		     */
		    sPro.init = function(options){
		        this.setting = $.extend({}, Sticky.setting, options, true);
		        if (options.fixedNode) {
		            this.fixedNode = options.fixedNode[0] || options.fixedNode;
		            this.stickyNode = options.stickyNode[0] || options.stickyNode;
		            this.cssStickySupport = this.checkStickySupport();
		            this.stickyNodeHeight = this.stickyNode.clientHeight;
		            this.fixedClazz = options.fixedClazz;
		            this.top = parseInt(options.top, 10) || 0;
		            this.zIndex = parseInt(options.zIndex) || 1;
		            this.setStickyCss();
		            this.isfixed = false;
		            // 把改变定位的操作添加到节流函数与window.requestAnimationFrame方法中，确保一定事件内必须执行一次
		            this.onscrollCb = this.throttle(function() {
		                this.nextFrame(this.sticky.bind(this));
		            }.bind(this), 50, 100);
		            this.initCss = this.getInitCss();
		            this.fixedCss = this.getFixedCss();
		            this.addEvent();
		        }
		    };
		
		    /**
		     * 获取原始css样式
		     * @return {string} 定位的样式
		     */
		    sPro.getInitCss = function() {
		        if (!!this.fixedNode) {
		            return "position:" + this.fixedNode.style.position + ";top:" + this.fixedNode.style.top + "px;z-index:" + this.fixedNode.style.zIndex + ";";
		        }
		        return "";
		    };
		
		    /**
		     * 生成fixed时的css样式
		     * @return {void}
		     */
		    sPro.getFixedCss = function() {
		        return "position:fixed;top:" + this.top + "px;z-index:" + this.zIndex + ";";
		    };
		
		    /**
		     * 给fixedNode设置fixed定位样式
		     * @param {string} style fixed定位的样式字符串
		     */
		    sPro.setFixedCss = function(style) {
		        if(!this.cssStickySupport){
		            if (!!this.fixedNode){
		                this.fixedNode.style.cssText = style;
		            }
		        }
		    };
		
		    /**
		     * 检查浏览器是否支持positon: sticky定位
		     * @return {boolean} true 支持 false 不支持
		     */
		    sPro.checkStickySupport = function() {
		        var div= null;
		        if(g.CSS && g.CSS.supports){
		            return g.CSS.supports("(position: sticky) or (position: -webkit-sticky)");
		        }
		        div = document.createElement("div");
		        div.style.position = "sticky";
		        if("sticky" === div.style.position){
		            return true;
		        }
		        div.style.position = "-webkit-sticky";
		        if("-webkit-sticky" === div.style.position){
		            return true;
		        }
		        div = null;
		        return false;
		    };
		
		    /**
		     * 给sticyNode设置position: sticky定位
		     */
		    sPro.setStickyCss = function() {
		        if(this.cssStickySupport){
		            this.stickyNode.style.cssText = "position:-webkit-sticky;position:sticky;top:" + this.top + "px;z-index:" + this.zIndex + ";";
		        }
		    };
		
		    /**
		     * 监听window的滚动事件
		     */
		    sPro.addEvent = function() {
		        $(g).on('scroll', this.onscrollCb.bind(this));
		    };
		
		    /**
		     * 让函数在规定时间内必须执行一次
		     * @param {Function} fn     定时执行的函数
		     * @param {int}      delay  延迟多少毫秒执行
		     * @param {[type]}   mustRunDelay 多少毫秒内必须执行一次
		     * @return {[type]}      [description]
		     */
		    sPro.throttle = function(fn, delay, mustRunDelay){
		        var timer = null;
		        var lastTime;
		        return function(){
		            var now = +new Date();
		            var args = arguments;
		            g.clearTimeout(timer);
		            if(!lastTime){
		                lastTime = now;
		            }
		            if(now - lastTime > mustRunDelay){
		                fn.apply(this, args);
		                lastTime = now;
		            }else{
		                g.setTimeout(function(){
		                    fn.apply(this, args);
		                }.bind(this), delay);
		            }
		        }.bind(this);
		    };
		
		    /**
		     * window.requestAnimationFrame的兼容性写法，保证在100/6ms执行一次
		     * @param  {Function} fn 100/16ms需要执行的函数
		     * @return {void}      
		     */
		    sPro.nextFrame = (function(fn){
		        var prefix = ["ms", "moz", "webkit", "o"];
		        var handle = {};
		        handle.requestAnimationFrame = window.requestAnimationFrame;
		        for(var i = 0; i < prefix.length && !handle.requestAnimationFrame; ++i){
		            handle.requestAnimationFrame = window[prefix[i] + "RequestAnimationFrame"];
		        }
		        if(!handle.requestAnimationFrame){
		            handle.requestAnimationFrame = function(fn) {
		                var raf = window.setTimeout(function() {
		                    fn();
		                }, 16);
		                return raf;
		            };
		        }
		        return function(fn) {
		            handle.requestAnimationFrame.apply(g, arguments);
		        }
		    })();
		
		    /**
		     * 判断stickyNode的当前位置设置fixed|static|sticky定位
		     * @return {void}
		     */
		    sPro.sticky = function() {
		        this.setting.runInScrollFn && this.setting.runInScrollFn();
		        var stickyNodeBox = this.stickyNode.getBoundingClientRect();
		        if(stickyNodeBox.top <= this.top && !this.isfixed){
		            this.setFixedCss(this.fixedCss);
		            this.fixedClazz && $(this.fixedNode).addClass(this.fixedClazz);
		            this.isfixed = true;
		            $(this).trigger('onsticky', true);
		        } else if(stickyNodeBox.top > this.top && this.isfixed) {
		            this.setFixedCss(this.initCss.replace(/position:[^;]*/, "position:static"));
		            g.setTimeout(function() {
		                this.setFixedCss(this.initCss)
		            }.bind(this), 30);
		            this.fixedClazz && $(this.fixedNode).removeClass(this.fixedClazz);
		            this.isfixed = false;
		            $(this).trigger('onsticky', true);
		        }
		    };
		
		    $.initSticky = function(options){
		        return new Sticky(options);
	    	};
	})();
```

html 结构：

```html
	<div class="m-nav">
   		<div class="nav-fixed fixed" id="j-nav" style="position: fixed; top: 0px; z-index: 100;">
       	<ul class="f-cb">
          	<li class="active" anchor-id="j-understand">了解儿童编程</li>
             		<li anchor-id="j-join">参与公益直播课</li>
	                 <li anchor-id="j-upload">上传编程作品</li>
          </ul>
       </div>
	</div>
```

css 结构：

```css
	.g-page-box .m-nav {
	 	height: 1.33333rem;
	}
	
	.g-page-box .m-nav .nav-fixed {
		 height: .86667rem;
		 padding: .22667rem .50667rem;
		 background-color: #1aadbb;
		 position: relative;
		 transform: translate3d(0, 0, 0);
		 -webkit-transform: translate3d(0, 0, 0);
		 transition: height 4s;
	}
	
	.fixed {
		position: fixed;
		top: 0px;
		z-index: 100;
	}
	
```















	
