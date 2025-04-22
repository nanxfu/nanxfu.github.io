---
title: 爬虫基本功——加密数据定位
index_img: /img/scraping.jpg
date: 2024-10-11 14:40:34
tags:
---
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011144612.png)
# 企x科技——无混淆js的数据解密方法一：

打开抓包工具，切换列表抓取到数据后查看响应体发现有encrypt_data字段内容被加密。要找出加密函数可以在搜索栏直接搜索该字段
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011144326.png)
进入到js内格式化后搜索encrypt_data字段在可疑处打断点随后分析即可
## 小技巧
对于大部分请求如果断点断下后若不确定是否是自己跟踪的接口，可以使用调试工具的XHR功能来确定

# 烯x数据——无混淆js的数据解密方法二
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011152152.png)

## 解密请求响应
特征1.在/industry/newest路径下的页面为静态页面
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011145911.png)
特征2.页面切换时会出现加密数据
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011150017.png)

检查发起程序时不存在js混淆，如果要按照方法一找出解密函数搜索关键字d显然不现实。此时可以尝试搜索JSON.parse函数。在全局搜索栏里搜索会有大量使用JSON.parse的函数出现。为了减小搜索范围，可以在请求处点击"发起程序"在函数调用处的文件里直接搜索JSON.parse。

然后分析代码，在疑似处理加密数据处的JSON.parse断点。随后加载数据，程序在第30行处停下。我们再在控制台输出JSON.parse(y)即可发现网页的明文数据，此处即为加密数据的处理段
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011150843.png)

## 表单请求加密
特征3.除此之外，同样的接口。在程序请求接口时依然会出现以下两个加密数据：payload、sig
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011151958.png)

要找出加密程序，老方法**先搜索请求字段**payload或者sig。结果搜不出来想要的结果
![payload](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011153914.png)
![sig](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011154003.png)

接着尝试**搜索请求路径**，找到了唯一一个包含该路径的js文件。然后在响应面板中右键在源面板中打开
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011152654.png)
在industry-newest文件中搜索payload和sig分析js代码发现都不太像加密处理的位置。于是我们可以跳转到其它中，发现有一块可疑处。即可在此处添加断点
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011153310.png)
刷新页面调用接口后开发工具并没有断下，所以我们只能另寻他处。再搜索栏搜索sig =时发现又一处比较可疑。所以我们添加断点并尝试请求接口
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011153429.png)
结果程序立马断下。观察源代码即可解出加密函数
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011153819.png)

# 招标与企xx
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241011154911.png)