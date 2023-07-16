---
title: 使用IIFE优化性能
index_img: /img/IIFE.png
date: 2023-07-16 16:03:36
tags:
- javascript
- IIFE
---
## 1.问题引入
考虑一个兼容浏览器事件添加的代码段
```javascript
function addEvent(ele, eventName, handler){
    if(ele.addEventListener){
        ele.addEventListener(eventName, handler)
    }else if(ele.attachEvent){
        ele.attachEvent('on'+eventName, handler)
    }else{
        ele['on'+eventName] = handler
    }
}
```


代码实现了兼容包括谷歌浏览器、IE浏览器等的事件监听方法。代码本身不存在BUG，但是在执行时会有性能问题。用户在每次添加事件时都需要走一遍if-else流程，但实际上我们在第一次调用 ```addEvent``` 方法甚至在脚本加载时就得知当前浏览器环境适用的事件监听方法。我们可以使用添加flag的方法来跳过后续重复调用函数时的性能损耗

## 2.flagFunction优化
```javascript
//flagFunction
let flagFunction

function addEvent(ele, eventName, handler){
    if(flagFunction){
        flagFunction.call(ele, eventName, handler)
    }else {
        if(ele.addEventListener){
            ele.addEventListener(eventName, handler)
            flagFunction = ele.addEventListener
        }else if(ele.attachEvent){
            ele.attachEvent('on'+eventName, handler)
            flagFunction = ele.attachEvent
        }else{
            ele['on'+eventName] = handler
            flagFunction = function (eventName, handler){
                ele['on'+eventName] = handler
            }
        }
    }
    
}
```
## 3.IIFE方式优化
实际上，这种方法代码会造成难以维护。我们接着采用IIFE的方式优化

```javascript
//IIFE
var addEvent = (function(){
    if(window.addEventListener){
        return function (ele, eventName, handler){
            ele.addEventListener(eventName, handler)
        }
    }else if(window.attachEvent){
        return function (ele, eventName, handler){
            ele.attachEvent('on'+eventName, handler)
        }
    }
    //...
})
```
可以看到相比第一中方式，采用IIFE使得代码更易阅读，更易维护